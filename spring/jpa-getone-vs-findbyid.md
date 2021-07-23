# JPA 의 getById vs findById

# 1. Overview

JPA 를 사용할 때 `ID` 값으로 엔티티를 가져오는 두 가지 메소드가 존재합니다.

비슷하지만 다른 이 두가지 메소드의 차이점에 대해서 알아봅시다.

<br>

## 1.1. getById

```java
// JpaRepository 내부 정의

@Override
public T getById(ID id) {

    Assert.notNull(id, ID_MUST_NOT_BE_NULL);
    return em.getReference(getDomainClass(), id);
}
```

`getById()` 는 이전에는 `getOne()` 이었으나 해당 메소드가 Deprecated 되고 대체되었습니다.

내부적으로 `EntityManager.getReference()` 메소드를 호출하기 때문에 엔티티를 직접 반환하는게 아니라 프록시만 반환합니다.

프록시만 반환하기 때문에 실제로 사용하기 전까지는 DB 에 접근하지 않으며, 만약 나중에 프록시에서 DB 에 접근하려고 할 때 데이터가 없다면 `EntityNotFoundException` 이 발생합니다.

<br>

## 1.2. findById

```java
// CrudRepository 내부 정의

@Override
public Optional<T> findById(ID id) {

    Assert.notNull(id, ID_MUST_NOT_BE_NULL);

    Class<T> domainType = getDomainClass();

    if (metadata == null) {
        return Optional.ofNullable(em.find(domainType, id));
    }

    LockModeType type = metadata.getLockModeType();

    Map<String, Object> hints = new HashMap<>();
    getQueryHints().withFetchGraphs(em).forEach(hints::put);

    return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints));
}
```

우리가 잘 알고 있는 메소드입니다.

실제 DB 에 요청해서 엔티티를 가져옵니다.

정확히 따지자면 영속성 컨텍스트의 1차 캐시를 먼저 확인하고 데이터가 없으면 실제 DB 에 데이터가 있는지 확인합니다.

<br>

# 2. 차이점

실제로 차이가 있는 부분은 성능 부분입니다.

`getById()` 는 해당 엔티티를 사용하기 전까진 DB 에 접근하지 않기 때문에 성능상으로 좀더 유리합니다.

따라서 특정 엔티티의 ID 값만 활용할 일이 있다면 DB 에 접근하지 않고 프록시만 가져와서 사용할 수 있습니다.

<br>

# Reference

- [Difference between getOne and findById in Spring Data JPA?](https://www.javacodemonk.com/difference-between-getone-and-findbyid-in-spring-data-jpa-3a96c3ff)