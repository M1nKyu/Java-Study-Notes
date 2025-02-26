---
created: 2025-02-26 16:38
summary: RefreshToken과 Redis 도입
tags:
  - spring
  - springboot
last_modified: 2025-02-26T18:43:00
---
> [출처 블로그](https://coasis.tistory.com/74), [깃허브](https://github.com/Ssspil/blog-code/tree/main/security)

---
### 🍪 구현 순서
1. JWT accessToken 인증 필터 구현
2. JWT 인증필터 검증
3. refreshToken도 추가로 발급하여 Redis에 저장
4. refreshToken으로 accessToken 재발급 (API 개발)
5. 로그아웃
---
### 🍪 JWT AccessToken AuthenticationFilter 구현 
#### 🍬 JwtAuthenticationFilter
```java
@Slf4j
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtHelper jwtHelper;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // AccessToken 값 가져오기
        String accessToken = "";
        Optional<Cookie> optionalCookie = Optional.ofNullable(request.getCookies())
                .map(cookies -> Arrays.stream(cookies)
                        .filter(cookie -> cookie.getName().equals("accessToken"))
                        .findFirst())
                .orElse(Optional.empty());
        if(optionalCookie.isPresent()){
            accessToken = optionalCookie.get().getValue();
        }

        // 토큰이 없으면 (비로그인) 다음 필터로 넘김
        if(accessToken.isEmpty()){
            filterChain.doFilter(request, response);
            return;
        }

        // JWT 토큰 검증
        if(!jwtHelper.validation(accessToken)){
            response.setCharacterEncoding(StandardCharsets.UTF_8.toString());
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("유효하지 않은 토큰입니다.");

            return;
        }

        // 토큰에서 정보 획득
        String email = jwtHelper.extractSubject(accessToken);
        UserRole role = UserRole.valueOf(jwtHelper.extractRole(accessToken));

        User user = new User(email, "password", role);
        CustomUserDetailsDto customUserDetails = new CustomUserDetailsDto(user);

        JwtAuthenticationToken authToken = new JwtAuthenticationToken(customUserDetails, null, customUserDetails.getAuthorities());

        SecurityContextHolder.clearContext();
        SecurityContextHolder.getContext().setAuthentication(authToken);

        filterChain.doFilter(request, response);
    }
}
```

#### 🍬 구현한 필터 Security Config에 등록
```java
// (생략)
public class SecurityConfig {
	
	// (생략)
	
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http, JwtHelper jwtHelper) throws Exception{ // 📌 매개변수 추가 (JwtHelper)
		
		// (생략)
		
        http // 📌 필터 등록
                .addFilterBefore(new JwtAuthenticationFilter(jwtHelper), LoginAuthenticationFilter.class)
                .addFilterAt(loginAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
			
        return http.build();
    }
	// (생략)
}
```

#### 🍬 궁금점
###### ❓ addFilterBefore의 역할? 
---
### 🍪 JWT 인증 필터 검증
- 이제 accessToken을 가진 사용자라면 서버에 어떠한 요청이든 넣을 수 있게 되었다.
- 특정 경로가 특정 권한만 접근할 수 있도록 설정했을 때, 잘 작동하는지 확인해본다.
```java
// SescurityConfig 내 filterChain 메서드에 설정
http.authorizeHttpRequests((auth) -> auth
                .requestMatchers(ALL_URL).permitAll()
                .requestMatchers("/api/admin").hasAnyAuthority(UserRole.ADMIN.getCode())
                .anyRequest().authenticated());
```

- POSTMAN을 사용하여
	- `http://localhost:8080/api/admin` 경로에 권한이 각각 ADMIN, USER인 사용자로 요청했을 때,
		- ADMIN 권한인 경우 `200 OK`
		- ADMIN 권한이 아닌 경우 `403 Forbidden` 에러를 확인할 수 있다.
---
### 🍪 RefreshToken으로 AccessToken 재발급하기
#### 🍬 토큰을 내려준다면 저장 위치가 어디가 좋은가?
- AccessToken: 
	- 대체로 헤더로 내려주기 때문에 localStorage에 저장한다.
	- XSS 공격에 노출될 수 있다.
- RefreshToken:
	- HttpOnly를 이용하여 쿠키로 내려준다.
	- CSRF 공격에 취약하다.
---
### 🍪 로그인 시 RefreshToken도 발급하고 Redis에 저장하기 (LoginSuccessHandler)
> - LoginSuccessHandler 에서 AccessToken을 내려주고 있다.
> 	- RefreshToken도 내려주도록 수정하면 된다.
> 	- 사용자 email을 key 값으로, refreshToken 값을 value로 하여 RefreshToken 유효시간 만큼 Redis에 캐싱한다.

#### 🍬 의존성 추가
```yml
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

#### 🍬 RedisService
```java
@Service
@RequiredArgsConstructor
public class RedisService {
    private final RedisTemplate<String, String> redisTemplate;

    // 값 넣기
    public void save(String key, String value, Duration ttl){
        redisTemplate.opsForValue().set(key, value, ttl);
    }

    // 값 가져오기
    public Optional<String> find(String key){
        return Optional.ofNullable(redisTemplate.opsForValue().get(key));
    }

    // 값 삭제하기
    public void delete(String key){
        redisTemplate.delete(key);
    }
}
```

#### 🍬 LoginSuccessHandler
```java
@Slf4j
@RequiredArgsConstructor
@Component
public class LoginSuccessHandler implements AuthenticationSuccessHandler  {

    private final JwtHelper jwtHelper;
    private final RedisService redisService;

    /*
    로그인 성공 시 실행되는 메서드
    */
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        // 사용자 정보 추출
        UserDetails userDetails = (UserDetails) authentication.getPrincipal(); // 인증된 사용자 객체를 반환
        String email = userDetails.getUsername(); 

        // 사용자 권한(Role) 추출
        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();   // 사용자의 권한 목록을 가져온다.
        String role = authorities.stream()      // 스트림을 사용하여
                .findFirst()// 첫 번째 권한을 가져오며,
                .map(GrantedAuthority::getAuthority)
                .orElse(UserRole.ADMIN.getCode()); // 만약 존재하지 않으면, 기본적으로 ADMIN 역할 부여. 권한을 문자열로 반환하는 메서드 
        
        // JWT 액세스 토큰 및 리프레시 토큰 생성
        String accessToken = jwtHelper.generateAccessToken(email, role); // email, role 기반으로 액세스 토큰 생성
        String refreshToken = jwtHelper.generateRefreshToken(email);
        
        // 📌 Redis에 RefreshToken을 유효시간만큼 캐싱 
        redisService.save(email, refreshToken, Duration.ofMillis(jwtHelper.extractExpiredAt(refreshToken)));
        
        // 쿠키 생성 및 응답 추가
        Cookie accessTokenCookie = createCookie(accessToken, "accessToken");    // 액세스 토큰 저장 쿠키
        Cookie refreshTokenCookie = createCookie(refreshToken, "refreshToken"); // 리프레시 토큰 저장 쿠키

        // response.addHeader("Authorization", "Bearer " + accessToken);

        // JSON 응답 설정 및 쿠키 추가
        String jsonResponse = new ObjectMapper().writeValueAsString(new LoginResponseDto(HttpServletResponse.SC_OK, "성공적으로 로그인이 되었습니다."));
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);          // 응답의 Content-type을 JSON으로 설정
        response.setCharacterEncoding(StandardCharsets.UTF_8.toString());   // 한글이 깨지지 않도록 UTF-8 인코딩
        response.setStatus(HttpServletResponse.SC_OK);                      // HTTP 상태 코드를 200OK로 설정
        response.getWriter().write(jsonResponse);                           // 클라이언트에 JSON 응답을 전송

        // 생성한 JWT 쿠키를 클라이언트에게 전달
        response.addCookie(accessTokenCookie);
        response.addCookie(refreshTokenCookie);
    }

    private Cookie createCookie(String accessToken, String cookieName){ ... }
}
```
---
### 🍪 RefreshToken으로 AccessToken 재발급하기 (API 개발)
#### UserService, UserServiceImpl
```java
// UserService
public interface UserService {
    User saveUser(JoinRequestDto joinrequest);

    User findUserByEmail(String email);

    String reissue(String refreshToken);
}
```

```java
// UserServiceImpl
@Slf4j                     
@Service                    
@Transactional               
@RequiredArgsConstructor     
public class UserServiceImpl implements UserService, UserDetailsService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;
    private final JwtHelper jwtHelper; // 📌
    private final RedisService redisService; // 📌

	// (생략)
	
	// 📌
    @Override
    public User findUserByEmail(String email){
        return userRepository.findByEmail(email);
    }
    
    // 📌
    @Override
    public String reissue(String refreshToken){
        String email = jwtHelper.extractSubject(refreshToken);

        Optional<String> optionalToken = redisService.find(email);

        if(optionalToken.isPresent()){
            if(!optionalToken.get().equals(refreshToken)){
                throw new RuntimeException("유효하지 않은 refreshToken 입니다.");
            }
            User user = findUserByEmail(email);
            return jwtHelper.generateAccessToken(email, user.getRole().getCode());
        }else{
            throw new RuntimeException("존재하지 않는 토큰입니다.");
        }
    }
}
```

#### UserController
- 간단한 유효성 검사를 하며, Service에서 AccessToken을 재발급한 토큰 값을 쿠키에 다시 저장한다.
```java
@Slf4j                       
@RestController             
@RequiredArgsConstructor   
@RequestMapping("/api")     
public class UserController {
    
    private final UserService userService;
    private final JwtHelper jwtHelper; // 📌

	...

	// 📌 추가
    @PostMapping("/token/refresh")
    public ResponseEntity<String> reissue(HttpServletRequest request, HttpServletResponse response,
                                          @CookieValue(name="refreshToken", required = false) String refreshToken){
        if(refreshToken == null){
            return ResponseEntity.badRequest().body("refreshToken null");
        }

        if(!jwtHelper.validation(refreshToken)){
            return ResponseEntity.badRequest().body("refreshToken expired");
        }

        try {
            String accessToken = userService.reissue(refreshToken);
            Cookie accessTokenCookie = createCookie(accessToken);

            response.addCookie(accessTokenCookie);

            return ResponseEntity.ok().build();
        }catch (Exception e){
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(e.getMessage());
        }
    }
	
	// 📌 추가
    private Cookie createCookie(String accessToken){
        Cookie accessTokenCookie = new Cookie("accessToken: " accessToken);
        long expiration = jwtHelper.extractExpiredAt(accessToken);
        int maxAge = (int) ((expiration - new Date(System.currentTimeMillis()).getTime()) / 1000);
        accessTokenCookie.setMaxAge(maxAge);
        accessTokenCookie.setPath("/");
        accessTokenCookie.setHttpOnly(true);
        accessTokenCookie.setSecure(false);

        return accessTokenCookie;
    }
}
```

#### Redis 접속하여 값 확인하기
```
# 캐시된 모든 키 값 출력
KEYS *

# 해당 키로 캐시된 값 출력
get [KEY]
```

#### 궁금점
###### Redis의 장점?

---
### 🍪 로그아웃
> 로그아웃 시 토큰을 강제로 만료시킨다.
```java
...
public class UserController {
	...
	
	// 📌
    @DeleteMapping("/user/logout")
    public ResponseEntity<String> logout(HttpServletResponse response){
        SecurityContextHolder.clearContext();
		
        Cookie accessToken = deleteCookie("accessToken");
        Cookie refreshToken = deleteCookie("refreshToken");
		
        response.addCookie(accessToken);
        response.addCookie(refreshToken);
		
        return ResponseEntity.ok().body("Logout success");
    }
	
	...
	
	// 📌
    private Cookie deleteCookie(String cookieName){
        Cookie cookie = new Cookie(cookieName, null);
        cookie.setMaxAge(0);
        cookie.setPath("/");
        return cookie;
    }
}
```
#### 🍬 궁금점
###### ❓ 
---
### 🍪 추가 구상
- JWT 블랙리스트 생성
	- 로그아웃된 JWT를 서버 or Redis or DB에 저장하고 이후에 토큰이 재사용 될 때 인증을 거부하는 방식이다.
	- 이후에 JwtAuthenticationFilter(Jwt 인증 필터)에서도 이 블랙리스트를 검사하는 로직을 추가하기

---
> **참고**
> - [01](https://coasis.tistory.com/74)
> - [02-블랙리스트 도입](https://velog.io/@hyojhand/Refresh-Token-%EB%B8%94%EB%9E%99-%EB%A6%AC%EC%8A%A4%ED%8A%B8-%EB%8F%84%EC%9E%85%EA%B8%B0)
> - [03-JWT 블랙리스트](https://slimeland.tistory.com/41)
