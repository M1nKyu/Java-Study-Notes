---
created: 2025-02-17 19:34
tags:
  - spring
  - springboot
last_modified: 2025-02-17T19:35:00
---
### 🍪 현업에서 사용되는 인증/인가 기술 스택
#### 1. 핵심 기술 스택
- Spring Security + JWT
- OAuth2/OIDC (소셜 로그인)
- Spring Data JPA + QueryDSL
- Redis (세션/토큰 관리)
- Spring Cloud Config (설정 관리)

#### 2. 기술 스택의 특징 및 장점
- 보안성
	- JWT + Redis 조합 -> 안전한 토큰 관리
	- Refresh Token 활용 -> 보안 강화
	- 비밀번호 암호화 (BCrypt)

- 확장성
	- MSA 환경 고려 (Spring Cloud Config)
	- 다양한 소셜 로그인 지원
	- Redis를 통한 분산 세션 관리

- 유지 보수성
	- 계층별 명확한 책임 분리
	- 예외 처리 일원화
	- 설정 중앙화

- 성능
	- Redis -> 빠른 토큰 검증
	- JPA + QueryDSL -> 효율적 쿼리 처리
	- 비동기 처리 지원

### 구현 방식 예제
#### 보안 설정
> `SecurityConfig`는 Spring Security 설정을 정의하는 클래스이다.
- `SecurityFilterChain`을 설정하여 HTTP 요청에 대한 보안 규칙을 정의한다.
- `csrf().disable()`은 CSRF 공격을 방어하는 기능 비활성화한다. API 서버는 일반적으로 비활성화한다.
- `sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)`는 상태없는 인증 방식을 사용하겠다는 설정, 서버가 세션을 관리하지 않음을 의미.
- `addFilterBefore`는 `JwtAuthenticationFilter`라는 커스텀 필터를 `UsernamePasswordAuthenticationFilter` 앞에 배치하여 JWT 인증을 먼저 처리하게 한다.
- `ouath2Login`을 통해 외부 OAuth2 로그인을 처리하는 설정을 추가한다.
```java
@Configuration
@RequiredArgsConstructor
public class SecurityConfig {
    private final JwtTokenProvider jwtTokenProvider;
    private final RedisTemplate redisTemplate;
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), 
                           UsernamePasswordAuthenticationFilter.class)
            .oauth2Login()
                .userInfoEndpoint()
                    .userService(customOAuth2UserService);
        return http.build();
    }
}
```
#### 토큰 관리(JWT + Redis)
> `AuthService` 클래스는 로그인 로직을 처리한다.
- `login()` 메서드는 사용자 이메일과 비밀번호를 통해 인증을 수행한다.
- 인증이 성공하면 JWT `accessToken`과 `refreshToken`을 생성한다.
- `refreshToken`은 Redis에 저장되며, 키는 `RT: 사용자이름` 형태로 구성된다. 이 방식은 사용자마다 고유한 Refresh Token을 Redis에 저장하는 방법이다.
```java
@Service
@RequiredArgsConstructor
public class AuthService {
    private final JwtTokenProvider tokenProvider; // JWT 토큰 생성 및 검증을 위한 클래스
    private final RedisTemplate<String, String> redisTemplate; // Redis에 데이터 저장을 위한 템플릿
     
    public TokenDto login(LoginRequest request) {
		
        // 1. 인증 (사용자 이메일과 비밀번호로 인증)
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.getEmail(), request.getPassword())
        );
        
        // 2. JWT 토큰 발급: 사용자 인증 정보를 바탕으로 JWT 생성
        TokenDto tokenDto = tokenProvider.generateToken(authentication);
        
        // 3. Refresh Token을 Redis에 저장 (RT: 사용자 이름 형태로 저장)
        redisTemplate.opsForValue()
            .set("RT:" + authentication.getName(), tokenDto.getRefreshToken(), 
                tokenDto.getRefreshTokenExpirationTime(), TimeUnit.MILLISECONDS);
        
        return tokenDto;
    }
}
```

- TokenDTO 
```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class TokenDto {
    private String grantType;     // Bearer 인증 타입
    private String accessToken;   // 실제 인증에 사용되는 토큰
    private String refreshToken;  // Access Token 재발급용 토큰
    private Long accessTokenExpirationTime;    // Access Token 만료 시간
    private Long refreshTokenExpirationTime;   // Refresh Token 만료 시간
}
```

#### 사용자 Entity 설계
```java
@Entity
@Table(name = "users")
@Builder
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
public class User extends BaseTimeEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String email;
    
    private String password;
    
    @Enumerated(EnumType.STRING)
    private Role role; // 사용자의 역할 (ex: ADMIN, USER)
    
    @Enumerated(EnumType.STRING)
    private AuthProvider provider;    // GOOGLE, KAKAO, NAVER etc.
    
    private String providerId; // 외부 OAuth2 인증에서 받은 고유 ID
}
```

#### Redis 설정
> `redisConfig` 클래스는 Redis와의 연결을 설정한다.
- `RedisTemplate`를 사용하여 Redis에 데이터를 저장하거나 조회할 수 있다.
- `setKeySerializer`와 `setValueSerializer`는 Redis에 저장할 데이터의 직렬화 방법을 설정한다.
	- `StringRedisSerializer`는 키 문자열로, 
	- `Jackson2JsonRedisSerializer`는 객체를 JSON 형태로 직렬화한다.

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory); // Redis 연결 설정
        template.setKeySerializer(new StringRedisSerializer()); // Redis 키 직렬화 설정
        template.setValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class)); // Redis 값 직렬화 설정
        return template;
    }
}
```

#### Spring Cloud Config 설정
```java
# application.yml
spring:
  config:
    import: "optional:configserver:"
  cloud:
    config:
      uri: http://config-server:8888
      name: auth-service
```

#### 추가적인 보안 강화
```java
@Configuration
public class SecurityDetailsConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("https://domain.com"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

#### 예외 처리
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuthenticationException(AuthenticationException e) {
        ErrorResponse response = ErrorResponse.builder()
            .code("AUTH_001")
            .message(e.getMessage())
            .build();
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(response);
    }
}
```
---
## 다음 학습: [[로그인, 로그아웃, 회원가입의 구현 (JWT + Redis) (1)]]
---
> **참고**
> - [01](https://coasis.tistory.com/70)
> - [02-csrf.disable()을 하는 이유?](https://velog.io/@wonizizi99/SpringSpring-security-CSRF%EB%9E%80-disable)
> - [03-csrf 보호 기능](https://developer-youn.tistory.com/156)
> - [📌 04-마이크로서비스 구조(MSA)의 인증 및 인가](https://medium.com/spoontech/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EA%B5%AC%EC%A1%B0-msa-%EC%9D%98-%EC%9D%B8%EC%A6%9D-%EB%B0%8F-%EC%9D%B8%EA%B0%80-authorization-authentication-a595179ab88e)
