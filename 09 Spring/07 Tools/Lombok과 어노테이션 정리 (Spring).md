---
created: 2025-02-18 16:43
tags:
  - spring
  - springboot
last_modified: 2025-02-20T16:18:00
---
### 🍪 Lombok 이란?
- 어노테이션 기반으로 코드 자동완성 기능을 제공하는 라이브러리.
- 만약 필드가 50개라면 하나하나 Getter, Setter를 작성하는 것은 매우 번거로운 작업이며, 가독성이 떨어질 것이다.
#### Lombok의 장점
- 어노테이션을 통한 코드 자동 생성 -> 생산성과 편의성 증가
- 코드의 길이 감소 -> 가독성과 유지보수성 증가
---
### Lombok 주요 어노테이션


|어노테이션|설명|
|---|---|
|**@Setter, @Getter**|각각 Getter, Setter 메소드를 자동으로 생성한다.|
|**@ToString**|toString() 메서드를 자동 생성하며, 기본적으로 모든 필드를 포함하지만, `exclude` 속성으로 특정 필드를 제외할 수 있다.|
|**@EqualsAndHashCode**|`equals()` 및 `hashCode()` 메서드를 자동 생성하며, `callSuper` 옵션을 사용하면 부모 클래스의 필드까지 고려할 수 있다.|
|**@Data**|`@Getter + @Setter + @ToString + @EqualsAndHashCode + @RequiredArgsConstructor`를 포함하는 종합 패키지 어노테이션이다.|
|**@NoArgsConstructor**|기본 생성자(매개변수가 없는 생성자)를 자동 생성한다. `force=true`를 사용하면 `final` 필드도 초기화한다.|
|**@RequiredArgsConstructor**|`final` 또는 `@NonNull`이 붙은 필드를 포함하는 생성자를 자동 생성한다.|
|**@AllArgsConstructor**|모든 필드를 포함하는 생성자를 자동 생성한다.|
|**@Builder**|빌더 패턴을 적용하여 객체 생성을 유연하게 할 수 있다. (`@AllArgsConstructor`와 함께 사용하면 필수 필드를 지정 가능)|
|**@Log, @Log4j, @Slf4j**|클래스에 로거(Logger) 객체를 자동으로 생성한다. (`@Slf4j`는 Logback, `@Log4j`는 Log4j 사용)|
|**@SneakyThrows**|`try-catch` 없이 체크 예외(Checked Exception)를 던질 수 있도록 한다. (`Throwable`을 기반으로 예외를 처리)|
|**@Synchronized**|메서드에 동기화(Synchronized)를 적용하여 멀티스레드 환경에서 안전하게 사용할 수 있도록 한다.|
|**@NonNull**|필드, 메서드 파라미터, 리턴값이 `null`이 될 수 없음을 명시한다. (`@RequiredArgsConstructor`에서 활용됨)|
|**@Value**|`@Data`와 유사하지만, 모든 필드를 `private final`로 설정하여 불변(Immutable) 클래스를 생성한다. (`@Getter`, `@ToString`, `@EqualsAndHashCode` 포함)|

---
> **참고 사이트**
> - [01](https://velog.io/@wngus4278/lombok-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-%EC%A0%95%EB%A6%AC)
> - [02](https://lucas-owner.tistory.com/26)
