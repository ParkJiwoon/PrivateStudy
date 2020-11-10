# Swap

Python 만큼은 아니지만 Kotlin 에서도 문법을 활용하여 값의 Swap 을 쉽게 짤 수 있다.

```kotlin
var a = 1
var b = 2

a = b.also { b = a }

println(a) // print 2
println(b) // print 1
```
