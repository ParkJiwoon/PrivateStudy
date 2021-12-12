# 영속성 컨텍스트 (Persistence Context)

# 1. 영속성 컨텍스트

- JPA 를 이해하는데 가장 중요한 용어
- 영속성 컨텍스트는 논리적인 개념이고 눈에 보이지 않음
- 엔티티 매니저 (EntityManager) 를 통해서 영속성 컨텍스트에 접근

엔티티 매니저 안에 영속성 컨텍스트가 존재

객체와 DB 사이의 중간 단계라고 생각하면 됨

<br>

# 2. 엔티티의 생명 주기

- 비영속 (new/transient): 객체가 생성되어 있지만 영속성 컨텍스트에 속하지 않는 상태
- 영속 (managed): 객체가 영속성 컨텍스트에 관리되는 상태
- 준영속 (detached): 영속화 되었다가 비영속화 된 상태
- 삭제 (removed)

<br>

## 2.1. 영속/비영속 예제

```java
// 비영속 (객체만 생성)
Member member = new Member()
member.setId(100L);
member.setName("HelloJPA");

// 영속
em.persist(member);
```

<br>

# 3. 영속성 컨텍스트의 장점

- 1차 캐시
- 동일성 (identity) 보장
- 트랜잭션을 지원하는 쓰끼 지연 (transactional write-behind)
- 변경 감지 (Drity Checking)
- 지연 로딩 (Lazy Loading)

<br>

## 3.1. 1차 캐시

영속성 컨텍스트 내부에는 1차 캐시라는게 존재합니다.

객체를 영속화하면 1차 캐시에 저장디ㅗ고 나중에 해당 객체를 조회할 때 1차 캐시를 먼저 확인합니다.

식별값은 `@Id` 이고 1차 캐시에 없는 경우에만 DB 를 조회하기 때문에 같은 객체를 조회할 때는 같은 쿼리를 여러 번 날릴 필요가 없습니다.

다만, 영속성 컨텍스트는 API 요청마다 생성되고 종료되기 때문에 완전 복잡한 비즈니스 로직을 갖는게 아닌 이상 성능 상의 큰 이점은 없습니다.

<br>

## 3.2. 영속 엔티티의 동일성 보장

1차 캐시로 **반복 가능한 읽기 (Repetable Read)** 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공합니다.

영속성 컨텍스트 내에서 같은 식별값을 가진 엔티티는 항상 동일한 객체를 반환합니다.

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

assertEquals(a == b);
```

<br>

## 3.3. 엔티티 등록 - 트랜잭션을 지원하는 쓰기 지연

엔티티 매니저는 객체를 영속화한다고 바로 쿼리를 날리는게 아니라 트랜잭션이 종료되는 순간 한번에 날립니다.

사용자가 작성한 쿼리는 **쓰기 지연 SQL 저장소**에 쌓여있다가 `transaction.commit()` 이 호출되는 순간 호출합니다.

```java
// 새로운 객체 생성
Memeber member1 = new Member(150L, "A");
Memeber member2 = new Member(160L, "B");

// 영속화
em.persist(member1);
em.persist(member2);

// 트랜잭션 커밋 (쓰기 지연 저장소 쿼리 호출)
tx.commit();
```

<br>

가장 큰 장점은 **버퍼링**이 가능하다는 점입니다.

예를 들면 Batch Size 라는 옵션이 있는데 비슷한 쿼리를 모아서 조회할 때 `IN` 쿼리로 호출하는 등 성능적인 이점을 제공할 수 있습니다.

<br>

## 3.4. 엔티티 수정 - 변경 감지 (Dirty Checking)

JPA 를 통해 조회한 객체는 필드값을 변경 후 **다시 영속화하지 않아도 자동으로 update** 쿼리가 호출됩니다.

JPA 는 트랜잭션 커밋 직전에 flush 를 호출합니다.

flush 이후에 엔티티에 저장되어 있는 스냅샷과 엔티티의 객체를 비교하여 변경된 부분에 대해 자동으로 쿼리가 발생합니다.

그래도 명시적으로 영속화하는게 낫지 않을까?? 하지 않는게 낫습니다. 됩니다.

<br>

# 4. 객체와 테이블 매핑

- `@Entity`: 객체를 엔티티로 지정
- `@Table`: 테이블명을 임의로 지정할 수 있음

<BR>

# 5. 데이터베이스 스키마 자동 생성

- DDL 을 애플리케이션 실행 시점에 자동 생성
- (장점) 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 을 생성해줌
- 옵션
  - create: 기존 테이블 삭제 후 생성 (drop -> create)
  - create-drop: create 와 동일하나 애플리케이션 종료 시점에 drop
  - update: 변경분만 반영 (운영 DB 사용 x)
  - validate: 엔티티와 테이블이 정상 매핑되었는지 확인해줌
  - none: 사용하지 않음
- **(주의) 운영 장비에서는 절대 create, create-drop, update 를 사용하면 안됨**
  - 개발 초기 단계는 create 또는 update
  - 테스트 서버 또는 운영 서버는 웬만하면 하지 않거나 적용해도 validate 정도 하는걸 추천