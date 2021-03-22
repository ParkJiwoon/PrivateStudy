# Reference

- [Inflearn: 스프링 핵심 원리 - 기본편 (김영한)](https://www.inflearn.com/course/스프링-핵심-원리-기본편#)

<br>

# 객체 지향 설계와 스프링

스프링은 DI 를 사용하여 다형성 + OCP, DIP 를 가능하게 지원합니다.

그래서 클라이언트 코드의 변경 없이 기능 확장이 가능합니다.

<br>

## 항상 Interface 를 사용해야 할까?

이상적으로는 모든 설계에 인터페이스를 만드는 게 좋습니다.

하지만 확장할 가능성이 없는 클래스에 대해서도 모두 인터페이스를 만든다면 개발 비용이 늘어나고 이후에 다른 사람이 코드를 분석할 때도 한번씩 더 코드를 들여다봐야 합니다.

그러므로 하나만 사용할 때는 우선 구현 클래스를 먼저 사용하고 추후에 확장이 필요하게 되면 리팩토링 하는 방향으로 설계합니다.

<br><br>

# 예제 코드 만들어보기

스프링 부트를 활용해서 예제 코드를 간단하게 만들어봅니다.

- [Github Code](https://github.com/ParkJiwoon/practice-webmvc)

<br>

## 비지니스 요구사항과 설계

- 회원
  - 회원을 가입하고 조회할 수 있다.
  - 회원은 일반과 VIP 두가지 등급이 있다.
  - 회원 데이터는 자체 DB 를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)
- 주문과 할인 정책
  - 회원은 상품을 주문할 수 있다.
  - 회원 등급에 따라 할인 정책을 적용할 수 있다.
  - 할인 정책은 모든 VIP 는 1000 원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경될 수 있다)
  - 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수도 있다. (미확정)

<br><br>

# IoC, DI, Container

## 제어의 역전 IoC (Inversion of Control)

- 기존 프로그램은 클라이언트에서 직접 서버 객체를 생성해서 실행했습니다.
- 하지만 AppConfig 에게 서버 객체 연결 부분을 넘기고 클라이언트 입장에서는 어떤 구현체가 오든지 신경쓰지 않게 합니다.
- 이렇게 프로그램이 직접 흐름을 제어하는게 아니라 외부에 흐름 제어 권한을 넘기는게 제어의 역전이라고 합니다.

<br><br>

# Spring Container

스프링 컨테이너에 대해서 알아봅니다.

<br>

## 1.  스프링 컨테이너 사용

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

- `ApplicationContext` 를 스프링 컨테이너라고 합니다.
- `ApplicationContext` 는 인터페이스입니다.
- 스프링 컨테이너는 xml 기반으로 만들거나 Annotation 기반으로 만들 수 있습니다.
  - 요즘에는 xml 대신 Annotation 기반으로 많이 합니다.

<br>

더 정확히 말하자면 스프링 컨테이너를 부를 때 `BeanFactory` 와 `ApplicationContext` 로 구분해서 이야기합니다.

하지만 `BeanFactory` 를 직접 사용하는 경우는 거의 없기 때문에 일반적으로 `ApplicationContext` 를 스프링 컨테이너라고 합니다.

<br>

## 2. 스프링 컨테이너에서 Bean 가져오기

```java
@Bean(name = "aaa")  // 이름을 명시적으로 지정했기 때문에 "aaa" 가 이름이 됨
public AaaService aaaService() {
    return new AaaService();
}

@Bean  // 이름이 따로 지정되지 않았기 때문에 "bbbService" 가 이름이 됨
public BbbService bbbService() {
    return new BbbService();
}

AaaService aaaService = applicationContext.getBean("aaa", AaaService.class);
BbbService bbbService = applicationContext.getBean("bbbService", BbbService.class);
```

- `ApplicationContext` 는 `getBean(name)` 메서드를 통해 컨테이너에서 빈을 직접 가져올 수 있습니다.
- Bean 이름은 따로 지정하지 않는 경우 메서드 이름으로 자동 등록됨
- Bean 이름은 중복되지 않아야 합니다.
  - 중복되면 다른 Bean 이 무시되거나 기존 Bean 을 덮어버리는 등의 문제가 발생할 수 있습니다.

<br><br>

# Spring Bean

Spring Bean 에 대해서 알아봅니다.

- [Github Code](https://github.com/ParkJiwoon/practice-webmvc/pull/4)

<br>

## Bean 상속 관계

- Spring Bean 을 가져오면 자식 관계에 있는 Bean 도 전부 가져옵니다.
- 그래서 객체의 최고 부모인 `Object` 타입으로 조회하면 모든 스프링 빈을 조회합니다.
- 실제로 `ApplicationContext` 에서 직접 Bean 을 가져올 일은 없습니다.
  - Spring Container 가 자동으로 주입해 주는 걸 사용

<br>

# BeanFactory 와 ApplicationContext

BeanFactory 와 ApplicationContext 에 대해서 알아봅니다.

<br>

## 1. 의존 관계

```java
BeanFactory <- ApplicationContext <- ApplicationConfig, ApplicationContext
(Interface)     (Interface)
```

- BeanFactory 를 ApplicationContext 가 상속받습니다.

<br>

## 2. BeanFactory

- 스프링 컨테이너의 최상위 인터페이스
- 스프링 빈을 관리하고 조회하는 역할을 담당
- `getBean()` 등을 제공

<br>

## 3. ApplicationContext

- BeanFactory 기능을 모두 상속받아서 제공
- 애플리케이션을 개발할 때는 스프링 빈 관리와 조회 기능은 물론, 수많은 부가 기능이 필요
- ApplicationContext 는 빈 관리 뿐만 아니라 여러가지 부가 기능을 제공

<br>

### 3.1. ApplicationContext 가 제공하는 부가 기능 (상속받는 Interface 들)

- `MessageSource`
  - 메세지 소스를 활용한 국제화 기능
  - 예를 들면 한국에서 오는 요청은 한국어로, 영어권에서 오는 요청은 영어로 출력
- `EnvironmentCapable`
  - 환경변수
  - 로컬, 개발, 운영 등을 구분해서 처리
- `ApplicationEventPublisher`
  - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- `ResourceLoader`
  - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

<br><br>

# Singleton (싱글톤) 패턴 활용

Spring 에서 Singleton 패턴을 어떻게 활용하는 지 알아봅니다.

<br>

## 1. Singleton 패턴의 필요성

- Spring 은 웹 애플리케이션을 만들기 위한 프레임워크
- 웹 애플리케이션은 여러명의 사용자가 동시에 요청 가능
- 만약 1000 명의 사용자가 요청을 했다면 1000 개의 객체를 생성했다가 소멸하는 과정이 들어감
- 불필요한 생성, 소멸 과정을 생략하기 위해 싱글톤 패턴을 도입

<br>

## 2. Singleton 패턴

```java
public class SingletonService {

    // 1. static 영역에 객체를 딱 1개만 생성
    private static final SingletonService instance = new SingletonService();

    // 2. 생성자 대신 외부에 인스턴스를 제공하는 메서드를 만듬
    public static SingletonService getInstance() {
        return instance;
    }

    // 3. 외부에서 임의로 생성하지 못하게 생성자를 private 으로 변경
    private SingletonService() { }
}
```

- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴
- `new` 로 외부에서 직접 생성하는 대신 `getInstance()` 메서드를 만들어서 제공
  - 생성자를 `private` 로 변경해서 외부에서 생성 불가능하게 처리
- 인스턴스가 이미 존재하면 해당 인스턴스를 리턴하고 그렇지 않으면 새로 생성해서 리턴

<br>

## 3. Spring 에서는?

- Spring 에서 항상 싱글톤 패턴으로 객체 제공
- 개발자가 따로 싱글톤 패턴을 적용할 필요 없음
- 스프링 컨테이너는 싱글톤 패턴이 모든 단점을 해결하면서 개발자에게 제공
- 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤이 아닌 새로운 객체 생성도 가능함
  - 하지만 자주 쓰지 않음 (99% 싱글톤만 사용)

<br>

## 4. 싱글톤 방식의 주의점

- 싱글톤 패턴을 사용하면 여러 클라이언트가 하나의 객체를 공유
- **싱글톤 객체는 상태를 유지 (stateful) 하게 설계하면 안됨**
- 무상태 (stateless) 로 설계 필요 !
  - 특정 클라이언트에 의존적인 필드 존재 X
  - 특정 클라이언트가 값을 변경할 수 있는 필드 존재 X
  - 가급적 읽기만 가능
  - 필드 대신 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용
- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있음

<br>

## 5. @Configuration 과 싱글톤

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public OrderService orderService() {
        return new OrderService(memberRepository(), discountPolicy());
    }

    @Bean
    public MemoryMemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

- `MemberService` Bean 을 호출하면 `new MemoryMemberRepository()` 를 호출하여 객체를 생성
- `OrderService` Bean 을 호출해도 `new MemoryMemberRepository()` 를 호출하여 객체를 생성
- `new` 키워드로 생성했기 때문에 싱글톤 패턴이 깨지는게 아닐까??
  - 아니다 !
- 스프링 컨테이너는 스프링 빈이 싱글톤이 되도록 보장해줌
  - 우리는 `AppConfig` 클래스를 작성했지만 스프링 컨테이너에 등록되는건 임의로 조작된 클래스
  - 스프링이 바이트 코드를 조작하여 싱글톤으로 동작하도록 함
  - 그렇다고 완전히 다른 클래스가 아닌 `AppConfig` 를 상속받는 클래스를 만듬 `AppConfig@CGLIB`
  - 만약 `@Configuration` 어노테이션을 제거하면 `AppConfig` 클래스가 그대로 생성되고 각 Bean 들에 대해 싱글톤이 보장되지 않음

<br><br>

# Component Scan 과 의존 관계 주입

Bean 을 등록할때는 자바 `@Configuration` 클래스에서 `@Bean` 어노테이션을 추가하거나 xml 에 bean 설정을 통해서 일일히 등록해야 합니다.

하지만 이런 과정을 사람이 직접 하면 중복으로 등록하거나 누락되는 경우도 발생하고 전부 등록하는게 귀찮습니다.

그래서 Spring 에서는 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공합니다.

또한 의존관계도 자동으로 주힙하는 `@Autowired` 라는 기능도 제공합니다.

<br>

## 1. Component Scan

```java
@Configuration
@ComponentScan(
        excludeFilters = @ComponentScan.Filter(
                type = FilterType.ANNOTATION, 
                classes = Configuration.class
        )
)
public class AutoAppConfig {
}
```

- `AutoAppConfig` 클래스에 별다른 설정 이 없어도 ComponentScan 어노테이션만 붙으면 자동으로 모든 클래스의 컴포넌트를 찾아서 Bean 으로 등록
- `@ComponentScan`
  - `@Component` 라는 어노테이션이 붙은 모든 Bean 을 찾아서 등록해줌
- `excludeFilters`
  - 컴포넌트 스캔을 사용하면 `@Configuration` 어노테이션이 붙은 설정 파일들도 전부 등록하게 됨
  - Bean 을 등록하는 설정이 있으면 중복으로 등록하는 일이 발생할 수 있기 때문에 스캔 대상에서 제거
  - 실무에서는 그냥 신경쓰지 않고 따로 제외하지 않음
  - 공부할 때 원하는 값을 얻기 위해 사용

<br>

컴포넌트 스캔을 사용하면 각 컴포넌트에 `@Component` 를 붙여야합니다.

그런데 설정 파일에서는 Bean 등록 뿐만 아니라 의존관계 주입도 직접 해주고 있었는데 여기서는 어떻게 할 수 있을까요?

`@Autowired` 어노테이션을 사용하면 자동으로 의존 관계를 주입해줍니다.

<br>

## 2. Component 빈 이름

- `@ComponentScan` 은 `@Component` 가 붙은 모든 클래스를 스프링 빈으로 등록
- 이때 스프링 빈의 기본 이름은 클래스명을 사용하되 맨 앞글마잔 소문자를 사용
  - 빈 이름 기본 전략: `MemberService` 클래스 → memberService
  - 빈 이름 직접 지정: `@Component("memberServiceBean")` 처럼 직접 지정 가능

<br>

## 3. @Autowoired 의존관계 자동 주입

- 생성자에 `@Autowoired` 를 지정하면 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입
- 기본 조회 전략은 타입이 같은 빈을 찾아서 주입
  - `getBean(MemberRepository.class)` 와 동일
  - 타입이 안맞거나 중복된 타입의 경우 어떻게 할까??

<br>

## 4. 탐색 시작 위치 지정 - basePackages

```java
@Configuration
@ComponentScan(basePackages = "com.jiwoon.practicewebmvc.member")
public class AutoAppConfig {
}
```

- 컴포넌트 스캔을 시작할 위치를 지정할 수 있음
- `basePackages = { "com.jiwoon.practicewebmvc.member", "com.jiwoon.adfd" }` 으로 여러 위치도 지정 가능
- 만약 지정하지 않으면 `@ComponentScan` 이 붙은 설정 정보 클래스의 패키지가 시작 위치가 됨
- 일반적으로는 basePackages 를 따로 추가하지 않고 프로젝트 최상단에 설정 파일을 둠 (권장 방법)
- 참고로 스프링 부트를 사용하면 스프링 부트의 대표 시작 정보인 `@SpringBootApplication` 안에 `@ComponentScan` 이 들어있음
  - 그래서 별다른 설정 없이도 Component Scan 을 사용할 수 있음

<br>

## 5. 스캔 기본 대상

컴포넌트 스캔은 `@Component` 뿐만 아니라 다음 어노테이션들도 대상에 포함됩니다.

각 어노테이션들은 내부적으로 `@Component` 를 포함하고 있습니다.

어노테이션은 명시적으로 컴포넌트의 역할을 나누기도 하지만 Spring 에서 부가적으로 처리해주는 기능도 포함되어 있습니다.

- `@Component`
  - 컴포넌트 스캔에서 사용
- `@Controller`
  - 스프링 MVC 컨트롤러에서 사용
  - Spring 에서는 부가적으로 MVC 컨트롤러로 인식
- `@Service`
  - 스프링 비즈니스 로직에서 사용
  - Spring 에서 부가적인 처리는 해주지 않음
  - 대신 코드를 보는 개발자들이 핵심 비즈니스 로직이 여기에 있겠구나 하고 비즈니스 계층을 인식하는 데 도움을 줌
- `@Repository`
  - 스프링 데이터 접근 계층에서 사용
  - Spring 에서 데이터 접근 계층으로 인식하고 데이터 계층의 예외를 스프링 예외로 변환해줌
- `@Configuration`
  - 스프링 설정 정보에서 사용
  - Spring 에서 설정 정보로 인식하고 스프링 빈이 싱글톤을 유지하도록 추가 처리

<br>

## 6. 중복 등록과 충돌

컴포넌트 스캔에서 같은 빈 이름을 등록하면 어떻게 될까요?

다음 두 가지 상황이 있습니다.

<br>

### 6.1. 자동 빈 등록 vs 자동 빈 등록

컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데 이름이 같은 경우 스프링은 오류를 발생시킵니다.

- `ConflictingBeanDefinitionException` 발생

<br>

### 6.2. 자동 빈 등록 vs 수동 빈 등록

```java
@Component
public class MemoryMemberRepository implements MemberRepository {
		// ...
}

@Configuration
public class AutoAppConfig {

    @Bean(name = "memoryMemberRepository")
    MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

- `MemoryMemberRepository` Bean 을 중복해서 등록
- 하지만 에러가 발생하지 않음
- 수동 빈이 우선권을 가짐
- 자동 빈을 수동 빈이 오버라이딩 해버림
  - `Overriding bean definition for bean 'memoryMemberRepository' with a different definition: replacing ...` 로그 출력됨

<br>

개발자가 의도적으로 수동 빈을 등록했다면 오버라이딩 되는 것이 자연스럽습니다.

하지만 실제로는 의도적으로 빈을 오버라이딩 하는게 아닌 여러 설정들이 꼬여서 발생하는 경우가 대부분입니다.

그래서 최근 스프링 부트에서는 **수동 빈 등록과 자동 빈 등록이 충돌나면 오류가 발생하도록 기본 값을 변경**했습니다.

그래서 `@SpringBootApplication` 이 붙은 메인 Application 을 실행시키면 오류를 볼 수 있습니다.

```java
Description:

The bean 'memoryMemberRepository', defined in class path resource [{package}/AutoAppConfig.class], could not be registered. A bean with that name has already been defined in file [{package}/member/MemoryMemberRepository.class] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```

만약 기본 설정을 무시하고 충돌이 발생하지 않도록 하려면 위 로그에 나온 것처럼 `application.properties` 에  설정을 추가하면 됩니다.

<br>

```java
// application.properties

spring.main.allow-bean-definition-overriding=true
```

<br><br>

# 의존 관계 주입 (DI)

스프링에서 중요한 의존관계 주입 방법에 대해서 알아봅니다.

스프링에는 크게 4 가지 방법으로 의존관계 주입을 해줄 수 있습니다.

현재는 생성자 주입만을 사용하는 것이 좋으며 이외에도 어떤 점이 좋고 어떤 점이 나쁜지 알아봅니다.

<br>

## 1. 다양한 의존관계 주입 방법

- 생성자 주입
- 수정자 주입 (Setter 주입)
- 필드 주입
- 일반 메서드 주입

<br>

### 1.1. 생성자 주입

```java
@Component
public class OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderService(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

- 빈을 등록하면서 동시에 주입됨
- 이름 그대로 생성자를 통해서 의존 관계를 주입하는 방법
- 생성자 호출 시점에 딱 1번만 호출되는 것이 보장됨
- "불변, 필수" 의존관계에 사용
- 생성자가 딱 1 개만 있다면 `@Autowired` 를 생략해도 됨
  - 여러 개 존재한다면 어떤 생성자로 의존관계를 주입할지 명시해야함

<br>

### 1.2. 수정자 주입 (Setter)

```java
@Component
public class OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```

- 생성자 주입과 다르게 모든 빈 생성 후에 의존관계 주입은 나중에 발생

<br>

**# 자바 빈 프로퍼티 규약**

Class 의 필드에 접근할 때는 직접 접근하지 않고 Getter, Setter 를 사용해서 접근한다.

<br>

### 1.3. 필드 주입

```java
@Component
public class OrderService {

    @Autowired private MemberRepository memberRepository;
    @Autowired private DiscountPolicy discountPolicy;

}
```

- 굉장히 간결해서 예전에 많이 씀
- 그러나 이제 인텔리제이에서 사용하지 않는걸 권장함
- 외부에서 값 변경이 어려워 테스트가 힘듬
- 하지만 테스트 코드는 애플리케이션과 관계 없기 때문에 필드 주입 방식으로 많이 사용

<br>

### 1.4. 메서드 주입

생성자랑 비슷하게 메서드에 `@Autowird` 를 붙이고 파라미터로 받아와서 주입하는 겁니다.

거의 사용하지 않습니다.

<br>

## 2. 옵션 처리

주입할 스프링 빈이 없어도 동작해야 하는 경우가 있습니다.

`@Autowired` 는 `required` 기본값이 `true` 입니다.

자동 주입 대상을 옵션으로 처리하는 방법은 다음 세 가지가 있습니다.

- `@Autowired(required = false)` : 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안됨
- `org.springframework.lang.@Nullable` : 자동 주입할 대상이 없으면 Null 이 입력됨
- `Optional`: 자동 주입할 대상이 없으면 `Optional.empty` 가 입력됨

```java
public class AutowiredTest {

    @Test
    void AutowiredOption() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);
    }

    static class TestBean {

        @Autowired(required = false)
        public void setNoBean1(Member noBean1) {
            // 호출이 아예 안됨
            System.out.println("noBean1 = " + noBean1);
        }

        @Autowired
        public void setNoBean2(@Nullable Member noBean2) {
            // noBean2 = null
            System.out.println("noBean2 = " + noBean2);
        }

        @Autowired
        public void setNoBean3(Optional<Member> noBean3) {
            // noBean3 = Optional.empty
            System.out.println("noBean3 = " + noBean3);
        }
    }
} 
```

<br>

## 3. 생성자 주입을 사용해야 하는 이유

과거에는 수정자 주입, 필드 주입을 많이 사용했지만 최근에는 생성자 주입을 사용하는 것이 권장됩니다.

기본으로 생성자 주입을 사용하면서 필요에 의해 옵션으로 수정자 주입을 같이 사용할 수도 있지만 필드 주입은 권장되지 않습니다.

<br>

### 3.1. 불변

- 대부분의 의존관계는 한번 주입하면 변경할 일이 없음
- 생성자 주입을 사용하면 `Setter` 메서드를 `public` 으로 열어두어야 하기 때문에 누군가에 의해 변경 될 가능성이 존재

<br>

### 3.2. 누락 방지

- 생성자 주입을 사용하면 순수한 Java 단위의 테스트 코드 작성이 가능
- Java 단위 테스트 코드를 짤 때 수정자 주입을 사용하면 필요한 의존관계 주입을 빼먹을 가능성이 있음

<br>

### 3.3. final 키워드

- 누락 방지와 비슷한데 생성자 주입만 유일하게 `final` 키워드가 사용 가능
- 의존관계가 설정되지 않은 버그를 미리 방지해줌
- 수정자 주입, 필드 주입은 생성자 이후에 호출되기 때문에 `final` 키워드 사용이 불가능

<br>

## 4. Lombok 으로 생성자 주입

롬복이라는 라이브러리가 반복적이고 귀찮은 코드 작업을 대신해줍니다.

최근에는 생성자 주입을 많이 사용하는데 롬복의 `@RequiredArgsConstructor` 와 조합하면 코드를 간단하게 짤 수 있습니다.

<br>

### Before

```java
@Component
public class OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderService(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

<br>

### After

```java
@Component
@RequiredArgsConstructor
public class OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
}
```

<br>

## 5. 조회 빈이 2개 이상

같은 인터페이스를 상속하는 두 가지의 구현체가 존재할 때 두 개 모두 빈으로 등록하고 사용하면 `NoUniqueBeanDefinitionException` 이 발생합니다.

스프링 입장에서는 어떤 빈을 사용하려고 하는지 알 수 없기 때문에 발생합니다.

<br>

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy {
    // ...
}
@Component
public class FixDiscountPolicy implements DiscountPolicy {
    // ...
}
@Component
public class OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    // discountPolicy 에는 어떤 빈이 들어갈까??
    public OrderService(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

- 예를 들어 `DiscountPolicy` 를 상속하는 두 개의 구현체가 동시에 빈으로 등록됩니다.
- `DiscountPolicy` 를 자동 주입 받아서 사용하는 `OrderService` 는 둘 중 어떤게 실제로 사용하려는 빈인지 알 수 없습니다.
- 이런 문제를 해결하기 위해서는 세 가지 방법이 존재합니다.

<br>

### 5.1. @Autowired 사용 시 빈 이름 지정

```java
@Component
public class OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderService(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = rateDiscountPolicy;
    }
}
```

- 생성자 파라미터로 `rateDiscountPolicy` 를 명시적으로 지정하면 해당 빈을 사용합니다.

<br>

### 5.2. @Qualifier

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {
    // ...
}
@Component
public class OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderService(MemberRepository memberRepository,
                        @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

- `@Qualifier` 는 빈의 힌트를 추가
- 빈 이름을 변경하는 것은 아님
- 만약 일치하는 힌트가 없을 시 빈의 이름을 갖고 찾음
- 하지만 혼란을 방지하기 위해 `@Qualifier` 는 힌트를 찾는 데만 사용하는게 좋음
- `@Configuration` 에서도 동일하게 `@Qualifier` 를 사용할 수 있음

<br>

### 5.3. @Primary

```java
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {
    // ...
}
@Component
public class OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderService(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

- 빈의 우선순위를 등록
- `OrderService` 에서는 아무런 처리도 하지 않았지만 우선 순위에 의해 `RateDiscountPolicy` 가 주입됨
- 코드를 보면 알 수 있지만 `OrderService` 에는 아무런 수정도 하지 않고 빈을 등록 가능
- 겹치는 빈이 있을 때 자주 사용하는 빈을 `@Primary` 로 등록하고 가끔 사용하는 다른 빈은 이름이나 `@Qualifier` 로 필요할 때 지정하는 방식을 주로 사용

<br>

`@Qualifer` 와 동시에 사용한다면 우선순위는 더 자세히 지정한 `@Qualifer` 가 더 높습니다.

<br>

### 5.4. Annotation 직접 만들기

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {
    // ...
}
```

- `@Qualifer` 를 사용하면 안에 String 이 들어가기 때문에 컴파일 시 잡기가 어려움
- 따라서 별도의 어노테이션으로 만들어서 사용하는게 더 깔끔

<br>

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}
```

- `@Qualifer` 에 사용되는 어노테이션들을 복사
- 그리고 `@Qualifier("mainDiscountPolicy")` 사용

<br>

```java
@Component
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {
    // ...
}
@Component
public class OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderService(MemberRepository memberRepository,
                        @MainDiscountPolicy DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

- `@Qualifer` 와 동일한 방식으로 사용 가능
- `@MainDiscountPolicy` 을 사용하기 때문에 재사용도 가능하고 어디에서 사용하는지 추적하기도 쉬워짐

<br>

## 6. 조회한 빈이 모두 필요할 때 List, Map

```java
public class AllBeanTest {

    /**
     * 의도적으로 해당 타입의 스프링 빈이 모두 필요한 경우가 있습니다.
     *
     * 예를 들어 할인 서비스를 제공하는 데 클라이언트가 할인의 종류 (rate, fix) 를 선택할 수 있다고 가정합니다.
     *
     * 스프링을 사용하면 소위 말하는 전략 패턴을 매우 간단하게 구현 가능합니다.
     *
     * Map, List 를 사용하면 같은 타입의 여러 빈을 모두 가져올 수 있습니다.
     *
     * DiscountService 는 discountCode 에 따라서 어떤 할인을 적용할 지 유연하게 적용 가능합니다.
     *
     */

    @Test
    void findAllBean() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L, "userA", Grade.VIP);
        int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");

        assertThat(discountService).isInstanceOf(DiscountService.class);
        assertThat(discountPrice).isEqualTo(1000);

        int rateDiscountPrice = discountService.discount(member, 20000, "rateDiscountPolicy");
        assertThat(rateDiscountPrice).isEqualTo(2000);
    }

    static class DiscountService {
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;
            System.out.println("policyMap = " + policyMap);
            System.out.println("policies = " + policies);
        }

        public int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member, price);
        }
    }
}
```

<br><br>

# 빈 생명주기 콜백 (Bean Life Cycle)

스프링 빈은 다음과 같은 라이프사이클을 가집니다.

```java
"객체 생성" -> "의존관계 주입"
```

<br>

개발자는 의존 관계 주입이 모두 완료된 시점을 어떻게 알 수 있을까요?

스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공합니다.

또한 스프링은 스프링 컨테이너가 종료되기 직전에 소멸 콜백을 줍니다.

따라서 안전하게 종료 작업을 진행할 수 있습니다.

<br>

**## 스프링 빈의 이벤트 라이프사이클**

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존관계 주입
4. 초기화 콜백 (빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출)
5. 사용
6. 소멸 전 콜백 (빈이 소멸되기 직전에 호출)
7. 스프링 종료

<br>

**## 객체의 생성과 초기화를 분리하자**

객체의 생성자에서 모든 초기화를 처리하면 되지 않을까?

생성자와 초기화 작업은 책임이 다르기 때문에 두 개의 책임을 분리해두는 게 좋습니다.

단순히 값 세팅하는 것 같은 단순한 동작이라면 생성자에서 해도 괜찮지만 무거운 동작은 별도의 메서드로 분리해서 처리해야 합니다.

그리고 생성과 초기화를 분리하면 실제로 이벤트가 발생할 때까지 수행하지 않도록 "초기화 지연" 이 가능합니다.

- 생성자 : 필수 정보 (파라미터) 를 받고 메모리를 할당해서 객체를 생성
- 초기화 : 생성된 값들을 활용해서 외부 커넥션을 연결 (무거운 동작)

<br>

스프링은 크게 3 가지 방법으로 빈 생명주기 콜백을 지원합니다.

결론부터 말하자면 3 번 방법을 사용하면 됩니다.

1. 인터페이스 (InitializingBean, DisposableBean)
2. 설정 정보에 초기화 메서드, 종료 메서드 지정
3. @PostConstruct, @PreDestory 어노테이션

<br>

각 방법에 대해서 적용하기 전에 임의의 TestClient 를 만들어둡니다.

```java
// 가짜 네트워크 클라이언트
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
        connect();
        call("초기화 연결 메시지");
    }

    public void setUrl(String url) {
        System.out.println("setUrl");
        this.url = url;
    }

    // 서비스 시작시 호출
    public void connect() {
        System.out.println("connect: " + url);
    }

    public void call(String message) {
        System.out.println("call: " + url + " message = " + message);
    }

    // 서비스 종료시 호출
    public void disconnect() {
        System.out.println("disconnect: " + url);
    }
}
public class BeanLifeCycleTest {

    @Test
    public void lifeCycleTest() {
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration
    static class LifeCycleConfig {

        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("<http://hello-spring.dev>");
            return networkClient;
        }
    }
}
```

<br>

Client 생성 후 `setUrl` 을 호출하기 전까지는 `url` 필드에 아무런 값이 없습니다.

```java
생성자 호출, url = null
connect: null
call: null message = 초기화 연결 메시지
```

<br>

## 1. 인터페이스 (InitializingBean, DisposableBean)

```java
// 가짜 네트워크 클라이언트
public class NetworkClient implements InitializingBean, DisposableBean {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
    }

    ....

    /**
     * InitializingBean 인터페이스의 메서드
     * 의존관계 주입이 끝나면 호출
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("afterPropertiesSet");
        connect();
        call("초기화 연결 메시지");
    }

    /**
     * DisposableBean 인터페이스의 메서드
     * 빈 종료 시에 호출
     */
    @Override
    public void destroy() throws Exception {
        System.out.println("destroy");
        disconnect();
    }
}
```

- `InitializingBean` 은 객체 의존관계 주입이 끝나면 호출하는 `afterPropertiesSet` 메서드를 가짐
- `DisposableBean` 은 객체가 소멸할 때 호출하는 `destroy` 메서드를 가짐

<br>

### 1.1. 출력 결과

`BeanLifeCycleTest` 를 실행하면 다음과 같은 결과가 나옵니다.

```java
생성자 호출, url = null
setUrl
afterPropertiesSet
connect: <http://hello-spring.dev>
call: <http://hello-spring.dev> message = 초기화 연결 메시지
18:00:39.834 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@16610890, started on Sun Mar 21 18:00:39 KST 2021
destroy
disconnect: <http://hello-spring.dev>
```

- 여기서 중요한 점은 객체의 생성 시점인 `new NetworkClient();` 가 아니라 의존관계 주입 시점인 `return networkClient;` 이후에 메서드들이 호출됨
- 따라서 `setUrl` 이 먼저 호출되어 의존관계 주입 시점에는 url 이 존재

<br>

### 1.2. 단점

- 이 인터페이스는 스프링 전용 인터페이스라서 의존적
- 초기화, 소멸 메서드의 이름 변경 불가능
- 내가 코드를 고칠 수 없는 외부 라이브러리에 적용 불가능

<br>

이 방법은 스프링 초창기에 나온거라 지금은 거의 사용하지 않습니다.

<br>

## 2. 설정 정보에 초기화 메서드, 종료 메서드 지정

```java
// 가짜 네트워크 클라이언트
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
    }

    ...

    public void init() {
        System.out.println("init");
        connect();
        call("초기화 연결 메시지");
    }

    public void close() {
        System.out.println("close");
        disconnect();
    }
}
```

- 일반 메서드를 자유롭게 생성

<br>

```java
@Configuration
static class LifeCycleConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public NetworkClient networkClient() {
        NetworkClient networkClient = new NetworkClient();
        networkClient.setUrl("<http://hello-spring.dev>");
        return networkClient;
    }
}
```

- Bean 을 선언할 때 직접 지정

<br>

### 2.1. 출력 결과

```java
생성자 호출, url = null
setUrl
init
connect: <http://hello-spring.dev>
call: <http://hello-spring.dev> message = 초기화 연결 메시지
18:11:42.267 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@505fc5a4, started on Sun Mar 21 18:11:42 KST 2021
close
disconnect: <http://hello-spring.dev>
```

<br>

### 2.2. 특징

- 메서드 이름 자유롭게 설정 가능
- 스프링 빈이 스프링 코드에 의존 가능
- 코드가 아니라 설정 정보를 사용하기 때문에 코드를 직접 고칠 수 없는 외부 라이브러리에서도 지정 가능

<br>

### 2.3. destroyMethod 속성의 비밀

- `destroyMethod` 는 기본값으로 `(inferred)` (추론) 으로 등록되어 있음
- 이 추론 기능은 `close`, `shutdown` 이름 메서드를 자동으로 호출해줌
- 라이브러리는 대부분 `close`, `shutdown` 등을 종료 메서드로 많이 사용
- 따라서 `destroyMethod` 를 지정해주지 않아도 잘 동작함
- 만약 추론 기능을 쓰기 싫으면 `destroyMethod = ""` 설정해주면 됨

<br>

실제로 테스트 해보니 `@Bean(initMethod = "init")` 만 등록해도 `close` 메서드가 정상적으로 호출됩니다.

그렇다면 `shutdown` 과 `close` 가 동시에 존재한다면 어떻게 동작할까요?

- `shutdown` 만 존재 : `shutdown` 메서드 정상 호출
- `close` 만 존재 : `close` 메서드 정상 호출
- `close`, `shutdown` 동시에 존재 : `close` 메서드 호출

<br>

## 3. @PostConstruct, @PreDestory 어노테이션

결론부터 말하자면 이 방법을 쓰면 됩니다.

<br>

```java
// 가짜 네트워크 클라이언트
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
    }

    ...

    @PostConstruct
    public void init() {
        System.out.println("init");
        connect();
        call("초기화 연결 메시지");
    }

    @PreDestroy
    public void close() {
        System.out.println("close");
        disconnect();
    }
}


@Configuration
static class LifeCycleConfig {

    @Bean
    public NetworkClient networkClient() {
        NetworkClient networkClient = new NetworkClient();
        networkClient.setUrl("<http://hello-spring.dev>");
        return networkClient;
    }
}
```

<br>

### 3.1. 출력 결과

```java
생성자 호출, url = null
setUrl
init
connect: <http://hello-spring.dev>
call: <http://hello-spring.dev> message = 초기화 연결 메시지
18:22:06.293 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@505fc5a4, started on Sun Mar 21 18:22:06 KST 2021
close
disconnect: <http://hello-spring.dev>
```

<br>

### 3.2. 특징

- 최신 스프링에서 가장 권장하는 방법
- 어노테이션 하나만 붙이면 되므로 편리
- 스프링에 종속적인 기술이 아니라 자바 표준이기 때문에 스프링이 아닌 다른 컨테이서에서도 동작
- 컴포넌트 스캔과 잘 어울림
- 유일한 단점은 코드를 직접 고치지 못하기 때문에 외부 라이브러리에 적용 불가능
  - 이럴 때는 2 번째 방법인 `initMethod`, `destroyMethod` 를 사용하자

<br><br>

# 빈 스코프 (Bean Scope)

스프링 빈은 스프링 컨테이너의 시작과 함께 생성되어 종료될 때까지 유지됩니다.

이것은 스프링 빈이 기본적으로 싱글톤 스코프로 생성되기 때문입니다.

스코프는 번역 그대로 빈이 존재할 수 있는 범위를 뜻합니다.

스프링은 다음과 같은 다양한 스코프가 존재합니다.

- 싱글톤
  - 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
- 프로토타입
  - 스프링 컨테이너가 빈의 생성과 의존관계 주입까지만 존재하고 더는 관리하지 않는 스코프
- 웹 관련 스코프
  - `request` : 웹 요청이 들어오고 나갈때 까지 유지되는 스코프
  - `session` : 웹 세션이 생성되고 종료될 때까지 유지되는 스코프
  - `application`: 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프

<br>

## 1. 빈 스코프 지정 방법

### 1.1. 컴포넌트 스캔 자동 등록

```java
@Scope("prototype")
@Component
public class HelloBean { }
```

<br>

### 1.2. 수동 등록

```java
@Scope("prototype")
@Bean
PrototypeBean HelloBean() {
    return new HelloBean();
}
```

<br>

## 2. 프로로타입 스코프

싱글톤 스코프의 빈을 조회하면 스프링 컨테이너는 `항상 같은 인스턴스`의 스프링 빈을 반환합니다.

반면에 프로토타입 스코프를 조회하면 스프링 컨테이너는 `항상 새로운 인스턴스를 생성`해서 반환합니다.

핵심은 `스프링 컨테이너는 프로토타입 빈을 생성하고 의존관계 주입, 초기화까지만 처리`한다는 겁니다.

클라이언트에 빈을 반환하고 나면 스프링 컨테이너는 생성된 프로토타입 빈을 관리하지 않습니다.

프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에 있습니다.

그래서 `@PreDestroy` 같은 종료 메서드가 호출되지 않습니다.

<br>

### 2.1. 싱글톤 스코프 테스트

```java
public class SingletonTest {

    @Test
    void singletonBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class);

        // 미리 생성해둔 싱글톤 빈을 반환함
        System.out.println("find singletonBean1");
        SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);

        // 미리 생성해둔 싱글톤 빈을 반환함
        System.out.println("find singletonBean2");
        SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);

        System.out.println("singletonBean1 = " + singletonBean1);
        System.out.println("singletonBean2 = " + singletonBean2);
        assertThat(singletonBean1).isSameAs(singletonBean2);

        ac.close();
    }

    @Scope("singleton")
    static class SingletonBean {
        @PostConstruct
        public void init() {
            System.out.println("SingletonBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("SingletonBean.destroy");
        }
    }
}
```

- 싱글톤은 스프링 컨테이너에서 빈을 주입할 때 미리 생성해둠
- 그리고 항상 같은 인스턴스를 반환해줌
- 애플리케이션 종료 시에 `@PreDestroy` 호출됨

<br>

```java
SingletonBean.init
find singletonBean1
find singletonBean2
singletonBean1 = com.jiwoon.practicewebmvc.scope.SingletonTest$SingletonBean@4671115f
singletonBean2 = com.jiwoon.practicewebmvc.scope.SingletonTest$SingletonBean@4671115f
19:25:14.335 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@3bf9ce3e, started on Sun Mar 21 19:25:14 KST 2021
SingletonBean.destroy
```

<br>

### 2.2. 프로토타입 스코프 테스트

```java
public class ProtoTypeTest {

    @Test
    void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        // getBean 을 호출하는 시점에 생성해서 반환
        System.out.println("find prototypeBean1");
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);

        // getBean 을 호출하는 시점에 생성해서 반환
        System.out.println("find prototypeBean2");
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

        // Prototype Scope 라서 둘은 다름
        System.out.println("prototypeBean1 = " + prototypeBean1);
        System.out.println("prototypeBean2 = " + prototypeBean2);
        assertThat(prototypeBean1).isNotSameAs(prototypeBean2);

        // 프로토타입 스코프의 관리 책임은 이제 클라이언트에 있어서 @Destroy 호출되지 않음
        // 필요하다면 prototypeBean1.destroy(); 처럼 클라에서 직접 호출
        ac.close();
    }

    @Scope("prototype")
    static class PrototypeBean {
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

- `getBean` 을 호출할 때마다 새로 생성해서 반환해서 인스턴스가 다름
- 이미 생생해서 전달된 프로토타입 빈은 스프링 컨테이너가 관리하지 않기 때문에 `@Predestroy` 가 호출되지 않음
- 그래도 `@PostConsrcut` 는 해줌 (초기화까지는 관리해줌)

<br>

```java
find prototypeBean1
PrototypeBean.init
find prototypeBean2
PrototypeBean.init
prototypeBean1 = com.jiwoon.practicewebmvc.scope.ProtoTypeTest$PrototypeBean@291f18
prototypeBean2 = com.jiwoon.practicewebmvc.scope.ProtoTypeTest$PrototypeBean@17d88132
19:27:04.835 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@3d6f0054, started on Sun Mar 21 19:27:04 KST 2021
```

<br>

## 3. 프로토타입 빈과 싱글톤 빈을 함께 사용할 시 문제점
