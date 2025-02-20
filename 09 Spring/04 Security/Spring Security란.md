---
created: 2025-02-10 19:27
tags:
  - spring
  - springboot
last_modified: 2025-02-10T19:28:00
---
## ⭐ Spring Security 란?
- Spring Security는 스프링 프레임워크 기반의 보안(인증과 권한 부여)을 다루기 위한 프레임워크이다.
### 🍪 주요 기능
#### 1. 인증 (Authentication)
- 사용자의 신원을 확인하는 과정.
- 기본적으로 다양한 인증 프로토콜을 지원한다.
	- form 기반의 로그인,
	- HTTP 기본 인증,
	- Remmember Me 인증,
	- OAuth, 
	- LDAP 등 
#### 2. 권한 부여 (Authorization)
- 인증된 사용자에게 특정한 권한과 역할(Role)을 부여하여 보호된 리소스(URL, 경로)에 대한 접근 권한을 관리한다.
#### 3. 세션 관리 (Session Management)
- 웹 애플리케이션의 세션 관리를 보호하고, 세션 공격에 대비하여 세션 관리 기능을 제공한다.
#### 4. CSRF (Cross-Site Request Forgery) 보호
- CSRF 공격으로부터 애플리케이션을 보호하기 위해 CSRF 토큰을 생성하고 검증하는 기능을 제공한다.
---
### SecurityConfig 클래스 파일 생성하기
> - Spring Sercurity 환경 설정을 구성하기 위한 클래스.
> - 웹 애플리케이션의 인증과 권한부여를 관리하며, 특정 URL에 대한 접근 제어, 로그인 및 로그아웃 처리, CSRF 보호 등을 설정 가능하다.
#### 의존성 (gradle)
```
implementation 'org.springframework.boot:spring-boot-starter-security'
```
#### 주요 문법과 기능
- `@Configuration`, `@EnableWebSecurity`
	- Spring Security 설정 클래스임을 명시
	- 웹 보안 활성화
- `HttpSecurity`: HTTP에 대한 보안을 설정
- `SecurityFilterChain`
	- `http.authorizeHttpRequests()`: URL별 접근 권한 설정
		- `permitAll()`: 모든 사용자 접근 허용
		- `hasRole()`: 특정 역할을 가진 사용자만 접근 허용
		- `authenticated()`: 인증된 사용자만 접근 허용
- 로그인/로그아웃 설정
	- `formLogin()`: 폼 기반 로그인 설정
	- `logout()`: 로그아웃 관련 설정
	- `sessionManagement()`: 세션 관리 정책 설정
- CSRF/CORS 설정
	- `csrf().disable()`: CSRF 보호 비활성화
	- `cors()`: CORS 정책 설정
- `PasswordEncoder`
	- 비밀번호 암호화 방식 설정
	- BCrypt 등의 암호화 알고리즘 사용
#### ex
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // CSRF 설정
            .csrf(csrf -> csrf.disable())
            
            // URL별 권한 설정
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/user/**").hasRole("USER")
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            
            // 로그인 설정
            .formLogin(login -> login
                .loginPage("/login")
                .loginProcessingUrl("/login-proc")
                .defaultSuccessUrl("/")
                .permitAll()
            )
            
            // 로그아웃 설정
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/")
                .invalidateHttpSession(true)
            )
            
            // 세션 관리
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .maximumSessions(1)
                .maxSessionsPreventsLogin(false)
            );

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```
---
### SecurityFilterChain
- `SecurityFilterChain`은 Spring Security에서 제공하는 인증, 인가를 위한 필터 모음이다.
- Spring Security에서 가장 핵심이 되는 기능을 제공한다.
	- 대부분의 서비스는 `Security Filter Chain`에서 실행된다고 보면 된다.
- `HttpSecurity`를 통해 설정한 내용은 이 필터 체인에 적용되며, 요청이 들어올 때마다 보안 검사를 수행한다.

#### Filter Chain
- HTTP 요청 -> Web Application Server(Servlet Container) -> 필터1 -> 필터2 -> ... -> 필터n -> Servlet -> 컨트롤러

#### Application Context 초기화
> Application Context 초기화 과정에서 사용자가 정의한 Security Filter Chain이 생성된다.

- `SecurityFilterChain` Bean을 반환하는 `filterChain` 메서드의 매개변수인 `HttpSecurity`를 통해 사용할 필터와 사용자가 직접 정의한 필터를 정의할 수 있다.
---
### HttpSecurity
- Spring Security에서 HTTP 요청에 대한 보안 설정을 구성하는 데 사용하는 클래스.
#### 주요 메서드
- `auhorizeRequest()`: URL 패턴별 접근 권한 설정.
```java
http.authorizeRequests()
    .antMatchers("/public/**").permitAll()
    .antMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated();

// `antMatchers()`: 특정 URL 패턴에 대한 접근 권한 설정.
```

- `formLogin()`: 폼 기반 로그인을 설정한다.
```java
http.formLogin()
    .loginPage("/login")
    .permitAll();
```

- `logout()`: 로그아웃 설정을 구성한다.
```java
http.logout()
    .logoutUrl("/logout")
    .logoutSuccessUrl("/login?logout")
    .permitAll();
```

- `csrf()`: CSRF 보호를 설정한다.
```java
http.csrf().disable(); // CSRF 보호 비활성화
```

- `sessionManagement()`: 세션 관리 설정을 구성한다.
```java
http.sessionManagement()
    .maximumSessions(1)
    .expiredUrl("/login?expired");
```

- `exceptionHandling()`: 예외 처리 설정을 구성한다.
```java
http.exceptionHandling()
    .accessDeniedPage("/403");
```

- `headers()`: HTTP 헤더 설정을 구성한다.
```java
http.headers()
    .frameOptions().sameOrigin();
```

- `httpBasic()`: HTTP 기본 인증을 설정한다.
```java
http.httpBasic();
```

- `rememberMe()`: Remember-Me 인증을 설정한다.
```java
http.rememberMe()
    .key("uniqueAndSecret")
    .tokenValiditySeconds(86400);
```

- `return http.build()`: 
	- HttpSecurity 객체를 통해 보안 설정을 최종적으로 빌드하고, 이를 Spring Security에 적용한다.
---
### 시나리오: 온라인 쇼핑몰 웹 애플리케이션
> 기능
	1. 모든 사용자가 접근할 수 있는 공개 페이지 (ex: 메인페이지, 상품목록)
	2. 로그인한 사용자만 접근할 수 있는 페이지 (ex: 장바구니, 주문내역)
	3. 관리자만 접근할 수 있는 페이지 (ex: 상품등록, 사용자관리)
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests(authorize -> authorize
                .antMatchers("/", "/home", "/products/**").permitAll() // 모든 사용자 접근 허용 -> 로그인하지 않아도 접근 가능
                .antMatchers("/cart", "/orders").hasRole("USER") // 로그인한 사용자만 접근 허용 
                .antMatchers("/admin/**").hasRole("ADMIN") // 관리자만 접근 허용 
                .anyRequest().authenticated() // 그 외 모든 요청은 인증 필요
            )
            .formLogin(form -> form
                .loginPage("/login") // 커스텀 로그인 페이지
                .permitAll() // 로그인 페이지는 모든 사용자 접근 허용
            )
            .logout(logout -> logout
                .logoutUrl("/logout") // 로그아웃 URL
                .logoutSuccessUrl("/login?logout") // 로그아웃 성공 시 이동할 URL
                .permitAll() // 로그아웃은 모든 사용자 접근 허용
            )
            .csrf(csrf -> csrf
                .disable() // CSRF 보호 비활성화 (개발 단계에서만 사용)
            );

        return http.build(); // 설정을 완료하고 SecurityFilterChain 반환
    }

    @Bean
	// 패스워드 "12345" → DB에는 "$2a$10$8Tl5FqR..." 형태로 암호화 저장
	// 데이터베이스 유출되어도 원본 비밀번호 알 수 없음
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // 비밀번호 암호화
    }

}
```
#### SecurityConfig 적용 효과
- SecurityConfig 적용 X
```
- URL만 알면 누구나 관리자 페이지 접근 가능
- 로그인 없이도 주문 페이지 접근 가능 (/orders)
- 악의적인 스크립트 삽입 가능 (XSS 공격)
- 세션 하이재킹 위험
```
- SecurityConfig 적용 O
```
- 관리자 페이지 접근 시도 → 로그인 페이지로 자동 리다이렉트
- 주문 시도 → 먼저 로그인 필요
- XSS 공격 차단
- 세션 탈취 시도 방지
```

### Form Login?
- `forLogin()`은 커스텀 로그인 페이지를 제공하고 로그인 과정에서의 흐름을 제어하기 위해 설정한다.
- 커스텀 디자인이나 특별한 로그인 절차를 구현해야 할 필요가 있을 때 `formLogin()`을 설정하여 로그인 페이지를 사용자 정의한다.

#### 폼 로그인?
- FormLogin은 Spring Security에서 제공하는 인증방식이다.
- 사용자가 웹 페이지에 제공된 로그인 폼을 통해 아이디와 비밀번호를 입력하여 인증을 수행하는 방식.
- Spring Security는 기본적으로 폼 로그인인을 지원하며, 이를 통해 사용자 인증을 처리하고 보안 설정을 제공한다.

#### 로그인 성공 여부를 어떻게 판단하는가?
- Spring Security에서는 AuthenticationManager가 인증을 담당한다.
- 로그인 시 제출된 아이디와 비밀번호를 사용자저장소에 있는 정보와 비교하여 인증 여부를 결정한다.
```java
http
    .formLogin(form -> form
        .loginPage("/login")  // 로그인 페이지 URL
        .permitAll()  // 로그인 페이지는 누구나 접근 가능
        .defaultSuccessUrl("/home", true)  // 로그인 성공 후 리디렉션할 페이지
        .failureUrl("/login?error=true")  // 로그인 실패 시 이동할 페이지
    );
```

#### 로그인 폼이 필요한 이유
1. 커스텀 로그인 페이지 제공
2. 모든 사용자 접근 허용: 
	- `permitAll()`을 사용하여 커스텀 로그인 페이지는 인증된 사용자뿐만 아니라 인증되지 않은 사용자도 접근할 수 있도록 한다.
	- 이 설정이 없다면 사용자가 로그인 페이지에 접근할 수 없게 된다.
3. 로그인 URL 커스터마이징:
	1. `loginPage("/login")`를 설정함으로써 로그인 페이지의 URL을 명확히 정의할 수 있다.

#### 로그인 폼이 필요 없는 경우
- 애플리케이션이 기본 로그인 폼(Spring Security의 자동 제공 로그인 폼)을 사용하고, 디자인이나 절차적 변경이 필요없다면 `formLogin()` 설정이 없어도 된다.
---
> **참고**
> - [01-Spring Security란?](https://www.elancer.co.kr/blog/detail/235)
> - [02-Spring Security 주요 기능](https://kejdev.github.io/spring/2024/01/06/spring-security.html)
> - [03-오키(Security와 JWT를 같이 사용하나요?)](https://okky.kr/questions/1254648)
> - [04-JWT와 OAuth의 적용 사례](https://f-lab.kr/insight/jwt-vs-oauth2)
> - [05-](https://kylo8.tistory.com/entry/Springboot-spring-security-%EA%B8%B0%EB%B3%B8-%EC%84%A4%EC%A0%95-SecurityConfig-%EC%84%A4%EC%A0%95-%ED%9A%8C%EC%9B%90%EA%B0%80%EC%9E%85-%EA%B5%AC%ED%98%84-authorizeHttpRequests-anyRequest-%EC%84%A4%EC%A0%95-%EA%B0%92-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)
> - [06-Spring Security Filter Chain](https://velog.io/@choidongkuen/Spring-Security-Spring-Security-Filter-Chain-%EC%97%90-%EB%8C%80%ED%95%B4)
> - [07-FormLogin](https://velog.io/@seowj0710/Spring-Security-Form-Login-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EC%84%A4%EC%A0%95)
