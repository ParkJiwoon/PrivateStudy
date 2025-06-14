# Kotlin 에서 JPA 사용하기 1 (all-open, no-arg 플러그인)

# Overview

Kotlin 으로 Spring Boot 를 만들다보면 JPA 를 함께 사용하는 일이 많습니다.

JPA 는 대표적인 Spring Boot 의 ORM 이지만 Kotlin 과 함께 사용하려면 몇가지 불편한 점이 존재합니다.

이런 불편한 점들을 해결하기 위해 Kotlin 측에서는 JPA 에서 사용하기 적합한 몇가지 플러그인을 제공합니다.

<br>

# 1. Kotlin JPA plugin

Kotlin 에서 제공하는 `plugin.jpa` 에는 다음 두 가지 플러그인이 포함되어 있습니다.
- `plugin.allopen`
- `plugin.noarg`

<br>

## 1.1. all-open plugin

Kotlin 에서는 기본적으로 모든 클래스와 메서드가 `final` 입니다.

이 말은 클래스나 메서드를 상속/오버라이드 할 수 없다는 뜻입니다.

Java 에서는 기본적으로 `open` 이라 상속에 열려있는 반면에 Kotlin 은 의도된 상속 구조를 사용하기 위해 기본적으로 `final` 이고 상속 가능한 클래스만 명시적으로 `open` 을 선언합니다.

JPA 에서는 지연 로딩 (lazy loading), 더티 체킹 (dirty checking) 등에 사용되는 프록시 객체를 만들기 위해서는 클래스가 상속 가능해야 합니다.

그래서 Kotlin 에서는 JPA 클래스를 `open` 으로 변경시켜주는 ["all-open" 플러그인](https://kotlinlang.org/docs/all-open-plugin.html)을 제공합니다.

<br>

## 1.2. no-arg plugin

JPA 에서는 엔티티 객체를 리플렉션 (Reflection) 으로 생성햐는데 구현상 반드시 파라미터 없는 기본 생성자가 필요합니다.  

Kotlin 에서는 정의된 생성자를 통해 모든 필드를 초기화하도록 요구하므로, 별도로 기본생성자를 만들어주지 않으면 JPA 에서 인스턴스를 생성할 수 없습니다.

사실 기본생성자가 없는 문제는 Kotlin 만의 문제는 아니고 Java 에서도 `@Entity` 객체는 항상 기본생성자를 직접 만들어주거나 Lombok 의 `@NoArgConstructor` 를 붙여줘야 합니다.

Kotlin 에서는 JPA 클래스에 기본생성자를 붙여주는 ["no-arg" 플러그인](https://kotlinlang.org/docs/no-arg-plugin.html)을 제공합니다.

<br>

# 2. 적용 방법

```kt
plugins {
    val kotlinVersion = "2.1.20"
    kotlin("plugin.jpa") version kotlinVersion
}
```

Kotlin `2.1.20` 버전 사용 기준으로 위와 같이 추가해주시면 됩니다.

`allopen`, `noarg` 플러그인을 별도 사용시 원래는 `allOpen`, `noArg` 블럭을 추가하여 적용 대상을 지정해야 했는데 `plugin.jpa` 에는 해당 설정 또한 내장되어 있기 때문에 따로 추가할 필요가 없습니다.

<br>

# Conclusion

Spring Initializr 에서는  Kotlin + JPA 선택 시 `plugin.jpa` 를 자동으로 추가해줍니다.

그래서 Spring Initializr 나 Gradle 설정을 통해 무심코 적용되는 경우가 많지만 실제로 Kotlin 에서 JPA 를 안정적으로 사용하기 위해 중요한 역할을 하는 플러그인들을 정리해봤습니다.