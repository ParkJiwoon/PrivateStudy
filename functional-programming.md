# 함수형 프로그래밍 (Functional Programming) 이란?

`함수형 프로그래밍(functional programming)` 은 자료 처리를 수학적 함수의 계산으로 취급하고 상태와 가변 데이터를 멀리하는 프로그래밍 패러다임의 하나이다. - [wiki](https://ko.wikipedia.org/wiki/%ED%95%A8%EC%88%98%ED%98%95_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)

`명령형 프로그래밍` 과 반대되는 의미로 `선언형 프로그래밍` 의 한 종류로 `함수형 프로그래밍` 이 존재합니다.

간단하게 설명하면 말그대로 함수를 사용해서 프로그래밍을 하는 겁니다.

명령형 프로그래밍은 코드에서 "how" 에 초점을 두었다면 함수형 프로그래밍은 "what" 에 초점을 두고 있습니다.

함수형 프로그래밍은 아래와 같은 특징을 갖고 있습니다.

- 함수는 1급 객체여야 한다. 고차함수 (HoF, Higher-order Function) 라고도 한다.
- 부수 효과 (Side-Effect) 가 없는 순수 함수 (Pure Function) 을 사용한다.
- 익명 함수 (Anonymous Function) 을 사용한다.

<br>

## 1급 객체 (First-class citizen) 란?

1급 객체는 아래 세 가지 조건을 충족해야 합니다.

1. 변수나 데이터에 할당 가능
2. 객체의 인자로 넘길 수 있다.
3. 객체의 리턴값으로 리턴할 수 있어야 한다.

<br>

간단하게 생각해보면 Java 는 되고 Kotlin 은 되지 않습니다.

왜냐하면 Java 는 함수를 변수에 할당할 수 없고 인자로 넘길수도 없으며 리턴값으로 리턴할 수도 없기 때문입니다.

그에 비해 Kotlin 은 `type 의 할당, 전달, 반환` 세 개가 전부 가능합니다.

Kotlin 또는 JavaScript 의 함수는 인자로 넘겨지거나 반환값으로 사용될 수 있기 때문에 고차함수라고 할 수 있습니다.

<br>

## 순수 함수 (Pure Function)

순수한 함수란 부작용 또는 부수효과 (Side Effect) 가 없는 함수를 말합니다.

Side Effect 가 없기 때문에 함수의 실행이 외부에 영향을 끼치지 않고, Thread Safe 하며 병렬 계산을 가능케 합니다.

순수 함수는 다음과 같은 특징을 가집니다.

- 입력으로 주어진 것 외에 연산은 실행하지 않는다.
- 상태의 변화가 없다.
- 입력이 동일하면 출력이 동일하다.

<br>

코드로 예시를 들어 보겠습니다.

```java
public static void main(String[] args) {
  List<String> names = new ArrayList<>();
  names.add("Kim");
  names.add("Lee");  // ["Kim", "Lee"]

  addName(names, "Park"); // ["Kim", "Lee", "Park", "Choi"]
  addName(names, "Park"); // ["Kim", "Lee", "Park", "Choi", "Park", "Choi"]
}

private static void addName(List<String> names, String newName) {
  String hiddenName = "Choi";
  names.add(newName);
  names.add(hiddenName);
}
```

<br>

이 함수는 순수함수의 세 가지 특징을 모두 어깁니다.

1. 입력으로 주어진 것 외에 `hiddenName` 을 함수 내부에서 선언해서 사용한다.
2. 함수를 사용하기 전과 후의 `names` 내부의 값은 다르다.
3. `addName(names, "Park")` 처럼 동일한 입력으로 두번 호출했지만 출력 결과는 다르다.

<br>

순수 함수는 외부 요소의 변화가 없고 동일한 입력에는 항상 동일한 출력을 해야 합니다.

```java
// 순수 함수의 예
int add(int a, int b) {
  return a + b;
}
```

<br>

## 익명 함수 (Anonymous Function)

익명 함수는 말그대로 이름이 없는 함수를 말합니다.

전통적인 명령형 언어에서는 모든 함수에 이름이 붙고 내용이 정의되지만, 함수형 언어에서는 익명 함수를 바로 실행하거나 인자로 넘길 수 있습니다.

```kotlin
// Kotlin 익명 함수
fun main() {
    add(fun(x: Int, y: Int) {
        println("Execute add Function $x and $y")
    }, 1, 2)
}

fun add(logFn: (x: Int, y: Int) -> Unit, a: Int, b: Int): Int {
    logFn.invoke(3, 4)
    return a + b;
}
```
