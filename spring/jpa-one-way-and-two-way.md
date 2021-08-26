# JPA 연관 관계 (일대일, 다대일, 단방향, 양방향)

# 1. Overview

JPA 의 Entity 는 그 자체로 테이블을 나타내기 때문에 어떻게 설계하는지가 중요합니다.

데이터는 테이블에 저장되어 있지만 그걸 사용하는 코드는 객체입니다.

객체와 테이블의 차이점을 알아야 제대로 된 엔티티 설계를 할 수 있습니다.

데이터베이스는 외래키를 이용해서 서로 다른 테이블끼리 상호 작용을 하는데, JPA 에서는 연관 관계라는 걸 이용합니다.

JPA 는 엔티티 사이의 연관 관계를 위해 `@OneToOne`, `@OneToMany`, `@ManyToOne` 라는 어노테이션을 제공합니다.

<br>

# 2. 연관 관계란?

JPA 에서 연관 관계를 매핑할 때 고려해야 할 점은 크게 두 가지가 있습니다.

- 단뱡향, 양방향
- 연관 관계의 주인

<br>

## 2.1. 단방향, 양방향

데이터베이스 관점에서는 Join 을 통해 여러 테이블을 한꺼번에 조회 가능하기 때문에 방향이라는 개념이 없습니다.

그러나 엔티티 (객체) 가 다른 연관된 엔티티를 조회하려면 필드값으로 참조해야 합니다.

예를 들어 `Member (1) : Car (N)` 관계의 테이블이 있다고 가정합니다.

`Member` 의 자동차 목록을 가져오기 위해선 `List<Car> cars` 필드값이 필요합니다.

이를 `Member -> Car` 단방향 참조라고 합니다.

반대로 `Car` 엔티티만 존재할 때, 이 자동차의 소유자를 알고 싶다면 `Member member` 필드값이 필요합니다.

이것 역시 `Car -> Member` 단방향 참조입니다.

이렇게 `Member <-> Car` 처럼 **각 엔티티가 서로를 단방향으로 참조하고 있는 걸 양방향 참조** 라고 합니다.

성능상 문제도 없는데 전부 양방향으로 참조 하면 되지 않을까? 하는 생각이 들 수도 있습니다.

하지만 양방향 참조를 한다는건 엔티티의 필드 갯수가 그만큼 늘어난다는 뜻입니다.

특히 사용자를 나타내는 User, Member, Account 등의 엔티티가 모든 엔티티에 대해 참조를 해버리면 필드 갯수가 어마어마하게 많은 복잡한 클래스가 될 겁니다.

그렇기 때문에 엔티티 설계를 할 때 **참조가 꼭 필요하지 않은 상황이라면 단방향 참조만 하는 것을 추천**합니다.

<br>

## 2.2. 연관 관계의 주인

두 엔티티가 양방향 관계일 때는 연관 관계의 주인를 정해야 합니다.

연관 관계의 주인이란 외래키를 관리하며 외래키 저장, 수정, 삭제의 권한을 갖는 실질적인 엔티티이고, 주인이 아닌 엔티티는 조회만 가능합니다.

주인이 아닌 엔티티에서 `mappedBy` 속성을 사용해서 주인 엔티티 필드에 붙이면 됩니다.

연관 관계의 주인은 `mappedBy` 속성을 사용하지 않습니다.

예를 들어 다대일에서 `@ManyToOne` 을 사용하는 다(N) 쪽은 항상 연관 관계의 주인이기 때문에 `mappedBy` 옵션을 지원하지 않습니다.

일반적으로 외래키가 있는 곳을 연관 관계의 주인로 많이 설정합니다. (다대일에서 다 쪽)

<br>

# 3. 일대일 (1:1)

요구사항
- 도메인: 사람(Person), 회사(Compnay), 집(House)
- 회사는 소유자가 존재한다 (Company -> Person)
- 집은 주인이 존재한다 (House -> Person)
- 사람은 집 정보를 갖고 있다 (Person -> House)

우선 사람이라는 공통 Domain Entity 가 존재합니다.

```java
@Entity
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "person_id")
    private Long id;

    @Column(name = "name")
    private String name;

    @OneToOne(mappedBy = "person")
    private House house;
}
```

<br>

## 3.1. 일대일 (1:1) 단방향

```java
@Entity
public class Company {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "company_id")
    private Long id;

    @Column(name = "name")
    private String name;

    @OneToOne(cascade = CascadeType.PERSIST)
    @JoinColumn(name = "person_id")
    private Person person;
}
```

- `Company(1) -> Person(1)`

두 테이블은 일대일 관계이기 때문에 어느 쪽에서 외래키를 관리할 지만 정하면 됩니다.

어느 쪽에서 참조하냐에 따라 외래키 위치가 정해지기 때문에 설계할 때 신중하게 정하는게 좋습니다.

예를 들어, 사람은 회사를 소유하지 않을 수도 있지만 주인 없는 회사는 일반적으로 없습니다.

**객체 지향 관점**에서 보면 사람이 회사를 소유하고 있기 때문에 `Person` 이 `Company` 필드를 갖는게 자연스럽습니다.

하지만 그렇게 되면 `person` 테이블이 `company_id` 라는 외래키 컬럼을 갖게 되는데, 회사를 소유하지 않은 사람은 해당 컬럼값이 `null` 로 세팅됩니다.

`null` 값을 갖는 건 **데이터베이스 관점**에서는 바람직하지 않기 때문에 `company` 테이블에 `person_id` 컬럼을 만들면 `null` 데이터는 사라집니다.

이처럼 객체 지향 관점이냐 데이터베이스 관점이냐에 따라 외래키의 위치가 달라집니다.

또한 나중에 회사를 여러 사람이 공동 소유하거나, 한 사람이 소유할 수 있는 회사가 많아지면서 **일대일 관계가 다대일 관계로 확장될 수 있습니다.**

어느 쪽이 옳다고 정해진 것은 없으므로 여러 가지 상황과 확장성을 고려해서 테이블 및 엔티티를 설계하는게 좋습니다.

<br>

## 3.2. 일대일 (1:1) 양방향

```java
@Entity
public class House {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "house_id")
    private Long id;

    @Column(name = "address")
    private String address;

    @OneToOne(cascade = CascadeType.PERSIST)
    @JoinColumn(name = "person_id")
    private Person person;
}
```

- `House(1) <-> Person(1)`

양쪽에 `@OneToOne` 으로 모두 참조값을 넣어주고 `mappedBy` 로 연관 관계의 주인을 설정해주면 됩니다.

두 엔티티가 서로를 참조할 수 있기 때문에 객체지향 관점에서는 걱정할 것이 없고 데이터베이스 관점으로 정하거나 추후 확장성을 고려해서 연관 관계의 주인을 설정해주는게 좋습니다.

여기서는 `Person` 객체에서 `mappedBy` 가 설정되어 있고 `House` 는 설정하지 않았기 때문에 `House` 가 연관 관계의 주인이 되어 외래키를 관리합니다.



<br>

# 4. 다대일 (N:1)

요구사항
- 도메인: 학교(School), 학생(Student), 선생(Teacher)
- 한 학교에 여러 학생이 다닐 수 있다 (Student -> School)
- 한 학교에 여러 선생이 근무할 수 있다 (Teacher -> School)
- 학교는 선생님들의 정보를 갖고 있다 (School -> Teacher)

우선 학교라는 공통 Domain Entity 가 존재합니다.

```java
@Entity
public class School {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "school_id")
    private Long id;

    @OneToMany(mappedBy = "school")
    private List<Teacher> teachers = new ArrayList<>();
}
```

<br>

## 4.1. 다대일 (N:1) 단방향

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "student_id")
    private Long id;

    @Column(name = "name")
    private String name;

    @ManyToOne(cascade = CascadeType.PERSIST)
    @JoinColumn(name = "school_id")
    private School school;
}
```

- `Student(N) -> School(1)`

다대일은 다(N) 쪽에 외래키가 있는 게 일반적입니다.

`@ManyToOne` 을 사용해서 `School` 필드값을 참조하면 되며, `School` 엔 별다른 설정을 하지 않아도 됩니다.

코드상으로는 학생은 학교의 정보가 있지만 학교 입장에서는 학생들의 정보를 직접적으로 알 수 없습니다.

<br>

## 4.2. 다대일 (N:1) 양방향

```java
@Entity
public class Teacher {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "teacher_id")
    private Long id;

    @Column(name = "name")
    private String name;

    @ManyToOne(cascade = CascadeType.PERSIST)
    @JoinColumn(name = "school_id")
    private School school;
}
```

- `Teacher(N) <-> School(1)`

단방향과 동일하지만 `School` 도메인에 `@OneToMany` 적용이 필요합니다.

다대일이기 때문에 참조하는 필드는 `List` 로 설정합니다.

학교는 `teachers` 를 통해 선생들의 정보를 알 수 있습니다.

<br>

# 5. 다대일 양방향 관계 데이터 저장

다대일 양방향 관계에서 데이터를 저장할 때는 `Teacher.setSchool()` 메소드와 `School.getTeachers().add()` 메소드를 모두 호출해서 객체끼리 동기화를 해주는 것이 중요합니다.

특히 주의할 점은 **연관 관계의 주인이 데이터를 세팅해야 외래키에 값이 제대로 들어간다는 사실**입니다.

<br>

## 5.1. 연관 관계의 주인이 아닌 일(1) 쪽에서만 세팅

```java
School school = new School();
Teacher teacher = new Teacher();

school.getTeachers().add(teacher);  // 주인이 아닌 엔티티가 세팅

schoolRepository.save(school)
teacherRepository.save(teacher);
```

- 위 코드처럼 List 에만 데이터를 넣고 저장하면 외래키가 세팅되지 않습니다.
- `school`, `teacher` 각각 데이터는 들어가지만 `teacher.school_id` 값이 `null` 이 됩니다.

<br>

## 5.2. 연관 관계의 주인인 다(N) 쪽에서만 세팅

```java
School school = new School();
Teacher teacher = new Teacher();

teacher.setSchool(school);  // 주인인 엔티티가 세팅

schoolRepository.save(school)
teacherRepository.save(teacher);
```

- 위 코드를 실행하면 외래키까지 데이터가 제대로 저장됩니다.
- 연관 관계의 주인이 외래키의 저장, 수정, 삭제를 담당하기 때문입니다.
- 다만, 순수한 객체끼리의 데이터 동기화를 위해 `school.getTeachers().add(teacher)` 도 호출하는 것이 좋습니다.
  - 만약 List 에 데이터를 넣지 않으면 `school.getTeachers()` 를 호출해도 리스트가 비어 있습니다.
  - 물론 다른 트랜잭션에서 호출하면 DB 를 조회해서 가져오긴 하지만 같은 트랜잭션 내에서 추가로 작업을 한다면 혼란이 생길 수 있습니다.

<br>

# Reference

- [JPA 연관관계 매핑](https://ultrakain.gitbooks.io/jpa/content/chapter5/chapter5.html)
