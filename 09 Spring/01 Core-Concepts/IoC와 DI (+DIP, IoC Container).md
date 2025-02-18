---
created: 2025-02-04 18:24
category:
  - design-pattern
  - spring
tags:
  - TIL
last_modified: 2025-02-04T22:09:00
---
## ⭐ IoC, DIP, DI, IoC Container 
- ![이미지|600x400](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*bl03kmmTIbWHbfFCsW63Sg.png)
	- [이미지 출처](https://medium.com/sjk5766/dependency-injection-ioc-dip-ioc-container%EC%A0%95%EB%A6%AC-310885cca412)
- `IoC`와 `DIP`는 특정 원리에 대한 Guideline을 제공하지만, 디테일한 구현 방법을 제공하진 않는다.
- `DI`는 Design Pattern, `IoC Container`는 Framework이다.
---
### 🍪 IoC (제어의 역전, Inversion of Control)
- IoC는 프로그래밍에서 **제어의 흐름을 외부에서 관리하는 디자인 패턴**이다.
- IoC는 **객체의 생성 및 생명주기 관리** 등을 개발자가 아닌 **외부**(프레임워크, 컨테이너 등)에게 위임한다.
- 객체 간 결합도를 낮추고 유연한 구조를 제공한다.

#### 🍬 IoC를 구현하는 디자인 패턴들
- Factory Pattern
	- 객체 생성을 전담하는 Factory 클래스에서 객체를 반환
	- 직접 객체를 생성하는 것이 아닌, Factory가 객체를 생성해주는 방식.
- Service Locator Pattern
	- 특정 서비스를 호출할 때 Service Locator를 통해 객체를 가져오는 방식.
- **Dependency Injection(DI)**
	- 객체의 의존성을 외부에서 주입하는 방식.
	- 현재 가장 많이 사용하는 IoC 구현 방식이다.
---
### 🍪 DIP (Dependency Inversion Principle)
- **SOLID 원칙** 중 하나. 
- 상위 모듈(High-level module)이 하위 모듈(Low-level module)에 직접 의존하지 않고, **추상화(인터페이스 또는 추상 클래스)에 의존해야 한다**는 원칙이다.
#### 🍬 전통적 방식 (DIP 위반)
- `WeatherApp`은 `WeatherService`에 직접 의존한다. (**강한 결합**, High Coupling)
- 만약 `WeatherService`의 구현이 바뀌면 `WeatherApp`도 변경해야 한다. (유지보수의 어려움)
```java
public class WeatherService {
    public String getWeather() {
        return "Sunny";
    }
}

public class WeatherApp {
    private WeatherService weatherService; // 하위 모듈에 직접 의존

    public WeatherApp() {
        this.weatherService = new WeatherService(); // 📌 직접 객체 생성 (강한 결합)
    }

    public void showWeather() {
        System.out.println("Weather: " + weatherService.getWeather());
    }
}
```

#### 🍬 DIP 적용 (인터페이스 사용)
- `WeatherApp`은 `WeatherService` **인터페이스에 의존**한다. (**추상화에 의존**)
- `SunnyWeatherService` -> `RainyWeatherService`로 변경해도 `WeatherApp`의 코드 변경이 필요없다.
- 유연한 구조를 유지할 수 있고, OCP(개방-폐쇄 원칙)도 만족한다.
```java
public interface WeatherService {
    String getWeather();
}

// 구현체 1
public class SunnyWeatherService implements WeatherService {
    public String getWeather() {
        return "Sunny";
    }
}

// 구현체 2
public class RainyWeatherService implements WeatherService {
    public String getWeather() {
        return "Rainy";
    }
}

// 상위 모듈 (WeatherApp) - 인터페이스에 의존
public class WeatherApp {
    private final WeatherService weatherService;

    // 📌 의존성 주입 (DI 적용)
    public WeatherApp(WeatherService weatherService) {
        this.weatherService = weatherService;
    }

    public void showWeather() {
        System.out.println("Weather: " + weatherService.getWeather());
    }
}

// 실행 예제
public class Main {
    public static void main(String[] args) {
        WeatherService weatherService = new SunnyWeatherService(); // 📌 구현체 직접 생성
        WeatherApp weatherApp = new WeatherApp(weatherService); // 📌 직접 주입
        weatherApp.showWeather();
    }
}
```
> 여전히 `WeatherService`구현체를 **수동으로 주입해야하는 번거러움**이 있다. -> **DI를 추가하여 Spring이 자동으로 객체를 관리**하도록 개선 가능.
---
### 🍪 DI (Dependency Injection)
> `의존성`: 한 객체가 다른 객체를 사용할 때, 이를 의존한다고 말한다.
- **IoC를 구현하는 디자인 패턴** 중 하나이다.
- 객체가 필요로 하는 의존성(다른 객체)을 외부에서 주입받는 방식이다.
	- 코드의 유연성과 유지보수성 향상.
- Spring에서 `IoC Container`를 활용하여 주입을 수행한다.

#### 🍬 DI의 유형 (주입 방식)
1. 생성자 주입 (Constructor Injection)
```java
@Component
public class ClothingService {
    private final WeatherService weatherService;

	// 📌
    @Autowired
    public ClothingService(WeatherService weatherService) {
        this.weatherService = weatherService;
    }
}
```
2. 세터 주입 (Setter Injection)
```java
@Component
public class ClothingService {
    private WeatherService weatherService;

	// 📌
    @Autowired
    public void setWeatherService(WeatherService weatherService) {
        this.weatherService = weatherService;
    }
}
```
3. 필드 주입 (Filed Injection)
```java
@Component
public class ClothingService {
	// 📌
    @Autowired
    private WeatherService weatherService;
}
```

#### 🍬 위 [[IoC와 DI (+DIP, IoC Container)#DIP 적용|DIP 적용]] 예시에서 DI까지 적용 (DIP + DI)
- **DIP 원칙을 적용할 때, DI를 함께 사용하면 더욱 효과적**이다.
- Spring이 `SunnyWeatherService`객체를 자동으로 생성하고 `WeatherController`에 주입한다.
	- `new`로 객체를 **직접 생성하고, 주입할 필요가 없어진다**.
- `WeatherService`의 구현체만 변경하면 전체 시스템 수정없이 유연하게 동작한다.
```java
@Component // ➡️ SunnyWeatherService 객체를 Spring이 자동으로 관리
public class SunnyWeatherService implements WeatherService {
    public String getWeather() {
        return "Sunny";
    }
}

@RestController
@RequestMapping("/weather")
public class WeatherController {
    private final WeatherService weatherService;

    @Autowired // ➡️ WeatherService 구현체를 WeatherController에 자동 주입 
    public WeatherController(WeatherService weatherService) { // 생성자 주입 (DI 적용)
        this.weatherService = weatherService;
    }

    @GetMapping
    public String getWeather() {
        return weatherService.getWeather();
    }
}
```
---
### 🍪 적용 단계별 정리

| 단계                    | 문제점                                                   | 개선점                                                                        |
| --------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------- |
| **DIP 미적용 (강한 결합)**   | - 새로운 기능 추가 시 기존 코드 변경 필요                             |                                                                            |
| **DIP 적용 (인터페이스 도입)** | - 객체를 수동으로 생성해야 하고, 주입 방식이 번거로움  <br>- 객체 관리를 직접 해야 함 | - 특정 구현체가 아니라 **인터페이스에 의존**<br>- 다양한 구현체를 쉽게 교체 가능  <br>- OCP(개방-폐쇄 원칙) 만족 |
| **DI 적용 (Spring)**    |                                                       | - Spring이 **객체 생명주기를 관리**하고 자동으로 주입  <br>- 유지보수와 테스트가 편리해짐                 |

---
### 🍪 IoC Container
> IoC 컨테이너는 **DI를 구현하는 프레임워크**이다.
- **자동으로 의존성을 주입**하기 위한 프레임워크이다.
- 객체의 생성, 생명 주기, 의존성 있는 객체를 다른 객체에 주입하는 역할을 한다.
- 개발자가 의존성 주입을 수동으로 할 필요가 없다.

#### 🍬 Spring 에서의 IoC Container (Spring Container)
- Spring에서의 두가지 주요 IoC 컨테이너는 `BeanFactory`와 `ApplicationContext`가 있다.
- `BeanFactory`는 가장 기본적인 IoC 컨테이너로, `Lazy-loading` 방식을 사용하여 Bean을 필요할 때 생성한다.
- `ApplicationContext`는 `BeanFactory`를 확장한 보다 고급 IoC 컨테이너이다.
	- 즉시 초기화(Eager-loading)을 기본으로 하고, 다양한 부가 기능을 제공한다.
- SpringBoot에서는 **일반적으로 ApplicationContext를 사용**한다.
	- 대표적 구현체: `AnnotationConfigApplicationContext`, `ClassPathXmlApplicationContext` 등

> (참고) Spring Container가 관리하는 객체를 `빈(Bean)`이라고 한다.
---
## 🧙‍♂️ 요약
- **IoC (제어의 역전, Inversion of Control)**: 
	- 객체의 생성과 제어 권한을 개발자가 아닌 프레임워크(Spring 등)에 위임하는 개념.
- **DIP (의존성 역전 원칙, Dependency Inversion Principle)**: 
	- 상위 모듈이 하위 모듈에 직접 의존하지 않고, 추상화(인터페이스)에 의존해야 한다는 원칙.
- **DI (의존성 주입, Dependency Injection)**: 
	- IoC를 구현하는 디자인패턴 중 하나로, 객체의 의존성을 외부에서 주입하는 방식. 생성자 주입, 세터 주입, 필드 주입 방식이 있음.
- **IoC Container**: 
	- DI를 구현하는 프레임워크.
	- 자동으로 의존성을 주입하기 위한 프레임워크.
	- Spring에서 `BeanFactory`와 `ApplicationContext`가 주요 컨테이너이며, Spring Boot에서는 보통 `ApplicationContext`를 사용.
---
> **참고사이트**
> - [01-블로그(DI, IoC, DIP, IoC container)](https://medium.com/sjk5766/dependency-injection-ioc-dip-ioc-container%EC%A0%95%EB%A6%AC-310885cca412)

