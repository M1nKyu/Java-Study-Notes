---
created: 2025-02-20 20:10
summary: spring security 주요 구조 및 동작 과정
tags:
  - spring
  - springboot
last_modified: 2025-02-20T20:11:00
---
## ⭐ Spring Security의 구조와 동작 과정
### 🍪 Spring Security의 주요 구조
#### ![image||600x350](https://ucarecdn.com/214ecf66-9e12-4579-8a1b-48b4bc86077a/-/stretch/off/-/resize/2200x/-/format/webp/)
- [이미지 출처](https://hyperskill.org/learn/step/27770)

#### 🍬 1. Security Filter Chain
- Spring Security는 서블릿 필터 체인을 기반으로 동작한다.
- 클라이언트의 요청은 여러 보안 필터를 거쳐 최종적으로 애플리케이션에 도달한다.
- 주요 필터:
	- `SecurityContextPersistenceFilter`: SecurityContext를 유지
	- `UsernamePasswordAuthenticationFilter`: 폼 기반 로그인 처리
	- `FilterSecurityInterceptor`: 권한 부여 처리
	- `ExceptionTranslationFilter`: 인증/인가 실패 예외 처리
#### 🍬 2. AuthenticationManager
- 인증을 처리하는 핵심 인터페이스.
- 사용자 정보(ID, PW)를 검증하여 인증 성공 여부를 판단한다.
- `ProviderManager`가 구현체로, 여러 `AuthenticationProvider`를 가질 수 있다.
	- `AuthenticationProvider`: 실제 인증 로직을 처리.
- `DaoAuthenticationProvider`가 기본적으로 사용되며, `UserDetailsService`를 통해 사용자 정보를 조회한다.
#### 🍬 3. UserDetailsService & UserDetails
- 사용자 정보를 로드하는 인터페이스.
- 데이터베이스나 다른 소스에서 사용자 정보를 조회합니다.
- `UserDetailsService`는 `UserDetails` 객체를 반환하며, DB에서 사용자 정보를 조회하는 역할을 한다.
- 비밀번호 검증은 `PasswordEncoder`를 통해 처리한다.
#### 🍬 4. SecurityContext & SecurityContextHolder
- SecurityContext: 인증된 사용자 정보를 저장하는 객체.
- SecurityContextHolder: 현재 요청의 보안 컨텍스트를 저장하고 관리한다.
- 인증이 성공하면 `Authentication` 객체가 `SecurityContext`에 저장된다.
#### 🍬 AccessDecisionManager & Voter
- 인가(Authorization) 처리를 담당한다.
- 특정 요청을 수행할 수 있는 권한이 있는지 검사한다.
- 여러 개의 `AccessDecisionVoter`(투표자)가 모여 투표를 진행하고, 승인 여부를 결정한다.
---
### 🍪 Spring Security의 동작 과정
```
1. 로그인 요청 → Security Filter가 요청을 가로챔
2. AuthenticationManager가 인증 수행 (UserDetailsService 활용)
3. 인증 성공 시 SecurityContext에 저장
4. AuthenticationSuccessHandler 실행 (성공 시 리다이렉트 / 토큰 발급)
5. AuthenticationFailureHandler 실행 (실패 시 로그인 페이지로 이동 / 오류 메시지 반환)
6. 인가 검사 → 권한이 있는지 확인 (AccessDecisionManager)
7. 권한이 있으면 요청 처리, 없으면 403 응답
```
#### 🍬 1. 사용자의 로그인 요청
- 사용자가 `/login` 엔드포인트를 통해 아이디와 비밀번호를 입력하고 로그인 요청을 보낸다.
#### 🍬 2. Security Filter Chain이 요청을 가로챔
- 요청이 `UsernamePasswordAuthenticationFilter`를 통과하면서 인증(Authentication) 과정이 시작된다.
- 사용자의 ID/PW 가 포함된 `UsernamePasswordAuthenticationToken` 객체가 생성된다.
#### 🍬 3. AuthenticationManager가 인증 수행
- `AuthenticationManager`는 `AuthenticationProvider`를 호출하여 사용자 인증 정보를 검증한다.
- `DaoAuthenticationProvider`는 `UserDetailsService`를 통해 DB에서 사용자를 조회하고(`loadUserByUsername()`), 비밀번호를 확인한다.
#### 🍬 4. 인증 성공 시 SecurityContext에 저장
- 인증이 성공하면, `Authentication` 객체가 `SecurityContextHolder`에 저장된다.
- 이후 요청에서 사용자의 인증 정보가 유지된다. (세션 or JWT 활용)
#### 🍬 5. 인가(Authorization) 검증 (권한 부여)
- 사용자가 특정 API (ex: `/admin`)에 접근하면, `FilterSecurityInterceptor`가 권한 검사를 수행한다.
- `AccessDecisionManager`가 현재 사용자의 Role(역할)과 접근 권한을 비교하여 접근 가능 여부를 결정한다.
#### 🍬 6. 인가 성공 시 요청 처리
- 권한 검사가 통과되면 컨트롤러에서 요청을 정상 처리한다.
- 만약 권한이 없으면 403 Forbidden 응답을 반환한다.

#### ![image|700x300](https://ucarecdn.com/a84f232e-6b7b-4f5a-b60e-b0b318d215bd/-/stretch/off/-/resize/2200x/-/format/webp/)
- [이미지 출처](https://hyperskill.org/learn/step/27770)
---
> **참고**
> - [01](https://myeongju00.tistory.com/88)
> - [02](https://hyperskill.org/learn/step/27770)
