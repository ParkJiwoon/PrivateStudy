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
