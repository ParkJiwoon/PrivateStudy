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

## 3.1. JPA 에서 사용 시

JPA 에는 Dirty Checking 이라는 기능이 있습니다.

개발자가 임의로 UPDATE 쿼리를 사용하지 않아도 트랜잭션 커밋 시에 1차 캐시에 저장되어 있는 Entity 와 스냅샷을 비교해서 변경된 부분이 있으면 UPDATE 쿼리를 날려주는 기능입니다.

하지만 `readOnly = true` 옵션을 주면 스프링 프레임워크가 하이버네이트의 FlushMode 를 `MANUAL` 로 설정해서 Dirty Checking 에 필요한 스냅샷 비교 등을 생략하기 때문에 성능이 향상됩니다.

<br>

# 4. rollbackFor

- 기본값: RuntimeException
- 사용법: `@Transactional(rollbackFor = Exception)`

트랜잭션은 예외가 발생하지 않으면 변경된 데이터를 커밋하지만 특정 Exception 이 발생하면 데이터를 롤백합니다.

기본값으로 `RuntimeException` 만 설정되어 있으나 특정 Exception 발생 시 롤백하도록 지정할 수 있습니다.

<br>

# 5. timeout

지정한 시간 내에 해당 메소드 수행이 완료되이 않은 경우 rollback 수행. -1일 경우 no timeout(Default=-1)

@Transactional(timeout=10)

<br>

# Reference

- [Github 전체 코드](https://github.com/ParkJiwoon/practice-codes/tree/master/spring-transactional/src/test/java/com/practice/transactional)