# 람다 표현식 내에서 사용하는 변수가 fianl 처럼 사용되어야 하는 이유

# 1. Overview

Java 의 람다 표현식은 익명 함수처럼 자유 변수 (Free Variable) 를 활용할 수 있습니다.

여기서 자유변수란 **파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수** 를 의미합니다.

<br>

# 2. 자유 변수 제약

하지만 모든 자유 변수를 사용할 수 있는건 아닙니다.

람다 표현식에서 사용되는 변수는 **final 이거나 final 처럼 사용** 되어야 합니다.

여기서 `final` 처럼 사용되어야 한다는건 `effectively final` 라고 표현하는데 **`final` 선언이 되어 있지 않지만 변수의 재할당이 발생하지 않는 변수** 를 의미합니다.

<br>

# 3. Effectively final

람다 표현식으로 사용 불가능한 변수는 인텔리제이에서 다음과 같이 경고해줍니다.

`Variable used in lambda expression should be final or effectively final`

간단한 예를 들어서 `effectively final` 가 뭔지 쉽게 이해해 봅니다.

<br>

## 3.1. 가능

```java
int number = 1357;
Runnable r = () -> System.out.println(number);
```

위 코드에서 `number` 변수는 `final` 선언이 되어 있지 않지만 에러가 발생하지 않습니다.

함수의 재할당이 이루어지지 않았기 때문입니다.

<br>

## 3.2. 불가능

```java
int number = 1357;
number = 2;
Runnable r = () -> System.out.println(number);
```

위 코드는 `number` 가 한번 선언된 후 2 로 재할당 되었기 때문에 람다 표현식 내부에서 사용 불가능합니다.

<br>

```java
int number = 1357;
Runnable r = () -> System.out.println(number);
number = 3;
```

마찬가지로 사용한 이후에 변경되어도 사용 불가능합니다.

<br>

# 4. 이유가 뭘까?






<br>

# Reference

- [Modern Java in Action](http://www.yes24.com/Product/Goods/77125987)
- [Why Do Local Variables Used in Lambdas Have to Be Final or Effectively Final? - Baeldung](https://www.baeldung.com/java-lambda-effectively-final-local-variables)
