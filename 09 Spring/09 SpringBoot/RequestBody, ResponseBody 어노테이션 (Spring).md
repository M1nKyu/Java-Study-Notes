---
created: 2025-02-18 19:34
tags:
  - spring
  - springboot
last_modified: 2025-02-20T17:28:00
---
## ⭐ @RequestBody, @ResponseBody
> 스프링 프레임워크에서 비동기 통신, 즉 API 통신을 구현하기 위해 `@RequestBody`와 `@ResponseBody` 어노테이션을 사용한다.
### 🍪 @RequestBody
> JSON 기반의 HTTP Body를 자바 객체로 변환한다.
```java
// 객체
public class Entity{
	private Long id;
    private String name;
    private String address;
}

// API Controller
@PostMapping("/api/post")
public void requestTest(@RequestBody Entity entity) {
    System.out.println("id = " + entity.id);
    System.out.println("name = " + entity.name);
    System.out.println("address = " + entity.address);
}
```
- 인자에서 사용이 가능하다.
---
### 🍪 @ResponseBody
> 자바 객체를 JSON 기반의 HTTP Body로 변환한다.
- `@RestController`를 사용하는 경우 반환값에 자동으로 `@ResponseBody` 어노테이션이 붙게 된다.
	- 따라서 자바 객체를 반환하면 알아서 JSON 형식으로 매핑해서 응답한다.
```java
// @Conteoller + @ResponseBody 를 통해 JSON으로 직렬화되어 HttpResponse 객체로 전달한다.

@Controller
public class HelloResource {

	@ResponseBody
	@PostMapping("/api/test")
	public HelloDTO showHelloDTO(@RequstBody HelloDTO dto){
		return dto;
	}
}
```
---
### 🍪 @RestController
- 위와 같이 `@Controller + @ResponseBody` 형식으로 작성을해서 Json API를 만들 수도 있지만,
- `@RestController`를 가지고도 위와같은 효과를 가질 수 있다.
- `@RestController` = `@Controller + @ResponseBody`이기 때문이다.
---
> **참고**
> - [01](https://cheershennah.tistory.com/179)
> - [02-@RequestBody, @ResponseBody 이해하기](https://velog.io/@nomonday/Spring-RequestBody-ResponseBody-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)
