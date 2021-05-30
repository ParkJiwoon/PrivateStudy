# Spring @Transactional 옵션

# Overview

Spring 에서 `@Transactional` 을 사용할 때 지정할 수 있는 옵션들을 알아봅니다.

- isolation
- propagation
- readOnly
- rollbackFor
- timeout

<br>

# 1. isolation

데이터베이스 트랜잭션이 안전하게 수행된다는 것을 보장하기 위한 성질 네가지가 존재합니다. (ACID)

- 원자성(Atomicity): 한 트랜잭션 내에서 실행한 작업들은 하나로 간주함 (모두 성공 또는 모두 실패)
- 일관성(Consistency): 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 함
- 격리성(Isolation): 동시에 실행되는 트랜잭션들이 서로 영향을 미치지 않아야 함
- 지속성(Durability): 트랜잭션을 성공적으로 마치면 결과가 항상 저장되어야 함

<br>

`@Transactional` 의 `isolation` 은 **동시에 여러 사용자가 데이터에 접근할 때 어디까지 허용할까?** 를 정하는 옵션이라고 생각하면 됩니다.

트랜잭의 격리 수준 (Isolation) 과 데이터의 일관성 (Consistency) 는 비례합니다.

격리 수준이 약할수록 데이터 접근 및 수정이 자유롭지만 일관성이 떨어지고 격리 수준이 강해진다면 데이터의 일관성이 증가합니다.

Spring 의 `@Transactional` 에서는 총 5가지 `isolation` 옵션을 제공합니다.

<br>

## 1.1. DEFAULT

사용하는 DB 의 기본 격리 수준을 따름

<br>

## 1.2. READ_UNCOMMITTED

한 트랜잭션이 처리 중인 **커밋되지 않은 데이터를 다른 트랜잭션에서 접근 가능**합니다.

DB 에 커밋하지 않은, 즉 존재하지 않는 데이터를 읽는 현상을 Dirty Read 라고 합니다.

데이터 정합성에 문제가 많아서 웬만하면 권장되지 않고 아예 지원하지 않는 경우도 있습니다.

**Dirty Read** 가 가능하기 때문에 잘못된 데이터를 읽을 수 있습니다.

- A 트랜잭션이 데이터 1 을 조회하여 2 로 변경하고 아직 커밋하지 않음
- B 트랜잭션이 동일한 데이터를 조회해서 2 라는 값을 받음 (Dirty Read)
- A 트랜잭션에서 오류가 발생해서 데이터를 롤백 (2 -> 1)
- 실제 데이터는 1 이지만 B 트랜잭션은 2 라는 잘못된 데이터를 읽은 셈

<br>

## 1.3. READ_COMMITTED

**트랜잭션은 커밋한 데이터만 읽을 수 있습니다.**

A 트랜잭션이 데이터를 변경해도 커밋하기 전이라면 B 트랜잭션은 변경되기 전의 데이터를 조회할 수 있습니다.

이 때, B 트랜잭션은 Undo 영역에서 데이터를 가져옵니다. (MVCC - Multi Version Concurrency Control 참조)

매 조회 시마다 새로운 스냅샷을 뜨기 때문에 다른 트랜잭션이 커밋한 후 다시 조회하면 변경된 데이터를 볼 수 있습니다.

대부분의 DB 기본 격리 수준이며 `REPEATABLE_READ` 와 함께 가장 많이 사용되는 방식입니다.

**Non-Repeatable Read** 현상이 발생할 수 있습니다.

트랜잭션에서 조회한 데이터가 트랜잭션이 끝나기 전에 다른 트랜잭션에 의해 변경되면 다시 읽었을 때 새로운 값이 읽히며 데이터 불일치하는 현상을 말합니다.

하나의 트랜잭션 내에서 똑같은 `SELECT` 쿼리를 실행했을 때 항상 같은 결과를 가져와야 한다는 `REPEATABLE READ` 정합성 정의에 어긋납니다.

- A 트랜잭션이 데이터 (row) 를 읽음
- B 트랜잭션이 같은 데이터를 수정하고 커밋
- A 트랜잭션이 다시 같은 데이터를 읽었는데 데이터가 달라짐

<br>

## 1.4. REPEATABLE_READ

간단히 말하면 **하나의 트랜잭션은 하나의 스냅샷만 사용**하는 겁니다.

A 트랜잭션이 시작하고 처음 조회한 데이터의 스냅샷을 저장하고 이후에 동일한 쿼리를 호출하면 스냅샷에서 데이터를 가져옵니다.

따라서 중간에 B 트랜잭션이 새로 커밋해도 A 트랜잭션이 조회하는 데이터는 변하지 않습니다.

**Phantom Read** 라는 다른 트랜잭션에서 수행한 작업에 의해 안보였던 데이터가 보이는 현상이 발생할 수 있습니다.

`REPEATABLE_READ` 격리 수준은 조회한 데이터에 대해서만 Shared Lock 이 걸리기 때문에 다른 트랜잭션이 새로운 데이터를 추가할 수 있습니다.

- A 트랜잭션이 조회한 데이터는 0 건
- B 트랜잭션이 새로운 데이터를 추가하고 커밋
- A 트랜잭션이 같은 쿼리로 다시 조회했더니 B 트랜잭션이 추가한 데이터까지 같이 조회됨
  
<br>

## 1.5. SERIALIZABLE

가장 단순하고 엄격한 격리 수준입니다.

이름 그대로 순차적으로 트랜잭션을 진행시키며 읽기 작업에도 잠금을 걸어 여러 트랜잭션이 동시에 같은 데이터에 접근하지 못합니다.

가장 안전하지만 성능 저하가 발생하기 때문에 극도의 안정성을 필요로 하지 않으면 자주 사용되지 않습니다.

<br>

# 2. propagation

현재 진행중인 트랜잭션 (부모 트랜잭션) 이 존재할 때 새로운 트랜잭션 메소드를 호출하는 경우 어떤 정책을 사용할 지에 대한 정의입니다.

예를 들어, 기존 트랜잭션에 참여해서 그대로 이어갈 수도 있고, 새로운 트랜잭션을 생성할 수도 있으며 non-transactional 상태로 실행할 수도 있습니다.

처음에 non-transactional 상태로 실행한다라는 개념에 대해 착각을 했었는데 **트랜잭션은 존재하지만 커밋, 롤백이 되지 않는 상태**입니다. 

그래서 `NOT_SUPPORTED` 같은 트랜잭션은 `TransactionSynchronizationManager.getCurrentTransactionName()` 메소드로 조회했을 때 이름이 존재하지만 JPA Dirty Checking 은 동작하지 않습니다.

<br>

Spring 의 `@Transactional` 에서는 다음과 같은 `propagation` 옵션을 제공합니다.

- `REQUIRED`: 기본값이며 부모 트랜잭션이 존재할 경우 참여하고 없는 경우 새 트랜잭션을 시작
- `SUPPORTS`: 부모 트랜잭션이 존재할 경우 참여하고 없는 경우 non-transactional 상태로 실행
- `MANDATORY`: 부모 트랜잭션이 있으면 참여하고 없으면 예외 발생
- `REQUIRES_NEW`: 부모 트랜잭션을 무시하고 무조건 새로운 트랜잭션이 생성
- `NOT_SUPPORTED`: non-transactional 상태로 실행하며 부모 트랜잭션이 존재하는 경우 일시 정지시킴
- `NEVER`: non-transactional 상태로 실행하며 부모 트랜잭션이 존재하는 경우 예외 발생
- `NESTED`:
  - 부모 트랜잭션과는 별개의 중첩된 트랜잭션을 만듬
  - 부모 트랜잭션의 커밋과 롤백에는 영향을 받지만 자신의 커밋과 롤백은 부모 트랜잭션에게 영향을 주지 않음
  - 부모 트랜잭션이 없는 경우 새로운 트랜잭션을 만듬 (`REQUIRED` 와 동일)
  - DB 가 SAVEPOINT 를 지원해야 사용 가능 (Oracle)
  - `JpaTransactionManager` 에서는 지원하지 않음

<br>

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

## 5.1. timeout 과 noRollbackFor 옵션을 같이 사용한다면?

호기심에 `noRollbackFor = {RuntimeException.class, JpaSystemException.class}` 옵션을 추가하고 타임아웃 테스트를 해보았습니다.

Exception 은 발생하지만 롤백 처리가 됩니다.

<br>

# Reference

- [Github 전체 코드](https://github.com/ParkJiwoon/practice-codes/tree/master/spring-transactional/src/test/java/com/practice/transactional)