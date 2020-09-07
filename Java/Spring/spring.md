```html
간단히 정리하면 스프링은 "자바 엔터프라이즈 개발을 편하게 해주는 오픈소스 프레임워크" 이며

코드의 가독성과 의존성 (클래스가 다른 클래스에 종속적임) 을 해결하기 위해 Application Context 라는 컨테이너를 제공한다.

이 컨테이너에서는 Bean 들을 관리하며 각 클래스에서 사용할 수 있도록 Bean 을 생성해주는 것을 DI (의존성 주입) 이라고 한다.

개발자가 아닌 컨테이너가 직접 Bean 을 생성, 관리하기 때문에 IoC (제어의 역전) 컨테이너라고도 한다.
```

<br>

# Spring 이란?

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
