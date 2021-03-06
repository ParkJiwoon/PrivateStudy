# Content

- [Java Static 이란?](#java-static-이란)
- [Map](#map)
- [Future](#future)

<br>

# Java Static 이란?

변수나 메소드 앞에 `static` 키워드를 붙여 사용한다.

`static` 은 같은 클래스에 있는 경우 같은 메모리 주소를 바라본다는 점을 이용해서 메모리 효율 최적화, 데이터 공유 등을 위해 사용한다.

한글로는 "정적인" 이라는 뜻인데 한번에 이해하기가 쉽지 않다.

동적인 다른 자원들과 달리 프로그램 상의 변하지 않는 자원이라고 생각하면 된다.

<br>

## 생성 시기와 소멸 시기

`static` 키워드를 사용하면 객체가 생성되는 시점이 아닌 Class Loader 가 클래스를 Load 할 때 Data 영역에 메모리가 할당되게 된다.

이 영역은 같은 유형의 클래스마다 공유되며 Process 가 종료되는 시점에서 해제되므로 `static` 키워드의 생명주기 역시 Class Load 시 생성되고 Process 종료 시 해제되게 된다.

<br>

## static 변수

클래스 내에서 `static` 으로 선언된 변수는 모든 클래스 객체에서 같은 메모리 주소를 바라본다.

예를 들어 `Student` 라는 클래스가 있다. 이 클래스에는 학생들이 살고 있는 도시인 `teacher` 변수를 갖는다.

```java
public class Student {
  String name;
  String teacher;
}
```

학생들은 모두 같은 반이라서 선생님이 같다고 가정하자.

만약 반의 선생님이 바뀌어도 모든 학생들이 동시에 바뀌어야 한다.

보통 한 반에 학생이 30 명이라고 가정하면, 선생님이 바뀌었을 때 30 명의 학생들을 전부 바꿔줘야 한다.

하지만 이럴 때 `teacher` 변수를 `static` 으로 선언한다면 `Student` 객체의 모든 `teacher` 가 같은 메모리를 공유해서 데이터를 바꾸면 동시에 바뀐다.

<br>

## static 메소드

`static` 메소드는 객체 생성 없이 사용할 수 있으며 보통 유틸리티 클래스에서 주로 사용한다.

`static` 메소드 내에서는 `static` 변수만 사용 가능하고 `this` 호출이 불가능하다.

"이메일 형식 검사", "현재 시간 구하기" 등은 새로운 객체를 할당할 필요가 없기 때문에 `static` 으로 작성하는 게 좋다.

<br>

## static 을 응용항 싱글톤 패턴 (Singleton pattern)

디자인 패턴 중 하나인 싱글톤 패턴은 `static` 의 개념을 이용한 패턴이다.

싱글톤은 단 하나의 객체만 생성하는 패턴이다.

싱글톤 클래스를 만드는 규칙은 다음과 같다.

1. 싱글톤 클래스의 생성자를 `private` 로 하여 외부에서 생성하지 못하게 한다.
2. 클래스 내부에 `static` 변수를 선언한다.
3. `getInstance()` 메소드를 `static` 으로 만들고 내부에서 싱글턴 객체를 선언하여 리턴한다.

```java
class Singleton {
  private static Singleton instance;

  // 생성자를 private 로 하여 외부에서 생성 불가능하게 만듬
  private Singleton() { }

  public static Singleton getInstacne() {
    if (instance == null) {
      instance = new Singleton();
    }

    return instance;
  }
}
```

<br>

# Map

`Key`, `Value` 형태로 데이터를 저장하는 자료구조

- [HashMap](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#hashmap)
    - [선언](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#%EC%84%A0%EC%96%B8)
    - [삽입](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%82%BD%EC%9E%85)
    - [가져오기](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#%EB%8D%B0%EC%9D%B4%ED%84%B0-%EA%B0%80%EC%A0%B8%EC%98%A4%EA%B8%B0)
    - [삭제](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%82%AD%EC%A0%9C)
    - [데이터 확인](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%99%95%EC%9D%B8)
    - [크기](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#%ED%81%AC%EA%B8%B0%EA%B8%B8%EC%9D%B4-%ED%99%95%EC%9D%B8)
    - [Key / Value 묶음](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#key-value-%EB%AC%B6%EC%9D%8C-%EA%B0%80%EC%A0%B8%EC%98%A4%EA%B8%B0)
    - [순회](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#map-%EC%88%9C%ED%9A%8C)
    - [compute](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#compute)
- [LinkedHashMap](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#linkedhashmap)
    - [순서 유지](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#%EC%88%9C%EC%84%9C-%EC%9C%A0%EC%A7%80)
    - [접근 빈도에 따른 순서 변경](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#%EC%A0%91%EA%B7%BC-%EB%B9%88%EB%8F%84%EC%97%90-%EB%94%B0%EB%A5%B8-%EC%88%9C%EC%84%9C-%EB%B3%80%EA%B2%BD)

<br>

# HashMap

## 선언

일반적으로 변수 부분은 인터페이스로 선언하는게 확장에 유리하다.

```java
Map<String, String> map = new HashMap<>();
```

<br>

## 데이터 삽입

`V put(K key, V value)` 로 값을 넣을 수 있다.

`put(k, v)` 을 했을 때 이미 키값이 존재한다면 데이터를 덮어쓴다.

`putIfAbsent` 을 이용하면 `Map` 에 `Key` 값이 없을 때에만 데이터를 넣을 수 있다.

```java
map.put("animal", "cat");           // {animal=cat}
map.put("food", "pizza");           // {animal=cat, food=pizza}


// 이미 "animal" 키값이 있기 때문에 "dog" 로 갱신되지 않음
map.putIfAbsent("animal", "dog");   // {animal=cat, food=pizza}
map.putIfAbsent("animal2", "dog");  // {animal=cat, animal2=dog, food=pizza}
```

<br>

## 데이터 가져오기

`V get(Object key)` 로 `value` 값을 가져올 수 있다.

`V getOrDefault(Object key, V defaultValue)` 을 사용하면 `key` 값이 없을 때 `null` 대신 설정된 값을 리턴한다.

```java
map.get("animal");  // "cat"

map.getOrDefault("food", "chicken");     // "pizza"
map.getOrDefault("food2", "chicken");    // "chicken"
```

<br>

## 데이터 삭제

`V remove(Object key)` 로 데이터를 삭제 할 수 있다.

```java
map.remove("animal2");
```

<br>

## 데이터 확인

`boolean containsKey(Object key)` 또는 `boolean containsValue(Object value)` 로 `key` 나 `value` 값이 존재하는 지 확인할 수 있다.

```java
map.containsKey("food");    // true
map.containsValue("dog");   // false
```

<br>

## 크기(길이) 확인

```java
map.size();
```

<br>

## Key, Value 묶음 가져오기

`Set<K> keySet()` 은 `Key`들로 이루어진 `Set` 자료구조를 리턴한다.

`Collection<V> values()` 은 `Value` 들로 이루어진 `Collection` 을 리턴한다.

```java
// map: {animal=cat, food=pizza}

map.keySet();   // [animal, food]
map.values();   // [cat, pizza]
```

<br>

## Map 순회

`forEach` 를 사용해서 `Map` 을 순회할 수 있다.

람다식을 이용하면 좀더 간단하게 나타낼 수 있으며 코드가 한줄이라면 중괄호 `{ }` 를 생략할 수 있다.

```java
// 출력
// animal: cat
// food: pizza
map.forEach((k, v) -> {
    System.out.println(k + ": " + v);
});

// 한 줄이면 중괄호 { } 생략 가능
map.forEach((k, v) -> System.out.println(k + ": " + v));

// 람다식 안쓰고 for 문으로 구현. forEach 도 내부적으로는 이렇게 구현되어있음
for (Map.Entry entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

<br>

## Compute

`compute` 를 사용해 원하는 로직을 실행하고 데이터를 넣을 수 있다.

만약 `Key` 가 없으면 새로운 데이터를 넣어주고 `Key` 값이 있으면 데이터를 갱신해준다.

만약 기존에 없는 데이터로 `compute` 연산을 하게 될 때 `value` 값은 `null` 이 된다.

```java
map.compute("animal", (k, v) -> {
    System.out.println(k + "'s value is " + v  + " -> seoul");      // animal's value is cat -> seoul
    return "lion";
});
// map: {animal=lion, food=pizza}
```

<br>

`computeIfAbsent` 와 `computeIfPresent` 조건을 걸어서 `compute` 연산을 실행한다.

`computeIfAbsent` 는 `Key` 가 없을 때만 실행되기 때문에 람다식으로도 `key` 값 하나 밖에 받지 않는다.

`computeIfPresent` 는 `Key` 값이 존재할 때만 실행된다.

만약 `Key` 가 없거나 있어서 조건이 일치하지 않으면 로직이 아예 실행되지 않는다.

```java
map.computeIfAbsent("fruit", (k) -> {
    System.out.println("New value of " + k + " is apple");      // New value of fruit is apple
    return "apple";
});
// map: {fruit=apple, animal=lion, food=pizza}


map.computeIfPresent("animal", (k, v) -> {
    System.out.println(k + "'s value is " + v +  " -> tiger");      // animal's value is lion -> tiger
    return "tiger";
});
// map: {fruit=apple, animal=tiger, food=pizza}
```

<br><br>

# LinkedHashMap

`HashMap` 은 `hashcode` 를 사용하기 때문에 순서가 일정하지 않다.

`LinkedHashMap` 은 내부를 `Double-Linked List` 로 구성하여 `HashMap` 의 순서를 유지한다.

`HashMap` 에서 상속받기 때문에 `HashMap` 의 모든 메소드를 사용할 수 있다.

<br>

## 순서 유지

데이터는 먼저 들어간 데이터가 무조건 앞에 위치하게 된다.

`forEach` 문에서도 동일하다.

```java
Map<String, String> map = new LinkedHashMap<>();

map.put("animal", "cat");
map.put("fruit", "apple");

System.out.println(map);        // {animal=cat, fruit=apple}

map.put("animal", "dog");       
System.out.println(map);        // {animal=dog, fruit=apple}

map.forEach((k, v) -> System.out.print(k + ": " + v + ", "));       // animal: dog, fruit: apple, 
```

<br>

## 접근 빈도에 따른 순서 변경

`LinkedHashMap` 은 생성자 파라미터로 `accessOrder` 라는 값을 받는다.

`accessOrder` 는 기본값이 `false` 인데, 만약 `true` 로 설정한다면 `LinkedHashMap` 의 접근 빈도에 따라서 순서가 바뀌게 된다.

```java
public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

<br>

아래 코드는 위와 완전히 동일하지만 `accessOrder` 만 `true` 로 생성한 `LinkedHashMap` 이다.

`"animal"` 키를 먼저 넣었지만 중간에 `put` 메소드로 `"animal"` 키에 접근을 했기 때문에 순서가 바뀌었다.

`put` 뿐만 아니라 `get` 이나 `compute` 에도 동일하게 동작한다 (`containsKey` 는 바뀌지 않음)

```java
Map<String, String> map = new LinkedHashMap<>(16, 0.75f, true);
new HashMap<>();
map.put("animal", "cat");
map.put("fruit", "apple");

System.out.println(map);        // {animal=cat, fruit=apple}

map.put("animal", "dog");       
System.out.println(map);        // {fruit=apple, animal=dog}

map.forEach((k, v) -> System.out.print(k + ": " + v + ", "));       // fruit: apple, animal: dog, 
```

<br>

`accessOrder` 를 이용하면 가장 첫번째에 존재하는, 즉 가장 사용되지 않은 `Entry` 를 알 수 있다.

```java
Map.Entry leastUsedEntry = map.entrySet().iterator().next();
int leastUsedKey = map.keySet().iterator().next();
int leastUsedValue = map.values().iterator().next();

// 아래처럼 구할 수도 있다.
leastUsedKey = leastUsedEntry.getKey();
leastUsedValue = leastUsedEntry.getValue();
```

<br>

# Future

## 개념
- 비동기 처리를 위해서 사용

## Method
- ```V get()```
    - Call 작업의 실행이 완료될 때까지 블로킹 되며 완료되면 그 결과값을 리턴
    - CancellationException: 작업이 취소되는 경우
    - ExecutionException: 작업 도중 예외가 발생한 경우
    - InterruptedException: 현재 쓰레드가 인터럽트된 경우

- ```V get(long timeout, TimeUnit unit)```
    - 지정된 시간동안 작업의 실행 결과를 기다림
    - TimeoutException: 대기시간이 초과한 경우
    - ```get()``` Exception 은 전부 같음

- ```boolean cancel(boolean mayInterruptIfRunning)```
    - 작업 취소 시도
    - 작업이 이미 완료 또는 취소되었거나 취소할 수 없는 경우 실패
    - 작업이 시작되기 전이었다면 실행하지 않음
    - ```cancel``` 메소드가 리턴된 후 ```isDone()``` 메소드는 항상 true
    - ```cancel``` 메소드가 true를 리턴하면 ```isCancelled()``` 메소드는 항상 true

- ```boolean isCancelled()```
    - 작업이 완료되기 전에 최소된 경우 true

- ```boolean isDone()```
    - 작업이 완료된 경우 true
    - 작업의 완료란 정상 종료, 예외, 취소 등을 전부 말함
