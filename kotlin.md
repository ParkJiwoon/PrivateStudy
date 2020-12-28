# Kotlin == 비교 연산자

- [Kotlin == 비교 연산자](#kotlin--비교-연산자)

<br>

## Java 에서 두 값이 같은 지 알고싶다면?

원시 타입 (Primitive Type) `int, char, ...` 인 경우 `==` 을 사용해서 값 비교할 수 있다.

참조 타입 (Reference Type) `Integer, String, ...` 인 경우 `==` 을 사용하면 주소 값을 비교하고 다른 객체는 다른 주소를 바라보기 때문에 값이 같아도 `false` 가 된다.

`equals` 를 사용하면 값을 비교할 수 있음

<br>

## Kotlin 에서는?

타입에 관계없이 `==` 을 사용하면 된다.

코틀린 `==` 은 내부적으로 `equals` 호출을 하기 때문에 `String` 끼리의 비교 여도 상관 없다.

만약 주소 값을 비교하고 싶다면 `===` 을 사용하면 된다.

<br>

# Swap

Python 만큼은 아니지만 Kotlin 에서도 문법을 활용하여 값의 Swap 을 쉽게 짤 수 있다.

```kotlin
var a = 1
var b = 2

a = b.also { b = a }

println(a) // print 2
println(b) // print 1
```
