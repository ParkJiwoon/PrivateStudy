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
