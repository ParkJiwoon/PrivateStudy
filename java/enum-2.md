# Java Enum 2편 : 여러가지 활용법

# 1. Overview

[Java Enum 1편 : Enum 기본적인 사용](./enum-1.md)에 대해서는 이미 학습했습니다.

이번에는 Enum 에 메소드를 추가하여 원하는 동작을 만들어내는 방법과 그밖의 활용법을 알아봅니다.

<br>

# 2. 메소드 추가 1: Enum 상수 별로 다른 동작이 필요할 때

가장 쉽게 떠올릴 수 있는 방법은 `switch` 문입니다.

하지만 Enum 클래스에는 **상수별 메소드 구현 (Constant-specific Method Implementation)** 이라는 좀더 깔끔한 방법이 있습니다.

<br>

## 2.1. Before

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    // 상수가 뜻하는 연산을 수행한다.
    public double apply(double x, double y) {
        switch (this) {
            case PLUS: return x + y;
            case MINUS: return x - y;
            case TIMES: return x * y;
            case DIVIDE: return x / y;
        }
        throw new AssertionError("알 수 없는 연산: " + this);
    }
}
```

- 깔끔해보이지만 뭔가 아쉬움
- 마지막 `AssertionError` 는 실제로는 도달하지 않지만 기술적으로는 도달 가능하기 때문에 생략 불가능
- 새로운 상수를 추가하면 `case` 문도 추가해야함

<br>

## 2.2. After

```java
public enum Operation {
    PLUS   { public double apply(double x, double y) { return x + y; }},
    MINUS  { public double apply(double x, double y) { return x - y; }},
    TIMES  { public double apply(double x, double y) { return x * y; }},
    DIVIDE { public double apply(double x, double y) { return x / y; }};

    public abstract double apply(double x, double y);
}
```

- Enum 상수값 별로 다르게 동작하는 코드를 구현
- `apply` 라는 추상 메소드를 선언하고 각 상수에서 재정의
- 이를 상수별 메소드 구현 (constant-specific method implementation) 이라고 함
- 추상 메소드로 정의되어 있기 때문에 새로운 상수를 추가해도 실수할 가능성이 적음

<br>

# 3. 메소드 추가 2: Enum 상수 일부가 같은 동작을 공유할 때

위에서 본 방법은 Enum 에 있는 각각의 상수가 모두 다른 동작을 할 때 사용했습니다.

만약 일부 상수끼리 같은 동작을 공유해야 할 때는 어떻게 해야 할까요?

일반적으로 생각 가능한 방법은 두가지가 있습니다.

1. 상수별로 메소드를 구현해서 같은 동작 코드를 중복해서 넣는다.
2. 별도의 메소드를 하나 만들어서 상수별 메소드에서 호출한다.

위 두가지 방법 모두 중복된 코드를 작성해야 한다는 단점이 있습니다.

다행히 Enum 클래스에서는 이러한 상황에서 **전략 열거 타입 (Enum)** 이라는 방법이 있습니다.

<br>

## 3.1. Before

```java
public enum Fruit {
    APPLE, ORANGE, BANANA, STRAWBERRY;

    public void printColor() {
        switch (this) {
            case APPLE:
            case STRAWBERRY:
                System.out.println("This is Red");
                break;
            default:
                System.out.println("This is Not Red");
        }
    }
}
```

- 과일을 나타내는 `Fruit` Enum 클래스
- `printColor()` 메소드를 호출하면 빨간색 과일들과 나머지 과일들의 출력 결과문이 다름
- 위의 문제점과 마찬가지로 새로운 빨간색 과일을 추가했을 때 `switch` 문에도 추가하지 않으면 빨간색 과일인데 "This is Not Red" 가 출력됨

<br>

## 3.2. After

```java
public enum Fruit {
    APPLE(ColorType.RED),
    ORANGE(ColorType.OTHER),
    BANANA(ColorType.OTHER),
    STRAWBERRY(ColorType.RED);

    private final ColorType colorType;

    Fruit(ColorType colorType) {
        this.colorType = colorType;
    }

    public void printColor() {
        colorType.printColor();
    }

    enum ColorType {
        RED {
            void printColor() {
                System.out.println("This is Red");
            }
        },
        OTHER {
            void printColor() {
                System.out.println("This is Not Red");
            }
        };

        abstract void printColor();
    }
}
```

- `Fruit` Enum 클래스 내부에 `ColorType` 이라는 Inner Enum 클래스를 정의
- `printColor()` 의 동작을 `ColorType` 에 위임
- 새로운 빨간색 과일이 추가되더라도 `ColorType` 을 지정해야 하므로 실수할 일이 적음

<br>

# 4. 메소드 추가 3: 여러 상수별 동작이 혼합될 때

한 Enum 상수값의 동작에 다른 Enum 상수값이 필요하다면 그냥 `switch` 문을 쓰는 것이 좋습니다.

```java
public enum Direction {
    NORTH, EAST, SOUTH, WEST;

    public static Direction rotate(Direction dir) {
        switch (dir) {
            case NORTH: return EAST;
            case EAST:  return SOUTH;
            case SOUTH: return WEST;
            case WEST:  return NORTH;
        }
        throw new AssertionError("알 수 없는 방향: " + dir);
    }
}
```

<br>

# 5. ordinal 메서드 대신 인스턴스 필드를 사용하라

Enum 클래스에는 기본적으로 `ordinal` 이라는 메소드를 제공합니다.

0 부터 시작되며 특정 상수값의 위치 (Index) 를 리턴해줍니다.

Enum API 문서를 보면 `ordinal` 에 대해서 이렇게 쓰여 있습니다.

"대부분의 개발자는 이 메소드를 쓸 일이 없다. 이 메소드는 `EnumSet` 과 `EnumMap` 같이 열거 타입 기반 범용 자료구조에 쓸 목적으로 설계되었다."

`oridnal` 을 사용할 때의 단점은 여러 가지 있습니다.

- 나중에 추가될 Enum 상수값이 꼭 순서대로라는 보장이 없다
- 중복된 숫자를 가져야 할 때 구분이 불가능하다

그러므로 `ordinal` 메소드를 사용하지 말고 별도의 인스턴스 필드를 선언해서 사용합시다.

<br>

# 6. ordinal 인덱싱 대신 EnumMap 을 사용하라

Enum 값을 Index 로 사용하고 싶을 때 `배열 + ordinal` 을 사용하는 것보다 `EnumMap` 을 사용하는 것이 좋습니다.

`EnumMap` 도 내부적으로 `ordinal` 을 사용하기 때문에 성능 상의 차이도 없습니다.

위에서도 한번 언급했었지만 개발자가 직접 `ordinal` 을 쓸 상황은 없습니다.

<br>

# 7. 비트 필드 대신 EnumSet 을 사용하라

과거에는 여러 값들을 집합으로 사용해야 할 경우 비트로 사용했습니다.

```java
public class Text {
    public static final int STYLE_BOLD          = 1 << 0;   // 1
    public static final int STYLE_ITALIC        = 1 << 1;   // 2
    public static final int STYLE_UNDERLINE     = 1 << 2;   // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3;   // 8

    // 매개변수 styles 는 0 개 이상의 STYLE_ 상수를 비트별 OR 한 값
    public void applyStyles(int styles) {
        // ...
    }
}

// usage
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

- 여러 개의 상수값을 OR 하여 사용하면 집합을 나타낼 수 있음
- 이렇게 만들어진 집합을 비트 필드 (bit field) 라고 함
- 비트 필드 값은 해석하기 어려움
- 최대 몇 비트가 필요한지 API 작성 시 미리 예측하여 적절합 타입 (int, long) 을 선택해야 함

<br>

## 7.1. EnumSet 클래스

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public void applyStyles(Set<Style> styles) {
        // ...
    }
}

// usage
text.applyStyles(EnumSet.of(Text.Style.BOLD, Text.Style.ITALIC));
```

- `java.util` 패키지
- `Set` 인터페이스를 구현하며, 타입 안전하고, 다른 어떤 `Set` 구현체와도 함께 사용 가능
- `EnumSet` 내부는 비트 벡터로 구현됨

<br>

# Reference

- [이펙티브 자바 Effective Java 3/E](http://www.yes24.com/Product/Goods/65551284) Item 34 ~ 37