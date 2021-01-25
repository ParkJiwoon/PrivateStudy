# Content

- [Spring 이란?](#spring-이란)
- [Lombok 이란?](#lombok-이란)
- [Property 값 주입](#property-값-주입)
- [Spring JPA](#spring-jpa)
- [@Autowired 와 @Qualifier](#autowired-와-qualifier)
- [@Before @BeforeClass @BeforeEach @BeforeAll](#before-beforeclass-beforeeach-beforeall)
- [PostConstruct](#postConstruct)

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

# Lombok 이란?

- [Lombok 사용상 주의점(Pitfall)](https://kwonnam.pe.kr/wiki/java/lombok/pitfall)
- [[Java] Lombok이란? 및 Lombok 활용법](https://mangkyu.tistory.com/78?category=872426)

<br>

Lombok은 애너테이션을 기반으로 `constructor`, `getter`, `setter` 등 반복적으로 작성해야 하는 메서드를 자동으로 생성하는 라이브러리입니다.

`Model` 이나 `Entity` 같은 도메인 클래스에서 반복되는 코드를 `@` 어노테이션을 이용하여 간단하게 적용할 수 있습니다.

개발자는 어노테이션만 사용하면 컴파일 되는 과정에서 자동으로 그에 맞는 코드들을 생성해줍니다.

<br>

## Lombok 을 사용하지 않는 클래스

```java
class Person {
  private String name;
  private Integer age;

  public Person() { }
  public Person(String name, Integer age) {
    this.name = name;
    this.age = age;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public Integer getAge() {
    return age;
  }

  public void setAge(Integer age) {
    this.age = age;
  }

  @Override
  public String toString() {
    return "Person{" +
           "name='" + name + '\'' +
           "age='" + age + '\'' +
           '}';
  }
}
```

클래스에 필요한 메서드들을 직접 입력해야 합니다.

(요즘은 IDE 의 도움을 받아 자동으로 Generate 할 수 있습니다)

<br>

## Lombok 을 사용하는 클래스

```java
@ToString
@Getter
@Setter
class Person {
  private String name;
  private Integer age;
}
```

롬복을 사용하면 위와 같이 코드를 간단하게 줄일 수 있으며 어떤 롬복이 구현되어있는지 어노테이션만 보고도 확인할 수 있습니다.

Lombok 의 더 많은 종류와 기능을 확인하시려면 Reference 의 활용법 링크를 참고해주세요.

<br>

## 장점

1. 코드의 길이가 짧아진다.
2. 사용하려는 어노테이션이 명시적이다.

<br>

## 단점

1. Lombok 을 모르는 사람들에게는 한눈에 와닿지 않고 어노테이션의 무분별한 사용은 오히려 가독성을 해칠 수 있다.
2. Lombok 을 제대로 이해하지 않고 코드를 작성하거나 수정하면 의도치 않은 결과를 얻을 수 있다. (아래 Reference 의 주의점 링크 참고)

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
    
    
<br>

# @Autowired 와 @Qualifier

## @Autowired 로 자동 의존 주입

Spring 에서 bean 을 설정할 때 `<context:annotation-config />` 을 설정해주고 `@Autowired` 를 사용하면  `<constructor-arg>` 나 `<property>` 태그를 추가하지 않아도 의존성 주입이 가능하다.

<br>

## @Resource 와의 차이점

`@Autowired` 는 "객체의 타입" 이 일치해야 한다.

`@Resource` 는 "객체의 이름" 이 일치해야 한다.

<br>

## 객체의 타입이 두 개 이상일 경우

만약 동일한 타입을 가진 Bean 이 여러 개라면 Spring Container 를 초기화 하는 과정에서 Exception 이 발생한다.

주입 대상이 여러 개라서 어떤 객체에 주입해야 할 지 판단할 수 없기 때문이다.

<br>

## @Qualifer 로 주입할 대상을 명시할 수 있음

`@Autowired` 어노테이션이 적용된 대상에 `@Qualifer` 를 사용하면 어떤 값에 주입할 지 설정할 수 있다.

하지만 만약 설정한 value 를 찾지 못하면 Exception 이 발생한다.

```xml
// something ...

<context:annotation-config />

<bean id="example1" class="Example">
  <qualifer value="e1" />
</bean>

<bean id="example2" class="Example" />

// something ...
```

```java
public class ExampleService {
  
  @Autowired
  @Qualifer("e1")
  private Exmaple example;

  public ExampleService(Example example) {
    this.example = example;
  }
}
```

<br>

## Spring 이 @Autowired 로 객체를 찾는 과정

1. 타입이 같은 빈 객체를 검사
2. 만약 같은 타입이 여러 개 존재한다면
    1. Qualifer 가 설정되어 있으면 그 객체를 찾는다.
    2. Qualifer 설정이 없으면 이름이 같은 객체를 찾는다.
3. 위 경우 모두 해당되지 않으면 Exception 발생


<br>

# @Before @BeforeClass @BeforeEach @BeforeAll

Spring 에서 테스트 코드를 작성할 때, 모든 테스트 코드 전에 반복적으로 해 주어야 하는 작업이 필요할 때가 있습니다.

예를 들어, 사용자 인증이 선행되어야 하는 테스트의 경우, 매 테스트 코드마다 인증하는 코드를 넣어야 합니다.

```java
public class Test {
    @Test
    public void test1(){
        authenticateForTest();  // login
        System.out.println("test 1");
    }

    @Test
    public void test2(){
        authenticateForTest();  // login
        System.out.println("test 2");
    }

    private void authenticateForTest() {
        System.out.println("authenticate");
    }
}
```

<br>

JUnit 에서는 이런 반복적인 코드를 없애기 위해 `@Before` 어노테이션을 제공합니다.

이 어노테이션에도 여러 종류가 있는데 간단하게 요약하면 아래와 같습니다

<br>

## @Before (JUnit 4), @BeforeEach (JUnit 5)

- 클래스 내에 존재하는 각각의 @Test 를 실행하기 전에 매번 실행

<br>

## @BeforeClass (JUnit 4), @BeforeAll (JUnit 5)

- 모든 테스트를 실행하기 전 딱 한번만 실행
- `static` 으로 선언해야 함

<br>

## Example

```java
public class Test {

    @BeforeAll
    public static void beforeAll() {
        System.out.println("@BeforeAll");
    }

    @BeforeEach
    public void beforeEach() {
        System.out.println("@BeforeEach");
    }

    @Test
    public void test1(){
        System.out.println("@Test 1");
    }

    @Test
    public void test2(){
        System.out.println("@Test 2");
    }

    @AfterEach
    public void afterEach() {
        System.out.println("@AfterEach");
    }

    @AfterAll
    public static void afterAll() {
        System.out.println("@AfterAll");
    }
}
```

<br>

## 전체 플로우

```html
@BeforeAll

@BeforeEach
@Test 1
@AfterEach

@BeforeEach
@Test 2
@AfterEach

@AfterAll
```

<br>

# PostConstruct

스프링 객체 초기화 방법 중 하나

해당 컴포넌트가 생성된 후 실행할 메소드를 지정한다.

<br>

## Bean 설정으로 할 경우

```xml
<bean id="testService" class="com.service.testService" init-method="init" />
```

```java
public class testService {

  public void init() {
    System.out.println("init");
  }
}
```

`init-method` 속성으로 실행할 메소드를 지정할 수 있다.

<br>

## Annotation 사용

```java
public class testService {

  @PostConstruct
  public void init() {
    System.out.println("init");
  }
} 
```

`@PostConstruct` 어노테이션을 설정해놓은 메소드는 WAS가 띄워질 때 실행된다.

`@Autowired` 이후에 실행되기 때문에 초기화 메소든 내에서 Autowired 된 값을 사용하려고 해도 에러가 나지 않는다.

<br>

# JUnit 5 에서 @Nested 와 @DisplayName 으로 가독성 있는 테스트 코드 작성하기

Spring 으로 개발하다보면 유닛 테스트 작성은 필수입니다.

많은 사람들이 거쳐가는 프로젝트는 테스트 코드의 크기도 어마어마합니다.

보통 테스트 코드를 작성할 땐 함수명으로 어떤 테스트 인지 명시하는게 관례입니다.

하지만 복잡한 비즈니스 로직을 테스트 하는데 함수명에는 이 정보를 전부 담을 수가 없습니다.

주석을 추가해서 설명을 달아 놓아도 역시 깔끔하지 않습니다.

어떻게 하면 테스트 코드의 가독성을 높일 수 있을까요?

<br>

## 기존의 Test Code

```java
public class DisplayNameTest {

    @Test
    public void testAsuccess() { /* */ }

    @Test
    public void testAfail() { /* */ }

    @Test
    public void test1success() { /* */ }

    @Test
    public void test1success() { /* */ }

    @Test
    public void test2success() { /* */ }

    @Test
    public void test2fail() { /* */}
}
```

<br>

위의 테스트 코드는 일반적으로 우리가 작성하는 JUnit 테스트 코드입니다.

함수명이 굉장히 짧고 코드 부분을 생략해서 간단해보이지만 실제 업무에서 사용되는 테스트 코드는 이렇게 간단하지 않습니다.

<br>

## @Nested

코드를 보면 아시겠지만 관심사가 비슷한 메소드가 몇개 보입니다.

똑같은 기능인데 성공 / 실패 여부만 나눠져 있는 메소드죠

같은 기능이니까 한 메소드에 넣어서 성공 / 실패 여부 두가지를 테스트 하면 어떨까? 하는 생각도 들지만 테스트 코드 하나가 너무 비대해집니다.

그리고 여기선 간단하게 성공 / 실패 여부로만 표현했지만 실제 코드에서는 상황이나 조건에 따른 여러 종류의 Exception 을 던져야 할 수도 있습니다.

<br>

이런 경우에 `@Nested` 클래스로 비슷한 함수를 묶어주면 훨씬 알아보기 쉽습니다.

[Junit 5 User Guide - Nested Test](https://junit.org/junit5/docs/current/user-guide/#writing-tests-nested) 를 보면 좋은 예제를 제공해줍니다.

처음 코드를 `@Nested` 클래스를 사용해서 수정해보겠습니다.

```java
public class DisplayNameTest {

    @Nested
    class testA {

        @Test
        public void success() { /* */ }

        @Test
        public void fail() { /* */ }
    }

    @Nested
    class testNumber {
        
        @Nested
        class test1 {

            @Test
            public void success() { /* */ }

            @Test
            public void fail() { /* */ }
        }

        @Nested
        class test2 {

            @Test
            public void success() { /* */ }

            @Test
            public void fail() { /* */ }

        }
    }
}
```

<br>

전체적인 코드의 양은 늘어났지만 계층적인 구조가 되어 훨씬 알아보기 편해졌습니다.

게다가 클래스로 구분되어 있으니 `success` 와 `fail` 을 중복으로 사용해도 전혀 문제가 없습니다.

실제로 테스트를 돌리면 장점이 더 명확하게 나타납니다.

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/images/nested-display-name-1.png?raw=true)

<br>

테스트 결과에서도 비슷한 테스트끼리 묶고 결과를 좀더 심플하게 표현할 수 있습니다.

<br>

## @DisplayName

`@Nested` 클래스로 계층을 나누어도 여전히 함수명은 알아보기 어렵습니다.

우리가 영어권이 아니라서 그런것 같습니다..그런데 실제로 영어권이더라도 camelCase 또는 snake_case 로 이루어진 영어가 한눈에 읽히는건 쉬운 일이 아닙니다.

[JUnit 5 User Guide - Display Names](https://junit.org/junit5/docs/current/user-guide/#writing-tests-display-names) 에 나와있는 예제를 보면 함수명이 길어지니 한눈에 들어오지 않지만 `@DisplayName` 어노테이션을 사용하면 간단하게 표현할 수 있습니다.

`@DisplayName` 은 `@Nested` 클래스와 함께 쓰면 더 빛을 발합니다.

위의 코드를 한번 더 수정해보겠습니다.

```java
public class DisplayNameTest {

    @Nested
    @DisplayName("A 테스트")
    class testA {

        @Test
        @DisplayName("성공")
        public void success() { /* */ }

        @Test
        @DisplayName("실패")
        public void fail() { /* */ }
    }

    @Nested
    @DisplayName("숫자")
    class testNumber {

        @Nested
        @DisplayName("1 테스트")
        class test1 {

            @Test
            @DisplayName("성공")
            public void success() { /* */ }

            @Test
            @DisplayName("실패")
            public void fail() { /* */ }
        }

        @Nested
        @DisplayName("2 테스트")
        class test2 {

            @Test
            @DisplayName("성공")
            public void success() { /* */ }

            @Test
            @DisplayName("실패")
            public void fail() { /* */ }

        }
    }
}
```

<br>

어노테이션 때문에 조금 지저분해 보이지만 실제 테스트를 돌리면 결과가 이쁘게 나옵니다.

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/images/nested-display-name-2.png?raw=true)

<br>

여기서 한가지 의문이 생길 수 있습니다.

굳이 `@DisplayName` 어노테이션을 사용해서 코드량을 늘리는 것보다 메소드 명을 한글로 작성하는게 낫지 않을까?

실제로 현업에서도 테스트 코드 작성 시 메소드 이름을 한글로 작성하는 케이스가 많다는 이야기를 종종 들었습니다.

저도 처음엔 고민을 했었는데 다음과 같은 이유들로 `@DisplayName` 을 쓰기로 결정했습니다.

- 한글로 작성해도 언더바를 작성해야 해서 가독성이 좋지 않음
- `@Nested` 와 함께 쓰려면 클래스를 작성해야 하는데 한글명으로 만드는 것보단 `@DisplayName` 을 쓰는게 깔끔함
- 드문 일이지만 외국인과 협업해야하는데 테스트코드명이 전부 한글로 되어 있으면 당황하겠죠? (이러면 DisplayName 도 한글로 못적을 것 같지만..)
- 가장 주목해야 할 점은 JUnit 개발자들은 영어가 모국어 수준일텐데도 `@DisplayName` 어노테이션을 추가했다는 점

<br>

### 테스트 결과에서 한글이 제대로 나오지 않는다면?

1. `Preferences > Build, Execution, Deployment > Build Tools > Gradle` 로 이동
2. `Run tests using` 을 `IntelliJ IDEA` 로 변경
3. Apply and OK 후 적용 안되면 인텔리제이 재시작

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/images/nested-display-name-3.png?raw=true)
