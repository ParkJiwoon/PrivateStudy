# JUnit 에서 AssertJ 로 contains 포함 여부 테스트

# 1. Overview

Java 에서 테스트 코드를 짤 때 특정 자료구조의 원소 값을 확인해야 하는 테스트가 있습니다.

반복문을 돌면서 일일히 확인해야 하거나 그냥 코드 한줄 한줄 입력하는 방법도 있지만 `org.assertj.core.api.Assertions` 에서 제공하는 `assertThat().contains()` 를 사용하면 좀 더 깔끔하게 확인할 수 있습니다.

<br>

# 2. contains

`Assertions.assertThat` 이후에 사용하는 `contains` 메소드는 단순합니다.

중복여부, 순서에 관계 없이 값만 일치하면 테스트가 성공합니다.

<br>

# 3. 사용법

```java
void containsTest() {
    List<Integer> list = Arrays.asList(1, 2, 3);

    // Success: 모든 원소를 입력하지 않아도 성공
    assertThat(list).contains(1, 2);

    // Success: 중복된 값이 있어도 포함만 되어 있으면 성공
    assertThat(list).contains(1, 2, 2);

    // Success: 순서가 바뀌어도 값만 맞으면 성공
    assertThat(list).contains(3, 2);

    // Fail: List 에 없는 값을 입력하면 실패
    assertThat(list).contains(1, 2, 3, 4);
}
```

`assertThat(비교대상 자료구조).contains(원소1, 원소2, 원소3, ..)` 형식으로 사용합니다.

위 예시만 보면 사용법을 한눈에 알 수 있습니다.

<br>

# 4. String, Array, Set, List 모두 사용 가능

```java
@Test
void stringContainsTest() {
    String str = "abc";
    assertThat(str).contains("a", "b", "c");
}

@Test
void arrayContainsTest() {
    int[] arr = {1, 2, 3, 4};
    assertThat(arr).contains(1, 2, 3, 4);
}

@Test
void setContainsTest() {
    Set<Integer> set = Set.of(1, 2, 3);
    assertThat(set).contains(1, 2, 3);
}
```

`List` 는 위에서 테스트 했었고, 다른 자료구조도 가능합니다.

<br>

# 5. containsOnly, containsExactly

추가적으로 좀 더 구체적인 테스트를 위한 여러 가지 메소드가 제공됩니다.

그 중에서 두 가지만 추가로 알아봅니다.

<br>

## 5.1. containsOnly: 순서, 중복을 무시하는 대신 원소값과 갯수가 정확히 일치

```java
/*
 * containsOnly 실패 케이스
 *
 * assertThat(list).containsOnly(1, 2);       -> 원소 3 이 일치하지 않아서 실패
 * assertThat(list).containsOnly(1, 2, 3, 4); -> 원소 4 가 일치하지 않아서 실패
 */
@Test
void containsOnlyTest() {
    List<Integer> list = Arrays.asList(1, 2, 3);

    assertThat(list).containsOnly(1, 2, 3);
    assertThat(list).containsOnly(3, 2, 1);
    assertThat(list).containsOnly(1, 2, 3, 3);
}
```

`containsOnly` 는 원소의 순서, 중복 여부 관계 없이 값만 일치하면 됩니다.

`contains` 와 다른 점은 원소의 갯수까지 정확히 일치해야 한다는 점입니다.

예를 들어 위 `list` 에서 `contains(1, 2)` 는 성공하지만 `containsOnly(1, 2)` 는 실패합니다.

<br>

## 5.2. containsExactly: 순서를 포함해서 정확히 일치

```java
/*
 * containsExactly 실패 케이스
 *
 * assertThat(list).containsExactly(1, 2);       -> 원소 3 이 일치하지 않아서
 * assertThat(list).containsExactly(3, 2, 1);    -> list 의 순서가 달라서 실패
 * assertThat(list).containsExactly(1, 2, 3, 3); -> list 에 중복된 원소가 있어서 실패
 */
@Test
void containsExactlyTest() {
    List<Integer> list = Arrays.asList(1, 2, 3);

    assertThat(list).containsExactly(1, 2, 3);
}
```

`containsExactly` 는 원소가 정확히 일치해야 합니다.

중복된 값이 있어도 안되고 순서가 달라져도 안됩니다.

특정 자료구조의 정확한 값을 테스트 하고 싶은 경우에는 이 메소드를 사용할 수 있습니다.

<br>

# Referecne

- [Github 전체 코드](https://github.com/ParkJiwoon/practice-codes/blob/master/spring-test/src/test/java/com/practice/springtest/junit/ContainsTest.java)