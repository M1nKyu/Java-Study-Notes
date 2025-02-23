---
created: 2025-02-18 19:39
tags:
  - spring
  - springboot
last_modified: 2025-02-18T19:47:00
---
> [출처 블로그](https://coasis.tistory.com/72), [깃허브](https://github.com/Ssspil/blog-code/tree/main/security)
---
### 🍪 구현 순서
1. **로그인 인증 필터 (AuthentiacationFilter) 구현**
2. **실제 인증 로직 (AuthenticationProvider) 구현**
3. **UserDetailsService DB 로직 구현**
4. **SecurityConfig 설정**
5. JWT 모듈인 JwtHelper 제작
6. 인증 성공 시 실행하는 핸들러 (AuthenticationSuccessHandler) 구현
7. 인증 실패 시 실행하는 핸들러 (AuthenticationFailureHandler) 구현
8. 인증 성공 후 JWT 발급과 응답값 생성 및 결과
---
### 🍪 로그인 인증 필터 (Authentication Filter)
> `AbstractAuthenticationProcessingFilter`를 상속받은 `UsernamePasswordAuthenticationFilter`를 상속하여 구현한다.
#### 🍬 LoginAuthenticationFilter
- 사용자의 로그인 요청을 처리하는 필터로, `UsernamePasswordAuthenticationFilter`을 상속받아 Spring Security 인증 프로세스를 커스텀한다.
- 이 필터는 클라이언트의 로그인 요청을 감지하고, 아이디와 비밀번호를 추출한 후, 이를 `AuthenticationManager`에 전달하여 인증을 시도한다.
```java
@Slf4j
// 📌 UsernamePasswordAuthenticationFilter를 상속 -> 기존의 Spring Security에서 제공하는 로그인 필터의 동작을 확장
public class LoginAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    /*
    📌로그인 인증 시도
	    클라이언트가 로그인 요청을 하면 호출되는 메서드.
	    HttpServletRequest 에서 email과 password를 추출한 후 AuthenticationManager를 통해 인증 진행
    */
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

        // 📌 요청의 Content-Type 확인
        // contentType 기반으로 로그인 요청이 JSON 방식인지, Form 방식인지 구분한다.
        String contentType = request.getContentType();
        String username = "";
        String password = "";


        // 📌 JSON 요청 처리 (application/json 요청일 경우)
        if(contentType.equals(MediaType.APPLICATION_JSON_VALUE)){
            try {
                // 1. ObjectMapper를 사용하여 request.getReader()로 JSON 데이터를 읽어들인다.
                // 2. 이를 LoginRequestDto 객체로 변환한다.
                LoginRequestDto loginRequest = new ObjectMapper().readValue(request.getReader(), LoginRequest.class);

                // 3. LoginRequestDto에서 email과 password 값을 추출한다.
                username = loginRequest.getEmail();
                password = loginRequest.getPassword();

            // 4. 만약 JSON 형식이 올바르지 않다면 AuthenticationServiceException을 발생시킨다.
            } catch (IOException e){
                throw new AuthenticationServiceException("잘못된 key, name으로 요청했습니다.", e);
            }
        }
        // 📌 Form 요청 처리 (application/x-www-form-urlencoded 요청일 경우)
        else if(contentType.equals(MediaType.APPLICATION_FORM_URLENCODED_VALUE)){
            // 1. email값과 password 값을 가져온다.
            username = request.getParameter("email");
            password = this.obtainPassword(request);
        }

        // 📌 인증 객체 생성 및 전달
        // UsernamePasswordAuthenticationToken을 생성하여 username과 password를 담는다.
        UsernamePasswordAuthenticationToken unauthenticated
                = new UsernamePasswordAuthenticationToken(username, password);

        // 이를 AuthenticationManager에 전달하여 실제 인증을 시도한다.
        return super.getAuthenticationManager().authenticate(unauthenticated);
    }

    // 📌 인증 성공 시 실행
    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        // 로그를 출력하여 인증이 성공함을 기록한다.
        log.info("Securty login >> 인증 성공");

        // AuthenticationSuccessHandler를 가져와 onAuthenticationSuccess()를 실행한다.
        AuthenticationSuccessHandler handler = this.getSuccessHandler();
        handler.onAuthenticationSuccess(request, response, authResult);


    }
    // 📌 인증 실패시 실행
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        // 로그를 출력하여 인증 실패를 기록한다.
        log.info("Securty login >> 인증 실패");

        // AuthenticationFailureHandler를 가져와 onAuthenticationFailure()를 실행한다.
        AuthenticationFailureHandler handler = this.getFailureHandler();
        handler.onAuthenticationFailure(request, response, failed);
    }
}

```

#### 🍬 궁금점
###### ❓ AbstractAuthenticationProcessingFilter란?
- Spring Security에서 인증 요청을 처리하는 필터의 부모 클래스.
- 사용자가 로그인 요청을 보내면, 인증을 수행하는 핵심 역할을 한다.
- 주로 로그인 요청을 가로채고, AuthenticationManager에게 인증을 위임한다.
- UsernamePasswordAuthenticationFilter가 이 클래스를 상속받아 사용된다.

1️. **로그인 요청을 가로챔**
	- 특정 URL(예: `/login`)에 대한 요청을 감지
	- `attemptAuthentication()` 메서드를 호출하여 인증 시도
2️. **인증을 `AuthenticationManager`에 위임**
	- 사용자의 입력값을 토큰(`UsernamePasswordAuthenticationToken`)으로 변환
	- `AuthenticationManager.authenticate()`를 호출하여 인증을 시도
3️. **인증 성공/실패 처리**
	- 인증 성공 시 → `successfulAuthentication()` 호출
	- 인증 실패 시 → `unsuccessfulAuthentication()` 호출
###### ❓ Spring Security는 어떻게 인증 실패를 판단하는가?
- `AuthenticationManager.authenticate()`의 결과에 따라 인증 성공/실패가 결정된다.
1️. `AuthenticationManager.authenticate()` 호출
2️. 내부적으로 AuthenticationProvider가 비밀번호 검사
3️. 비밀번호 틀림, 계정 비활성화 등 예외 발생 시 인증 실패
4️. 예외가 발생하면 `unsuccessfulAuthentication()` 호출 → 401 Unauthorized 응답
###### ❓ HttpServletRequest와 HttpServletResponse
- HttpServletRequest
	- 클라이언트가 **서버로 보낸 요청**을 담는 객체.
	- 요청 헤더, 파라미터, 바디, 세션 정보 등을 **가져오는 데 사용**한다.
- HttpServletResponse
	- HTTP 응답 코드, 응답 헤더, 응답 바디 등을 설정할 수 있다.
###### ❓ content-Type이란? 구분의 필요성은?
- 클라이언트가 서버로 **보낼 데이터 형식**을 나타내는 HTTP 헤더.
- 서버가 Content-Type을 확인하고, 데이터 처리 방식을 결정하는데 사용한다.
- 어떤 클라이언트가 요청을 보내느냐에 따라 데이터 형식이 다를 수 있다. 따라서 구분을 하고 처리해야 한다. 
###### ❓ AuthenticationException란?
- 인증 과정에서 발생하는 예외 클래스, 로그인 실패시 던져지는 예외이다.
- **인증 실패 시 Spring Security에서 기본적으로 던지는 예외**이다.
###### ❓ AuthenticationManager란?
- 인증(Authentication)을 관리하는 인터페이스, 실제 인증을 수행하는 역할을 가진다.
```java
// authenticate()를 호출하면, 적절한 AuthenticationProvider가 인증을 수행함.
Authentication authentication = authenticationManager.authenticate(
    new UsernamePasswordAuthenticationToken(username, password)
);
```
1️. `LoginAuthenticationFilter`에서 `AuthenticationManager`에게 인증 요청 전달  
2️. `AuthenticationManager`가 적절한 `AuthenticationProvider`를 찾음  
3️. `AuthenticationProvider`가 DB에서 사용자 정보를 조회하고 인증  
4️. 인증 성공 → `Authentication` 객체 반환  
5️. 인증 실패 → `AuthenticationException` 발생
###### ❓ UsernamePasswordAuthenticationToken란?
- Spring Security에서 사용자의 아이디와 비밀번호를 담는 토큰 객체.
- 인증 전과 인증 후의 상태가 다르다.
- 인증 요청 시, password만 포함하고, 권한 정보는 null.
- 인증 성공 시, 사용자 정보(userDetails)와 권한(authorities) 포함.
```java
new UsernamePasswordAuthenticationToken(userDetails, password, userDetails.getAuthorities());
```
---
### 🍪 실제 인증 로직 (AuthenticationProvider) 구현
> DB에 존재하는 사람인지 아닌지를 검사한다.
#### 🍬 LoginAuthenticationProvider
```java
@Slf4j
@Component
@RequiredArgsConstructor
public class LoginAuthenticationProvider implements AuthenticationProvider {

    private final UserDetailsService userDetailsService;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        UserDetails user = userDetailsService.loadUserByUsername((String) authentication.getPrincipal());

        if(bCryptPasswordEncoder.matches((String)authentication.getCredentials(), user.getPassword())){
            return new UsernamePasswordAuthenticationToken(user, "", user.getAuthorities());
        }else {
            throw new LoginAuthenticationException("비밀번호가 다릅니다.");
        }
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

#### 🍬 LoginAuthenticationException
```java
public class LoginAuthenticationException extends AuthenticationException {

    public LoginAuthenticationException(String msg, Throwable cause){
        super(msg, cause);
    }

    public LoginAuthenticationException(String msg) {
        super(msg);
    }
}
```
#### 🍬 궁금점
###### ❓ UsernamePasswordAuthenticationToken와 UsernamePasswordAuthenticationFilter


---
### 🍪 UserDetailsService DB 로직 구현
#### 🍬 UserServiceImpl 코드 추가
```java
@Slf4j                      // 로그를 사용할 수 있게 해주는 Lombok 어노테이션
@Service                    // UserServiceImpl을 서비스 빈으로 등록
@Transactional              // 트랜잭션 관리를 자동으로 처리: saveUser()의 작업이 한 트랜잭션에 처리되도록 해준다.
@RequiredArgsConstructor    // final 선언 필드의 생성자를 자동으로 생성한다 -> 의존성 자동 주입
public class UserServiceImpl implements UserService, UserDetailsService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;

    @Override
    public User saveUser(JoinRequestDto joinRequest){  ...  }
    
	// 📌 추가
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User findUser = userRepository.findByEmail(username);

        if(findUser == null){
            throw new UsernameNotFoundException("해당 유저를 찾을 수 없습니다.");
        }
        return new CustomUserDetailsDto(findUser);
    }
}
```

#### 🍬 UserRepository 코드 추가
```java
@Repository
public interface UserRepository extends JpaRepository <User, Long>{
    Boolean existsByEmail(String email);

    User findByEmail(String email); // 📌 추가
}
```

#### 🍬 CustomUserDetailsDto
```java
@Slf4j
@RequiredArgsConstructor
public class CustomUserDetailsDto implements UserDetails {

    private final User user;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Collection<GrantedAuthority> authorities = new ArrayList<>();

        authorities.add((GrantedAuthority) () -> user.getRole().getCode());

        return authorities;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getEmail();
    }

    //
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired(){
        return true;
    }

    @Override
    public boolean isEnabled(){
        return true;
    }
}
```
---
### 🍪 SecurityConfig 설정 추가
```java
@Slf4j
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    @Value("${spring.security.cors.allowed-methods}")
    private String[] ALLOW_METHOD;
    @Value("${spring.security.cors.allowed-origins}")
    private String ALLOW_CROSS_ORIGIN_DOMAIN;

    private final AuthenticationConfiguration authenticationConfiguration;
    private final LoginSuccessHandler loginSuccessHandler;
    private final LoginFailHandler loginFailHandler;

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    public LoginAuthenticationFilter loginAuthenticationFilter() throws Exception{
        LoginAuthenticationFilter loginAuthenticationFilter = new LoginAuthenticationFilter();

        loginAuthenticationFilter.setAuthenticationManager(authenticationManager(authenticationConfiguration));
        loginAuthenticationFilter.setFilterProcessesUrl("/api/login");
        loginAuthenticationFilter.setAuthenticationSuccessHandler(loginSuccessHandler);
        loginAuthenticationFilter.setAuthenticationFailureHandler(loginFailHandler);

        return loginAuthenticationFilter;
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{

        final String[] ALL_URL = new String[]{"/api/login", "/api/user/save"};
        final String[] ADMIN_URL = new String[]{"/api/admin"};
        final String[] ROLES = new String[]{UserRole.SUPER.getCode(), UserRole.MANAGER.getCode(), UserRole.ADMIN.getCode()};

        http.cors((cors -> cors.configurationSource(new CorsConfigurationSource(){
            @Override
            public CorsConfiguration getCorsConfiguration(HttpServletRequest request){
                CorsConfiguration configuration = new CorsConfiguration();

                configuration.setAllowedOrigins(List.of(ALLOW_CROSS_ORIGIN_DOMAIN));
                configuration.setAllowedMethods(List.of(ALLOW_METHOD));
                configuration.setAllowedHeaders(Collections.singletonList("*"));
                configuration.setAllowCredentials(true);
                configuration.setMaxAge(3600L);

                return configuration;
            }
        })));


        http.csrf((auth) -> auth.disable());

        http.formLogin((auth) -> auth.disable());

        http.authorizeHttpRequests((auth) -> auth
                .requestMatchers(ALL_URL).permitAll()
                        .requestMatchers(ADMIN_URL).hasAnyAuthority(ROLES)
                        .anyRequest().authenticated());

        http.sessionManagement((session) -> session.sessionCreationPolicy((SessionCreationPolicy.STATELESS)));

        return http.build();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception{
        return authenticationConfiguration.getAuthenticationManager();
    }
}
```
#### 🍬 application.yaml 추가
```yaml
  security:
    jwt:
      secret: <시크릿 키 입력>
    cors:
      allowed-origins: "http://localhost:8080"
      allowed-methods: "GET,POST,PUT,DELETE,OPTIONS"
```
---
## 다음 학습: [[로그인, 로그아웃, 회원가입의 구현 (JWT + Redis) (3)]]
---
## 📝 추가 학습
- [[Spring Security의 인증 흐름 (Spring Security)]]
- [[AuthenticationManager와 AuthenticationProvider (Spring Security)]]
- [[Spring Security의 예외 처리 (Spring Security)]]
---
> **참고**
> - [01-SpringSecurity 구조와 흐름](https://coasis.tistory.com/71)
> - [02-AbstractAuthenticationProcessingFilter(SpringSecurity)](https://velog.io/@on5949/SpringSecurity-AbstractAuthenticationProcessingFilter-%EC%99%84%EC%A0%84-%EC%A0%95%EB%B3%B5)
