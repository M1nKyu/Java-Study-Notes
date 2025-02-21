---
created: 컬렉션 프레임워크란
tags:
  - java
  - collection-framework
last_modified: 2025-01-21T19:33:00
---
## ⭐ 컬렉션 프레임워크 (Collection Framework)
> 컬렉션 프레임워크는
>- 자료구조의 종류의 형태를 자바 클래스로 구현한 모음이라고 생각하면 된다.
>- 동작의 효율을 높이고 프로그래밍의 수고로움을 덜어준다.

### 🍪 컬렉션 프레임워크란?
- 다수의 데이터를 쉽고 효과적으로 처리할 수 있는 **표준화된 방법을 제공하는 클래스와 인터페이스의 집합**.
- 데이터를 저장하는 **자료구조와 데이터를 처리하는 알고리즘을 구조화** -> 클래스로 구현.
- 효율적으로 **데이터를 저장하고 검색하고 삭제**하는 등의 작업을 수행할 수 잇다.
- 컬렉션 프레임워크는 자바의 **인터페이스를 사용하여 구현**된다.
---
### 🍪 주요 구성 요소
> 컬렉션 인터페이스 + 구현 클래스 + 알고리즘
- ![CollectionFrameWork Hierarchy|700x400](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*qgcrVwo8qzF4muOQ-kKB8A.jpeg)

#### 🍬 1. 컬렉션 인터페이스 (Collection Interfaces)
> 각 인터페이스는 특정 종류의 컬렉션을 나타내며, 컬렉션을 구현하는 클래스들이 이 인터페이스들을 구현한다.

- **Collection**: 
	- 모든 컬렉션의 가장 기본적인 인터페이스.
	- `List`, `Set`, `Queue` 등 여러 하위 인터페이스를 포함한다.

- **List**:
	- 순서가 있는 데이터를 저장하며, 중복된 요소를 허용하는 컬렉션.
	- ex: `ArrayList`, `LinkedList`, `Vector`, `Stack` 
- **Set**: 
	- 중복을 허용하지 않는 컬렉션. 
	- ex: `HashSet`, `TreeSet`
- **Queue**:
	- FIFO(선입선출) 방식으로 요소를 처리하는 큐 컬렉션.
	- ex: `LinkedList`, `PriorityQueue`
- **Map**:
	- Key-Value 쌍으로 데이터를 저장하는 컬렉션.
	- `Collection`의 하위 인터페이스는 아니지만, 컬렉션 프레임워크의 일부로 취급된다.
	- ex: `HashMap`, `TreeMap`

#### 🍬 2. 구현 클래스 (Implementation Classes)
> 컬렉션 인터페이스를 구현한 클래스들이 실제로 데이터를 저장하고 처리하는 작업을 수행한다.


| 인터페이스     | 구현 클래스            | 설명                               |
| --------- | ----------------- | -------------------------------- |
| **List**  | **ArrayList**     | 순차적 데이터를 저장하는 동적 배열 기반의 컬렉션      |
|           | **LinkedList**    | 연결 리스트 기반의 컬렉션, 데이터 삽입/삭제가 빠름    |
|           | **Vector**        | 고정 크기의 배열을 기반으로 한 리스트, 동기화 지원    |
|           |                   |                                  |
| **Set**   | **HashSet**       | 중복되지 않는 데이터를 저장하는 해시 기반의 집합      |
|           | **LinkedHashSet** | 삽입 순서를 유지하는 해시 기반의 집합            |
|           | **TreeSet**       | 데이터를 자동으로 정렬하는 트리 기반의 집합         |
|           |                   |                                  |
| **Queue** | **PriorityQueue** | 우선순위에 따라 데이터를 처리하는 큐 구현체         |
|           | **ArrayDeque**    | 배열 기반의 양방향 큐 구현체, 양방향에서 삽입/삭제 가능 |
|           |                   |                                  |
| **Map**   | **HashMap**       | 해시 테이블을 사용하여 키-값 쌍으로 데이터를 저장하는 맵 |
|           | **LinkedHashMap** | 삽입 순서를 유지하는 해시 기반의 맵             |
|           | **TreeMap**       | 키가 자동으로 정렬되는 트리 기반의 맵            |
#### 🍬 3. 알고리즘
- 컬렉션 프레임워크는 `Collections` 클래스와 같은 유틸리티 클래스를 제공하여 데이터를 정렬, 검색, 변환, 병합, 분할 등의 작업을 수행할 알고리즘을 제공한다.

- ex
	- `Collections.sort()`
	- `Collections.shuffle()`
	- `Collections.reverse()
---
### 🍪 Collection Interface에서 제공하는 주요 메소드
| 소드                         | 설명                                        |
| -------------------------- | ----------------------------------------- |
| boolean add(E e)           | 해당 컬렉션(collection)에 전달된 요소를 추가함. (선택적 기능) |
| void clear()               | 해당 컬렉션의 모든 요소를 제거함. (선택적 기능)              |
| boolean contains(Object o) | 해당 컬렉션이 전달된 객체를 포함하고 있는지를 확인함.            |
| boolean equals(Object o)   | 해당 컬렉션과 전달된 객체가 같은지를 확인함.                 |
| boolean isEmpty()          | 해당 컬렉션이 비어있는지를 확인함.                       |
| Iterator<E> iterator()     | 해당 컬렉션의 반복자(iterator)를 반환함.               |
| boolean remove(Object o)   | 해당 컬렉션에서 전달된 객체를 제거함. (선택적 기능)            |
| int size()                 | 해당 컬렉션의 요소의 총 개수를 반환함.                    |
| Object[] toArray()         | 해당 컬렉션의 모든 요소를 Object 타입의 배열로 반환함.        |


---
### 🍪 컬렉션 프레임워크의 장점

- **일관성**: 일관된 인터페이스 설계를 통해 데이터를 처리하는 방식이 통일되어 있어 사용하기 쉽다.
- **효율성**: 특정 용도에 최적화되어 있어, 성능이 뛰어나다. (예를 들어, `ArrayList`는 검색이 빠르고, `HashSet`은 중복을 제거하는 데 효율적)
- **유연성**: 다양한 자료구조와 알고리즘을 제공하므로, 문제에 맞는 최적의 솔루션을 선택할 수 있다.
- **반복자([[Iterator란|Iterator]])**: 데이터의 요소를 순차적으로 접근할 수 있는 반복자(`Iterator`)를 제공한다.
---
>**참고**
> - [01-Tcp School](https://www.tcpschool.com/java/java_collectionFramework_concept)
> - [02-docs.oracle](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)
> - [03-Mastering the Java Collections Framework Hierarchy With Java Code and JUnit Testing]
> - [04-Inpa Dev](https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-Collections-Framework-%EC%A2%85%EB%A5%98-%EC%B4%9D%EC%A0%95%EB%A6%AC)
> - [[Map, Set, List|노트-주요 컬렉션 인터페이스의 특징(List, Set, Map)]]

