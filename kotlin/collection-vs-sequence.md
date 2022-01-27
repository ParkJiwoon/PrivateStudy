# Kotlin Collections 와 Sequences 의 차이점 (feat. Java Stream)

# Overview

Java8 에서는 Collection 을 다루기 위해 Stream 을 사용합니다.

Kotlin 은 Collections 자체에서 `filter`, `map` 등의 여러 가지 API 를 제공하기 때문에 매번 `.streams()` 를 붙이지 않아도 사용 가능하다는 장점이 있습니다.

하지만 비슷해보이는 두 코드 사이에는 큰 차이점이 하나 있는데요.

바로 Lazy Evaluation 입니다.

<br>

# 1. Lazy Evaluation

Lazy Evaluation 이란 쉽게 말해서 **필요하지 않은 연산을 하지 않는다** 라고 이해할 수 있습니다.

어떤 로직이나 연산을 그 즉시 수행하지 않고 실제로 사용되기 전까지 미루는 걸 의미합니다.

반대의 의미인 Eager Evaluation 은 연산이 있으면 그때그때 수행하는 것을 의미합니다.

Java Stream 의 예시와 함께 보면 쉽게 이해할 수 있습니다.

<br>

# 2. Java Streams

```java
Stream.of(1, 2, 3, 4, 5, 6)
        .filter(e -> {
            System.out.println("filter: " + e);
            return e < 3;
        })
        .map(e -> {
            System.out.println("map: " + e);
            return e * e;
        })
        .anyMatch(e -> {
            System.out.println("anyMatch: " + e);
            return e > 2;
        });
```

- (1, 2, 3, 4, 5, 6) 의 숫자 묶음 존재
- `filter`: 3 보다 작은 수만 추출
- `map`: 제곱으로 변환
- `anyMatch`: 2 보다 큰 수가 있는지 확인

조금 지저분해 보이지만 이해를 돕기 위해 연산 중간중간마다 print 문을 추가했습니다.

위 코드를 있는 그대로 나열하면 6 개의 숫자 묶음에서 3 보다 작은 수만 뽑아서 제곱한 뒤 그 중에서 2 보다 큰 수가 있는지 확인하는 겁니다.

위 코드의 결과값은 아래와 같습니다.

<br>

```text
filter: 1
map: 1
anyMatch: 1
filter: 2
map: 2
anyMatch: 4
```

총 6 개의 숫자가 있었지만 실제로 연산된 것은 두 개 뿐입니다.

`anyMatch` 조건에 해당하는 숫자가 나오자 이후 숫자들은 볼 필요가 없다고 판단하여 전부 생략했습니다.

이게 바로 Lazy Evaluation (필요하지 않는 연산은 하지 않는다) 입니다.

<br>

# 3. Kotlin Collections

```kt
listOf(1, 2, 3, 4, 5, 6)
    .filter {
        println("filter: $it")
        it < 3
    }
    .map {
        println("map: $it")
        it * it
    }
    .any {
        println("any: $it")
        it > 2
    }
```

그럼 이제 같은 로직을 Kotlin 으로 작성해보았습니다.

위 로직을 실행하며 어떻게 될까요?

<br>

```text
filter: 1
filter: 2
filter: 3
filter: 4
filter: 5
filter: 6
map: 1
map: 2
any: 1
any: 4
```

한 눈에 봐도 결과가 다른 것을 알 수 있습니다.

Kotlin Collections 는 매 연산마다 모든 원소에 대해서 수행합니다.

데이터의 양이 많으면 많을수록 성능 차이는 더욱 벌어질 겁니다.

<br>

# 4. Kotlin Sequences

Kotlin 에서도 Lazy Evaluation 을 수행하게 하는 방법이 있습니다.

바로 [Sequence](https://kotlinlang.org/docs/sequences.html) 를 사용하는 겁니다.

위 코드에서 한줄만 추가하면 됩니다.

<br>

```kt
listOf(1, 2, 3, 4, 5, 6)
    .asSequence()   // 이 부분을 추가해서 Sequence 로 변경
    .filter {
        println("filter: $it")
        it < 3
    }
    .map {
        println("map: $it")
        it * it
    }
    .any {
        println("any: $it")
        it > 2
    }
```

Collection 에서 수행하지 말고 `asSequence()` 를 사용해서 Sequence 로 변경한 뒤에 연산을 수행하면 됩니다.

위 코드의 결과는 다음과 같습니다.

<br>

```text
filter: 1
map: 1
any: 1
filter: 2
map: 2
any: 4
```

이제 불필요한 연산을 하지 않는 것을 볼 수 있습니다.

<br>

# 5. 왜 그럴까?

Lazy Evaluation 에 대해 좀더 설명하면 **중간 단계 (intermediate step)** 의 결과를 바로 리턴하냐 아니냐의 차이에 있습니다.

Kotlin Collections 은 매 연산을 수행할 때마다 결과 Collection 을 반환합니다.

이에 비해 Kotlin Sequences 또는 Java Streams 는 종료 (terminate) 함수가 호출되기 전까지는 연산을 수행하지 않습니다.

위에서 사용한 `any()` 함수 또한 종료 함수입니다.

이 차이를 쉽게 알려면 종료 함수가 없는 Sequences 를 사용해보면 됩니다.

<br>

## 5.1. Kotlin Sequences 의 Lazy Evaluation 확인

```kotlin
val sequence: Sequence<Int> = listOf(1, 2, 3)
    .asSequence()
    .filter {
        println("filter: $it")
        it < 2
    }
    .map {
        println("map: $it")
        it * it
    }

println("종료함수를 아직 호출하지 않음")
sequence.toList()
```

Sequences 는 매 함수의 결과로 `Sequence` 를 반환합니다.

그래서 최종적으로 Collection 으로 변환하려면 다시 `toList()` 를 호출해야 합니다.

`toList()` 역시 종료함수라서 호출되는 순간에 모든 연산이 수행됩니다.

Java Streams 의 `collect(Collectors.toList())` 와 같다고 생각하시면 됩니다.

<br>

```text
종료함수를 아직 호출하지 않음
filter: 1
map: 1
filter: 2
filter: 3
```

<br>

## 5.2. Kotlin Collections 의 Eager Evaluation 확인

```kotlin
val list: List<Int> = listOf(1, 2, 3)
    .filter {
        println("filter: $it")
        it < 2
    }
    .map {
        println("map: $it")
        it * it
    }

println("Collection 은 매번 List 를 반환하기 때문에 이미 연산됨")
```

Sequences 와 다르게 Collections 은 매 함수의 결과로 `Collection` 을 반환합니다.

사실상 매 함수가 모두 종료 함수라고 볼 수 있으며, 그래서 결과를 다음 단계로 넘기지 못하고 매번 전부 연산을 하는겁니다.

<br>

```text
filter: 1
filter: 2
filter: 3
map: 1
Collection 은 매번 List 를 반환하기 때문에 이미 연산됨
```

<br>

# Conclusion

이제 Kotlin Sequences 는 Lazy Evaluation 때문에 불필요한 연산을 생략한다는 점을 알았습니다.

하지만 Sequences 가 항상 좋은 것은 아닙니다.

[Sequences by Kotlin Reference](https://kotlinlang.org/docs/sequences.html) 를 보면 다음과 같은 문구가 있습니다.

> So, the sequences let you avoid building results of intermediate steps, therefore improving the performance of the whole collection processing chain. However, the lazy nature of sequences adds some overhead which may be significant when processing smaller collections or doing simpler computations. Hence, you should consider both Sequence and Iterable and decide which one is better for your case.

<br>

요약하자면 Sequences 는 중간 단계의 결과를 생략하기 때문에 성능 향상이 되지만, 오버헤드가 있기 때문에 데이터가 적거나 연산이 단순한 컬렉션을 처리할 때는 오히려 안좋을 수가 있다고 합니다.

그러므로 각자 상황에 맞춰 적절한 방법을 선택하는게 가장 좋습니다.

<br>

# Reference

- [Sequences by Kotlin Reference](https://kotlinlang.org/docs/sequences.html)
- [Kotlin - Collections와 Sequences 의 차이점 by codechacha](https://kotlinlang.org/docs/sequences.html)