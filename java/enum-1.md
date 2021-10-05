# Java Enum 1편 : Enum 기본적인 사용

# 1. Overview

Java Enum 타입은 일정 개수의 상수 값을 정의하고, 그 외의 값은 허용하지 않습니다.

과거에는 특정 상수값을 사용하기 위해선 모두 상수로 선언해서 사용했습니다.

```java
public static final String MON = "Monday";
public static final String TUE = "Tuesday";
public static final String WED = "Wednesday";
```

이렇게 사용하면 개발자가 실수하기도 쉽고 한눈에 알아보기도 쉽지 않습니다.

그리고 관련있는 값들끼리 묶으려면 접두어를 사용해서 점점 변수명도 지저분해집니다.

Enum 클래스는 이러한 문제점을 말끔히 해결해주는 굉장히 유용한 클래스입니다.

추가적인 활용법은 [Java Enum 2편 : 여러가지 활용법](./enum-2.md) 에서 다루기로 하고 여기서는 기본적인 사용법에 대해서 알아봅니다.

<br>

# 2. 정의

```java
public enum Day {
    MON, TUE, WED, THU, FRI, SAT, SUN
}
```

위처럼 단순하게 요일을 열거한 Enum 클래스를 만들 수 있습니다.

하지만 각각의 요소들이 특정한 값을 갖게 하고 싶을 수도 있습니다.

예를 들어, 각 요일의 풀네임 (full-name) 이 필요할 때도 있을겁니다.

<br>

# 3. 생성자와 final 필드 추가

```java
public enum Day {
    MON("Monday"),
    TUE("Tuesday"),
    WED("Wednesday"),
    THU("Thursday"),
    FRI("Friday"),
    SAT("Saturday"),
    SUN("Sunday")
    ;

    private final String label;

    Day(String label) {
        this.label = label;
    }

    public String label() {
        return label;
    }
}
```

Enum 요소에 특정 값을 매핑하고 싶다면 위 코드처럼 필드값을 추가하면 됩니다.

여기서는 `label` 이라는 String 값을 추가했습니다.

필드값을 추가하면 생성자도 함께 추가해야하는데 Enum 클래스는 생성자가 있다고 하더라도 `new` 연산으로 생성할 수 없습니다.

<br>

```java
System.out.println(Day.MON.name());      // MON
System.out.println(Day.MON.label());     // Monday
```

이렇게 규칙이 존재하는 특정 요소들을 하나의 Enum 클래스로 묶어두면 가독성도 좋아지고 if 문으로 일일히 검사할 필요도 없어서 편리합니다.

필드값을 추가할 때 이름을 `name` 으로 정하는건 피하는게 좋습니다.

Enum 클래스 자체에서 `name()` 이라는 메소드를 제공하기 때문에 헷갈릴 수 있습니다.

<br>

# 4. 필드값으로 Enum 값 찾기

Enum 은 자체적으로 `name()` 값으로 Enum 값을 찾는 `valueOf()` 메소드를 제공합니다.

특정 필드값으로 찾는 기능은 제공하지 않기 때문에 직접 만들어야 합니다.

<br>

## 4.1. 직접 Enum values() 순회하며 찾기

```java
public enum Day {
    // ..codes

    public static Day valueOfLabel(String label) {
        return Arrays.stream(values())
                    .filter(value -> value.label.equals(label))
                    .findAny()
                    .orElse(null);
    }
}
```

다른 필드값으로 찾기 위해선 위 코드와 같이 모든 Enum 값을 순회하면서 일치하는 값이 있는지 찾아야 합니다.

<br>

## 4.1. 캐싱해서 순회 피하기

```java
public enum Day {
    // ..codes

    private static final Map<String, Day> BY_LABEL =
            Stream.of(values()).collect(Collectors.toMap(Day::label, e -> e));

    public static Day valueOfLabel(String label) {
        return BY_LABEL.get(label);
    }
}
```

`HashMap` 을 사용해서 값을 미리 캐싱해두면 조회할 때마다 모든 값을 순회할 필요가 없습니다.

처음부터 값을 캐싱해두는게 싫다면 `valueOfLabel()` 에 처음 접근할 때 Lazy Caching 할 수 있습니다.

다만, 이 때는 동시성 문제 해결을 위해 `HashMap` 을 동기화 해야 합니다.

그리고 위 코드에서는 `valueOfLabel()` 메소드를 그냥 리턴했기 때문에 없는 `label` 값으로 호출하면 `null` 값이 리턴됩니다.

이런 경우에 사용자에게 nullable 가능성을 알려주기 위해 반환 값을 `Optional<Day>` 로 넘겨주는 방법도 있습니다.

<br>

# 5. 여러 개의 값 연결하기

```java
public enum Day {
    MON("Monday", 10),
    TUE("Tuesday", 20),
    WED("Wednesday", 30),
    THU("Thursday", 40),
    FRI("Friday", 50),
    SAT("Saturday", 60),
    SUN("Sunday", 70)
    ;

    private final String label;
    private final int number;

    Day(String label, int number) {
        this.label = label;
        this.number = number;
    }

    public String label() {
        return label;
    }

    public int number() {
        return number;
    }

    private static final Map<String, Day> BY_LABEL =
            Stream.of(values()).collect(Collectors.toMap(Day::label, Function.identity()));

    private static final Map<Integer, Day> BY_NUMBER =
            Stream.of(values()).collect(Collectors.toMap(Day::number, Function.identity()));

    public static Day valueOfLabel(String label) {
        return BY_LABEL.get(label);
    }

    public static Day valueOfNumber(int number) {
        return BY_NUMBER.get(number);
    }
}
```

위 코드처럼 여러 개의 필드값을 세팅할 수 있습니다.

여기서 사용한 예시는 요일이라서 그냥 `number` 필드를 추가했지만 만약 과일이라면 사과의 이름, 색, 무게, 가격 등등 여러 요소를 한번에 매핑시켜서 정의할 수 있습니다.

<br>

# Reference

- [Attaching Values to Java Enum - Baeldung](https://www.baeldung.com/java-enum-values)
