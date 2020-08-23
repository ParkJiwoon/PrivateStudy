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
