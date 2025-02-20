---
created: 2025-02-19 17:24
tags:
  - spring
  - springboot
summary: MVC와 WebFlux 방식의 설명과 차이점, 선택 기준
last_modified: 2025-02-20T16:44:00
---
## ⭐ Spring WebFlux와 MVC
- Spring WebFlux는 Spring 5에서 도입된 비동기 웹 프레임워크이다.
- 이전 버전은 Servlet API를 기반으로 동기적 처리 모델을 사용하였으나,
	- 최근에는 높은 트래픽과 데이터량을 효율적으로 처리하기 위해 반응형 웹 프로그래밍을 사용한다.
- 반응형 웹 프로그래밍 패러다임을 구현하기 위한 여러 라이브러리와 표준 중 하나가 Reactor이며, 이를 기반으로 한 것이 Spring WebFlux이다.
---
### 🍪 Spring MVC와 Spring WebFlux
#### 🍬 공통점
- 어노테이션 기반의 프로그래밍 모델이다.
	- `@Controller`, `@RestController`, `RequestMapping` 등 대부분 어노테이션을 그대로 사용할 수 있다.
- Spring Security, Spring Data 등 다른 스프링 생태계와의 호환성을 유지한다.
- DI, AOP, 서비스 추상화 등 핵심 스프링 프레임워크 기능을 제공한다.
#### 🍬 차이점 (Spring WebFlux만의 특징, 장점)
- 비동기와 논블로킹 처리:
	- Servlet API 대신 Reactor API를 사용하여 요청을 처리한다.
- 백프레셔(Backpressure) 지원:
	- 소비자가 처리할 수 있는 만큼만 데이터를 전달하여 시스템 오버로드를 방지한다.
- 다양한 환경 지원:
	- Servlet 컨테이너뿐만 아니라 Netty, Undertow 등 다양한 실행환경에서 동작이 가능하다.
#### 🍬 Spring WebFlux의 단점
- 비동기 프로그래밍은 코드가 복잡해지고 디버깅이 어려울 수 있다.
- 대부분 관계형 데이터베이스 드라이버는 블로킹 방식으로 작동하므로, 이들은 WebFlux와 잘 호환되지 않는다.
- WebFlux와 Reactor API에 익숙해지는 데 시간이 필요하다.
---
### 🍪 GPT 설명
>**Spring MVC**와 **Spring WebFlux**는 웹 애플리케이션을 처리하는 방식에서 차이가 있지만, 기본적으로 **Spring의 설정** 및 **구조**는 비슷합니다. 그러나 **WebFlux**는 **reactive programming** 모델을 채택하고, 비동기적이고 논블로킹 방식으로 동작하기 때문에, 코드에서 사용되는 **API**와 **문법**에 차이가 있습니다.
#### 1. **기본 차이점**
- **Spring MVC**: 전통적인 블로킹 방식의 서버 사이드 애플리케이션.
- **Spring WebFlux**: 비동기적이고 논블로킹 방식으로 동작, 리액티브 프로그래밍 패러다임 사용.
#### 2. **컨트롤러 및 핸들러 메서드 문법 차이**
###### Spring MVC (블로킹 방식):
```java
@Controller
public class MyController {

    @GetMapping("/hello")
    public String hello(Model model) {
        model.addAttribute("message", "Hello, World!");
        return "hello"; // hello.html 뷰로 반환
    }
}
```
- `@Controller`를 사용하여 일반적인 서블릿 기반 컨트롤러를 설정합니다.
- 메서드는 **동기적**으로 실행되며, 결과를 반환하고 그에 맞는 뷰를 렌더링합니다.
###### Spring WebFlux (비동기/논블로킹 방식):
```java
@RestController
public class MyController {

    @GetMapping("/hello")
    public Mono<String> hello() {
        return Mono.just("Hello, World!");  // Mono는 1개의 값 또는 빈 값을 반환하는 비동기형 리액티브 스트림
    }
}
```
- `@RestController`를 사용하여 RESTful API를 설정합니다.
- 메서드는 **Mono** 또는 **Flux**를 반환합니다. `Mono`는 하나의 값을 반환하는 비동기 리액티브 스트림이고, `Flux`는 여러 개의 값을 반환할 수 있습니다.
- **비동기** 방식으로 데이터를 처리하므로, HTTP 요청을 처리하는 동안 스레드를 차단하지 않고 결과를 비동기적으로 반환합니다.
#### 3. **요청 처리 차이**
###### Spring MVC:
```java
@GetMapping("/getData")
public String getData() {
    String data = fetchDataFromDatabase();  // 동기적 호출
    return data;
}
```
###### Spring WebFlux:
```java
@GetMapping("/getData")
public Mono<String> getData() {
    return Mono.fromCallable(() -> fetchDataFromDatabase())  // 비동기 호출
            .subscribeOn(Schedulers.boundedElastic());  // 비동기적으로 데이터를 처리
}
```
- **Spring WebFlux**는 `Mono`와 `Flux`를 사용하여 비동기적으로 데이터 처리 및 외부 시스템과의 상호작용을 처리합니다.
- 데이터베이스나 다른 리소스와 상호작용할 때 비동기적으로 요청을 처리하여 성능을 높입니다.
#### 4. **응답 처리 차이**
###### Spring MVC:
```java
@GetMapping("/hello")
public String hello() {
    return "Hello, World!";  // String 반환 (동기적 응답)
}
```
###### Spring WebFlux:
```java
@GetMapping("/hello")
public Mono<String> hello() {
    return Mono.just("Hello, World!");  // Mono로 감싸서 비동기적으로 처리
}
```
- **Spring MVC**에서는 동기적으로 값을 반환하는 반면, **Spring WebFlux**에서는 비동기적이고 논블로킹 방식으로 `Mono`나 `Flux`를 반환하여 리액티브 처리합니다.
#### 5. **기타 설정 차이점**
- **Spring MVC**: `@EnableWebSecurity`, `@EnableWebMvc` 등 서블릿 기반 설정을 사용합니다.
- **Spring WebFlux**: `@EnableWebFluxSecurity`와 같은 리액티브 보안 설정을 사용합니다.
#### 6. **서버 설정 차이**
- **Spring MVC**는 서블릿 기반 컨테이너(Tomcat 등)에서 실행됩니다.
- **Spring WebFlux**는 **Netty**나 **Undertow**와 같은 비동기 웹 서버를 사용할 수 있습니다. 또한 서블릿 컨테이너를 사용할 수도 있지만, 논블로킹 특성을 살리기 위해 비서블릿 서버를 사용하는 경우가 많습니다.
#### 요약
- **Spring MVC**는 **동기적**이고 **서블릿 기반**의 웹 애플리케이션 모델로 동작하며, 반환 타입으로 `String`, `ModelAndView`, `ResponseEntity` 등을 사용합니다.
- **Spring WebFlux**는 **비동기적**이고 **리액티브** 방식으로 동작하며, 반환 타입으로 `Mono`, `Flux`를 사용합니다. 이러한 타입은 비동기적으로 데이터를 처리하고 스트리밍합니다.
따라서 문법은 **주요 차이점**이 있으며, **WebFlux**에서는 리액티브 프로그래밍을 사용해야 하므로, `Mono`와 `Flux`와 같은 리액티브 타입을 적극적으로 사용해야 합니다.
---
### 🍪 SpringMVC와 SpringWebFlux의 선택
#### 🍬 WebFlux가 유리한 경우 
- 동시 처리 요청이 많고, I/O 작업이 많은 경우
- 리소스 사용량이 제한된 경우
- 리액티브 프로그래밍이 필요한 경우
> 하나라도 Sync or Blocking된 코드가 있다면, MVC보다 느려진다고 한다.

#### 🍬 MVC가 유리한 경우
- 대부분 요청이 빠르게 끝나는 경우
- 스레드 기반의 기존 기술과의 호환이 되는 경우
- 팀원이 WebFlux에 익숙하지 않은 경우
---
> **참고**
> - [01](https://velog.io/@shdrnrhd113/Spring-MVC-Vs.-Spring-WebFlux)
> - [02](https://codenme.tistory.com/123)
> - [03](https://velog.io/@c65621/Spring-WebFlux-%ED%83%84%EC%83%9D-%EB%B0%B0%EA%B2%BD-MVC%EC%99%80%EC%9D%98-%EB%B9%84%EA%B5%90-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%9E%A5%EB%8B%A8%EC%A0%90)
> - [04-MVC와 WebFlux의 차이점](https://pearlluck.tistory.com/726)
> - [📌 05-오키(Webflux 진짜로 대용량 처리에서 MVC 보다 빠를까?)](https://okky.kr/questions/1412539)
> - [06-테스트를 통한 MVC와 WebFlux 비교](https://m.blog.naver.com/joebak/222008524688)



