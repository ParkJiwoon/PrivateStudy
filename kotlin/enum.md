# Kotlin Enum

# 1. Overview

[Java Enum](../java/enum-1.md) 에 이어 Kotlin Enum 사용법에 대해서도 알아봅니다.

사용법과 이유에 대해서는 Java 에서 알아보았으니 간단하게 코드만 작성합니다.

<br>

# 2. 기본 사용법

```kotlin
enum class Day {
    MON, TUE, WED, THU, FRI, SAT, SUN
}
```

<br>

# 3. 필드값 추가

```kotlin
enum class Day(
    val label: String
) {
    MON("Monday"),
    TUE("Tuesday"),
    WED("Wednesday"),
    THU("Thursday"),
    FRI("Friday"),
    SAT("Saturday"),
    SUN("Sunday")
    ;
}
```

Kotlin 은 원래 `;` 을 안쓰지만 Enum 클래스에 필드값을 추가하는 경우 마지막에 꼭 추가해야합니다.

<br>

# 4. 필드값 캐싱

```kotlin
enum class Day(
    val label: String
) {
    MON("Monday"),
    TUE("Tuesday"),
    WED("Wednesday"),
    THU("Thursday"),
    FRI("Friday"),
    SAT("Saturday"),
    SUN("Sunday")
    ;

    companion object {
        private val LABEL_CACHE: Map<String, Day> =
            values().associateBy { it.label }
        
        fun findByLabel(label: String) = LABEL_CACHE[label]
    }
}
```

<br>

# 5. 상수별 메소드 구현

```kotlin
enum class Operation {
    PLUS {
        override fun apply(x: Double, y: Double): Double {
            return x + y
        }
    },

    MINUS {
        override fun apply(x: Double, y: Double): Double {
            return x - y
        }
    },

    TIMES {
        override fun apply(x: Double, y: Double): Double {
            return x * y
        }
    },

    DIVIDE {
        override fun apply(x: Double, y: Double): Double {
            return x / y
        }
    };

    abstract fun apply(x: Double, y: Double): Double
}
```

<br>

# Reference

- [Kotlin Enum Classes](https://kotlinlang.org/docs/enum-classes.html)