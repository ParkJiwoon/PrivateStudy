# Content

- [Spring 이란?](#spring-이란)
- [Property 값 주입](#property-값-주입)
- [Spring JPA](#spring-jpa)

<br>

# Spring 이란?

```html
간단히 정리하면 스프링은 "자바 엔터프라이즈 개발을 편하게 해주는 오픈소스 프레임워크" 이며

코드의 가독성과 의존성 (클래스가 다른 클래스에 종속적임) 을 해결하기 위해 Application Context 라는 컨테이너를 제공한다.

이 컨테이너에서는 Bean 들을 관리하며 각 클래스에서 사용할 수 있도록 Bean 을 생성해주는 것을 DI (의존성 주입) 이라고 한다.

개발자가 아닌 컨테이너가 직접 Bean 을 생성, 관리하기 때문에 IoC (제어의 역전) 컨테이너라고도 한다.
```

<br>

스프링을 한 문장으로 표현하자면 **자바 엔터프라이즈 개발을 편하게 해주는 오픈소스 경량급 애플리케이션 프레임워크**다.

스프링은 **스프링 애플리케이션 컨텍스트** (Spring Application Context) 라는 **컨테이너** (Container) 를 제공하는데, 이것은 애플리케이션 컴포넌트들을 생성하고 관리한다.

애플리케이션 컴포넌트 또는 빈 (Bean) 들이 컨테이너 내부에서 서로 연결되어 완전한 애플리케이션을 만든다.

<br>

## 의존성 주입 (Dependency Injection, DI)

빈의 상호 연결은 의존성 주입 이라고 알려진 패턴을 기반으로 수행된다.

컨테이너에서 모든 애플리케이션 컴포넌트를 생성, 관리하고 해당 컴포넌트를 필요로 하는 빈에 주입 (연결) 한다.

일반적으로 생성자 인자 또는 속성의 접근자 메서드를 통해 처리된다.

지금까지 스프링 버전에서는 XML 파일을 사용해서 빈을 상호 연결하도록 컨테이너에 전달했다.

그러나 최신 버전의 스프링에서는 `@Configuration` 애노테이션을 사용하여 각 빈을 컨테이너에 제공하는 클래스라는 걸 명시한다.

아래 두 코드는 똑같은 설정을 각각 XML 과 애노테이션을 사용한 예시이다.

```xml
<bean id="inventoryService" class="com.example.InventoryService" />
<bean id="productService" class="com.example.ProductService" />
  <constructor-arg ref="inventoryService" />
</bean>
```

```java
@Configuration
public class ServiceConfiguration {
  
  @Bean
  public InventoryService inventoryService() {
    return new InventoryService();
  }

  @Bean
  public ProductService productService() {
    return new ProductService(inventoryService());
  }
}
```

<br>

> ### 애노테이션 (Annotation)
>
> 애노테이션은 클래스, 인터페이스, 함수, 매개변수, 속성, 생성자에 어떤 의미를 추가할 수 있는 기능이며, 자바 컴파일러가 컴파일 시에 처리한다. 소스 코드에 추가된 애노테이션 자체는 바이트 코드로 생성되지 않고 주석으로 처리되지만, 컴파일러가 작업을 수행해준다.

<br>

## 제어의 역전 (Inversion of Control, IoC)

간단히 말하면 **객체에 대한 제어권이 개발자로부터 컨테이너로 넘어간 것**

객체의 생성부터 생명주기 관리까지 전부 컨테이너가 맡아서 하기 때문에 제어를 컨테이너가 갖고 있다.

스프링에서 제공하는 컨테이너를 IoC 컨테이너라고 하기도 한다.

컨테이너가 직접 빈을 생성/관리하기 때문에 개발자는 코드에 `new` 등으로 선언하지 않아도 되며 이는 각 클래스들의 의존도를 줄여준다.

<br>

### IoC 용어 정리

- `bean` : 스프링에서 제어권을 가지고 직접 만들어 관계를 부여하는 오브젝트
    - 스프링을 사용하는 애플리케이션에서 만들어지는 모든 오브젝트가 빈은 아니다. 스프링의 빈은 스프링 컨테이너가 생성하고 관계설정, 사용을 제어해주는 오브젝트를 말한다.
- `bean factory` : 스프링의 IoC를 담당하는 핵심 컨테이너
    - Bean을 등록/생성/조회/반환/관리 한다. 보통 bean factory를 바로 사용하지 않고 이를 확장한 application context를 이용한다. BeanFactory는 bean factory가 구현하는 interface이다. (`getBean()` 등의 메서드가 정의되어 있다)
- `application context` : bean factory를 확장한 IoC 컨테이너
    - Bean의 등록/생성/조회/반환/관리 기능은 bean factory와 같지만, 추가적으로 spring의 각종 부가 서비스를 제공한다. ApplicationContext 는 application context 가 구현해야 하는 interface이며, BeanFactory를 상속한다.
- `configuration metadata` : application context 혹은 bean factory가 IoC를 적용하기 위해 사용하는 메타정보
    - 스프링의 설정정보는 컨테이너에 어떤 기능을 세팅하거나 조정하는 경우에도 사용하지만 주로 bean을 생성/구성하는 용도로 사용한다.
- `container (ioC container)` : IoC 방식으로 bean을 관리한다는 의미에서 bean factory나 application context를 가리킨다.
    - application context는 그 자체로 ApplicationContext 인터페이스를 구현한 오브젝트를 말하기도 하는데, 하나의 애플리케이션에 보통 여러개의 ApplicationContext 객체가 만들어진다. 이를 통칭해서 spring container라고 부를 수 있다.

<br>

## 자동 구성 (Auto Configuration)

**자동 연결** (Autowiring) 과 **컴포넌트 스캔** (Component Scanning) 이라는 스프링 기법을 기반으로 한다.

스프링은 컴포넌트 스캔을 사용하여 애플리케이션의 classpath 에 지정된 컴포넌트를 찾은 후 컨테이너의 빈으로 생성한다.

스프링 부트의 Auto Configuration 라이브러리는 다음 일들을 수행한다.

- 스프링 MVC 를 활성화 하기 위해 컨테이너 (스프링 애플리케이션 컨텍스트) 에 관련된 Bean 들을 구성한다.
- 내장된 Tomcat 서버를 컨테이너에 구성한다.
- 스프링 MVC 뷰를 나타내기 위해 사용하는 템플릿 (JSP, Thymeleaf, Mustache 등) 의 **뷰 리졸버** (View Resolver) 를 구성한다.

<br><br>

# Property 값 주입

- [Spring @Value annotation tricks](https://dev.to/habeebcycle/spring-value-annotation-tricks-1a80)

Spring Boot 프로젝트가 커지면 공통으로 사용되는 글로벌 값을 별도로 관리할 필요가 생긴다.

`@Value` 어노테이션은 properties 파일에 세팅한 내용을 Spring 변수에 주입하는 역할을 한다.

`@Value` 어노테이션의 사용법에 대해 알아보도록 하자.

<br>

## 1. @Value("") 사용

`greetingMessage` 변수에 `"Hello World"` 를 주입해서 사용할 수 있다.

```java
@RestController
public class ValueController {

    @Value("Hello World")
    private String greetingMessage;

    @GetMapping("")
    public String sendGreeting(){
        return greetingMessage;
    }
}
```

<br>

`String` 뿐만 아니라 다른 타입에도 사용할 수 있다.

```java
@Value("1.234")
private double doubleValue; //could be Double

@Value("1234")
private Integer intValue; //could be int

@Value("true")
private boolean boolValue; //could be Boolean

@Value("2000")
private long longValue;
```

<br>

## 2. @Value("${...}") 사용

`application.properties` 에 정의한 내용을 가져와서 사용할 수 있다.

```yaml
# application.properties
greeting.message=Hello World!
```

```java
@RestController
public class ValueController {

    @Value("${greeting.message}") 
    private String greetingMessage;

    @GetMapping("")
    public String sendGreeting(){
        return greetingMessage;
    }
}
```

<br>

속성 값은 런타임에 변수로 주입되며 만약 속성값이 properties 파일에 없으면 아래와 같은 오류가 발생한다.

```java
Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
2020-02-29 21:54:43.953 ERROR 2996 --- [main] o.s.boot.SpringApplication: Application run failed
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'valueController': Injection of autowired dependencies failed; nested exception is java.lang.IllegalArgumentException: Could not resolve placeholder 'greeting.message' in value "${greeting.message}"
```

<br>

오류를 방지하기 위해 `@Value` 어노테이션에는 기본값을 세팅할 수 있다.

`@Value("${greeting.message:Greeting not found!}")` 처럼 콜론을 붙이고 뒤에 기본값을 붙이면 된다.

만약 콜론 앞에 공백이 있으면 에러가 발생한다.

```yaml
# application.properties

my.int.value=20
my.double.value=3.142
my.boolean.value=true
```

```java
@Value("${my.int.value:0}")
private int intValue; // 런타임에 20 주입

@Value("${my.double.value: 0.0}")
private double doubleValue; // 런타임에 3.142 주입

// property 에 값이 있어도 공백 때문에 기본값이 들어감
@Value("${my.boolean.value :false}")
private boolean boolValue; // 공백 때문에 false 주입

// proprety 에 값이 없어서 기본값 사용
@Value("${my.long.value:300}")
private long longValue; // 런타임에 300 주입
```

<br>

## 3. @Value("${...}") 로 List 주입

```yaml
# application.properties
my.weekdays=Mon,Tue,Wed,Thu,Fri
```

```java
@Value("${my.weekdays}")
private List<String> strList; // injects [Mon, Tue, Wed, Thu, Fri]

//Providing default value
@Value("${my.weekends:Sat,Sun,Fri}")
private List<String> strList2; // injects [Sat, Sun, Fri]
```

<br>

## 4. @Value("#{${...}}") 로 Map 주입

```yaml
# application.properties
database.values={url:'http://127.0.0.1:3306/', db:'mySql', username:'root', password:'root'}
```

```java
@RestController
public class ValueController {

    @Value("#{${database.values: {url: 'http://127.0.0.1:3308/', db: 'mySql', username: 'root', password: ''}}}")
    private Map<String, String> dbValues;

    @GetMapping("")
    public Map getDBProps(){
        return dbValues;
    }
}
```

<br>

## 5. @Value("${...}") 생성자 파라미터에 주입

생성자에 파라미터로 넘기면서 값을 주입할 수도 있다.

```yaml
# application.properties

company.name= Scopesuite Pty ltd
# company.location= Sydney
```

```java
@Service
public class CompanyService {
   private String compName;
   private String location;

   public CompanyService(
    @Value("${company.name}") String compName,
    @Value("${company.location:Washington}") String location
   ) {
       this.compName = compName;
       this.location = location;
   }
}
```

<br><br>

# Spring JPA

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

## SQL 중심 개발의 문제점

MyBatis 와 같은 SQL Mapper 는 데이터베이스의 쿼리를 직접 작성하기 때문에 **객체지향 프로그래밍 보다는 데이터베이스 테이블 모델링에 집중**하게 된다.

- 반복적인 SQL 작업

  - RDB 를 사용하면서 개발자들은 객체 지향 관점보다는 SQL 중심으로 코드를 짜게 된다.
  - RDB 는 SQL 만 인식할 수 있기 때문에 반복되는 SQL 의 사용을 피할 수 없었고 수십, 수백 개의 테이블마다 각각 SQL 문을 작성해줘야 했다.

- RDB 와 객체지향 프로그래밍의 목적은 다르다

  - RDB 는 데이터 저장에 초점이 맞추어져 있다.
  - 객체지향 프로그래밍은 기능과 속성을 한 곳에서 관리하는 기술이다.
  - 추상화, 캡슐화, 다형성 등 객체지향의 패러다임을 RDB 로는 표현할 수 없다.

<br>

## JPA (Java Persistence API) 의 등장

RDB 를 사용하는 프로젝트에서 객체지향 프로그래밍을 할 수 있게 하는 자바 표준 ORM (Object Relational Mapping) 기술이 생겼다.

JPA 는 Java 에서 제공하는 기술 명세 인터페이스이다.

JPA 는 RDB 와 객체지향 프로그래밍 두 개의 영역을 연결해주는 역할을 한다.

개발자는 객체 지향적으로 프로그래밍을 하고, JPA 는 RDB 에 맞게 SQL 을 대신 생성해서 실행해준다.

<br>

## ORM 과 SQL Mapper 의 차이

- ORM (Object-Relation Mapping)

  - 객체를 매핑하여 간접적으로 DB 를 다룸
  - 개발자는 SQL 쿼리 대신 메서드로 데이터를 조작하며 ORM 이 SQL 을 자동으로 생성해준다.
  - JPA

- SQL Mapper

  - 쿼리를 매핑하여 SQL 문으로 직접 DB 를 조작한다.
  - MyBatis, jdbcTemplate

<br>

## Spring Data JPA (Repository)

JPA 는 인터페이스이기 때문에 구현체가 필요하다.

대표적으로 Hibernate, Eclipse Link 등이 있다.

Spring 에서는 이 구현체들을 좀 더 쉽게 사용할 수 있도록 추상화시킨 Spring Data JPA 모듈을 사용한다.

개발자가 Repository 인터페이스에 정해진 규칙대로 메소드를 입력하면, Spring 이 알아서 해당 메소드 이름에 적합한 쿼리를 날리는 구현체를 만들어서 Bean으로 등록해준다.

Hibernate 와 Spring Data JPA 를 사용하는 데에는 사실 큰 차이가 없지만 Spring Data JPA 가 권장되는 이유는 크게 두 가지가 있다.

- 구현체 교체가 쉽다

  - Hibernate 외에 다른 구현체로 쉽게 교체가 가능하다.
  - Spring Data JPA 내부에서 구현체 매핑을 지원해주기 때문에 언젠가 Hibernate 외의 다른 구현체로 넘어갈 일이 생겨도 쉽게 교체 가능하다.

- 저장소 교체가 쉽다

  - RDB 외의 다른 DB 로 쉽게 교체 가능하다.
  - Spring Data 의 하위 프로젝트들은 기본적인 CRUD 인터페이스가 같기 때문에 의존성만 교체하면 쉽게 변경이 가능하다.
  - 예를 들어 MongoDB 로 교체가 필요하다면 Spring Data JPA 에서 Spring Data MongoDB 로 의존성만 교체하면 된다.

<br>

## JPA 의 장단점

- 장점
    - CRUD 쿼리를 직접 작성할 필요가 없음
    - 부모-자식 관계 표현, 1:N 관계 표현, 상태와 행위를 한 곳에서 관리 등 객체 지향 개발에 집중 가능
- 단점
    - 높은 러닝커브 (객체지향 프로그래밍과 RDB 를 둘 다 이해해야 함)
    - 제대로 사용하지 못하면 성능 문제가 발생함
