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
