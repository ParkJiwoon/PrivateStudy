```html
요약

1. ORM (Object-Relational Mapping) 이란?
  - 객체 지향 프로그래밍과 관계형 데이터베이스 사이의 구조적 문제를 해결해주는 프레임워크
  - 개발자가 객체 지향 프로그래밍에 집중할 수 있게 해준다.

2. JPA (Java Persistence API) 란?
  - 자바 ORM 기술에 대한 API 표준 명세로, Java 에서 제공한다 (Spring 아님)
  - JPA 의 구현체로 Hibernate 가 존재한다.

3. Spring Data JPA 란?
  - Hibernate 와 같은 JPA 구현체를 좀 더 쉽게 사용할 수 있도록 Spring 에서 제공하는 모듈
```

<br>

# SQL 중심 개발의 문제점

MyBatis 와 같은 SQL Mapper 는 데이터베이스의 쿼리를 직접 작성하기 때문에 **객체지향 프로그래밍 보다는 데이터베이스 테이블 모델링에 집중**하게 된다.

### 1. 반복적인 SQL 작업

- RDB 를 사용하면서 개발자들은 객체 지향 관점보다는 SQL 중심으로 코드를 짜게 된다.
- RDB 는 SQL 만 인식할 수 있기 때문에 반복되는 SQL 의 사용을 피할 수 없었고 수십, 수백 개의 테이블마다 각각 SQL 문을 작성해줘야 했다.

### 2. RDB 와 객체지향 프로그래밍의 목적은 다르다

- RDB 는 데이터 저장에 초점이 맞추어져 있다.
- 객체지향 프로그래밍은 기능과 속성을 한 곳에서 관리하는 기술이다.
- 추상화, 캡슐화, 다형성 등 객체지향의 패러다임을 RDB 로는 표현할 수 없다.

<br>

# JPA (Java Persistence API) 의 등장

RDB 를 사용하는 프로젝트에서 객체지향 프로그래밍을 할 수 있게 하는 자바 표준 ORM (Object Relational Mapping) 기술이 생겼다.

JPA 는 Java 에서 제공하는 기술 명세 인터페이스이다.

JPA 는 RDB 와 객체지향 프로그래밍 두 개의 영역을 연결해주는 역할을 한다.

개발자는 객체 지향적으로 프로그래밍을 하고, JPA 는 RDB 에 맞게 SQL 을 대신 생성해서 실행해준다.

<br>

# ORM 과 SQL Mapper 의 차이

### ORM (Object-Relation Mapping)

- 객체를 매핑하여 간접적으로 DB 를 다룸
- 개발자는 SQL 쿼리 대신 메서드로 데이터를 조작하며 ORM 이 SQL 을 자동으로 생성해준다.
- JPA

### SQL Mapper

- 쿼리를 매핑하여 SQL 문으로 직접 DB 를 조작한다.
- MyBatis, jdbcTemplate

<br>

# Spring Data JPA (Repository)

JPA 는 인터페이스이기 때문에 구현체가 필요하다.

대표적으로 Hibernate, Eclipse Link 등이 있다.

Spring 에서는 이 구현체들을 좀 더 쉽게 사용할 수 있도록 추상화시킨 Spring Data JPA 모듈을 사용한다.

개발자가 Repository 인터페이스에 정해진 규칙대로 메소드를 입력하면, Spring 이 알아서 해당 메소드 이름에 적합한 쿼리를 날리는 구현체를 만들어서 Bean으로 등록해준다.

Hibernate 와 Spring Data JPA 를 사용하는 데에는 사실 큰 차이가 없지만 Spring Data JPA 가 권장되는 이유는 크게 두 가지가 있다.

### 1. 구현체 교체가 쉽다

- Hibernate 외에 다른 구현체로 쉽게 교체가 가능하다.
- Spring Data JPA 내부에서 구현체 매핑을 지원해주기 때문에 언젠가 Hibernate 외의 다른 구현체로 넘어갈 일이 생겨도 쉽게 교체 가능하다.

### 2. 저장소 교체가 쉽다

- RDB 외의 다른 DB 로 쉽게 교체 가능하다.
- Spring Data 의 하위 프로젝트들은 기본적인 CRUD 인터페이스가 같기 때문에 의존성만 교체하면 쉽게 변경이 가능하다.
- 예를 들어 MongoDB 로 교체가 필요하다면 Spring Data JPA 에서 Spring Data MongoDB 로 의존성만 교체하면 된다.

<br>

# JPA 의 장단점

- 장점
    - CRUD 쿼리를 직접 작성할 필요가 없음
    - 부모-자식 관계 표현, 1:N 관계 표현, 상태와 행위를 한 곳에서 관리 등 객체 지향 개발에 집중 가능
- 단점
    - 높은 러닝커브 (객체지향 프로그래밍과 RDB 를 둘 다 이해해야 함)
    - 제대로 사용하지 못하면 성능 문제가 발생함
