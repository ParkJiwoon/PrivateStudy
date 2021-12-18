# Spring Boot Kotlin 연습

# Skills

- Spring Boot 2.6.1
- Kotlin
- Jar
- Spring Data JPA
- MySQL
- AWS
- Multi Module

<br>

# Multi Module 세팅

[Commit Log](https://github.com/Project-DC-Inside/dcinside-server/commit/b8c46419b6b3e2fa911adf35cfc50b2173af0d78) 참고

<br>

# All-open Plugin: 클래스에서 open 접근자 세팅

`build.gradle.kts` 에서 `plugin.spring` 을 추가해야힙니다.

위 플러그인은 allOpen 플러그인이라고 합니다.

Kotlin 은 클래스가 기본적으로 `final` 로 정의되기 때문에 상속하려면 `open` 키워드를 사용해야 합니다.

그리고 Spring AOP 는 CGLIB 를 사용하여 상속을 통한 Proxy 패턴을 사용하기 때문에 `final` 클래스를 사용하면 사용이 불가능합니다.

이 플러그인은 모든 클래스를 기본으로 `open` 으로 만들어줍니다.

Kotlin 자체에서 제공하는 플러그인이며 [Spring 인 경우 spring 플러그인을 사용](https://kotlinlang.org/docs/all-open-plugin.html)하면 된다고 합니다.

아래 Annotation 들에 대해 제공하기 때문에 JPA 의 `@Entity` 처럼 Proxy 패턴을 패턴을 사용하는 다른 키워드는 직접 추가해야 합니다.

- @Component
- @Async
- @Transactional
- @Cacheable
- @SpringBootTest

<br>

멀티 모듈 (멀티 프로젝트) 로 사용한다면 [Gradle 공식 가이드](https://docs.gradle.org/current/userguide/plugins.html#sec:subprojects_plugins_dsl)에 따라서 아래와 같이 사용하면 됩니다.

```kt
// build.gradle.kts
plugins {
    kotlin("plugin.spring") version "1.6.0"
}

// api/build.gradle.kts
plugins {
    kotlin("plugin.spring")
}
```

<br>

# No-arg Plugin: 기본 생성자 추가

[No-arg compiler plugin](https://kotlinlang.org/docs/no-arg-plugin.html) 을 참고했습니다.

다음 어노테이션들이 붙은 클래스들에 대해 기본 생성자를 만들어줍니다.

- `@Entity`
- `@Embeddable`
- `@MappedSuperclass`

<br>

Spring Initializr 로 Kotlin + Spring Data JPA 프로젝트를 생성하면 아래 플러그인을 자동으로 추가해줍니다.

```kt
// build.gradle.kts
plugins {
    kotlin("plugin.spring") version "1.6.0"
}
```
