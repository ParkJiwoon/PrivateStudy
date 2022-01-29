# Effective Java Item 06. 불필요한 객체 생성을 피하라

# Overview

Java 는 새로운 인스턴스를 만들 때 메모리에 객체를 저장합니다.

불필요한 객체를 생성해서 메모리를 낭비하고 성능을 떨어트리는 몇가지 예시를 알아봅니다.

<br>

# 1. 공유 객체는 new 로 생성하지 말자

```java
// bad case
String s = new String("example");

// good case
String s = "example";
```

똑같은 기능의 객체를 매번 생성하는 것보다는 객체 하나를 재사용하는 것이 낫습니다.

새로운 객체를 생성할 때마다 `new` 키워드를 사용하는 걸 자제해야 합니다.

위 두 개의 코드는 새로운 객체를 생성한다는 사실은 같습니다.

하지만 새로운 `String` 을 사용하는 경우에 차이가 발생합니다.

<br>

```java
// bad case : 새로운 인스턴스를 생성함 (s != a)
String s = new String("example");
String a = new String("example");

// good case : 같은 인스턴스를 사용함 (s == a)
String s = "example";
String a = "example";
```

이렇게 동일한 값을 하나 더 선언해서 사용하는 경우, `new` 를 사용하면 새로운 인스턴스를 생성합니다.

반면에 good case 처럼 사용하면 하나의 인스턴스를 서로 공유하게 되고 불필요한 객체를 생성하지 않습니다.

<br>

# 2. 객체 생성 비용이 비싸면 캐싱하자

객체의 생성 비용이 비싼 경우도 있습니다.

이런 비싼 객체가 반복해서 사용된다면 캐싱해서 재사용하는 게 좋습니다.

가장 대표적인 예로 정규표현식 활용이 있습니다.

Java 에서는 문자열 정규표현식을 사용하기 위해 `String.matches` 메서드를 사용하는데, 이 메서드는 내부적으로 `Pattern` 인스턴스를 생성합니다.

`Pattern` 인스턴스는 정규표현식에 해당하는 유한 상태 머신 (finite state machine) 을 만들기 때문에 인스턴스 생성 비용이 높습니다.

따라서 이런 경우에는 `Pattern` 인스턴스를 미리 생성해서 캐싱해두고 인스턴스가 호출될 때마다 재사용하는 방법이 있습니다.

```java
// bad case: 내부적으로 매번 Pattern 인스턴스를 생성해서 비효율적
boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*");
}

// good case: Pattern 인스턴스를 미리 생성해서 캐싱해두자
private static final Pattern ROMAN = Pattern.compile("^(?=.)M*");

boolean isRomanNumeral(String s) {
    return ROMAN.matcher(s).matches();
}
```

<br>

# 3. 오토 박싱에 유의하자

Java 에는 원시 타입 (Primitive Type) 과 참조 타입 (Reference Type) 이 존재합니다.

두 타입을 같이 사용할 때 개발자가 명시적으로 타입 변환을 하지 않아도 두 타입을 상호 변환시켜주는데, 이걸 오토 박싱 (Auto Boxing) 이라고 합니다.

<br>

```java
Long sum = 0L;

for (long i = 0; i < 10; i++) {
    sum += i;
}

return sum;
```

위 코드를 보면 `Long sum` 변수는 참조 타입을 사용하고 매번 더하는 `long i` 변수 원시 타입을 사용합니다.

이 때 매 반복문마다 `i` 값을 `Long` 으로 감싸는 (오토 박싱) 작업이 발생합니다.

결국 필요 없는 `Long` 인스턴스가 무수히 생성되고, 성능상으로도 메모리 상으로도 굉장히 비효율적입니다.

경우에 따라 다르겠지만, 박싱된 타입보다는 기본 타입을 사용하고 의도치 않은 오토 박싱에 주의해야합니다.

<br>

# 4. 꼭 필요한 상황이 아니라면 객체 풀 (pool) 을 생성하지 말자

재사용이 필요한 인스턴스는 캐싱해두는 게 좋다고 했지만 단순히 객체 생성을 피하고자 풀 (pool) 을 만드는 것은 좋지 않습니다.

요즘의 JVM 가비지 컬렉터는 많이 좋아져서 작고 간단한 객체를 생성하고 회수하는 일이 크게 부담되지 않습니다.

데이터베이스 연결처럼 생성 비용이 워낙 비싸서 재사용 하는 편이 나은 경우를 제외하면, 객체 풀 생성은 가독성을 떨어트리고 잘못된 경우 성능까지 떨어트립니다.

<br>

# Conclusion

이번 주제의 핵심은 "불필요한" 객체 생성을 피하자입니다.

당연히 프로그램이 돌아가다보면 객체의 생성과 소멸은 자연스러운 일이기 때문에 완전히 피할 수는 없습니다.

하지만 굳이 만들 필요도 없는 객체를 생성하는 습관을 줄인다면 조금 더 건강한 코드를 작성할 수 있을 겁니다.