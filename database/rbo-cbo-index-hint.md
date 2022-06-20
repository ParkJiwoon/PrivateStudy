# MySQL Optimizer 와 USE INDEX vs FORCE INDEX

# Overview

MySQL 테이블을 설계할 때 보통 자주 사용하는 조건 컬럼에는 Index 를 추가합니다.

다양한 쿼리를 사용하는 경우 인덱스를 여러 개 추가하는데, 인덱스의 컬럼이 겹치면 원하지 않는 인덱스를 타서 Slow Query 가 발생할 수 있습니다.

Slow Query 의 발생가능한 원인과 해결 방법에 대해 간단히 알아봅니다.

<br>

# 1. Optimizer

SQL 쿼리를 호출하면 DBMS 에 존재하는 Optimizer 가 SQL 실행계획을 만들어줍니다.

옵티마이저는 두 가지 종류로 나뉩니다.

- **RBO (Rule Based Optimizer)**
  - 미리 정해진 규칙대로만 쿼리를 수행
  - 인덱스가 존재하면 무조건 인덱스를 탐
  - 예측 가능하기 때문에 설계하기 쉽지만 그만큼 쿼리 자체를 잘 짜야함
- **CBO (Cost Based Optimizer)**
  - 통계 정보 (Record 수, Index 컬럼 값 갯수) 를 기반으로 옵티마이저가 생각하는 가장 효율적인 쿼리를 수행
  - 인덱스가 존재해도 Table Full Scan 이 더 효율적이라고 생각하면 Full Scan 함
  - 예측이 힘들기 때문에 우리가 생각한 실행계획과 달라 Slow Query 가 발생할 수 있음
    - 특히 로컬, 개발 환경과 실제 운영 환경에서는 데이터의 갯수가 달라 같은 쿼리여도 다르게 수행될 수 있음

<br>

최근 대부분의 RDBMS 는 CBO 를 사용한다고 합니다.

예를 들어 우리가 `WHERE ~ AND ~` 조건을 사용할 때 Index 컬럼 순서를 지키지 않았지만 자동으로 인덱스를 태우는 것도 다 CBO 덕분입니다.

하지만 CBO 를 사용하는 경우 우리 생각과는 다르게 동작할 수 있습니다.

여러 인덱스가 존재할 때 A 인덱스를 타는게 더 효율적임에도 B 인덱스를 타는 경우가 있습니다.

이런 경우에 사용하는 것이 바로 [Index Hint](https://dev.mysql.com/doc/refman/8.0/en/index-hints.html) 입니다. (쿼리 최적화의 또다른 방법으로 [Optimizer Hint](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html) 라는 것도 있는데 이번 포스팅에서는 다루지 않습니다)

<br>

# 2. Index Hint

인덱스 힌트는 이름 그대로 옵티마이저가 인덱스를 선택할 때 도움을 줍니다.

특정 인덱스를 사용하지 않길 원하면 `IGNORE`, 사용하길 원하면 `USE`, `FORCE` 를 사용할 수 있습니다.

그렇다면 `USE` 와 `FORCE` 의 차이는 뭘까요?

[Stackoverflow - MySQL : FORCE INDEX vs USE INDEX](https://stackoverflow.com/questions/20797475/mysql-force-index-vs-use-index) 를 참고해보면 둘은 다음과 같은 차이가 있습니다.

- `USE INDEX`
  - DB 옵티마이저에게 지정한 인덱스를 사용하라고 권장
  - 하지만 만약 Table Scan 이 더 빠르다면 옵티마이저는 인덱스 대신 Table Scan 수행 가능
- `FORCE INDEX`
  - Table Sacan 이 더 효율적이어도 무조건 인덱스 사용
  - Index 를 사용할 수 없는 쿼리 (인덱스가 걸려있지 않은 컬럼이 조건) 인 경우에만 다른 방법 선택 가능

<br>

대부분의 경우에는 `USE INDEX` 만 사용해도 되지만 Full Scan 도 허용하지 않는 경우에는 `FORCE INDEX` 를 사용하면 될 것 같습니다.

<br>

# Reference

- [Stackoverflow - MySQL : FORCE INDEX vs USE INDEX](https://stackoverflow.com/questions/20797475/mysql-force-index-vs-use-index)
- [Index Hint](https://dev.mysql.com/doc/refman/8.0/en/index-hints.html)