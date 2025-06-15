# Kotlin 에서 JPA 사용하기 3 (@Id 선언)

# Overview

Kotlin 에서 엔티티를 정의할 때 `@Id` 값을 정의하는 방법에는 여러가지가 있습니다.

Java 에서는 기본적으로 모든 변수가 nullable 하기 때문에 딱히 의견이 갈릴 일이 없었는데요.

Kotlin 에서 많이 사용하는 대표적인 `@Id` 정의 스타일과 장단점을 알아보겠습니다.

<br>

# 1. val + nullable 정의

```kt
@MappedSuperclass
abstract class BaseEntity1 {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null
}
```

아마 Kotlin + JPA 를 사용하면서 가장 많이 보이는 스타일일 것 같습니다.

특징
- 엔티티의 특성상 DB 에 저장되기 전까지는 `null` 값이므로 nullable 타입을 사용

이 스타일의 단점이라고 하면 `id` 가 nullable 이기 때문에 도메인 로직에서 사용할 때는 `id!!` 나 `requireNotNull(id)` 등을 사용해야 할 수 있습니다.

<br>

# 2. id 기본값을 0L 으로 지정

```kt
@MappedSuperclass
abstract class BaseEntity2 {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0L
}
```

이 스타일 역시 Kotlin + JPA 에서 많이 보이는 스타일입니다.

특징
- Notnull 타입으로 정의했기 때문에 매번 `!!` 을 붙이지 않아도 됨 (NPE 방지)
- DDD 관점에서는 도메인 객체가 "불완전한 상태(null ID)"를 가지는 것이 자연스럽지 않기 때문에 이를 해결

가장 큰 특징으로는 `nullable` 을 지양하는 Kotlin 의 철학을 지킬 수 있다는 점입니다.

엔티티의 id 는 DB 에 저장되기 전까지 `null` 인데 어떻게 `0L` 을 사용할 수 있는걸까요? 

<br>

## 2.1. Repository#save

```kt
@Override
@Transactional
public <S extends T> S save(S entity) {

    Assert.notNull(entity, ENTITY_MUST_NOT_BE_NULL);

    if (entityInformation.isNew(entity)) {
        entityManager.persist(entity);
        return entity;
    } else {
        return entityManager.merge(entity);
    }
}
```

JPA 의 Repository `save` 는 `isNew` 라는 함수를 사용해서 엔티티가 새로 생성된건지 기존 데이터인지 검사합니다.

새로 생성된 데이터라면 `persist` 를 진행해서 데이터를 저장, id 에 값을 주입하고 기존 데이터라면 `merge` 를 사용해서 업데이트 합니다.

<br>

## 2.2. isNew

```kt
@Override
public boolean isNew(T entity) {

    ID id = getId(entity);
    Class<ID> idType = getIdType();

    if (!idType.isPrimitive()) {
        return id == null;
    }

    if (id instanceof Number n) {
        return n.longValue() == 0L;
    }

    throw new IllegalArgumentException(String.format("Unsupported primitive id type %s", idType));
}
```

Spring Data JPA 내부의 Repository 구현을 보면 `isNew` 함수에서 `id == null` 뿐만 아니라 `id == 0L` 또한 "새로운 객체" 라고 판단해줍니다.

그래서 `null` 대신 `0L` 을 입력해도 JPA 가 정상적으로 동작하는 겁니다.

하지만 JPA 의 구현체로 Hibernate 가 아닌 다른 걸 사용한다면 0L 을 저장되지 않은 구현체로 보장하지 않기 때문에 문제가 발생할 수 있습니다.

그리고 `id` 를 0L 로 정의한다는 것 자체가 어색하거나 불편하게 느껴질 수도 있습니다.

<br>

# 3. var + nullable 정의

```kt
@MappedSuperclass
abstract class BaseEntity3 {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long? = null
}
```

Java 와 가장 유사하게 동작하는 스타일입니다.

Jetbrain 이나 Spring 에서 제공하는 예제 코드에서 등장하는데 오픈소스 예제에서는 테스트 시 직접 `id` 값을 설정하거나 유연하게 다루기 위해 때문에 사용하는 것 같습니다.

<br>

# 4. 개인적인 사용 방식

저는 위의 스타일들 중에서 마음에 안드는 부분이 하나씩 있었습니다.

- 저장되지 않은 `id` 에 `0L` 을 넣는 것
- nullable 하게 정의하면 호출할 때 `id!!` 를 사용해야 하는 것
- 그렇다고 별도의 `id()` 메서드를 정의하는 것도 마음에 안듬

그래서 Backing Field 를 사용했습니다.

<br>

```kt
@MappedSuperclass
abstract class BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    protected val _id: Long? = null

    val id: Long = _id ?: throw IllegalStateException("엔티티의 ID 값이 존재하지 않습니다.")
}
```

`id` 정의하는데 이렇게까지 해야하나? 하는 생각이 들수도 있지만 `BaseEntity` 에만 정의해두면 다시 건들 일이 없긴 합니다.

- `@Id` 가 사용되는 `_id` 변수는 nullable 이라 저장되지 않았을 때 `null`
- `protected` 로 정의하여 상속 후 프록시객체로 필드 주입 가능
- 외부에서 `id` 호출시에도 `!!` 를 붙이거나 함수처럼 `id()` 형태로 사용하지 않아도 됨

<br>

# Conclusion

그래서 뭘 써야하나?

정답은 없습니다.

팀 컨벤션이나 개인적인 선호도에 따라 1번을 쓰기도 하고 2번을 쓰기도 합니다.

현재 대중화된 Spring Boot + JPA Hibernate 기준으로는 어떤 스타일을 해도 정상 동작합니다.

가장 많이 쓰는 방식은 1번 (`val id: Long? = null`) 으로 알고 있습니다.