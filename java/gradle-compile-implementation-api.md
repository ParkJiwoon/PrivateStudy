# [Gradle] compile, implementation, api 차이점

# Overview

```groovy
compile 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.springframework.boot:spring-boot-starter-web'

testCompile 'org.springframework.boot:spring-boot-starter-test'
testImplementation 'org.springframework.boot:spring-boot-starter-test'
```

`build.gradle` 에 관한 설정을 검색하다보면 의존 라이브러리를 추가할 때 두가지 방법을 자주 봅니다.

`compile` 과 `implementation` 은 무슨 차이가 있을까요?

<br>

## 1. compile 은 상위 모듈까지 가져온다

`compile` 은 `implementation` 보다 더 많은 라이브러리를 빌드합니다.

예를 들어 다음과 같이 의존하는 관계의 프로젝트 세 개가 있다고 가정합니다.

```text
myApp -> mySpring -> myJava
```

<br>

myApp 에서 mySpring 을 의존하고 mySpring 은 myJava 를 의존합니다.

이 때 `compile` 을 사용해서 mySpring 을 빌드하게 되면 mySpring 이 의존하고 있는 myJava 까지 함께 빌드합니다.

그래서 myApp 에서 myJava 에서 제공하는 API 까지 사용할 수 있습니다.

만약 myJava 를 직접적으로 사용할 필요가 없다면 쓸데없는 API 들이 노출되고 빌드 시간도 오래 걸리게 하는 결과를 만듭니다.

대신 `implementation` 을 사용해서 빌드하면 mySpring 만 빌드하기 때문에 빌드 속도가 빠르고 필요한 API 만 노출해서 사용할 수 있습니다.

<br>

## 2. compile 은 deprecated 되었다

그리고 `compile` 은 deprecated 되고 `api` 로 대체되었습니다.

그러니 만약 상위 모듈까지 전부 가져오고 싶을 땐 `compile` 대신 `api` 를 사용하면 됩니다.

일반적인 경우에는 `implementation` 을 사용해서 빌드 속도를 향상시키는 것이 좋습니다.

<br>

# Reference

- [What's the difference between implementation, api and compile in Gradle? - StackOverflow](https://stackoverflow.com/questions/44493378/whats-the-difference-between-implementation-api-and-compile-in-gradle)