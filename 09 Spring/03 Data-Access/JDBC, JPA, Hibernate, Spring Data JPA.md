---
created: 2025-01-23 17:06
category:
  - database
  - spring
tags:
  - TIL
last_modified: 2025-01-23T18:34:00
---
## ⭐ JDBC, JPA, Hibernate
- ![Image|400x300](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4lkueej7s72jhmsccjlw.png)

### 🍪 JDBC (Java Database Connectivity)
- 데이터베이스와 통신하기 위한 API.
- **가장 기본적**이고 직접적인 방법.
- 모든 Java의 Data Access 기술의 근간  
	- 즉, **모든 Persistence Framework는 내부적으로 JDBC API를 이용**한다
- JDBC를 사용하면 개발자가 직접 **SQL 쿼리를 작성**하고 실행해야 한다.
#### 🍬 특징
- 직접적인 SQL 쿼리 실행: SQL를 직접 작성한다.
- 높은 유연성: 모든 유형의 쿼리 및 데이터베이스 조작이 가능하다.
- 저수준 API: 데이터베이스 작업을 처리하기 위해 많은 양의 코드가 필요하다.
- 데이터베이스의 **종류에 상관없이 똑같은 코드로 해결** 
	- ![Image|600x400](https://codingnomads.com/images/91623cb8-cb37-474a-e930-dbe65fcf9800/public)
#### 🍬 JDBC 프로그래밍의 흐름
1. JDBC 드라이버 로드
2. DB 연결
3. DB에 데이터의 CRUD 수행
4. DB 연결 종료

> **JDBC 드라이버?**
> - **DBMS와의 통신**을 당당하는 자바 클래스(구현체)

#### 🍬 JdbcTemplate
- **Spring Framework의 일부로 제공**되는 JDBC의 확장.
- JDBC의 저수준 API를 **추상화하여 효율적으로 데이터베이스와 상호작용**을 가능하게 한다.

```
List<User> users = jdbcTemplate.query("SELECT * FROM users", 
                                        (rs, rowNum) -> new User(rs.getString("username")));
```

- 특징
	- 간결한 코드
	- 자동 리소스 관리: 커넥션 생성, 예외 처리, 리소스 해제 등 작업을 자동으로 처리
	- SQL 파라미터화: 자동으로 SQL 쿼리 파라미터를 처리 -> SQL Injection 공격 방지.
---
### 🍪 [[MyBatis와 JPA#^1524e3|JPA]] (Java Persistence API)
- ORM을 위한 Java 표준 API.
- **ORM을 구현하기 위한 표준 인터페이스**
- 객체와 데이터베이스간 매핑을 자동으로 처리한다.
- 인터페이스이므로 이를 구현할 수 있는 기술들이 필요하고, 가장 대표적으로는 `Hibernate`가 있다
	- 구현체를 사용하여 실제 데이터베이스와 상호작용한다.
#### 🍬 특징
- **객체** 지향적 접근
- **자동화된 쿼리 생성**
- **캐시** 및 **지연 로딩**

#### 🍬 장점
- 객체와 데이터베이스 간 패러다임 불일치 해결
	- 테이블 간의 매핑을 설정하는 것만으로 JPA가 객체의 상태를 데이터베이스에 맞춰 저장하고 관리해 준다

- 생산성 향상
	- 기본적 SQL 쿼리를 자동으로 작성해준다.

- 유지보수성 향상
	- JPA는 데이터베이스 **구조가 변경되어도 대부분의 경우 알아서 처리**한다.
	- 객체 중심의 코드 -> 가독성이 좋고 이해하기 쉽다.

- 데이터베이스 독립성
	- JPA는 데이터베이스에 독립적이므로 **데이터베이스 변경 시 코드 수정이 최소화**된다

- 성능 향상
	- **지연 로딩**, **쓰기 지연**, **1차 캐시** 등 다양한 성능 최적화 기능을 제공 (+배치 처리, 쿼리 캐시)
	- `지연 로딩: 필요할 때까지 관련된 엔티티를 로딩하지 않음, 필요한 시점에만 데이터를 가져오는 방식`
	- `1차 캐시: 영속성 컨텍스트 내에 1차 캐시를 유지 -> 반복 조회 요청시 데이터베이스에 여러 번 접근하는 것을 방지`

#### 🍬 동작 원리
> JPA는 애플리케이션과 JDBC 사이에서 동작한다. (JDBC API를 이용하여 데이터베이와 통신)

1. 엔티티 상태 변경 감지 (Dirty Checking)
	- JPA의 Persistence Context(영속성 컨텍스트)가 엔티티의 최초 상태를 스냅샷으로 보관
	- 트랜잭션 커밋 시점에 현재 엔티티와 스냅샷을 비교해 변경사항 확인
2. 엔티티를 SQL로 변환
	- 변경된 엔티티의 속성을 분석
	- 해당 변경사항에 맞는 **적절한 SQL 쿼리 자동 생성**
	- Hibernate의 경우 HQL(Hibernate Query Language)을 SQL로 변환
3. JDBC를 통한 데이터베이스 통신
	- 생성된 SQL을 **JDBC Connection을 통해 데이터베이스로 전송**
	- PreparedStatement 등을 사용해 실제 데이터베이스 작업 수행
4. 트랜잭션 관리
	- 데이터베이스 작업의 원자성 보장
	- 여러 데이터베이스 작업을 하나의 논리적 작업 단위로 처리
	- 성공 시 커밋, 실패 시 롤백 수행

---
### 🍪 Hibernate
- **JPA의 구현체**.
- JPA를 사용하기위해선 Hibernate를 사용하면 된다.
- 따라서 이도 **JDBC API를 내부적으로 사용**하고, **JPA의 특징, 장점을 그대로** 가지고 우리가 직접 사용할 수 있게 해 준다.
- 자바객체를 통해 데이터베이스가 Oracle, MySql, MSSQL 등 에 상관없이 다룰수 있도록 하는 추상화를 목표로 한다.

#### 🍬 특징
> JPA 구현체이므로 JPA의 특징과 동일하다.
- **ORM 기능** : 객체와 관계형 데이터베이스 간의 매핑
- **JDBC 추상화**
- **JPQL**(Java Persistence Query Language), 네이티브 SQL, Querydsl 등 **다양한 쿼리 언어 지원**
- **캐싱 기능**
- **데이터베이스 독립성**
---
### 🍪 Spring Data JPA
- **JPA를 한 단계 더 추상화**하여 개발 용이성을 상당히 올려주는 인터페이스.
- **[[Boilerplate Code]]를 줄이는 것이 목표**이다.
- 기본적으로 스프링 데이터는 ORM 공급자로 Hibernate를 사용한다. (변경 가능)
- Spring Data JPA는 `Repository` 인터페이스를 제공한다.
- JpaRepository를 확장하는 인터페이스를 만들기만 하면 `save()`, `update()`, `delete()`, `select()`와 같은 모든 CRUD 메서드를 즉시 사용할 수 있다.
```java
// findByUsername(String username) -> 해당 매서드를 내부에서 자동으로 만들어 줌
public interface IUserDAO extends JpaRepository<User, Long> {  
	User findByName(String name);  
}
```

---
## 🧙‍♂️ 요약
- **JDBC**
	- 가장 기본적인 Java와 데이터베이스의 연결 기술

- **JPA & Hibernate**
	- 객체-관계형 매핑(ORM) 제공으로 코드 간소화.
	- JDBC 위에서 동작한다.
	- SQL 대신 객체 중심으로 작업이 가능하다.
	- Hibernate는 JPA의 구현체로, HQL을 사용한다.

- **Spring Data JPA**
	- Spring Framework의 JPA 추상화 계층
	- Hibernate를 기본 ORM 제공자로 사용한다.
	- JPA/Hibernate의 BoilerPlate Code를 제거한다.
---
> **참고사이트**
> - [JDBC, JPA, Hibernate, Spring Data JPA 차이](https://velog.io/@pppp0722/JDBC-JPA-Hibernate-Spring-Data-JPA-%EC%B0%A8%EC%9D%B4)
> - [JDBC, JdbcTemplate, JPA 차이점 및 활용법!](https://perfect-dev.tistory.com/30)
> - [JDBC, JPA/Hibernate, Mybatis의 차이](https://proglish.tistory.com/78)
> - [JPA와 하이버네이트](https://chaeyami.tistory.com/256)
> - [Spring Data JPA란?](https://velog.io/@yebali/Spring-Data-JPA%EB%9E%80)

