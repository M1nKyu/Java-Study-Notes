---
created: 2025-02-17 20:17
tags:
  - spring
  - springboot
last_modified: 2025-02-17T20:17:00
---
> [블로그의 글](https://coasis.tistory.com/70) 을 참고하여 구현해본다.([깃허브](https://github.com/Ssspil/blog-code/tree/main/security))
### 🍪 Security Config

#### 🍬 구현 내용
- CSRF 비활성화:
	- 일반적 **RESTful API**에서는 **CSRF 보호가 필요하지 않아** 비활성화한다.
- 폼 로그인 방식 비활성화:
	- RESTful API 방식으로 할 것, **JWT를 이용하므로 폼 로그인을 비활성화한**다.
- HTTP Basic 인증 방식 비활성화:
	- **보안이 약한 방식**이다. 대부분 사용하지 않는다.
- 요청 인증 및 권환 설정:
	- 모두 접근할 수 있는 URL과 일부 URL 요청에 따른 권한을 설정한다.
- 세션 관리 설정:
	- JWT 기반 인증을 사용 -> **세션을 사용하지 않는다**.
#### 🍬 코드
```java
@Configuration // 이 클래스가 설정 클래스임을 나타낸다.
@EnableWebSecurity // Spring Security를 활성화한다.
public class SecurityConfig {

	/*
	📌 BCryptPasswordEncoder
		비밀번호를 안전하게 저장하기 위해 인코딩(해싱)한다.
		해싱하여 DB에 저장할 때 사용한다. 
		로그인 시 입력한 비밀번호와 DB의 해싱된 비밀번호를 비교할 때 사용한다.	
	*/
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
	
	/*
	📌 SecurityFilterChain
		Spring Security의 필터 체인을 구성하는 핵심 메서드이다.
		HttpSecurity는 HTTP 보안 설정을 담당하는 객체이다.
	*/
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception{
    
	    // 📌 보안이 적용될 URL 및 역할 설정하기
        final String[] ALL_URL = new String[]{"/api/user/login", "/api/user/save"};
        final String[] ADMIN_URL = new String[]{"/api/admin"};
        final String[] ROLES = new String[]{UserRole.SUPER.getCode(), UserRole.MANAGER.getCode(), UserRole.ADMIN.getCode()};

		// 📌 CSRF 보안 비활성화
        http.csrf((auth) -> auth.disable()); 

		// 📌 폼 기반 로그인 비활성화
		// API 기반 로그인을 구현할 것이므로, 브라우저에서 제공하는 기본 로그인 폼이 필요없다.
		// 대신 사용자가 로그인하면 JWT 토큰을 발급하고 이를 활용한다.
        http.formLogin((auth) -> auth.disable()); 

		// 📌 URL별 접근권한 설정하기
        http.authorizeHttpRequests((auth) -> auth
                .requestMatchers(ALL_URL).permitAll() // "/api/user/login", "/api/user/save" 요청은 모든 사용자에게 허용된다. 누구나 인증없이 접근할 수 있다.
				.requestMatchers(ADMIN_URL).hasAnyAuthority(ROLES) // "/api/admin" 엔드포인트는 ROLES에 정의된 관리자 권한을 가진 사람만 접근 가능하다.
				.anyRequest().authenticated()); // 위에서 명시적으로 허용한 URL 이외의 모든 요청은 인증된 사용자만 접근 가능하다. 로그인하지 않으면 이외에 접근할 수 없다.
		
		/*
		📌 세션을 사용하지 않도록 설정
			API 서버는 세션을 사용하지 않고 JWT 같은 토큰 기반 인증을 사용해야 한다.
			요청마다 토큰을 보내야 하며, 서버가 세션 정보를 저장하지 않음으로써 확장성이 높아진다.
		*/
        http.sessionManagement((session) -> session.sessionCreationPolicy((SessionCreationPolicy.STATELESS)));

		// 📌 필터 체인 생성: 위의 보안 구성을 바탕으로 Securityfilterchain 객체 생성 후 반환
        return http.build();
    }
}
```

#### 🍬 궁금점
###### ❓ 1. JWT를 이용하는 경우, 왜 폼 로그인을 비활성화하는가?
- **폼 로그인이란**
	- Spring Security에서 기본적으로 제공하는 로그인 방식 중 하나
	- 사용자가 브라우저에서 로그인 페이지를 통해 ID/PW 를 입력하면 서버가 이를 처리하여 인증한다.
	- 기본적으로 `/login` 엔드포인트에서 HTML 폼을 제공하며, **서버에서 세션을 이용하여 로그인 상태를 유지**한다.
- JWT에서 폼 로그인이 필요 없는 이유
	- JWT는 세션을 사용하지 않고 `Stateless` 방식으로 동작한다. ([[JWT|JWT 노트]])
		- 서버가 사용자 **정보를 세션에 저장하는 기본 폼 로그인 방식이 필요 없다**.
	- **REST API 에서는 JSON 데이터를 주고받아**, 브라우저에서 HTML 로그인 폼을 제공할 필요가 없다.
		- 대신 사용자는 `POST /api/user/login` API를 호출하여 로그인하고, **JWT를 받아 클라이언트에서 관리**한다.
	- 폼 로그인 방식은 API 보안 모델과 맞지 않다.
		- 기본적으로 Spring Security의 폼 로그인은 웹브라우저 환경을 가정하므로, API 기반 인증에는 적합하지 않다.
###### ❓ 2. JWT 기반 인증을 사용하면 왜 세션을 사용하지 않는가?
- JWT 기반 인증은 서버가 상태를 관리하지않는다 -> 세션을 사용하지 않는다.
- 명시적으로 세션을 사용하지 않도록 설정하는 이유는 Spring Security의 기본 설정이 세션을 사용하기 때문이다.
```java
http.sessionManagement((session) -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

###### ❓3. (auth) -> auth 가 무엇인가?
```java
http.csrf((auth) -> auth.disable());
```
- 이 구문은 Java의 **람다 표현식**을 사용한 코드이다.
- `(auth) -> auth.disable()`은 `Consumer<HttpSecurity>` 인터페이스의 `accept()` 메서드 구현과 같다.
```java
// 기존 코드를 풀어서 보면
http.csrf(new Consumer<HttpSecurity>() {
    @Override
    public void accept(HttpSecurity auth) {
        auth.disable();
    }
});
```
- `http.csrf()` 메서드는 `Consumer<HttpSecurity>` 타입의 인자를 받는다.

###### ❓ 4. 세션을 사용하지 않도록 설정하는 코드를 자세히 설명
```java
http.sessionManagement((session) -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```
- `sessionManagement()`는 세션을 어떻게 관리할 지 결정하는 설정이다.
- `sessionCreationPolicy(SessioncreationPolicy.STATELESS)`의 옵션
	- `ALWAYS`: 요청이 오면 항상 세션 생성
	- `IF_REQUIRED`: (기본값) 필요한 경우 세션을 생성
	- `NEVER`: 세션을 생성하지 않지만, 기존 세션은 사용 가능
	- `STATELESS`: 세션을 아예 사용하지 않음
---
### 🍪 DB 연결하기
- 블로그에서는 postgreSql을 사용했지만, 나는 mySql을 사용한다.
```properties
spring.application.name=Auth_JWT_Redis

spring.datasource.url=jdbc:mysql://localhost:3306/<DB명>?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=<비밀번호>
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```
---
### 🍪 Model 작성
#### 🍬 User (엔티티)
```java
@Getter                     // lombok.Getter: 자동으롤 Getter 메서드 생성
@Builder                    // lombok.Builder: 빌더 패턴을 자동으로 적용
@Entity                     // jakarta.persistence.Entity: 클래스가 JPA 엔티티임을 나타냄
@NoArgsConstructor          // lombok.NoArgsConstructor: 기본 생성자를 자동으로 생성한다. JPA 프록시 객체를 만들 때 기본 생성자 필요 -> 필수
@AllArgsConstructor         // lombok.AllArgsConstructor: 모든 필드를 포함하는 생성자도 자동으로 생성
@Table(name = "user_info")  // jakarta.persistence.Table: DB에서 "user_info" 테이블과 매핑
public class User {
    
    // jakarta.persistence.Id : 해당 키가 기본 키(PK)임을 나타냄
    // @GeneratedValue(...): 기본 키의 자동 증가 (Auto Increment) 설정 
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;
    private String password;

    @Enumerated(EnumType.STRING) // Enum 타입을 DB에 문자열(String) 형태로 저장
    private UserRole role;
}
```
- `@Builder`: 객체를 생성할 때 유연한 방식으로 필드를 채울 수 있다.
	- ex: `User user = User.builder().name("John").password("secure123").role(UserRole.ADMIN).build();`

| 어노테이션                                                 | 역할                    |
| ----------------------------------------------------- | --------------------- |
| `@Getter`                                             | 모든 필드의 Getter 자동 생성   |
| `@Builder`                                            | 빌더 패턴 적용              |
| `@Entity`                                             | JPA 엔티티 지정            |
| `@NoArgsConstructor`                                  | 기본 생성자 자동 생성          |
| `@AllArgsConstructor`                                 | 모든 필드를 포함하는 생성자 자동 생성 |
| `@Table(name = "user_info")`                          | 테이블명을 `user_info`로 지정 |
| `@Id`                                                 | 기본 키(PK) 지정           |
| `@GeneratedValue(strategy = GenerationType.IDENTITY)` | `AUTO_INCREMENT` 설정   |
| `@Enumerated(EnumType.STRING)`                        | Enum 값을 문자열로 저장       |

#### 🍬 UserRole (Enum)
```java

@Getter // getCode(), getValue() 자동 생성
public enum UserRole implements GrantedAuthority { // Spring Security의 GrantedAuthority 인터페이스 구현 -> 권한 시스템에서 사용할 수 있도록 함.
	
	// UserRole의 4가지 권한
    SUPER("SUPER", "최고관리자")
    , MANAGER("MANAGER", "매니저")
    , ADMIN("ADMIN", "관리자")
    , USER("USER", "일반유저");
	
	/*
	📌 UserRole Enum은 두 개의 필드를 가진다.
		code → 권한 코드 (예: "SUPER", "MANAGER" 등)
		value → 권한 설명 (예: "최고관리자", "매니저" 등)
	*/
    private String code;
    private String value;
	
	// enum의 생성자는 기본적으로 private -> 직접 호출 불가
    UserRole(String code, String value){
        this.code = code;
        this.value = value;
    }
	/*
	📌 GeneratedAuthority 인터페이스 구현 -> Spring Security에서 권한 사용 가능
		getAuthority() 메서드는 UserRole의 code값을 반환한다.
		즉, "SUPER", "MANAGER", "ADMIN", "USER"가 권한 값으로 사용된다.
	*/
    @Override
    public String getAuthority() {
        return this.getCode();
    }
}
```

|어노테이션/구현|역할|
|---|---|
|`@Getter`|`code`, `value` 필드의 Getter 자동 생성|
|`Enum`|고정된 권한 목록 제공|
|`GrantedAuthority`|Spring Security에서 권한 역할 수행|
|`getAuthority()`|`code` 값을 반환하여 Security 권한으로 사용|

#### 🍬 JoinRequestDto (회원가입 DTO)
```java
@Getter
@Setter
@ToString // toString() 메서드 자동 생성
public class JoinRequestDto {

    private String email;
	private String password;
	
	// 주어진 정보를 바탕으로 User 엔티티 객체를 생성하는 메서드
    public User toUserEntity(JoinRequestDto joinRequest){
        return User
                .builder()
                .email(joinRequest.getEmail())
                .password(joinRequest.getPassword())
                .role(UserRole.ADMIN)
                .build();
    }
}
```
- `toString()` 메서드는 객체의 필드 값을 문자열로 변환하여 출력이 가능하다.
#### 🍬 궁금점
###### ❓ Enum 클래스에서 생성자를 직접 작성하지 않고, `@AllArgsConstructor`을 사용하면 안됐는가?
- Enum에 적용할 수 없다.
- Lombok의 `@AllArgsConstructor`는 **클래스에만 적용 가능**하다.
- Enum은 컴파일 타임에 미리 정의된 값만 사용 가능하다 -> Lombok이 동적으로 생성할 수 없다.
- `@Builder`, `NoArgsContsructor`, `AllArgsConstructor` 등 어노테이션은 모두 Enum에서 사용할 수 없다.
---
### 🍪 회원가입 비즈니스 로직
#### 🍬 UserController
> - **ServletUriComponentsBuilder.fromCurrentContextPath()** 를 이용하여 현재 요청의 컨텍스트 경로를 가져온다.
> - 새로 저장된 User 엔티티의 ID값을 `path(...)`경로를 생성해서 Location에 담아 새 리소스의 URI를 클라이언트에게 알려준다.
> - 사용자를 생성했기 때문에 상태코드 Created(201)를 내려준다.
- 위 방법은 치명적임, 이후에 보완한다.
```java
@Slf4j                      // 📌 로깅 기능 제공 (log.info(), log.error() 등)
@RestController             // 📌 @Controller + @ResponseBody, 모든 메서드가 JSON 응답 반환
@RequiredArgsConstructor    // 📌 final이 붙은 필드를 포함하는 생성자를 자동 생성 -> 생성자를 통한 의존성 주입
@RequestMapping("/api")     // 기본 URL 설정, 이 컨트롤러의 모든 API는 /api 경로를 포함
public class UserController {
    
    private final UserService userService;

    /*
    📌 POST /api/user/save 요청이 오면 실행됨 (회원 가입 요청 받음)
	    @RequestBody JoinRequestDto joinRequest: JSON 데이터를 JoinRequestDto 객체로 변환하여 받는다.

     */
    @PostMapping("/user/save")
    public ResponseEntity<User> saveUser(@RequestBody JoinRequestDto joinRequest){
        User newUser = userService.saveUser(joinRequest); // 새로운 사용자를 저장, 저장된 사용자 정보 반환

        /*
        📌 URI.create(...): 새로 생성된 사용자의 URI을 생성
	        ServletUriComponentsBuilder를 사용하여 현재 컨텍스트 경로에 /api/user/{id} 추가
        */
        URI uri = URI.create(ServletUriComponentsBuilder
                .fromCurrentContextPath()
                .path("/api/user/" + newUser.getId())
                .toUriString());

        // HTTP 상태 코드 "201 Created"와 함께 새로 생성된 사용자 정보를 반환한다.
        // Locatioin 헤더에 생성된 사용자의 URI가 포함된다.
        return ResponseEntity.created(uri).body(newUser);
    }
}
```

| **어노테이션/코드**                                | **설명**                                   |
| ------------------------------------------- | ---------------------------------------- |
| `@Slf4j`                                    | 로깅 기능을 제공합니다.                            |
| `@RestController`                           | JSON 응답을 반환하는 컨트롤러입니다.                   |
| `@RequiredArgsConstructor`                  | `final` 필드를 포함하는 생성자를 자동 생성합니다.          |
| `@RequestMapping("/api")`                   | 기본 URL을 `/api`로 설정합니다.                   |
| `private final UserService userService`     | `UserService`를 주입받습니다.                   |
| `@PostMapping("/user/save")`                | POST 요청을 처리합니다.                          |
| `@RequestBody JoinRequestDto joinRequest`   | JSON 데이터를 `JoinRequestDto` 객체로 변환합니다.    |
| `userService.saveUser(joinRequest)`         | 회원 가입 로직을 실행합니다.                         |
| `URI.create(...)`                           | 새로 생성된 사용자의 URI를 생성합니다.                  |
| `ResponseEntity.created(uri).body(newUser)` | HTTP 상태 코드 `201 Created`와 사용자 정보를 반환합니다. |

#### 🍬 UserService
> 구현체가 필요한 인터페이스.
```java
public interface UserService {
    User saveUser(JoinRequestDto joinrequest);
}
```

#### 🍬 UserServiceImpl
```java

@Slf4j                      // 로그를 사용할 수 있게 해주는 Lombok 어노테이션
@Service                    // UserServiceImpl을 서비스 빈으로 등록
@Transactional              // 트랜잭션 관리를 자동으로 처리: saveUser()의 작업이 한 트랜잭션에 처리되도록 해준다.
@RequiredArgsConstructor    // final 선언 필드의 생성자를 자동으로 생성한다 -> 의존성 자동 주입
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;

    @Override
    public User saveUser(JoinRequestDto joinRequest){
        // 📌 1. 이메일 중복 체크 : 존재하면 예외를 던져 중복 가입을 방지한다.
        String email = joinRequest.getEmail();
        Boolean isExist = userRepository.existsByEmail(email);
        if(isExist) throw new EmailExistException(email);

        // 📌 2. 비밀번호 암호화
        String encPassword = bCryptPasswordEncoder.encode(joinRequest.getPassword());
        joinRequest.setPassword(encPassword);

        // 📌 3. JoinRequestDto를 엔티티로 변환: User 객체는 암호화된 비밀번호와 함께 저장
        User user = joinRequest.toUserEntity(joinRequest);

        // 📌 데이터베이스에 User 저장
        return userRepository.save(user);
    }
}
```
> (참고) `throws`는 책임 전가, `throw`는 강제 예외 발생

#### 🍬 UserRepository
```java
@Repository
public interface UserRepository extends JpaRepository <User, Long>{
    Boolean existsByEmail(String email);
}
```
- `existsByEmail(String email)`
	- Spring Data JPA에서 쿼리 메서드 기능을 활용한 메서드이다.
	- DB에서 이메일을 기반으로 사용자가 존재하는지 확인한다.
	- Spring Data JPA는 `existsByEmail` 이름을 보고 자동으로 아래와 같은 SQL 쿼리를 생성한다.
		- `SELECT COUNT(*) > 0 FROM user WHERE email = :email`
- 사용자가 입력한 비밀번호의 암호화는 필수이다.
	- 절대로 평문으로 저장히자 않아야 한다. (`BCryptPasswordEncoder` 사용)
### 🍪 예외 처리
#### 🍬 ErrorResultDto
> 예외 발생 시 클라이언트에게 전달할 에러 응답을 표현하는 DTO
```java
@Getter
@Setter
@AllArgsConstructor
public class ErrorResultDto {
    private Integer code; 
    private String message;
    private Map<String, Object> data;

    public ErrorResultDto(Integer code, String message){
        this.code = code;
        this.message = message;
    }
}
```

#### 🍬 EmailExistException
> 사용자 정의 예외, 이메일이 존재하는 경우에 발생하는 예외를 표현함
```java
public class EmailExistException extends RuntimeException{
    private static final String PREFIX_MESSAGE = "This Email Already Exists: ";

    public EmailExistException(String email){
        super(PREFIX_MESSAGE + email);
    }
}
```

#### 🍬 MyExceptionHandler
> 특정 예외가 발생하면 이를 처리하고 적절한 에러 응답을 클라이언트에 반환한다.
```java
/*
@RestControllerAdvice:
Spring 에서 전역적으로 예외를 처리하는 역할을 하는 어노테이션.
특정 컨트롤러에서 발생한 예외를 처리할 수 있도록 해준다.
@RestController가 붙은 컨트롤러에서 발생한 예외를 처리하도록 설정돼있다.
*/
@Slf4j
@RestControllerAdvice(annotations = RestController.class)
public class MyExceptionHandler {
    
    // 예외 처리 메서드
    public ErrorResultDto emailDuplicationExceptionHandler(EmailExistException e){

        // 예외가 발생한 경우 log.error(...) -> 로그로 에러 메시지를 기록한다.
        // e.getMessage()는 예외 메시지를, e는 예외 객체 자체를 기록한다.
        log.error("EmailEistException >> ", e.getMessage(), e);

        // 클라이언트에게 보낼 에러 응답을 ErrorResultDto 객체로 생성하여 반환한다.
        // HttpStatus.CONFLICT는 409 상태 코드 -> 리소스 충돌(이메일 중복)을 나타낸다.
        // 예외 메시지를 그대로 사용하여 ErrorResultDto에 포함시킨다.
        return new ErrorResultDto(HttpStatus.CONFLICT.value(), e.getMessage());
    }
}
```

#### 🍬 예외 처리 흐름
1. **클라이언트 요청**: 
	- 클라이언트가 이메일을 입력하여 서버에 요청을 보낸다.
2. **이메일 중복 검증**: 
	- 서버에서 이메일 중복 검사를 진행한다. 
	- 만약 이메일이 존재하면, `EmailExistException` 예외가 발생한다. -> `if(isExist) throw new EmailExistException(email);`
3. **예외처리**: 
	- `EmailExistException`이 발생하면 Spring은 이를 `MyExceptionHandler` 클래스에서 정의한 `emailDuplicationExceptionHandler` 메서드로 전달한다.
4. **에러 응답 생성**: 
	- `emailDuplicationExceptionHandler` 메서드는 `ErrorResultDto` 객체를 생성하여 HTTP 응답으로 반환한다.
	- 이 응답은 `code`와 `message`를 포함하여 클라이언트에게 이메일 중복 오류에 대한 정보를 제공한다.

#### 🍬궁금점
###### ❓ 핸들러와 예외, DTO의 관계
- 핸들러는 예외를 처리하고, DTO를 사용하여 클라이언트에게 반환할 응답을 생성한다.
- 예외는 특정 조건에서 발생하며, 이를 핸들러가 처리할 수 있도록 던진다.
- DTO는 핸들러에서 예외를 처리한 후, 클라이언트에게 전달할 에러 메시지와 상태 코드 등을 정의한다.

###### ❓ 예외 핸들러에서 에러 정보를 받환받는 주체는?
- Spring 프레임워크의 **DispatcherServlet**

- 흐름
	1. 클라이언트 요청
	2. 예외 발생 후 `EmailExistsException` 던짐 (throw)
	3. `@RestControllerAdvice` 핸들러 작동
		- 핸들러는 `EmailExistException`을 처리하고 `ErrorResultDto` 반환
	4. `ErrorResultDto` 객체 반환
		- Spring의 DispatcherServlet에 의해 클라이언트로 전달된다.
		- 이 객체는 자동으로 JSON 형태로 직렬화되어 HTTP 응답으로 클라이언트에 반환된다.
	5. 클라이언트는 HTTP 응답 수신
		- 예외 메시지와 `409 Conflict` 상태 코드 반환
---
### 🍪 테스트 하기 (POSTMAN)
> 현업에서는 User Entity를 그냥 내려주면 안된다. `ResponseDto`를 생성하여 중요한 정보는 무조건 제외한 상태로 내려줘야 한다.
```json
// 요청 
POST http://localhost:8080/api/user/save
body: {"email": "myEmail@naver.com", "password":"myPw1234"}
```
- ![[2025-02-18_study_springboot_jwt_redis_test.png|700x400]]

- ![[2025-02-18_study_springboot_auth_jwt_redis_test_2.png|700x250]]
	- Headers의 Location에서 생성된 path를 확인할 수 있다.

| id  | email             | password                                                     | role  |
| --- | ----------------- | ------------------------------------------------------------ | ----- |
| 1   | myEmail@naver.com | $2a$10$Io9zy5ZPFYONi66cQlQ08uYgs7QjMy95JQX536IuGErzLx7mwbYFi | ADMIN |
- 테이블을 조회하면, 위와 같은 결과가 표시된다.
	- role이 `ADMIN`으로 설정되어 있다.
	- `JoinRequestDto`의 `toUserEntity()` 메서드에 모든 요청에 대해 role을 ADMIN으로 설정하게 되어 있다.
	- 권한 부여를 어떻게 구현할 수 있을지도 고려해봐야겠다.
---
## 다음 학습 : [[로그인, 로그아웃, 회원가입의 구현 (JWT + Redis) (2)]]
---
## 📝 추가 학습
- [[Lombok]]
- [[Enum class (Java)]]
- [[Custom Exception (Spring)]]
- [[Runtime Exception (Java)]]
- [[REST API과 JSON]]
- [[RequestBody, ResponseBody 어노테이션 (Spring)]]
---
> **참고**
> - [01](https://coasis.tistory.com/70)
> - [02-Lombok 어노테이션 정리](https://lucas-owner.tistory.com/26)
> - [[03-RequestBody와 ResponseBody]]
> - [04-Spring 예외 처리](https://velog.io/@injoon2019/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8A%A4%ED%94%84%EB%A7%81%EC%97%90%EC%84%9C-%EC%98%88%EC%99%B8-%EC%B2%98%EB%A6%AC)

