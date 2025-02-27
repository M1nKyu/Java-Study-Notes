---
created: super와 super() (Java)
summary: 
tags:
  - java
  - collection-framework
last_modified: 2025-02-27T21:07:00
---
### 🍪 super: 상속받은 부모를 가리키는 참조 변수
- `super`는 자식 클래스에서 부모 클래스로부터 **상속받은 멤버를 참조**하는데 사용되는 변수이다.
#### 특징
- **상속받은 멤버와 자신의 멤버 이름이 같을 때 구분**하기 위해 사용한다.
- 부모 클래스의 멤버에 접근할 때 사용된다.
- 주로 오버라이딩된 메서드에서 부모 클래스의 메서드를 호출할 때 사용한다.
#### 예시
```java
class Parent {
    int x = 10;
}

class Child extends Parent {
    int x = 20;
    void method() {
        System.out.println("x=" + x);         // 20
        System.out.println("this.x=" + this.x); // 20
        System.out.println("super.x="+ super.x); // 10
    }
}
```
---
### 🍪 super(): 상속받은 부모의 생성자 호출 메서드
- `super()`는 자식 클래스의 생성자에서 부모 클래스의 생성자를 호출하기 위해 사용한다.
#### 특징
- 반드시 생성자의 첫 줄에서 호출해야 한다.
- 명시적으로 호출하지 않으면 컴파일러가 자동으로 `super();`를 삽입한다.
- 부모 클래스의 매개변수가 있는 생성자를 호출할 때 사용한다.
#### 예시
```java
class Parent {
    Parent(int x, int y) {
        // 부모 클래스 생성자
    }
}

class Child extends Parent {
    Child(int x, int y, int z) {
        super(x, y); // 부모 클래스의 생성자 호출
        // 자식 클래스 생성자 코드
    }
}
```
---
>**참고**
> - [01](https://velog.io/@rhdmstj17/java.-super%EC%99%80-super-%EC%99%84%EB%B2%BD%ED%95%98%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

