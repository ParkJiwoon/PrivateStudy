# Java 코드를 짤 때 주의할 점 몇가지

# Overview

[Java Coding Best Practices And Standards](https://javatechonline.com/java-coding-best-practices-and-standards) 에 있는 글을 번역한 글입니다.

여러 개의 항목이 있었는데, 제가 개발하면서 공감되었던 부분들만 간단하게 정리했습니다.

굉장히 기본적인 내용도 있는데 초심을 잊지 말자는 의미에서 기록합니다.

1. NullPointerException 을 고려하자
2. String 생성할 때 new 키워드를 사용하지 말자
3. 반복문 내에서 새로운 객체를 생성하지 말자
4. Collections 을 반복하는 동안 수정하지 말자
5. Switch-Case 문에서 break 키워드를 빼먹지 말자
6. 객체 비교 "==" 와 "eqauls()" 의 차이를 알자
7. 무작정 StringBuffer 를 사용하지 말자
8. Java 파일 작성 시 표준

<br>

## 1. NullPointerException 을 고려하자

개발할 때 NullPointerException, 즉 NPE 를 고려하지 않는 경우가 있습니다.

항상 발생가능한 상황을 고려해서 `null` 체크를 해주어야 합니다.

Java 8 부터는 `Optional` 을 사용할 수도 있습니다.

<br>

가장 흔한 케이스가 `String` 의 `equals()` 입니다.

```java
// Bad Case : name 으로 null 값이 넘어오면 NPE 발생
public boolean isKim(String name) {
    return name.equals("Kim");
}

// Good Case 1 : name 이 null 값이어도 NPE 발생하지 않음
public boolean isKim(String name) {
    return "Kim".equals(name);
}

// Good Case 2 : org.apache.commons.lang 에서 제공하는 StringUtils 클래스 사용해서 비교
// (내부적으로 null 체크해서 안전함)
public boolean isKim(String name) {
    return StringUtils.equals(name, "Kim");
}
```

<br>

빈 컬렉션에 `null` 값을 세팅하는 것도 나쁜 습관입니다.

`null` 값 대신에 `Collections.EMPTY_LIST` 같은 값을 넣어줍시다.

```java
// Bad Case : null 세팅
List<String> strings = null;

// Good Case : EMPTY_LISTY 세팅
List<String> strings = Collections.EMPTY_LIST
```

<br>

## 2. String 생성할 때 new 키워드를 사용하지 말자

```java
// Slow
String s = new String("hello");

// Fast
String s = "hello";
```

`new` 키워드를 사용하면 항상 새로운 객체를 만들고 힙에 추가합니다.

반면, 사용하지 않으면 String pool 을 먼저 확인하고 없는 경우에만 추가하기 때문에 좀더 효율적입니다.

<br>

## 3. 반복문 내에서 새로운 객체를 생성하지 말자

```java
for (int i = 0; i < 5; i++) {
    Foo f = new Foo();
    f.getMethod();
}
```

루프 반복횟수 만큼 많은 객체가 생성되기 때문에 지양해야 합니다.

<br>

## 4. Collections 을 반복하는 동안 수정하지 말자

```java
List<String> names = new ArrayList<>();
names.add("Kim");
names.add("Lee");
names.add("Park");

for (String name : names) {
    if ("Kim".equals(name)) {
        names.remove(name);
    }
}
```

예를 들어 이름 목록에서 Kim 을 삭제하려고 합니다.

위 코드는 `names` 라는 리스트를 반복하는 동시에 `remove()` 로 요소를 삭제하고 있습니다.

이러면 `ConcurrentModificationException` 가 발생할 수 있습니다.

<br>

## 5. Switch-Case 문에서 break 키워드를 빼먹지 말자

```java
int index = 0;

switch (index) {
    case 0:
        System.out.println("Zero");
    case 1:
        System.out.println("One");
        break;
    case 2:
        System.out.println("Two");
    break;
    // ...
    default:
        System.out.println("Default");
}
```

위 코드를 보면 case 0 에서 break 문을 빼먹었기 때문에 "Zero" 이후에 "One" 까지 출력됩니다.

<br>

## 6. 객체 비교 "==" 와 "eqauls()" 의 차이를 알자

`==` 연산자는 객체 참조가 같은 지 비교합니다.

`equals()` 메소드는 객체의 값을 비교합니다.

대부분은 `equals()` 메소드를 사용해서 비교하지만, 이 둘의 차이점을 잘 모르고 사용하는 경우가 있습니다.

<br>

## 7. 무작정 StringBuffer 를 사용하지 말자

`StringBuilder` 와 `StringBuffer` 의 차이를 잘 모르고 무작정 `StringBuffer` 를 사용하는 경우가 있습니다.

`StringBuffer` 는 기본적으로 동기화되기 때문에 많은 오버헤드를 생성할 수 있습니다.

따라서 동기화가 우선순위가 아니라면 `StringBuilder` 를 사용하는 것도 고려해보아야 합니다.

<br>

## 8. Java 파일 작성 시 표준

- 변수는 public, protected, private 순으로 정의합니다.
- 생성자는 필드 수가 적은 생성자를 먼저 정의합니다.
- 메소드는 접근성 (accessibility) 이 아닌 기능별 (functionality) 로 그룹화되어야 합니다. 예를 들어 public 메소드 사이에 private 메소드가 올 수도 있습니다.
- 코드 설명을 위해 주석을 사용할 수도 있지만, 최대한 주석을 줄이고 가독성 있는 코드를 작성하는 것이 좋습니다.

<br>

# Reference

- [Java Coding Best Practices And Standards](https://javatechonline.com/java-coding-best-practices-and-standards)