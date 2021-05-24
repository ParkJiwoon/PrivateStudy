# Spring @Transactional 옵션

# Overview

Spring 에서 `@Transactional` 을 사용할 시에 지정할 수 있는 옵션들을 알아봅니다.

# 1. propagation



# 2. isolation

# 3. readOnly

- 기본값: false
- 사용법: `@Transactional(readOnly = true)`

기본값은 `false` 이며 `true` 로 세팅하는 경우 트랜잭션을 읽기 전용으로 변경합니다.

만약 읽기 전용 트랜잭션 내에서 INSERT, UPDATE, DELETE 작업을 해도 반영이 되지 않거나 DB 종류에 따라서 아예 예외가 발생하는 경우도 있습니다.

성능 향상을 위해 사용하거나 읽기 외의 다른 동작을 방지하기 위해 사용하기도 합니다.

<br>

## 3.1. JPA 에서 Dirty Checking 무시

JPA 에는 Dirty Checking 이라는 기능이 있습니다.

개발자가 임의로 UPDATE 쿼리를 사용하지 않아도 트랜잭션 커밋 시에 1차 캐시에 저장되어 있는 Entity 와 스냅샷을 비교해서 변경된 부분이 있으면 UPDATE 쿼리를 날려주는 기능입니다.

하지만 `readOnly = true` 옵션을 주면 스프링 프레임워크가 하이버네이트의 FlushMode 를 `MANUAL` 로 설정해서 Dirty Checking 에 필요한 스냅샷 비교 등을 생략하기 때문에 성능이 향상됩니다.

<br>

# 4. rollbackFor

- 기본값: RuntimeException, Error
- 사용법: `@Transactional(rollbackFor = {IOException.class, ClassNotFoundException.class})`

사용할 때 `@Transactional(rollbackFor = IOException.class)` 처럼 Exception 을 하나만 지정한다면 중괄호를 생략할 수 있습니다.

기본적으로 트랜잭션은 종료 시 변경된 데이터를 커밋합니다.

하지만 `@Transactional` 에서 `rollbackFor` 속성을 지정하면 특정 Exception 발생 시 데이터를 커밋하지 않고 롤백하도록 변경할 수 있습니다.

기본값은 `{}` 라고 나와있지만 사실 `RuntimeException` 과 `Error` 가 세팅되어 있습니다.

내부 로직으로 들어가 설명을 보면 둘 다 예측 불가능한 예외 상황이기 때문에 기본값으로 들어가 있다고 합니다.

중요한 점은 이 값은 그냥 기본값이 아니라 아예 지정된 값이기 때문에 `rollbackFor` 속성으로 다른 Exception 을 추가해도 `RuntimeException` 이나 `Error` 는 여전히 데이터를 롤백합니다.

만약 강제로 데이터 롤백을 막고 싶다면 `noRollbackFor` 옵션으로 지정해주면 됩니다.

<br>

# 5. timeout

- 기본값: -1
- 사용법: `@Transactional(timeout = 2)`

지정한 시간 내에 해당 메소드 수행이 완료되이 않은 경우 `JpaSystemException` 을 발생시킵니다.

`JpaSystemException` 은 `RuntimeException` 을 상속받기 때문에 데이터 역시 롤백 처리 됩니다.

초 단위로 지정할 수 있으며 기본값인 -1 인 경우엔 timeout 을 지원하지 않습니다.

지정된 timeout 을 초과하면 다음과 같은 에러 로그를 보여줍니다.

```
org.springframework.orm.jpa.JpaSystemException: transaction timeout expired; nested exception is org.hibernate.TransactionException: transaction timeout expired
```

<br>

## 5.1. noRollbackFor 을 같이 사용한다면?

호기심에 `noRollbackFor = {RuntimeException.class, JpaSystemException.class}` 옵션을 추가하고 타임아웃 테스트를 해보았습니다.

Exception 은 발생하지만 롤백 처리가 됩니다.

<br>

# Reference

- [Github 전체 코드](https://github.com/ParkJiwoon/practice-codes/tree/master/spring-transactional/src/test/java/com/practice/transactional)