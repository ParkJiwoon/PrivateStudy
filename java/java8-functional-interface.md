# Java8 함수형 인터페이스 (Functional Interface)

# Overview

함수형 인터페이스란 1 개의 추상 메소드를 갖는 인터페이스를 말합니다.

Java8 부터 인터페이스는 기본 구현체를 포함한 디폴트 메서드 (default method) 를 포함할 수 있습니다.

여러 개의 디폴트 메서드가 있더라도 **추상 메서드가 오직 하나**면 함수형 인터페이스입니다.

자바의 람다 표현식은 함수형 인터페이스로만 사용 가능합니다.

<br>

# 1. Functional Interface

함수형 인터페이스는 위에서도 설명했듯이 추상 메서드가 오직 하나인 인터페이스를 의미합니다.

추상 메서드가 하나라는 뜻은 `default method` 또는 `static method` 는 여러 개 존재해도 상관 없다는 뜻입니다.

그리고 `@FunctionalInterface` 어노테이션을 사용하는데, 이 어노테이션은 해당 인터페이스가 함수형 인터페이스 조건에 맞는지 검사해줍니다.

`@FunctionalInterface` 어노테이션이 없어도 함수형 인터페이스로 동작하고 사용하는 데 문제는 없지만, 인터페이스 검증과 유지보수를 위해 붙여주는 게 좋습니다.

<br>

## 1.1. Functional Interface 만들기

```java
@FunctionalInterface
interface CustomInterface<T> {
    // abstract method 오직 하나
    T myCall();

    // default method 는 존재해도 상관없음
    default void printDefault() {
        System.out.println("Hello Default");
    }

    // static method 는 존재해도 상관없음
    static void printStatic() {
        System.out.println("Hello Static");
    }
}
```

위 인터페이스는 함수형 인터페이스입니다.

`default method`, `static method` 를 넣어도 문제 없습니다.

어차피 함수형 인터페이스 형식에 맞지 않는다면 `@FunctionalInterface` 이 다음 에러를 띄워줍니다.

> Multiple non-overriding abstract methods found in interface com.practice.notepad.CustomFunctionalInterface

<br>

## 1.2. 실제 사용

```java
CustomInterface<String> customInterface = () -> "Hello Custom";

// abstract method
String s = customInterface.myCall();
System.out.println(s);

// default method
customInterface.printDefault();

// static method
CustomFunctionalInterface.printStatic();
```

함수형 인터페이스라서 람다식으로 표현할 수 있습니다.

`String` 타입을 래핑했기 때문에 `myCall()` 은 `String` 타입을 리턴합니다.

마찬가지로 `default method`, `static method` 도 그대로 사용할 수 있습니다.

위 코드를 실행한 결과값은 다음과 같습니다.

```sh
Hello Custom
Hello Default
Hello Static
```


<br>

# 2. Java 에서 기본적으로 제공하는 Functional Interfaces

매번 함수형 인터페이스를 직접 만들어서 사용하는 건 번거로운 일입니다.

그래서 Java 에서는 기본적으로 많이 사용되는 함수형 인터페이스를 제공합니다.

기본적으로 제공되는 것만 사용해도 웬만한 람다식은 다 만들 수 있기 때문에 개발자가 직접 함수형 인터페이스를 만드는 경우는 거의 없습니다.

<br>

| 함수형 인터페이스 | Descripter      | Method                    |
| ----------------- | --------------- | ------------------------- |
| Predicate         | `T -> boolean`  | `boolean test(T t)`       |
| Consumer          | `T -> void`     | `void accept(T t)`        |
| Supplier          | `() -> T`       | `T get()`                 |
| Function<T, R>    | `T -> R`        | `R apply(T t)`            |
| Comparator        | `(T, T) -> int` | `int compare(T o1, T o2)` |
| Runnable          | `() -> void`    | `void run()`              |
| Callable          | `() -> T`       | `V call()`                |

<br>

## 2.1. Predicate

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

`Predicate` 는 인자 하나를 받아서 `boolean` 타입을 리턴합니다.

람다식으로는 `T -> boolean` 로 표현합니다.

<br>

## 2.2. Consumer

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

`Consumer` 는 인자 하나를 받고 아무것도 리턴하지 않습니다.

람다식으로는 `T -> void` 로 표현합니다.

소비자라는 이름에 걸맞게 무언가 (인자) 를 받아서 소비만 하고 끝낸다고 생각하면 됩니다.

<br>

## 2.3. Supplier

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

`Supplier` 는 아무런 인자를 받지 않고 T 타입의 객체를 리턴합니다.

람다식으로는 `() -> T` 로 표현합니다.

공급자라는 이름처럼 아무것도 받지 않고 특정 객체를 리턴합니다.

<br>

## 2.4. Function

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

`Function` 은 T 타입 인자를 받아서 R 타입을 리턴합니다.

람다식으로는 `T -> R` 로 표현합니다.

수학식에서의 함수처럼 특정 값을 받아서 다른 값으로 반환해줍니다.

T 와 R 은 같은 타입을 사용할 수도 있습니다.

<br>

## 2.5. Comparator

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

`Comparator` 은 T 타입 인자 두개를 받아서 `int` 타입을 리턴합니다.

람다식으로는 `(T, T) -> int` 로 표현합니다.

<br>

## 2.6. Runnable

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

`Runnable` 은 아무런 객체를 받지 않고 리턴도 하지 않습니다.

람다식으로는 `() -> void` 로 표현합니다.

`Runnable` 이라는 이름에 맞게 "실행 가능한" 이라는 뜻을 나타내며 이름 그대로 실행만 할 수 있다고 생각하면 됩니다.

<br>

## 2.7. Callable

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

`Callable` 은 아무런 인자를 받지 않고 T 타입 객체를 리턴합니다.

람다식으로는 `() -> T` 로 표현합니다.

`Runnable` 과 비슷하게 `Callable` 은 "호출 가능한" 이라고 생각하면 좀 더 와닿습니다.

<br>

### Supplier vs Callable

`Supplier` 와 `Callable` 은 완전히 동일합니다.

아무런 인자도 받지 않고 특정 타입을 리턴해줍니다.

둘이 무슨 차이가 있을까..?

사실 그냥 차이가 없다고 생각하시면 됩니다.

단지 `Callable` 은 `Runnable` 과 함께 병렬 처리를 위해 등장했던 개념으로서 `ExecutorService.submit` 같은 함수는 인자로 `Callable` 을 받습니다.

<br>

# 3. 두 개의 인자를 받는 Bi 인터페이스

특정 인자를 받는 `Predicate`, `Consumer`, `Function` 등은 두 개 이상의 타입을 받을 수 있는 인터페이스가 존재합니다.

<br>

| 함수형 인터페이스 | Descripter          | Method                   |
| ----------------- | ------------------- | ------------------------ |
| BiPredicate       | `(T, U) -> boolean` | `boolean test(T t, U u)` |
| BiConsumer        | `(T, U) -> void`    | `void accept(T t, U u)`  |
| BiFunction        | `(T, U) -> R`       | `R apply(T t, U u)`      |

<br>

# 4. 기본형 특화 인터페이스

지금까지 확인한 함수형 인터페이스를 **제네릭 함수형 인터페이스**라고 합니다.

자바의 모든 형식은 참조형 또는 기본형입니다.
- 참조형 (Reference Type) : Byte, Integer, Object, List
- 기본형 (Primitive Type) : int, double, byte, char

<br>

`Consumer<T>` 에서 T 는 참조형만 사용 가능합니다.

Java 에서는 기본형과 참조형을 서로 변환해주는 박싱, 언박싱 기능을 제공합니다.

- 박싱 (Boxing) : 기본형 -> 참조형 (`int -> Integer`)
- 언박싱 (Unboxing) : 참조형 -> 기본형 (`Integer -> int`)

<br>

게다가 개발자가 박싱, 언박싱을 신경쓰지 않고 개발할 수 있게 **자동으로 변환해주는 오토박싱 (Autoboxing)** 이라는 기능도 제공합니다.

예를 들어 `List<Integer> list` 에서 `list.add(3)` 처럼 기본형을 바로 넣어도 사용 가능한 것도 오토박싱 덕분입니다.

하지만 이런 변환 과정은 비용이 소모되기 때문에, 함수형 인터페이스에서는 이런 오토박싱 동작을 피할 수 있도록 **기본형 특화 함수형 인터페이스** 를 제공합니다.

`IntPredicate`, `LongPredicate` 등등 특정 타입만 받는 것이 확실하다면 기본형 특화 인터페이스를 사용하는 것이 더 좋습니다.

아래에서 소개하는 인터페이스 외에 `UnaryOperator` 나 `Bi` 인터페이스에도 기본형 특화를 제공합니다.

<br>

## 4.1 Predicate (`T -> boolean`)

기본형을 받은 후 `boolean` 리턴

- `IntPredicate`
- `LongPredicate`
- `DoublePredicate`

<br>

## 4.2. Consumer (`T -> void`)

기본형을 받은 후 소비

- `IntConsumer`
- `LongConsumer`
- `DoubleConsumer`

<br>

## 4.3. Function (`T -> R`)

기본형을 받아서 기본형 리턴

- `IntToDoubleFunction`
- `IntToLongFunction`
- `LongToDoubleFunction`
- `LongToIntFunction`
- `DoubleToIntFunction`
- `DoubleToLongFunction`

<br>

기본형을 받아서 R 타입 리턴

- `IntFunction<R>`
- `LongFunction<R>`
- `DoubleFunction<R>`

<br>

T 타입 받아서 기본형 리턴

- `ToIntFunction<T>`
- `ToDoubleFunction<T>`
- `ToLongFunction<T>`

<br>

## 4.4. Supplier (`() -> T`)

아무것도 받지 않고 기본형 리턴

- `BooleanSupplier`
- `IntSupplier`
- `LongSupplier`
- `DoubleSupplier`

<br>

# Conclusion

Java 8 에서 람다에 활용 가능한 함수형 인터페이스를 제공하고 있습니다.

직접 만들어서 쓸 수도 있지만 이미 제공하는 인터페이스로도 대부분 처리 가능하므로 어떤 게 있는지 잘 파악해서 활용해야 합니다.

<br>

# Reference

- [Modern Java in Action](http://www.yes24.com/Product/Goods/77125987)