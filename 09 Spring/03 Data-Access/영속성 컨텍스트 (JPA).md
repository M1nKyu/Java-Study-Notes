---
created: 2025-02-08 17:18
category:
  - spring
  - database
  - jpa
tags:
  - TIL
last_modified: 2025-02-08T19:43:00
---
- JPA에서 가장 중요한 두가지
	1. **객체와 관계형 데이터베이스의 매핑 (Object Relational Mapping)**
		- DB와 객체를 어떻게 설계하고, 이 둘 간의 매핑을 어떻게 할 것인가 
	2. **영속성 컨텍스트(Persistence Context)**
		- 실제 JPA가 내부에서 어떻게 동작하는가
---
## ⭐ 영속성 컨텍스트란?
### 🍪 개념
- **엔티티를 영구 저장하는 환경**이라는 뜻이다.
- JPA(Java Persistence API)에서 엔티티를 관리하는 **논리적 개념**이다.
- JPA에서 엔티티 객체를 저장하고 관리하는 **1차 캐시 역할**을 한다.
- 애플리케이션과 데이터베이스 사이에서 **객체를 관리**하는 **일종의 캐시**라고 볼 수 있다.
- 영속성 컨텍스트는 **엔티티 매니저를 생성할 때 하나** 만들어진다.
	- 엔티티 매니저를 통해 영속성 컨텍스트에 접근하고 관리할 수 있다.
- `em.persist(entity);`
	- 엔티티 매니저를 사용하여 `entity`를 **영속성 컨텍스트에 저장**하는 의미이다. (DB에 저장하는 것이 아닌)
---
### 🍪 EntityManagerFactory와 EntityManager
- 고객의 **요청**이 들어오게 되면 **EntityManagerFactory**를 이용하여 **각각 EntityManager를 생성**하게 된다.
- EntityManager은 내부적으로 **데이터베이스 커넥션**을 이용하여 DB와 소통한다.
#### 🍬 EntityManagerFactory
- EntityManager를 생성하는 팩토리 객체.
- JPA를 사용하기 위해서는 가장 먼저 **EntityMangerFactory 인터페이스 객체를 먼저 생성**해야 한다.
- 생성 비용이 매우 크므로 **애플리케이션 전체에서 한 번만 생성**하고, 공유해서 사용한다.
- 여러 스레드가 동시에 접근해도 안전하므로 서로 다른 스레드 간 공유해도 된다.
```java
// 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistence-unit-name");
```

#### 🍬 EntityManager
- 실제 **영속성 컨텍스트를 관리하는 객체**이다.
	- 엔티티를 관리하는 관리자로, **엔티티와 관련된 모든 일을 처리**한다.
- EntityManagerFactory에서 생성된다.
- 데이터베이스 연결이 필요한 시점까지 커넥션을 얻지 않는다.
- **스레드 간 공유가 불가능(Thread-unsafe)**, 요청이 들어올 때마다 생성해서 사용해야 한다.
- 엔티티를 저장(`persist`), 조회(`find`), 수정(`merge`), 삭제(`remove`)하는 기능을 제공한다.
```java
// 생성: 엔티티 매니저 팩토리를 생성했으면 엔티티 매니저를 생성할 수 있다.
EntityManagerFactory emf = Persistence.createManagerFactory("example");
EntityManager em = emf.createEntityManager();
```

- EntityManager와 영속성 컨텍스트 예제
```java
EntityManager em = emf.createEntityManager(); // 영속성 컨텍스트 생성
EntityTransaction tx = em.getTransaction();
tx.begin(); // 트랜잭션 시작

Member member = new Member();
member.setId("memeber");
member.setName("John");

// 영속성 컨텍스트에 저장됨 (비영속 -> 영속)
em.persist(member);  

tx.commit(); // 트랜잭션 커밋 (flush 실행 → DB 반영)
em.close();  // EntityManager 종료 (영속성 컨텍스트 제거)
```
---
### 🍪 엔티티의 4가지 상태 (생명 주기)
- ![엔티티 생명주기](https://psvm.kr/posts/tutorials/jpa/3-em-and-persistence-context/entity-lifecycle.svg)
	- [이미지 출처](https://psvm.kr/posts/tutorials/jpa/4-entity-lifecycle)
#### 🍬 비영속 (new/transient)
- 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태.
- `new` 키워드로 객체를 생성했지만 `EntityManager`에 저장되지 않은 상태.
#### 🍬 영속 (merged)
- `EntityManager.persist(entity)`를 통해 영속성 컨텍스트가 관리하는 상태.
#### 🍬 준영속 (detached)
- `EntityManager.clear()`, `EntityManager.detach(entity)`, `EntityManager.close()` 등을 호출하여 **관리가 해제**된 상태.
#### 🍬 삭제 (removed)
- `EntityManager.remove(entity)`를 호출하여 삭제된 상태.
---
### 🍪 영속성 컨텍스트 사용 이점
> 왜 DB와 바로 소통하지 않고, 영속성 컨텍스트의 개념을 사용?
#### 🍬 1. 1차 캐시
- **영속성 컨텍스트 내부에는 캐시**가 있는데 이를 **1차 캐시**라고 한다.
- 엔티티를 조회하면 **1차 캐시에 먼저 저장**되며, 이후 같은 엔티티를 조회할 때 데이터베이스 조회없이 1차 캐시에서 가져온다.
```java
// 엔티티를 생성한 상태 (비영속) 
Member member = new Member(); 
member.setId("member1"); 
member.setName("회원1");  

// 엔티티를 영속 -> 1차 캐시에 저장됨.
// 여기서 트랜잭션이 commit 되지 않았음으로 실제 DB에 들어가지 않음. 
entityManager.persist(member);  

// 1차 캐시에서 조회 
Member member = entityManager.find(Member.class, "member1");
```

- 1차 캐시에 없다면?
	- **DB에서 조회** -> DB에서 조회된 결과가 **1차 캐시에 저장**됨 -> 저장된 **1차 캐시의 엔티티 정보를 반환**한다.
	- ![1차 캐시에 없는 경우](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*S3FYyci6xXMyLiziD5kdrw.png)
		- [이미지 출처 (김영한 JPA 강의자료)](https://medium.com/ssonzk/jpa-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-c6a6ff476464)


#### 🍬 2. 동일성 보장 (Identity)
- 같은 트랜잭션 안에서는 **같은 id를 가진 엔티티** 객체가 **항상 동일한 객체를 보장**한다.
- 즉, `find()`로 조회한 엔티티와 `persist()`된 엔티티는 같은 인스턴스이다.
```java
Member member1 = em.find(Member.class, "member1");
Member member2 = em.find(Member.class, "member1");

System.out.println(member1 == member2); // true (같은 인스턴스)
```

- 동일성을 제공할수 있는 것은 영속성 컨텍스트의 **1차 캐시가 있기 때문에** 가능하다.
	- 1차 캐시는 영속성 컨텍스트에 있고, 영속성 컨텍스트는 트랜잭션의 시작과 함께 생성되고 죵료시 소멸되기 때문이다.
		- (물론, 같은 트랜잭션 안에서 허용되는 이야기.)
#### 🍬 3. 변경 감지 (Dirty Checking)
> 변경 감지는 영속성 컨텍스트가 관리하는 영속 상태의 엔티티만 적용된다.
- JPA로 **엔티티를 수정**할 때는 **단순히 엔티티를 조회해서 데이터를 변경**하면 된다.
- JPA는 트랜잭션이 커밋될 때 변경된 사항을 **자동으로 감지하여 반영**한다.
```java
em.getTransaction().begin();

Member member = em.find(Member.class, "member1");
member.setName("UpdatedName"); // 엔티티 값 변경

em.getTransaction().commit(); // UPDATE 쿼리 자동 실행
```
- 변경 흐름
	1. 트랜잭션이 커밋되면, 엔티티 매니저가 먼저 **플러시(flush)를 호출**한다. (`flush`: 영속성 컨텍스트에 저장된 엔티티 상태를 DB와 동기화하는 작업)
	2. **엔티티와 스냅샷**(엔티티가 최초 로딩된 시점의 상태)을 **비교**하여 변경된 엔티티를 찾는다.
	3. 변경된 엔티티가 있으면, **수정 쿼리를 생성**해서 **쓰기 지연 SQL 저장소**에 **저장**한다. (실제 DB에 반영할 쿼리를 일시적으로 보관)
	4. 쓰기 지연 저장소의 **SQL을 플러시(flush)** 하여 **데이터베이스에 반영**한다.
	5. 데이터베이스 트랜잭션을 **커밋**한다. 
- ![흐름](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*d96v7c_kXKjdwMQACgswcw.png)
	- [이미지 출처 (김영한 JPA 강의자료)](https://medium.com/ssonzk/jpa-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-c6a6ff476464)

> 영속성 컨텍스트를 플러시하는 방법
	1. EntityManager의 `.flush()` 메서드 직접 호출 
	2. Transaction의 Commit이 발생 시 자동 호출
	3. JPQL 쿼리 실행시 자동 호출

#### 🍬 4. 지연 로딩 (Lazy Loading)
- `ManyToOne(fetch = FetchType.LAZY)` 설정 시, 실제 사용될 때까지 데이터베이스 조회를 지연한다.
- `em.find()`를 호출해도 즉시 데이터를 가져오지 않고, **필요할 때 쿼리를 실행**한다.
```java
@ManyToOne(fetch = FetchType.LAZY)
private Team team;
```

#### 🍬 5. 트랜잭션을 지원하는 쓰기 지연 (Transactional write-behind)
- 쓰기 지연은 영속성 컨텍스트에서 엔티티 **상태 변경을 DB에 즉시 반영**하지 않고, 트랜잭션 **커밋 시점까지 지연** 시킨다는 개념이다.
- 트랜잭션 커밋 시점까지 **변경 사항을 메모리 상에서 관리**하고, **한 번에 DB에 반영**함으로써 **성능을 최적화**하는 기법이다.
- 영속성 컨텍스트 안에는 1차 캐시 뿐만 아니라 `쓰기 지연 SQL 저장소`가 존재한다.
- `.persist(entity)`가 발생하면 1차캐시에 저장하면서 `쓰기 지연 SQL 저장소`에 쿼리를 쌓아둔다.
```java
em.getTransaction().begin();

// 엔티티 상태 변경 (쓰기 지연)
// setName()을 호출해도 즉시 DB에 반영되지 않는다.
Member member1 = em.find(Member.class, "member1");
member1.setName("UpdatedName1");

Member member2 = em.find(Member.class, "member2");
member2.setName("UpdatedName2");

// 트랜잭션 커밋 시점에 DB에 반영 (한 번의 쿼리로 반영)
em.getTransaction().commit();
```

---
## 🧙‍♂️ 요약
- **영속성 컨텍스트는**
	- JPA에서 **엔티티를 관리**하는 1차 캐시 역할을 하는 **논리적 개념**이다.
	- 엔티티 생명주기를 관리하며, DB와의 동기화를 담당한다.
	- `EntityManager`를 통해 영속성 컨텍스트에 접근하고, 엔티티를 관리한다. 

- 엔티티 생명주기
	- 비영속(Transient) / 영속(Persistent) / 준영속(Detached) / 삭제(Removed)

- 영속성 컨텍스트 사용 이점은
	- **엔티티 조회**시 **영속성 컨텍스트에서 먼저** 저장된 엔티티를 **확인** 하는 **1차 캐시** 역할을 한다.
	- 같은 트랜잭션 내 **같은 ID**의 엔티티는 항상 **동일한 객체**를 보장한다.
	- 트랜잭션 커밋 시, **변경된 엔티티를 자동 감지**하여 데이터베이스에 반영한다.
	- 연관된 엔티티를 **실제 사용될 때까지** 데이터베이스 **조회를 지연**한다.
	- 엔티티 **상태 변경**은 트랜잭션 **커밋 시점까지 지연**되며, **한 번에 DB에 반영**된다.

- 엔티티 조회 ~ 커밋
	1. 엔티티 조회 시, 먼저 **1차 캐시에서 확인**하고 **없으면 데이터베이스에서 조회 후 캐시에 저장**한다.
	2. 엔티티 **상태가 변경되면** 영속성 컨텍스트가 이를 감지하고 **쓰기 지연 SQL 저장소에 쿼리를 쌓는다**.
	3. 트랜잭션 **커밋 시**, `flush`가 호출되어 **쓰기 지연 SQL 저장소의 쿼리가 실행**되고, 변경 사항이 DB에 반영된다.

---
> **참고사이트**
> - [01-영속성 컨텍스트](https://medium.com/ssonzk/jpa-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-c6a6ff476464)
> - [02-JPA 1차캐시,2차캐시](https://medium.com/ssonzk/jpa-1%EC%B0%A8-%EC%BA%90%EC%8B%9C%EC%99%80-2%EC%B0%A8-%EC%BA%90%EC%8B%9C-401ac9183a1d)
> - [03-EntityManager, EntityMangerFactory](https://bnzn2426.tistory.com/143)
> - [04-Entity Life Cycle](https://psvm.kr/posts/tutorials/jpa/4-entity-lifecycle)
