---
created: 2025-02-21 20:14
summary: 
tags:
  - spring
  - springboot
last_modified:
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
```java
@Slf4j
@Component
public class JwtHelper {

    private static final int ACCESS_TOKEN_VALIDITY = 30 * 60 * 1000;
    private static final int REFRESH_TOKEN_VALIDITY = 24 * 60 * 60 * 1000;

    @Value("${spring.security.jwt.secret}")
    private String secretKey;

    public String generateAccessToken(String subject, String role){
        return JWT.create()
                .withSubject(subject)
                .withClaim("role", role)
                .withIssuedAt(new Date(System.currentTimeMillis()))
                .withExpiresAt(new Date(System.currentTimeMillis() + ACCESS_TOKEN_VALIDITY))
                .sign(Algorithm.HMAC512(secretKey));
    }

    public String generateRefreshToken(String subject){
        return JWT.create()
                .withSubject(subject)
                .withIssuedAt(new Date(System.currentTimeMillis()))
                .withExpiresAt(new Date(System.currentTimeMillis() + REFRESH_TOKEN_VALIDITY))
                .sign(Algorithm.HMAC512(secretKey));
    }

    public boolean validation(String token){
        try {
            JWTVerifier verifier = JWT.require(Algorithm.HMAC512(secretKey)).build();
            verifier.verify(token);
        }catch (JWTVerificationException e){
            return false;
        }
        return true;
    }

    public String extractSubject(String token){
        return JWT.decode(token)
                .getSubject();
    }

    public long extractExpiredAt(String token){
        return JWT.decode(token)
                .getExpiresAt()
                .getTime();
    }

    public String extractRole(String token){
        return JWT.decode(token)
                .getClaim("role")
                .asString();
    }
}
```
---
### 🍪 인증 성공/실패 핸들러 구현
#### 🍬 LoginSuccessHandler
```java
@Slf4j
@RequiredArgsConstructor
@Component
public class LoginSuccessHandler implements AuthenticationSuccessHandler  {

    private final JwtHelper jwtHelper;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        String email = userDetails.getUsername();

        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();

        String role = authorities.stream()
                .findFirst()
                .map(GrantedAuthority::getAuthority)
                .orElse(UserRole.ADMIN.getCode());

        String accessToken = jwtHelper.generateAccessToken(email, role);
        String refreshToken = jwtHelper.generateRefreshToken(email);
        Cookie accessTokenCookie = createCookie(accessToken, "accessToken");
        Cookie refreshTokenCookie = createCookie(refreshToken, "refreshToken");

        // response.addHeader("Authorization", "Bearer " + accessToken);

        String jsonResponse = new ObjectMapper().writeValueAsString(new LoginResponseDto(HttpServletResponse.SC_OK, "성공적으로 로그인이 되었습니다."));
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding(StandardCharsets.UTF_8.toString());
        response.setStatus(HttpServletResponse.SC_OK);
        response.getWriter().write(jsonResponse);

        response.addCookie(accessTokenCookie);
        response.addCookie(refreshTokenCookie);
    }

    private Cookie createCookie(String accessToken, String cookieName){
        Cookie accessTokenCookie = new Cookie(cookieName, accessToken);
        long expiration = jwtHelper.extractExpiredAt(accessToken);
        int maxAge = (int) ((expiration - new Date(System.currentTimeMillis()).getTime() / 1000));

        accessTokenCookie.setMaxAge(maxAge);
        accessTokenCookie.setPath("/");
        accessTokenCookie.setHttpOnly(true);
        accessTokenCookie.setSecure(false);

        return accessTokenCookie;
    }
}
```

#### 🍬 LoginFailHandler
```java
@Slf4j
@Component
public class LoginFailHandler implements AuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {

        String jsonResponse = new ObjectMapper()
                .writeValueAsString(new LoginResponseDto(HttpServletResponse.SC_UNAUTHORIZED, exception.getMessage()));

        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setCharacterEncoding(StandardCharsets.UTF_8.toString());
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

        response.getWriter().write(jsonResponse);

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
> **참고**
> - [01]()
