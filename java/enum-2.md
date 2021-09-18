# Java Enum 2편 : 상수값에 메소드 추가

# 1. Overview

Java [Java Enum 1편 : Enum 기본적인 사용](./enum-1.md)에 대해서는 이미 학습했습니다.

이번에는 Enum 에 메소드를 추가하여 원하는 동작을 만들어내는 방법을 알아봅니다.

<br>

# 2. 상수별 메소드 구현: Enum 상수 별로 다른 동작이 필요할 때

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
- 마지막 `AssertionError` 는 실제로는 도달하지 기술적으로는 도달 가능하기 때문에 생략 불가능
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

# 3. 전략 열거 타입: Enum 상수 일부가 같은 동작을 공유할 때

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

# 4. 여러 상수별 동작이 혼합될 때

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

# Reference

- [이펙티브 자바 Effective Java 3/E](http://www.yes24.com/Product/Goods/65551284) Item 34 : int 상수 대신 열거 타입을 사용하라
