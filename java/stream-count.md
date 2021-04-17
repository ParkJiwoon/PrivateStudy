# Java Stream count() 의 비밀

# Overview

Java 8 Stream 에는 `count()` 라는 종결 함수가 있습니다.

현재 Stream 의 원소 갯수를 카운트 해서 `long` 타입으로 리턴합니다.

<br>

# count 의 중간 연산으로 peek 사용

```java
public class NotePad {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5)
              .peek(System.out::println)
              .count();
    }
}
```

위 코드는 어떻게 동작할까요?

아무것도 나오지 않습니다.

원인을 몰라서 구글링 해봤더니 [Java 9 Stream API Reference](https://docs.oracle.com/javase/9/docs/api/java/util/stream/Stream.html#count--) 에 다음과 같은 글이 있었습니다.

<br>

> The number of elements covered by the stream source, a List, is known and the intermediate operation, peek, does not inject into or remove elements from the stream (as may be the case for flatMap or filter operations). Thus the count is the size of the List and there is no need to execute the pipeline and, as a side-effect, print out the list elements.

<br>

요약하자면 `count()` 라는 종결함수는 **Stream 의 갯수를 세기 때문에 효율을 위해 중간 연산을 생략하기도 한다**는 뜻입니다.

그래서 중간 연산을 강제로 실행시키고 싶다면 `filter` 나 `flatMap` 과 같이 Stream 요소의 갯수를 변화시킬 수 있는 중간 연산을 추가하면 됩니다.

<br>

# filter 로 강제 출력

```java
public class NotePad {
    public static void main(String[] args) {
        Stream.of(1, 2, 3, 4, 5)
              .filter(e -> e > 0)
              .peek(System.out::println)
              .count();
    }
}
```

`filter` 를 추가하면 `peek` 도 정상적으로 동작합니다.

<br>

```text
1
2
3
4
5
```

<br>

# Reference

- [Java 8 Stream – The peek() is not working with count()?](https://mkyong.com/java8/java-8-stream-the-peek-is-not-working-with-count/)