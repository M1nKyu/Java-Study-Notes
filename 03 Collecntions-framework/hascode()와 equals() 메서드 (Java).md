---
created: hascode()와 equals() 메서드 (java)
summary: hashCode와 equals 메서드 오버라이딩
tags:
  - java
last_modified: 2025-02-21T17:00:00
---
## ⭐ 자바의 eqauls와 hashCode
### 🍪 자바에서 equals와 hashCode의 중요성
- 객체의 비교와 해시 기반 컬렉션에서 중요한 역할을 한다.
- equals는 객체의 동등성을 비교, hashCode는 객체를 해시 기반 자료구조에서 빠르게 찾을 수 있도록 해준다.
- 두 메서드는 자바의 모든 객체에서 기본적으로 제공된다.
- 올바르게 재정의하지 않으면 예상치 못한 동작을 초래할 수 있다.
- HashMap이나 HashSet에서 이 메서드들의 올바른 구현이 필수적이다.
---
### 🍪 equals와 hashCode의 개념
> 동일성: == 비교, 객체의 주소 값을 비교한다.
> 동등성: equals() 메서드를 이요하여 객체 내부의 값을 비교한다.
#### 🍬 equals 메서드
- 객체의 동등성 비교에 사용된다.
- 기본적으로 Object 클래스에서 제공되며, 참조값을 비교한다.
- 그러나 대부분 경우, 객체의 실제 값을 비교하도록 재정의해야 한다.
	- 기본 equals 메서드는 참조값만 비교하기 때문에, 동일값의 객체라도 다른 객체로 인식한다.
- equals 함수를 재정의한 대표적인 예: String class의 equals()는 번지 비교가 아닌 문자열 '값'을 비교한다.
#### 🍬 hashcode 메서드
- 객체의 메모리 번지를 이용하여 생성한 해시코드를 반환한다.
- 해시코드는 정수값, 객체를 빠르게 검색하는 데 사용된다.
#### 🍬 equals + hashCode
- 두 메서드는 반드시 함께 재정의해야 하며, equals가 true를 반환하는 두 객체는 동일한 hashCode를 가져야 한다.
---
### 🍪 hashCode와 equals의 동작 순서
- ![Image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsMxPO%2FbtrNAyvpegC%2FUPc1EBboouzu0WZnc9bUfk%2Fimg.png)
	- [이미지 출처](https://inpa.tistory.com/entry/JAVA-%E2%98%95-equals-hashCode-%EB%A9%94%EC%84%9C%EB%93%9C-%EA%B0%9C%EB%85%90-%ED%99%9C%EC%9A%A9-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0)
1. 데이터가 추가된다면, 그 데이터의 hashCode()의 리턴값이 컬렉션에 있는지 비교한다.
2. 해시코드가 같다면, equals() 메서드의 리턴값을 비교하며 true이면같은 객체라고 판단한다.
---
### 🍪 equals와 hashCode의 기본 구현 예제
```java
@Override
public boolean eqauls(Object obj){ 

	if (this == obj) return true; // 같은 객체이면 true 반환
	
	if (obj == null || getClass() != obj.getClass()) return false;  // null이거나 클래스 타입이 다르면 false 반환
	
	MyClass myClass = (MyClass) obj;  // obj를 MyClass 타입으로 캐스팅
	return Objects.equals(field1, myClass.field1) && Objects.equals(field2, myClass.field2); // 두 객체의 field1, field2 값이 모두 같으면 true 반환   
}

@Override
public int hashCode(){  
	return Objects.hash(field1, field2);  // field1, field2 값을 기반으로 해시코드 생성
}
```
---
### 🍪 equals와 hashCode 재정의 방법
> 재정의할 때는 몇 가지 규칙을 따라야 한다.
- equals 메서드는 대칭성, 반사성, 추이성을 만족해야 하며,
	- hashCode는 동일한 객체에 대해 항상 동일한 값을 반환해야 한다.

- equals 메서드는 객체의 필드를 비교하여 동등성을 판단하고,
	- hashcode 메서드는 equals에서 비교한 필드를 기반으로 해시코드를 생성해야 한다.
		- 이를 통해 equals와 hashCode의 일관성을 유지한다.
---
### 🍪 컬렉션 프레임워크에서의 활용
- 두 메서드는 HashMap, HashSet 같은 자료구조에서 중요한 역할을 한다.
- ex: 해시맵에서 키를 검색할 때 hashCode를 사용하여 버킷을 찾고, 그 버킷 안에서 equals를 사용하여 정확한 키를 찾는다.
- 따라서 equals와 hashCode가 일관성을 유지하지 않으면 데이터 검색이 제대로 이루어지지 않는다.
---
>**참고 사이트**
> - [01-자바에서 equals와 hashCode의 올바른 사용법](https://f-lab.kr/insight/java-equals-hashcode-20250108?gad_source=1)
> - [02-equals와 hashCode 함수](https://mangkyu.tistory.com/101)
> - [03-hashcode()와 equals() 메서드는 언제 사용하고 왜 사용할까?](https://jisooo.tistory.com/entry/java-hashcode%EC%99%80-equals-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B3%A0-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C)
> - [04-자바 equals/hasCode 오버라이딩](https://inpa.tistory.com/entry/JAVA-%E2%98%95-equals-hashCode-%EB%A9%94%EC%84%9C%EB%93%9C-%EA%B0%9C%EB%85%90-%ED%99%9C%EC%9A%A9-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0)

