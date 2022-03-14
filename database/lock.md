# MySQL Lock

# Overview

Database 의 lock 이란 동시성 이슈를 해결하기 위한 솔루션 중 하나입니다.

이름 그대로 내가 데이터를 사용하는 동안 다른 사람은 접근 못하게 잠금 (Lock) 기능을 거는 겁니다.

<br>

# Shared Lock

S-Lock, 읽기 락, 공유 락 이라고도 부릅니다.

S-Lock 끼리는 서로 호환이 가능해서 S-Lock 이 걸려 있는 곳에 S-Lock 을 걸 수 있습니다.

**InnoDB 에서는 SELECT 시 S-Lock 을 걸지 않기** 때문에 X-Lock 이 걸려 있어도 조회 가능합니다.

한 Row 에 여러 Shared Lock 을 걸 수 있기 때문에 **여러 트랜잭션이 동시에 읽을 수 있지만** Exclusive Lock 을 걸 수 없기 때문에 **다른 트랜잭션이 읽고 있는 Row 의 수정, 삭제는 불가능**합니다.

- `SELECT ... FOR SHARE`
  - 읽은 모든 행에 Shared Lock 설정
  - 트랜잭션이 끝날때까지 잠긴 Row 의 값이 변경되지 않음을 보장
  - 다른 트랜잭션이 동시에 조회 가능한 일반적인 Shared Lock
  - MySQL 5.7 버전에서는 `LOCK IN SHARE MODE` 로 사용
- `SELECT ... FOR UPDATE`
  - 트랜잭션이 끝날때까지 잠긴 Row 에 대한 다른 트랜잭션의 `SELECT`, `UPDATE`, `DELETE` 가 모두 잠김
  - `NOWAIT` 옵션 등을 사용해서 락을 바로 획득하지 못하는 경우 예외를 던지도록 할 수 있음

<br>

# Exclusive Lock

X-Lock, 쓰기 락, 베타 락 이라고도 부릅니다.

호환되지 않기 때문에 X-Lock 이 걸려있는 레코드에 S-Lock, X-Lock 을 걸려고 하면 대기 상태가 됩니다.

`UPDATE`, `DELETE` 등 수정 쿼리를 날릴 때 각 Row 에 걸립니다.

Exclusive Lock 이 걸린 Row 는 어떠한 Lock 도 걸수 없기 때문에 다른 트랜잭션이 읽거나 수정하거나 삭제가 불가능합니다.

하지만 위에서 언급했듯이 InnoDB 에서는 조회 시 S-Lock 을 걸지 않기 때문에 조회 가능합니다.

<br>

# Record Lock

Row 가 아닌 Index Record Lock 을 의미합니다.

마찬가지로 Shared Lock, Exclusive Lock 이 존재합니다.

InnoDB 는 명시적으로 인덱스를 생성하지 않아도 PK 값으로 Clustered Index 를 생성하기 때문에 Record Lock 을 사용할 수 있습니다.

<br>

# Gap Lock

Index 의 Gap 에 걸리는 Lock 입니다.

여기서 인덱스의 Gap 이란, 인덱스의 키 사이사이에 존재하는 실제로는 존재하지 않는 부분을 의미합니다.

예를 들어 1, 5 라는 ID 값이 존재한다면 이 사이에 있는 2, 3, 4 는 실제로 데이터가 존재하지 않습니다.

이런 부분을 Gap 이라고 하며 **기존 데이터의 수정을 방지하는 Record Lock 과 달리 새로운 데이터의 추가를 방지**합니다.

<br>

# Reference

- [MySQL Reference - 15.7.1 InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
- [MySQL Reference - 15.7.2.4 InnoDB Locking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)