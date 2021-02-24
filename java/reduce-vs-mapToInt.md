# Collection 의 합을 더하는 방법

Collection 의 합을 구하는 방법은 `reduce` 와 `sum` 두 가지가 존재합니다.

단, Stream 에서 `sum()` 을 사용하려면 `IntStream, LongStream, DoubleStream` 와 같은 기본형 (Primitive Type) 특화 스트림을 사용해야 합니다.

그래서 보통 `mapToInt, mapToLong, mapToDouble` 같은 메소드로 스트림을 변환시키고 사용합니다.

<br>

# reduce

`reduce(초기값, 연산)` 형식으로 사용합니다.

초기값부터 시작하여 각 원소를 차례대로 순회하며 연산을 수행합니다.

이전 연산의 결과를 다음 초기값으로 넘기면서 연산의 결과를 누적해서 총 결과값을 구하는 메서드입니다.

```java
int sum = Stream.of(1, 2, 3).reduce(0, Integer::sum);
```

<br>

# sum

Stream 의 총합을 구하는 메서드입니다.

기본형 특화 스트림에서만 사용 가능합니다.

```java
int sum = Stream.of(1, 2, 3).mapToInt(e -> e).sum();
```

<br>

# 그렇다면 합을 구할 때 어떤걸 사용할까?

처음 생각 할 때는 `reduce` 로 한번에 하는게 Stream 처리가 적어서 더 빠를거라고 생각했습니다.

가독성도 크게 차이 안납니다.

그런데 실제로 테스트 해보니 결과는 달랐습니다.

<br>

## Test Code

```java
public static void main(String[] args) {
    Map<String, Integer> hashmap = new HashMap<>();

    for (int i = 0; i < 10000000; i++) {
        hashmap.put(i + "", i);
    }

    // for-loop 시간측정
    long start1 = System.currentTimeMillis();
    int sum1 = 0;
    for (String key : hashmap.keySet()) {
        sum1 += hashmap.get(key);
    }
    long end1 = System.currentTimeMillis();

    // stream mapToInt 시간측정
    long start2 = System.currentTimeMillis();
    int sum2 = hashmap.values().stream().reduce(0, Integer::sum);
    long end2 = System.currentTimeMillis();

    // stream reduce 시간측정
    long start3 = System.currentTimeMillis();
    int sum3 = hashmap.values().stream().mapToInt(i -> i).sum();
    long end3 = System.currentTimeMillis();

    System.out.println("for-loop: " + (end1 - start1));
    System.out.println("int reduce: " + (end2 - start2));
    System.out.println("mapToInt: " + (end3 - start3));
}
```

## Result

```html
for-loop: 303
reduce: 368
mapToInt: 310
```

<br>

## 원인 & 결론

`reduce` 에는 박싱, 언박싱 비용이 들어갑니다.

내부적으로 합계를 계산하기 위해 `Integer` 를 `int` 형으로 언박싱 하고 다시 `int -> Integer` 로 박싱하는 과정이 숨겨져 있어서 시간이 더 오래 걸립니다.

그래서 `reduce` 의 성능이 더 느리니 `IntStream` 에서 제공하는 메서드가 있는 경우에는 해당 메서드를 사용하는 게 유용합니다.

<br>

# Reference

- [Java 8 - Stream (스트림) - soy.me](https://soy.me/2017/06/14/java8-stream/)
