---
created: Iterator란
tags:
  - java
  - collection-framework
last_modified: 2025-01-23T15:23:00
---
## ⭐ Iterator (반복자)
> Iterator는 자바의 **컬렉션 객체**에 저장된 요소들을 **하나씩 순회하며 접근**할 수 있는 인터페이스이다.

### 🍪 주요 메서드

|         메서드         | 설명                                                                                                                                  |
| :-----------------: | :---------------------------------------------------------------------------------------------------------------------------------- |
| `boolean hasNext()` | 다음 요소가 존재하면 `true`, 없으면 `false`를 반환.<br>반복의 끝에 도달했는지 확인할 때 사용된다                                                                     |
|     `E next()`      | 다음 요소를 반환한다.<br>요소를 가져오면서 커서를 다음 위치로 이동한다.<br>더 이상 가져올 요소가 없으면 `NoSuchElementEception` 발생                                           |
|   `void remove()`   | 현재 반복중인 컬렉션에서 마지막으로 반환된 요소를제거한다.<br>선택적(optional)이며, 모든 컬렉션이 이를 지원하지는 않는다.<br>`Collections.unmodiableList`같은 수정 불가능한 컬렉션은 지원하지 않는다. |

---
### 🍪 기본 선언 방법
- 컬렉션 객체의 `iterator()` 메서드를 호출하여 선언한다.
```java
// `Type`은 컬렉션이 저장하는 요소의 타입.
// `collection`은 `List`, `Set` 등 컬렉션 객체.
Iterator<Type> iterator = collection.iterator();
```

---
### 🍪 사용 예시
```java
import java.util.ArrayList;
import java.util.Iterator;

public class IteratorExample {
	public static void main(String[] args){
		ArrayList<String> list = new ArrayList<>();
		list.add("Apple");
		list.add("Banana");
		list.add("Cherry");
		
		Iterator<String> iterator = list.iterator(); // 📌
		
		while(iterator.hasNext()){
			String element = iterator.next();
			System.out.println(element);
			
			if(element.equals("Banana")){
				iterator.remove();
			}
			
			System.out.println("Updated List: " + list);
		}
	}
}
```
- 출력:
```
Apple
Banana
Cherry
Updated List: [Apple, Cherry]
```
---
### 🍪 Iterator의 특징
1. 읽기와 삭제 지원: 요소를 읽으면서 `remove()` 메서드를 통해 요소를 삭제할 수 있다.
2. 순방향 반복만 지원: `Iterator`는 컬렉션의 요소를 한 방향으로만 순회할 수 있다.
3. 다양한 컬렉션에서 사용 가능: `List`, `Set`, `Queue`등 대두분 컬렉션 클래스에서 사용할 수 있다.
4. ConCurrentModificationException: 반복 중 컬렉션이 수정되면 예외가 발생할 수 있다. (ex: `list.add()` 호출)

> 순서가 없는 컬렉션 (ex: `HashSet`)도 `Iterator`를 지원한다.
> - 저장된 요소들을 임의의 순서로 반환한다
---
### 🍪 Map에서의 Iterator
- Map은 `Iterator`를 사용하여 직접 순회를 할 수 없다.
	- **뷰(Collection Views)**를 통해 직접적으로 `Iterator`를 사용할 수 있다.
- 세 가지 뷰: `keySet()`, `values()`, `entrySet()`
#### 🍬 keySet()
- `Set<K>` 형식으로, `Map`의 모든 **키를 반환**한다.
- 키를 순회하고자 할 때 사용된다.
```java
public static void main(String[] args){
	Map<String, Integer> map = new HashMap<>();
	map.put("Apple", 1);
	map.put("Banana", 2);
	map.put("Cherry", 3);
	
	Iterator<String> keyIterator = map.keySet().iterator(); // 📌
	while(keyIterator.hasNext()){
		String key = keyIterator.next();
		System.out.println("Key: " + key);
	}
}

/*
[출력]
Key: Apple
Key: Banana
Key: Cherry
*/
```
#### 🍬 values()
- `Collection<V>` 형식으로, `Map`의 모든 **값을 반환**한다.
- 값만 순회하고자 할 때 사용된다.
```java
public static void main(String[] args){
	Map<String, Integer> map = new HashMap<>();
	map.put("Apple", 1);
	map.put("Banana", 2);
	map.put("Cherry", 3);
	
	Iterator<String> valueIterator = map.values().iterator(); // 📌
	while(valueIterator.hasNext()){
		Integer value = valueIterator.next();
		System.out.println("Value: " + value);
	}
}

/*
[출력]
Value: 1
Value: 2
Value: 3
*/
```
#### 🍬 entrySet()
- `Set<Map.Entry<K, V>>` 형식으로, `Map`의 **Key-Value 쌍(Entry)를 반환**한다.
- 키와 값을 동시에 순회하고자 할 때 사용된다.
```java
public static void main(String[] args){
	Map<String, Integer> map = new HashMap<>();
	map.put("Apple", 1);
	map.put("Banana", 2);
	map.put("Cherry", 3);
	
	Iterator<Map.Entry<String, Integer>> entryIterator = map.entrySet().iterator(); // 📌
	while(entryIterator.hasNext()){
		Map.Entry<String, Integer> entry = entryItertaor.next();
		System.out.println("Key: " + entry.getKey() + ", Value:" + entry.getVlaue());
	}
}

/*
[출력]
Key: Apple, Value: 1
Key: Banana, Value: 2
Key: Cherry, Value: 3
*/
```
> for-each문으로 요소들을 순회할 수도 있다.
---
### 🍪 Iterator vs for-each 문

| 특징         | Iterator                        | For-each                                           |
| ---------- | ------------------------------- | -------------------------------------------------- |
| **도입 시기**  | Java 2                          | Java 5                                             |
| **사용 방식**  | 명시적으로 `Iterator` 객체를 사용         | 간단한 구문으로 내부적으로 `Iterator`를 사용                      |
| **제어 가능성** | 순회 과정을 세밀하게 제어 가능 (`remove()`)  | 순회 과정은 제어 불가능                                      |
| **코드 가독성** | 다소 복잡함                          | 간결하고 직관적                                           |
| **요소 제거**  | `Iterator.remove()`로 안전하게 제거 가능 | 직접 제거 불가 (`ConcurrentModificationException` 발생 가능) |
| **용도**     | 요소를 수정하거나 제거할 때 적합              | 단순 순회가 필요할 때 적합                                    |

#### 🍬 언제 사용해야 할까?
- **Iterator**:
    - 요소를 제거하거나 수정해야 할 때.
    - 동기화된 컬렉션에서 안전한 순회가 필요할 때.
    - 고급 제어(조건부 반복, 중간 종료 등)가 필요한 경우.
- **For-each**:
    - 단순히 컬렉션이나 배열을 순회하는 경우.
    - 요소를 수정하거나 제거하지 않아도 되는 경우.
    - 가독성이 중요한 경우.
---
>**참고**
> - [01-블로그](https://jbiscode.tistory.com/entry/Java-Iterator%EC%9D%98-%EA%B8%B0%EB%8A%A5%EA%B3%BC-%EC%82%AC%EC%9A%A9-%EC%9D%B4%EC%9C%A0)
> - [02-블로그](https://thaud153.tistory.com/17)
> - [03-Map 출력하기](https://dongjin94.tistory.com/178)
> - ChatGPT 

