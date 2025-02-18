---
created: 2025-01-23 15:50
category:
  - database
  - spring
tags:
  - TIL
last_modified: 2025-01-23T17:07:00
---
## ⭐ MyBatis와 JPA
> - Java 애플리케이션에서 데이터베이스를 사용하는 프레임워크로 가장 많이 쓰이는 기술은 **MyBatis**와 **JPA**이다.
> - 두 기술은 모두 데이터를 관계형 데이터베이스에 저장시킨다는 측면에동일하지만, 서로 다른 접근 방식을 채택하고 있다.

> - `SQL Mapper : 개발자가 작성한 SQL 실행 결과를 객체에 매핑시켜주는 프레임워크.`
> - `ORM : 객체와 DB의 데이터를 자동으로 매핑시켜주는 프레임워크.`

---
### 🍪 MyBatis란
#### 🍬 개념
- **SQL Mapper Framework**.
- SQL을 명시적으로 작성하여 데이터베이스와 상호작용 한다.
- XML 파일을 통해 쿼리를 작성하거나 어노테이션으로 SQL을 정의한다.
- Java 소스에서 SQL문을 별도의 XML 파일로 저장 -> 이 둘을 연결시켜주는 기능 제공.

```java
@Mapper
public interface UserMapper {
	@Select("SELECT * FROM users WHERE id = #{id}")
	User getUserById(int id);
}
```

```xml
<select>
	SELECT * FROM users WHERE id = #{id}
</select>
```

#### 🍬 특징
> JDBC를 통해 데이터베이스에 액세스하는 작업을 캡슐화하고, 일반 SQL 쿼리, 저장 프로시저 및 고급 매핑을 지원하며 모든 JDBC 코드 및 매개변수의 중복 작업을 제거(간소화)한다.

- SQL을 직접 작성 -> 쿼리에 대한 제어가 높다.

- 복잡한 SQL 쿼리 작성과 최적화에 유리하다.

- Java 코드와 SQL 매핑
	- **XML 파일이나 어노테이션**으로 SQL과 Java 메서드를 매핑한다.
	- Java 메소드 선언과 SQL문만 작성하면, MyBatis가 자동으로 둘 간 연결을 시켜준다.

- 동적 SQL 생성 기능
	- Dynamic SQL 생성 기능을 제공하여 프로그램 실행 중에 입력되는 파라미터에 따라 서로 다른 SQL 문을 동적으로 생성해내는 기능을 제공한다.
	- MyBatis 내에 `if`, `choose`, `when`, `otherwise`, `foreach`등의 문법을 지원해서 동적인 SQL문 생성이 가능하다.
```sql
# 예시
SELECT * FROM BOARD WHERE AUTHOR = ‘파라미터’

SELECT * FROM BOARD WHERE TITLE LIKE ‘%파라미터%’

SELECT * FROM BLOG WHERE state = ‘active’ AND title like #{title}
```
---
### 🍪 JPA (Java Persistence API)
#### 🍬 개념
- **ORM**(Object Relational Mapping) 기술.
- Java **객체와 관계형 데이터베이스 간의 매핑**을 위한 API.
- 데이터베이스와의 상호작용을 객체지향적으로 처리한다.
- SQL을 자동으로 생성하고 관리해서 SQL Query에 대한 부담이 적다.
- 따라서 비즈니스 로직에 집중할 수 있고, 코드 가독성을 높일 수 있다.

```java
@Entity
public class User {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	private String name;
	...
}
```

```java
// JPA Repository Interface
public interface UserRepository extends JpaRepository <User, Long> {}
```

> ORM 기술을 실제 구현해서 만들어진 프레임워크가 [[JDBC, JPA, Hibernate, Spring Data JPA#🍪 Hibernate|Hibernate]]이다.
#### 🍬 특징
- 완전한 **ORM**
	- 객체(Entity)와 데이터베이스 테이블 간의 매핑을 자동으로 관리한다.
- **JPQL** (Java Persistence Query Language)
	- SQL과 유사한 쿼리 언어를 제공하며, 객체 중심으로 데이터를 조회할 수 있다.
- 생산성
	- CRUD같은 **반복 작업을 자동화** -> 개발속도를 높인다.
- **캐싱**
	- 영속성 컨텍스트(1차 캐시)를 통해 성능을 최적화한다.

- 객체와 데이터베이스 간 **패러다임 불일치를 해결**한다.
	- 테이블 간의 매핑을 설정하는 것만으로 JPA가 객체의 상태를 데이터베이스에 맞춰 저장하고 관리해 준다.

- MyBatis와 다르게 SQL문의 작성이 불필요하다.
---
### 🍪 MyBatis vs JPA

|기준|MyBatis|JPA|
|---|---|---|
|데이터베이스|튜닝 및 최적화 세밀한 제어 가능|제한적|
|복잡한 쿼리 처리|직접 작성|JPQL 사용|
|개발 생산성|낮음|높음|
|성능|높음|상대적으로 낮음|
|대규모 트래픽 처리|유리|트랜잭션 병목 발생 가능|
|객체 지향 모델링|제한적|우수|

#### 🍬 [선택 기준](https://jimoou.github.io/database/2024/10/11/post15.html)
1. **대규모 트래픽 처리와 병렬성**
	- *JPA*는 기본적으로 트랜잭션 범위 내에서 멀티스레드 안정성을 보장하기 위해 EntityManage 같은 구성요소를 통해 *동기화 매커니즘*을 사용한다.
		- 이로 인해 데이터 일관성과 안정성이 보장되지만,
			- 다수의 쓰레드가 *동일한 엔티티에 접근할 때 Lock* 경합이 발생할 수 있다. 
				- 이로 인해 *성능 병목*이 발생할 수 있다.
	- *MyBatis*는 다수의 쓰레드가 *단순히 데이터베이스와 SQL을 통해 직접 통신*하는 구조이기 때문에 동시성 제어가 필요없는 경우 MyBatis가 더 나은 성능을 보일 수 있다.

2. 설계 복잡성과 코드 관리
	- 개발에 있어서 오직 **성능만이 고려되는 것은 아니다**.
	- **인적자원, 개발속도, 유지보수, 가독성** 등 많은 요소들이 프로젝트에서 집약적으로 고려된다.
	- 테이블, SQL Query, Model Class를 생각했을 때 **MyBatis는 유연성이 매우 떨어진다**.
	- 또한 협업 시 알 수 없는 쿼리의 목적, 조인문을 통한 쿼리, 떨어지는 가독성, SQL에 대한 이해도 요구, 디버깅 등의 어려움이 있고, 성능이 좋은 만큼 수동으로 세팅해야 할 요소도 많다. 
---
## 🧙‍♂️ 요약
- MyBatis와 JPA 모두
	- Java 애플리케이션에서 데이터베이스와 상호작용하기 위한 기술이다.
	- 데이터베이스에 데이터의 CURD 작업을 수행한다.

- MyBatis
	- **SQL Mapper**
	- 복잡한 SQL 작업과 데이터베이스 튜닝에 적합하다.

- JPA
	- **ORM**
	- SQL 작성이 불필요하다.
	- 데이터베이스의 객체지향적인 설계가 가능하다.

- 선택 기준
	- 복잡한 SQL 제어와 성능 튜닝 필요: MyBatis
	- 자동화와 객체 지향적인 접근 필요: JPA

> 프로젝트에서 기술을 선택하고 사용하기 전 **왜 이 기술을 선택했는가에 대한 명확한 이유**를 고민하자.
---
> **참고사이트**
> - [01-MyBatis vs JPA](https://www.elancer.co.kr/blog/detail/231)
> - [02-MyBatis란](https://khj93.tistory.com/entry/MyBatis-MyBatis%EB%9E%80-%EA%B0%9C%EB%85%90-%EB%B0%8F-%ED%95%B5%EC%8B%AC-%EC%A0%95%EB%A6%AC)
> - [📌 03-MyBatis, JPA 선택 기준](https://jimoou.github.io/database/2024/10/11/post15.html)
