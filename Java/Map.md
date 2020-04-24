# Map

`Key`, `Value` 형태로 데이터를 저장하는 자료구조

- [HashMap](https://github.com/ParkJiwoon/PrivateStudy/blob/master/Java/Map.md#hashmap)

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



