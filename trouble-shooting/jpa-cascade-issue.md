# JPA 관련 Hibernate 에러: object references an unsaved transient instance - save the transient instance before flushing

# 에러 로그 

```sh
TransientPropertyValueException: object references an unsaved transient instance - save the transient instance before flushing
```

<br>

# 원인

JPA 연관 관계 테스트 중에 발생했습니다.

FK 로 사용되는 컬럼값이 없는 상태에서 데이터를 넣으려다 발생한 에러입니다.

예를 들어 `사람 (id, name)` 이라는 테이블과 `집 (id, address, person_id)` 라는 테이블 관계가 있을 때, 사람 데이터를 넣지 않고 집을 넣으려고 하면 `person_id` 값이 없어서 에러가 발생합니다.

```java
Person person = new Person("Alice");
House house = new House("Seoul", person);

houseRepository.save(house);    // 에러 발생 (person 의 id 값을 모르는데 넣으려고 해서)
```

<br>

# 해결 방법

연관 관계 매핑해줄 때 사용하는 `@ManyToOne`, `@OneToOne`, `@OneToMany` 어노테이션에 `cascade` 옵션을 설정해줍니다.

`cascade` 는 "영속성 전이" 라고 하는 개념인데 특정 엔티티를 영속화 할 때 연관된 엔티티도 함께 영속화 합니다.

저장할 때만 사용하려면 `cascade = CascadeType.PERSIST` 로 설정해주면 되며, 전체 적용인 `CascadeType.ALL` 로 설정해도 됩니다.

그럼 `House` 데이터를 저장하기 전에 `Person` 값부터 저장합니다.

<br>

# Reference

- [Stack Overflow - How to fix the Hibernate “object references an unsaved transient instance - save the transient instance before flushing” error](https://stackoverflow.com/questions/2302802/how-to-fix-the-hibernate-object-references-an-unsaved-transient-instance-save)