---
created: 2025-02-06 16:36
category:
  - spring
  - database
tags:
  - TIL
last_modified: 2025-02-06T17:34:00
---
## ⭐ JPQL과 QueryDSL
### 🍪 JPQL (Java Persistence Query Language)
- [[JDBC, JPA, Hibernate, Spring Data JPA|JPA(Java Persistence API)]]에서 제공하는 객체 지향 쿼리 언어.
- 관계형 데이터베이스의 테이블이 아닌, 엔티티 객체를 대상으로 쿼리를 작성한다.
- SQL과 유사하지만, SQL이 테이블과 컬럼을 대상으로 하는 반면,
	- JPQL는 엔티티와 그 속성을 대상으로 쿼리를 수행한다.
- JPQL은 데이터베이스에 독립적이며, JPA 구현체(ex: Hibernate)가 이를 적절한 SQL로 변환하여 실행한다.
#### 🍬 특징
- 객체 지향: 
	- 데이터베이스의 테이블 대신 엔티티 객체를 대상으로 한다.
	- 객체 모델과의 매핑이 자연스럽다.
	- 테이블에 매핑되는 객체가 존재해야 한다.
- 데이터베이스 독립적:
	- JPQL은 JPA 스펙에 정의되어 있어 특정 데이터베이스에 종속되지 않는다.
- 문자열 기반 쿼리(정적 쿼리):
	- 쿼리 자체가 문자열로 작성되며, 런타임에 파싱되어 실행된다.
		- 이로 인해 쿼리 오류는 실행 시점에 발견될 수 있다.
		- 컴파일 시점에는 오류를 잡기 어렵다.

#### 🍬 예제
```java
// 전체 회원 조회
String jpql = "SELECT m FROM Member m";
List<Member> members = em.createQuery(jpql, Member.class)
                        .getResultList();

// 특정 조건의 회원 조회
String jpql = "SELECT m FROM Member m WHERE m.name = :name";
List<Member> members = em.createQuery(jpql, Member.class)
                        .setParameter("name", "김철수")
                        .getResultList();
```
---
### 🍪 QueryDSL
- 자바에서 타입 안전한 쿼리를 작성할 수 있도록 도와주는 도메인 특화 언어(DSL)
- 엔티티 모델에 대해 컴파일 시점에 검증 가능한 쿼리를 생성할 수 있도록 지원한다.
- JPQL 빌더 역할을 하며, 코드 기반으로 사용이 단순하고 쉽다.

#### 🍬 특징
- [[타입 안전성(Type safety)]]:
	- 쿼리를 자바 코드로 작성한다 -> 문법 오류, 필드명 오타 등을 컴파일 시점에 검출할 수 있다.
- 자동 완성 지원:
	- IDE의 코드 자동 완성 기능을 활용할 수 있어 쿼리 작성이 더욱 편리하다.
- 유연한 쿼리 생성:
	- 동적 쿼리를 코드로 손쉽게 생성할 수 있으며, 조건에 따라 쿼리 변경을 간편하게 처리할 수 있다.
- 확장성:
	- JPA 외에도 SQL, MongoDB 등 다양한 데이터 소스에 대해 쿼리를 작성할 수 있다.
- 동적 쿼리:
	- 동적 쿼리를 작성하기에 매우 유리하다.

#### 🍬 예제
```java
// `QMember` 클래스는 엔티티 `Member`에 대해 QueryDSL이 생성한 타입 안전한 쿼리 클래스이다.
JPAQueryFactory queryFactory = new JPAQueryFactory(entityManager);
QMember member = QMember.member;

List<Member> members = queryFactory.selectFrom(member)
                                   .where(member.name.eq("John"))
                                   .fetch();
```
---
### 🍪 JPQL vs QueryDSL
> 대부분 QueryDSL을 선호한다.
- JPQL은 문자열 형태의 쿼리 전달 -> 오타같은 에러를 컴파일 단계에서 파악할 수 없다.
- 그에 비해, Query DSL은 자바 코드로 이루어져 있어 컴파일 단계에서 발견할 수 있고, 쿼리에 의존적이지 않을 수 있다.
#### 🍬 예시: 기본 조회
- JPQL
```java
String jpql = "SELECT m FROM Member m WHERE m.age > :age";  // ❌ 문자열 기반 → 오타 발생 가능, 컴파일 타임 오류 검출 불가
List<Member> members = entityManager.createQuery(jpql, Member.class)
                                    .setParameter("age", 18)  // ❌ 가독성 부족 → 필드명을 문자열로 직접 작성해야 함
                                    .getResultList();
```
- QueryDSL
```java
QMember member = QMember.member;
List<Member> members = queryFactory
    .selectFrom(member)
    .where(member.age.gt(18))  // ✅ 타입 안전성 → 필드명을 직접 참조하여 오타 방지
    .fetch();  // ✅ IDE 자동완성 지원 → 필드명 자동 추천, 가독성 향상
```
#### 🍬 예시: 동적 쿼리
- JPQL
```java
String baseQuery = "SELECT m FROM Member m WHERE 1=1";  // ❌ 문자열 결합 방식 → 코드가 지저분해지고 유지보수 어려움
if (username != null) {
    baseQuery += " AND m.username = :username";  // ❌ 조건이 많아질수록 쿼리 작성이 복잡해짐
}
if (age != null) {
    baseQuery += " AND m.age = :age";
}
TypedQuery<Member> query = entityManager.createQuery(baseQuery, Member.class);
if (username != null) {
    query.setParameter("username", username);
}
if (age != null) {
    query.setParameter("age", age);
}
List<Member> members = query.getResultList();
```
- QueryDSL
```java
QMember member = QMember.member;
BooleanBuilder builder = new BooleanBuilder();

if (username != null) {
    builder.and(member.username.eq(username));  // ✅ BooleanBuilder 사용 → 조건을 유연하게 추가 가능
}
if (age != null) {
    builder.and(member.age.eq(age));
}

List<Member> members = queryFactory
    .selectFrom(member)
    .where(builder)  // ✅ 가독성 우수 → 문자열 결합 없이 깔끔한 코드 작성 가능
    .fetch();  // ✅ 컴파일 타임 검증 가능 → 오류 발생 가능성 낮음

```
#### 🍬 예시: 조인(Join)
- JPQL
```java
String jpql = "SELECT m FROM Member m JOIN m.team t WHERE t.name = :teamName";  // ❌ 문자열 기반 → 필드명 오타 발생 가능
List<Member> members = entityManager.createQuery(jpql, Member.class)
                                    .setParameter("teamName", "TeamA")  // ❌ 복잡한 조인 쿼리는 가독성이 떨어짐
                                    .getResultList();
```
- QueryDSL
```java
QMember member = QMember.member;
QTeam team = QTeam.team;

List<Member> members = queryFactory
    .selectFrom(member)
    .join(member.team, team)  // ✅ 객체 필드를 직접 참조 → 필드명 오타 방지
    .where(team.name.eq("TeamA"))  // ✅ SQL-like 문자열 없이 간결한 조인 표현 가능
    .fetch();  // ✅ IDE 자동완성 지원 → 유지보수 용이
```

#### 🍬 예시: 그루핑 및 집계 함수
- JPQL
```java
String jpql = "SELECT COUNT(m), AVG(m.age) FROM Member m WHERE m.team.id = :teamId";
Object[] result = (Object[]) entityManager.createQuery(jpql)  // ❌ Object[] 배열로 결과를 반환 → 데이터 접근이 불편함
                                          .setParameter("teamId", 1L)
                                          .getSingleResult();
Long count = (Long) result[0];  // ❌ 형 변환이 필요 → 코드가 복잡해짐
Double avgAge = (Double) result[1];
```
- QueryDSL
```java
QMember member = QMember.member;

Tuple result = queryFactory
    .select(member.count(), member.age.avg())
    .from(member)
    .where(member.team.id.eq(1L))
    .fetchOne();  // ✅ Tuple 사용 → 값에 쉽게 접근 가능

long count = result.get(member.count());  // ✅ 형 변환 불필요 → 간결한 코드 작성 가능
double avgAge = result.get(member.age.avg());  // ✅ IDE 자동완성 지원 → 필드명 오타 방지
```
---
## 🧙‍♂️ 요약

| 항목              | JPQL                           | QueryDSL                          |
| --------------- | ------------------------------ | --------------------------------- |
| **쿼리 작성 방식**    | 문자열 기반                         | 코드 기반 (타입 안전)                     |
| **컴파일 타임 검증**   | 불가능 (런타임 오류 가능)                | 가능 (오타, 문법 오류 방지)                 |
| **가독성**         | SQL과 유사하지만 문자열 결합이 많아 복잡할 수 있음 | 객체 필드를 직접 참조하여 직관적                |
|                 |                                |                                   |
| **동적 쿼리**       | 문자열 결합이 필요하여 관리 어려움            | BooleanBuilder 활용으로 간결하고 유지보수 용이  |
| **자동 완성 지원**    | 없음                             | IDE 자동 완성 지원                      |
| **조인(Join) 표현** | 문자열 기반으로 필드명 오타 발생 가능          | 객체 필드를 직접 참조하여 안전하고 가독성 우수        |
| **그루핑 및 집계**    | Object[] 반환으로 형 변환 필요          | Tuple 사용으로 접근 용이                  |

- 정적 쿼리 위주의 단순한 조회라면 JPQL 사용도 가능하다.
- 동적 쿼리, 조인, 복잡한 쿼리를 자주 사용한다면 QueryDSL이 압도적으로 유리하다.
- IDE 자동 완성, 컴파일 타임 검증, 유지보수성 등을 고려할 때 대부분 QueryDSL을 선호한다.
---
> **참고사이트**
> - [01]()
> - [02-블로그(JPQL과 QueryDSL의 차이점)](https://kkoon9.tistory.com/504)
> - [03-QueryDSL과 JPQL 선택(김영한 답변)](https://www.inflearn.com/community/questions/38771/querydsl%EA%B3%BC-jpql%EC%9D%84-%EC%84%A0%ED%83%9D%ED%95%98%EB%8A%94-%EC%B0%A8%EC%9D%B4%EA%B0%80-%EA%B6%81%EA%B8%88%ED%95%A9%EB%8B%88%EB%8B%A4?srsltid=AfmBOooe-LGduqeqFLPSSEKbeiZMZTlFF2AfhUM-LPUcd9FZ1cl8k6WP)
> - ChatGPT
