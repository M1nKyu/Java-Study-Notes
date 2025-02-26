---
created: 2025-02-21 20:14
summary: 
tags:
  - spring
  - springboot
last_modified: 2025-02-24T19:00:00
---
> [출처 블로그](https://coasis.tistory.com/72), [깃허브](https://github.com/Ssspil/blog-code/tree/main/security)
---
### 🍪 구현 순서
1. 로그인 인증 필터 (AuthentiacationFilter) 구현
2. 실제 인증 로직 (AuthenticationProvider) 구현
3. UserDetailsService DB 로직 구현
4. SecurityConfig 설정
5. **JWT 모듈인 JwtHelper 제작**
6. **인증 성공 시 실행하는 핸들러 (AuthenticationSuccessHandler) 구현**
7. **인증 실패 시 실행하는 핸들러 (AuthenticationFailureHandler) 구현**
8. **인증 성공 후 JWT 발급과 응답값 생성 및 결과**
---
### 🍪 JWT 모듈 JwtHelper 제작
#### 🍬 JwtHelper
- JWT(JSON Web Token) **생성 및 검증**을 담당하는 클래스.
```java
@Slf4j 
@Component
public class JwtHelper {

    private static final int ACCESS_TOKEN_VALIDITY = 30 * 60 * 1000;        // 액세스 토큰 유효기간 (30분)
    private static final int REFRESH_TOKEN_VALIDITY = 24 * 60 * 60 * 1000;  // 리프레스 토큰 유효기간 (24시간)

    // 📌 JWT 서명에 사용할 비밀키
    @Value("${spring.security.jwt.secret}")
    private String secretKey;

    /*
    📌 JWT 생성:
	    generateAccessToken() : 유저정보(subject)와 권한(role) 정보를 포함한 액세스 토큰 발급
	    generateRefreshToken(): 리프레시 토큰 발급 (사용자 정보만 포함, 권한 정보는 없음)
    */
    public String generateAccessToken(String subject, String role){
        return JWT.create()
                // JWT 필드 설정
                .withSubject(subject)                                                           // 사용자 정보 저장
                .withClaim("role", role)                                                        // 📌 역할(권한) 저장
                .withIssuedAt(new Date(System.currentTimeMillis()))                             // 발급 시간 설정 (현재 시간)
                .withExpiresAt(new Date(System.currentTimeMillis() + ACCESS_TOKEN_VALIDITY))    // 만료 시간 설정 (현재 시간 + 30분)
                .sign(Algorithm.HMAC512(secretKey));                                            // HMAC512 알고리즘을 사용하여 서명
    }
    public String generateRefreshToken(String subject) {
        return JWT.create()
                .withSubject(subject)                                                           // 사용자 정보 저장 (이메일 또는 사용자 ID)
                .withIssuedAt(new Date(System.currentTimeMillis()))                             // 발급 시간 설정 (현재 시간)
                .withExpiresAt(new Date(System.currentTimeMillis() + REFRESH_TOKEN_VALIDITY))   // 만료 시간 설정 (현재 시간 + 24시간)
                .sign(Algorithm.HMAC512(secretKey));                                            // HMAC512 알고리즘을 사용하여 서명
    }

    /*
    📌 validation(): JWT 유효성 검증
    */
    public boolean validation(String token) {
        try {
            // JWTVerifier 객체 생성 - HMAC512 알고리즘과 시크릿 키를 사용하여 검증
            JWTVerifier verifier = JWT.require(Algorithm.HMAC512(secretKey)).build();

            // 전달받은 토큰이 유효한지 검증 (1.서명이 올바르고, 2.만료되지 않았는지, 3.변조되지 않았는지 확인)
            verifier.verify(token);
        } catch (JWTVerificationException e) {
            return false; // ❌ 검증 실패 시 false 반환
        }
        return true; // ✅ 검증 성공 시 true 반환
    }


    /*
    📌 JWT 정보 추출: extractSubject, extractExpiredAt, extractRole
    */
    // JWT의 사용자정보(subject) 추출
    public String extractSubject(String token){
        return JWT.decode(token) // 토큰을 디코드(decode) -> JWT 객체로 변환
                .getSubject();   // 토큰의 'subject' 값을 추출
    }
    // JWT의 만료시간 (Unix timestamp) 추출
    public long extractExpiredAt(String token){
        return JWT.decode(token)
		.getExpiresAt() // 만료 시간 추출
                .getTime();     // Unix timestamp로 반환
    }
    // JWT에서 사용자 역할(role) 추출
    public String extractRole(String token){
        return JWT.decode(token)
                .getClaim("role")        // "role" 클레임 값을 추출
                .asString();             // role 클레임을 문자열(String) 형태로 반환
    }
}
```
#### 예시 코드
```java
JwtHelper jwtHelper = new JwtHelper();
String token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";  // 예시 JWT 토큰

// 1. 사용자 정보 추출 (subject)
String subject = jwtHelper.extractSubject(token);  
System.out.println("Subject: " + subject);  // 예: user123

// 2. 만료 시간 추출 (expiresAt)
long expiredAt = jwtHelper.extractExpiredAt(token);  
System.out.println("Expires At: " + expiredAt);  // 예: 1677812400000 (Unix timestamp)

// 3. 사용자 역할 추출 (role)
String role = jwtHelper.extractRole(token);  
System.out.println("Role: " + role);  // 예: ROLE_USER
```
#### 🍬 궁금점
###### ❓ claim?
- JWT에서 **payload 부분에는 토큰에 담을 정보**가 들어있다.
- 이 **정보들을 클레임(Claim)** 이라고 부르고, 이는 **Json(Key/Value) 형태**의 한 쌍으로 이루어져 있다.
- **주요 Claim 예시**
	- `iss` (Issuer): 토큰 발급자 (토큰을 만든 서버)
	- `sub` (Subject): 토큰의 주제 (주로 사용자의 고유 식별자)
	- `aud` (Audience): 토큰을 받을 대상 (토큰의 수신자)
	- `exp` (Expiration Time): 토큰 만료 시간
	- `nbf` (Not Before): 토큰이 사용되기 시작할 시점
	- `iat` (Issued At): 토큰 발급 시간
	- `jti` (JWT ID): 토큰의 고유 ID
###### ❓ subject?
- JWT에서 subject는 **토큰의 주요 내용**을 나타내는 사용자 정보이다.
	- 일반적으로 사용자의 고유 **식별자(ID) 또는 이메일**을 주로 사용한다.
	- subject는 JWT에서 **중요한 claim**으로 사용되며, 보통 `sub`라는 이름으로 저장된다.
---
### 🍪 인증 성공/실패 핸들러 구현
#### 🍬 LoginSuccessHandler
> 로그인 성공 시 JWT 토큰을 생성하고, 이를 쿠키로 응답에 추가하는 역할을 한다.

```
// 정리
1. 사용자가 로그인하면 `onAuthenticationSuccess()`가 실행됨.
2. 사용자 이메일과 권한을 가져옴.
3. `jwtHelper`를 사용해 액세스 토큰과 리프레시 토큰을 생성함.
4. `createCookie()` 메서드로 JWT를 쿠키에 담음.
5. 응답으로 JSON 메시지(로그인 성공)와 함께 쿠키를 클라이언트에 전달함.
```
```java
@Slf4j
@RequiredArgsConstructor
@Component
public class LoginSuccessHandler implements AuthenticationSuccessHandler  {

    private final JwtHelper jwtHelper;

    /*
    📌 로그인 성공 시 실행되는 메서드
    */
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        // 📌 사용자 정보 추출
        UserDetails userDetails = (UserDetails) authentication.getPrincipal(); // 인증된 사용자 객체를 반환
        String email = userDetails.getUsername(); 

        // 📌 사용자 권한(Role) 추출
        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();   // 사용자의 권한 목록을 가져온다.
        String role = authorities.stream()      // 스트림을 사용하여
                .findFirst()// 첫 번째 권한을 가져오며,
                .map(GrantedAuthority::getAuthority)
                .orElse(UserRole.ADMIN.getCode()); // 만약 존재하지 않으면, 기본적으로 ADMIN 역할 부여. 권한을 문자열로 반환하는 메서드 
        
        // 📌 JWT 액세스 토큰 및 리프레시 토큰 생성
        String accessToken = jwtHelper.generateAccessToken(email, role); // email, role 기반으로 액세스 토큰 생성
        String refreshToken = jwtHelper.generateRefreshToken(email);
        
        // 📌 쿠키 생성 및 응답 추가
        Cookie accessTokenCookie = createCookie(accessToken, "accessToken")     // 액세스 토큰 저장 쿠키
        Cookie refreshTokenCookie = createCookie(refreshToken, "refreshToken"); // 리프레시 토큰 저장 쿠키

        // response.addHeader("Authorization", "Bearer " + accessToken);

        // 📌 JSON 응답 설정 및 쿠키 추가
        String jsonResponse = new ObjectMapper().writeValueAsString(new LoginResponseDto(HttpServletResponse.SC_OK, "성공적으로 로그인이 되었습니다."));
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);          // 응답의 Content-type을 JSON으로 설정
        response.setCharacterEncoding(StandardCharsets.UTF_8.toString());   // 한글이 깨지지 않도록 UTF-8 인코딩
        response.setStatus(HttpServletResponse.SC_OK);                      // HTTP 상태 코드를 200OK로 설정
        response.getWriter().write(jsonResponse);                           // 클라이언트에 JSON 응답을 전송

        // 📌 생성한 JWT 쿠키를 클라이언트에게 전달
        response.addCookie(accessTokenCookie);
        response.addCookie(refreshTokenCookie);
    }

    /*
    📌 JWT 토큰을 쿠키로 변환하는 유틸리티 메서드
    */
    private Cookie createCookie(String accessToken, String cookieName){
        Cookie accessTokenCookie = new Cookie(cookieName, accessToken);
        long expiration = jwtHelper.extractExpiredAt(accessToken); // 토큰의 만료 시간을 가져옴
        int maxAge = (int) ((expiration - new Date(System.currentTimeMillis()).getTime() / 1000)); //현재 시간과 비교하여 쿠키의 maxAge 를 설정한다.

        accessTokenCookie.setMaxAge(maxAge);    // 쿠키의 만료 시간 설정
        accessTokenCookie.setPath("/");         // 모든 경로에서 쿠키를 사용할 수 있도록 설정
        accessTokenCookie.setHttpOnly(true);    // JavaScript에서 쿠키를 읽지 못하도록 보호
        accessTokenCookie.setSecure(false);     // 보안이 필요한 경우 true로 변경해야 함

        return accessTokenCookie;
    }
}
```

#### 🍬 LoginFailHandler
```java
@Slf4j
@Component
public class LoginFailHandler implements AuthenticationFailureHandler {

    /*
    로그인 실패 시 Spring Security가 자동으로 이 메서드를 호출한다.
    */
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {

        // JSON 응답 생성
        String jsonResponse = new ObjectMapper() // LoginResponseDto 객체를 JSON 문자열로 반환한다.
                .writeValueAsString(new LoginResponseDto(HttpServletResponse.SC_UNAUTHORIZED, exception.getMessage())); 

        // HTTP 응답 설정
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);          // 응답의 Content-Type을 application/json으로 설정하여 JSON 응답을 반환하도록 함.
        response.setCharacterEncoding(StandardCharsets.UTF_8.toString());   //  UTF-8 인코딩 설정으로 한글 깨짐 방지
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);            // HTTP 상태 코드 401 (인증실패)를 설정

        response.getWriter().write(jsonResponse); // JSON 응답을 클라이언트로 전송하여 로그인 실패 메시지를 반환합
    }
}
```

#### 🍬 LoginResponseDto
```java
@Getter
public class LoginResponseDto {
    private int status;
    private String message;

    public LoginResponseDto(int status, String message) {
        this.status = status;
        this.message = message;
    }
}
```
---
## [[로그인, 로그아웃, 회원가입의 구현 (JWT + Redis) (4)]]
---
> **참고**
> - [01]()
