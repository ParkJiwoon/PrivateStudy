# Spring 에서 private 메소드, 변수 테스트하기

# 1. Overview

Spring Boot 에서 테스트 코드를 짜다 보면 `private` 메소드를 테스트 하고 싶을 때가 있습니다.

하지만 `private` 메소드나 변수는 다른 클래스에서 접근되지 않고 이는 테스트 클래스에서도 마찬가지입니다.

그렇다고 테스트가 필요한 `private` 를 모두 `public` 으로 바꾸는 것은 그다지 만족스러운 방법이 아닙니다.

다행히도 `private` 에 접근해서 사용할 수 있는 방법이 있습니다.

<br>

# 2. Class 선언

```java
public class Calculator {
    private String owner;

    public Calculator(String owner) {
        this.owner = owner;
    }

    private int add(int a, int b) {
        return a + b;
    }

    private void print() {
        System.out.println("This is private method");
    }
}
```

- 테스트를 위한 간단한 클래스 하나와 `private` 메소드를 선언합니다.
- 예시를 위해 파라미터를 받고 반환값이 존재하는 `add` 메소드와 값을 받지도 반환하지도 않는 `print` 메소드를 선언합니다.

<br>

# 3. Private Method

```java
class CalculatorTest {

    @Test
    void addTest() throws Exception {
        Calculator calculator = new Calculator("");

        // private method 를 가져옴
        Method method = calculator.getClass().getDeclaredMethod("add", int.class, int.class);
        method.setAccessible(true);

        // add 메소드 실행
        int result = (int) method.invoke(calculator, 1, 2);
        assertThat(result).isEqualTo(3);
    }


    @Test
    void printTest() throws Exception {
        Calculator calculator = new Calculator("");

        // private method 를 가져옴
        Method method = calculator.getClass().getDeclaredMethod("print");
        method.setAccessible(true);

        // print 메소드 실행
        method.invoke(calculator);
    }
}
```

- 코드만 보면 쉽게 이해할 수 있습니다.
- `java.lang.reflect.Method` 를 사용해서 원하는 메소드를 가져옵니다.
- `getDeclaredMethod` 에는 (메소드 이름, 파라미터들...) 형식으로 값을 넣어주면 됩니다.
- `print` 메소드는 아무런 파라미터를 받지 않기 때문에 좀더 심플합니다.

<br>

# 4. Private Field

```java
class CalculatorTest {

    @Test
    void ownerTest() throws Exception {
        Calculator calculator = new Calculator("My Name");

        // private field 를 가져옴
        Field field = calculator.getClass().getDeclaredField("owner");
        field.setAccessible(true);

        // owner 값 꺼내기
        String owner = (String) field.get(calculator);
        assertThat(owner).isEqualTo("My Name");
    }
}
```

- method 와는 다르게 `java.lang.reflect.Field` 를 사용합니다.
- `invoke` 대신 `get` 으로 값을 꺼내는 것 외에는 큰 차이점은 없습니다.

<br>

# 5. Conclusion

이제 우리는 `private` 로 선언된 값이나 변수도 테스트 가능합니다.

전체 테스트 코드는 [Github](https://github.com/ParkJiwoon/practice-codes/blob/master/spring-test/src/test/java/com/practice/springtest/privat/CalculatorTest.java) 에서 확인할 수 있습니다.