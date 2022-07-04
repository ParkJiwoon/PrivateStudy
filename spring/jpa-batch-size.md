# JPA Batch Size

# 1. Overview

BatchSize 는 JPA 의 성능 개선을 위한 옵션 중 하나입니다.

여러 개의 프록시 객체를 조회할 때 `WHERE` 절이 같은 여러 개의 `SELECT` 쿼리들을 하나의 `IN` 쿼리로 만들어줍니다.

간단한 테스트와 함께 사용법을 알아봅니다.

<br>

# 2. Domain 정의

```java
@Entity
public class Parent {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "parent_id")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent")
    private List<Child> children = new ArrayList<>();
}


@Entity
public class Child {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "child_id")
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Parent parent;
}
```

간단한 `Parent(1) <-> Child(N)` 관계의 도메인을 작성했습니다.

편의상 Getter/Setter 는 생략합니다.

<br>

# 3. BatchSize 정의

```java
@Target({TYPE, METHOD, FIELD})
@Retention(RUNTIME)
public @interface BatchSize {
	int size();
}
```

`@BatchSize` 클래스 파일을 보면 위과 같이 나와있습니다.

Type, Method, Field 에 사용할 수 있으며 `size` 를 설정해야 합니다. (Method 에 설정하는 건 자주 사용하지 않아서 이 포스트에선 제외합니다)

`size` 는 간단히 말해서 `IN` 절에 들어갈 요소의 최대 갯수를 의미합니다.

만약 `IN` 절에 `size` 보다 더 많은 요소가 들어가야 한다면 여러 개의 `IN` 쿼리로 나누어 날립니다.

<br>

## 3.1. Type (Class) 에 정의

```java
@BatchSize(size = 100)
@Entity
public class Parent {
    ...
}
```

Entity 클래스 위에 붙일 수 있습니다.

만약 다른 엔티티에서 여러 개의 `Parent` 객체를 프록시로 호출한다면 배치사이즈가 적용되어 `IN` 쿼리로 조회할 겁니다.

<br>

## 3.2. Field 에 정의

```java
@Entity
public class Parent {

    @BatchSize(size = 100)
    @OneToMany(mappedBy = "parent")
    private List<Child> children = new ArrayList<>();
}
```

`@OneToMany` 를 사용하는 Collections 에 붙이면 여러 `Parent` 객체가 `getChildren()` 호출할 때 하나의 쿼리로 가져옵니다.

<br>

## 3.3. application.yml 에 정의

```yaml
spring:
    properties:
      hibernate:
        default_batch_fetch_size: 100
```

`application.yml` 에 추가하면 프로젝트 전역으로 배치 사이즈를 적용할 수 있습니다.

<br>

# 4. 호출 테스트

```java
List<Parent> parents = parentRepository.findAll();

// 실제로 사용해야 쿼리가 나가기 때문에 size() 까지 호출해줌
parents.get(0).getChildren().size();
parents.get(1).getChildren().size();
```

`Parent`, `Child` 데이터가 이미 존재한다고 가정하고 테스트 코드를 작성했습니다.

`parents` 에서 `for` 문으로 간단하게 작성해도 되지만 명시적으로 두번 호출해봅니다.

<br>

## 4.1. Before

```sql
SELECT * FROM parent

SELECT * FROM child WHERE child.parent_id = 1
SELECT * FROM child WHERE child.parent_id = 2
```

배치 사이즈를 적용하지 않으면 `child` 테이블을 조회하기 위해 두 개의 쿼리가 날아갑니다.

만약 `parents` 의 갯수가 더 많다면 갯수만큼 쿼리가 날아갈겁니다.

<br>

## 4.2. After

```sql
SELECT * FROM parent

SELECT * FROM child WHERE child.parent IN (1, 2)
```

배치 사이즈를 추가하면 여러 쿼리를 하나의 `IN` 쿼리로 만들어줍니다.

`IN` 절에 들어가는 요소의 갯수는 설정 가능합니다.

만약 조건 갯수보다 설정한 배치사이즈 크기가 더 작다면 `IN` 쿼리가 추가로 날아갑니다.

예를 들어 `size` 를 100 으로 설정했기 때문에 데이터가 250 개라면 1 ~ 100, 101 ~ 200, 201 ~ 250 이렇게 세 번에 나누어서 `IN` 쿼리를 날립니다.

(사실 완전히 똑같은 사이즈로 분배해서 날리지는 않고 내부적으로 최적화한 사이즈로 나누어서 날립니다)

<br>

# 5. Conclusion

BatchSize 옵션을 사용하면 비슷한 조회 쿼리 데이터들을 한번에 가져올 수 있어 성능적으로 효과를 볼 수 있습니다.