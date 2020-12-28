# Contents

- [For](#for)
- [Kotlin 비교 연산자](#kotlin-비교-연산자)
- [Swap](#swap)
- [drop, dropLast, dropWhile, dropLastWhile](#drop,-dropLast,-dropWhile,-dropLastWhile)

<br>

# For

Kotlin 에서의 for 문은 자바와 마찬가지로 iterator 를 사용합니다.

따라서 `iterator()`, `next()`, `hasNext()` 을 제공하는 모든 컬렉션에서 for 문을 사용할 수 있습니다.

기본 for 문은 아래와 같습니다.

```kotlin
for (item in collection) {
    print(item)
}
```

<br>

특정 범위 (range) 를 지정해서 사용할 수도 있습니다.

`..` 을 사용하면 마지막 숫자까지 **포함하**고 `until` 을 사용하면 마지막 숫자는 **포함하지 않습니다.**

```kotlin
// 1, 2, 3
for (i in 1..3) {
    println(i)
}

// 1, 2
for (i in 1 until 3) {
    println(i)
}
```

<br>

`downTo` 를 이용하면 감소하는 for 문을 만들 수 있고 `step` 을 사용하면 증가되는 양을 수정할 수 있습니다.

```kotlin
// 6, 4, 2, 0
for (i in 6 downTo 0 step 2) {
    println(i)
}
```

<br>

배열을 순회할 때는 `index` 만 뽑아내거나 `(index, value)` 형태로 순회 가능합니다.

```kotlin
for (i in array.indices) {
    println(array[i])
}
```

```kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

<br>

컬렉션 타입에서는 `forEach`, `forEachIndexed` 를 사용하여 람다식으로 표현할 수 있습니다.

```kotlin
// 1, 2, 3
listOf(1, 2, 3).forEach { println(it) }

/**
 * index: 0, value: 4
 * index: 1, value: 5
 * index: 2, value: 6
 */
listOf(4, 5, 6).forEachIndexed { index, value -> 
    println("index: $index, value: $value")
}
```

<br>

## Reference

- [Control Flow: if, when, for, while](https://kotlinlang.org/docs/reference/control-flow.html#control-flow-if-when-for-while)

<br>

# Kotlin 비교 연산자

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

<br>

# drop, dropLast, dropWhile, dropLastWhile

`drop` 은 문자열의 앞이나 뒷부분을 자를 때 사용된다.

내부적으로는 `substring` 으로 구현되어 있다.


- `drop` : 앞에서부터 n 개의 문자를 제거한 String 을 반환
- `dropLast`:  뒤에서부터 n 개의 문자를 제거
- `dropWhile, dropLastWhile`: 조건을 만족하지 않는 문자가 나올때까지 제거

<br>

```kotlin
/* definition */
fun String.drop(n: Int): String

fun String.dropLast(n: Int): String

fun String.dropWhile(
  predicate: (Char) -> Boolean
): String

fun String.dropLastWhile(
  predicate: (Char) -> Boolean
): String


/* exmaple */
val string = "<<<First Grade>>>"

println(string.drop(6)) // st Grade>>>
println(string.dropLast(6)) // <<<First Gr
println(string.dropWhile { !it.isLetter() }) // First Grade>>>
println(string.dropLastWhile { !it.isLetter() }) // <<<First Grade
```

<br>

# Reference

[String](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/)
