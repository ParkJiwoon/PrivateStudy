# Content

- [Spring 이란?](#spring-이란?)
- [Property 값 주입](#property-값-주입)

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
