# Mockito @Mock @MockBean @Spy @SpyBean 정리

# 1. Overview

Spring 에서 Test Code 를 작성할 때 연관 클래스 관계 없이 특정 클래스만 테스트 하고 싶을 때 Mockito 라이브러리를 사용합니다.

<br>

# 2. @Mock

```java
public class ServiceTest {
    private Service service;

    @Test
    public void test() {
        // mocking
        Repository repository = Mockito.mock(Repository.class);
        service = new Service(repository);

        // stub
        Mockito.when(repository.findAll()).then(item -> {
            System.out.println("This is Repository");
            return Collections.emptyList();
        });

        // when
        service.findAllItems();

        // then
        Mockito.verify(repository, Mockito.times(1)).findAll();
    }
}
```

<br>

위 코드는 `@Mock` 을 사용해서 아래와 같이 바꿀 수 있습니다.

```java
@ExtendWith(MockitoExtension.class)
public class ServiceTest {
    private Service service;

    @Mock
    private Repository repository;

    @Test
    public void test() {
        service = new Service(repository);

        // .. 이하 동일
    }
}
```

쉽게 말하면 `Mockito.mock()` 을 사용하지 않고 쉽게 Mock 객체를 주입할 수 있습니다.

대신 `@Mock` 을 사용하려면 `@ExtendWith(MockitoExtension.class)` 을 꼭 추가해야 합니다.

<br>

추가로 `Service` 에 `@InjectMocks` 어노테이션을 붙이면 알아서 `@Mock` 으로 생성된 필요한 객체를 주입해서 만들어줍니다.

그래서 `service = new Service(repository)` 코드처럼 직접 객체를 생성하지 않아도 됩니다.

<br>

# 3. @Spy

만약 위 예제에서 `Service` 의 비즈니스 로직 중 특정 메소드는 실제 기능을 사용하고 싶을 수도 있습니다.

이럴 때는 `@Mock` 대신 `@Spy` 를 사용하고 실제 기능을 사용하고 싶은 메소드는 stub 하지 않으면 됩니다.

<br>

# 4. @MockBean, @SpyBean

`Bean` 이라는 suffix 가 붙으면 Mock 객체를 스프링 컨테이너에 주입합니다.

따라서 테스트 클래스 위에 `@SpringBootTest` 어노테이션이 붙어있을 때 사용합니다.

Mock 객체를 스프링 컨테이너에 등록하기 때문에 `@InjectMock` 대신 `@Autowired` 를 사용해서 직접 주입받아야 합니다.

<br>

# 5. Summary

- `@Mock`: 실제 인스턴스가 아닌 가상의 객체 (Mock) 를 만들어서 반환
- `@Spy`: 실제 인스턴스의 일부 메서드만 Mocking 하여 원하는 값을 반환하게 하고 나머지는 실제 메서드를 호출하는 객체를 반환
- `@InjectMocks`: 이 어노테이션이 붙어있는 클래스는 `@Mock` 또는 `@Spy` 로 선언된 객체가 자신의 필드값으로 존재하는 경우 주입
- `@MockBean`: `@SpringBootTest` 로 ApplicationContext 를 띄우는 경우 기존에 있는 객체를 Mock 객체로 교체함
- `@SpykBean`: `@SpringBootTest` 로 ApplicationContext 를 띄우는 경우 기존에 있는 객체를 Spy 객체로 교체함

<br>

`@MockBean` 과 `@SpyBean` 은 `spring-boot-starter-test` 하위에 있는 `org.springframework.boot.test.mock.mockito` 패키지에 존재

그러므로 스프링 컨테이너를 띄우지 않고 (`@SpringBootTest` 를 사용하지 않고) 단순 유닛 테스트만 한다면 `@Mock` 을 사용해서 가짜 객체를 만들면 되지만, 컨테이너를 띄우고 그 안의 Bean 을 모킹하고 싶다면 `@MockBean` 어노테이션을 사용해야 합니다.

`@Mock` 객체는 가짜 객체기 때문에 행위를 반드시 지정해줘야 합니다.

<br>

`when().thenReturn()` 과 `given().willReturn()` 의 차이는 Mockito 를 사용하냐 BDDMockito 를 하냐의 차이가 있습니다.
