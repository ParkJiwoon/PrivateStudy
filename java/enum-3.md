# Java Enum 3편

# 1. Overview

Java Enum 3편입니다.

이번 편은 Effective Java 에 있는 여러가지 Enum 활용법입니다.

<br>

# 2. ordinal 메서드 대신 인스턴스 필드를 사용하라

Enum 클래스에는 기본적으로 `ordinal` 이라는 메소드를 제공합니다.

0 부터 시작되며 특정 상수값의 위치 (Index) 를 리턴해줍니다.

Enum API 문서를 보면 `ordinal` 에 대해서 이렇게 쓰여 있습니다.

"대부분의 개발자는 이 메소드를 쓸 일이 없다. 이 메소드는 `EnumSet` 과 `EnumMap` 같이 열거 타입 기반 범용 자료구조에 쓸 목적으로 설계되었다."

`oridnal` 을 사용할 때의 단점은 여러 가지 있습니다.

- 나중에 추가될 Enum 상수값이 꼭 순서대로라는 보장이 없다
- 중복된 숫자를 가져야 할 때 구분이 불가능하다

그러므로 `ordinal` 메소드를 사용하지 말고 별도의 인스턴스 필드를 선언해서 사용합시다.

<br>

# 3. 비트 필드 대신 EnumSet 을 사용하라

과거에는 여러 값들을 집합으로 사용해야 할 경우 비트로 사용했습니다.

```java
public class Text {
    public static final int STYLE_BOLD          = 1 << 0;   // 1
    public static final int STYLE_ITALIC        = 1 << 1;   // 2
    public static final int STYLE_UNDERLINE     = 1 << 2;   // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3;   // 8

    // 매개변수 styles 는 0 개 이상의 STYLE_ 상수를 비트별 OR 한 값
    public void applyStyles(int styles) {
        // ...
    }
}

// usage
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

- 여러 개의 상수값을 OR 하여 사용하면 집합을 나타낼 수 있음
- 이렇게 만들어진 집합을 비트 필드 (bit field) 라고 함
- 비트 필드 값은 해석하기 어려움
- 최대 몇 비트가 필요한지 API 작성 시 미리 예측하여 적절합 타입 (int, long) 을 선택해야 함

<br>

## 3.1. EnumSet 클래스

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public void applyStyles(Set<Style> styles) {
        // ...
    }
}

// usage
text.applyStyles(EnumSet.of(Text.Style.BOLD, Text.Style.ITALIC));
```

- `java.util` 패키지
- `Set` 인터페이스를 구현하며, 타입 안전하고, 다른 어떤 `Set` 구현체와도 함께 사용 가능
- `EnumSet` 내부는 비트 벡터로 구현됨

<br>

# 4. ordinal 인덱싱 대신 EnumMap 을 사용하라

Enum 값을 Index 로 사용하고 싶을 때 `배열 + ordinal` 을

