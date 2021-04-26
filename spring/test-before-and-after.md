
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
