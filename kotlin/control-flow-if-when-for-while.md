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
