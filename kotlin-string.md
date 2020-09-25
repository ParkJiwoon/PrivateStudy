# drop, dropLast, dropWhile, dropLastWhile

- `drop` : 앞에서부터 n 개의 문자를 제거한 String 을 반환
- `dropLast`:  뒤에서부터 n 개의 문자를 제거한 String 을 반환
- `dropWhile, dropLastWhile`: 조건을 만족하지 않는 문자가 나올때까지 제거한 String 을 반환

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

# Reference

[String](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/)
