# Spring Request DTO 에 null 값이 들어가는 이유 (Jackson, Lombok)

# Overview

Spring Boot 로 REST API 를 테스트 하다가 이상한 이슈에 직면했습니다.

클라이언트에서 `@RequestBody` 로 요청을 받기 위한 DTO 클래스를 만들고 값을 입력 받았는데 null 값이 입력되는 겁니다.

처음에는 오타가 있거나 잘못 만든 건 줄 알았는데 변수명을 바꾸니 잘 동작했습니다.

예를 들어, 변수명이 `aCount` 일 때는 동작하지 않았는데 `aaCount` 로 바꾸니 제대로 값이 들어왔습니다.

그래서 이것저것 바꿔가면서 테스트를 하였고 Jackson, Lombok 에 대해서 알게 된 사실을 정리합니다.

쓰다보니 글이 장문이 되었는데 결론과 해결법만 알고 싶으면 마지막만 보면 됩니다.

<br>

# 1. Jackson

Spring 은 JSON 데이터를 매핑하기 위한 Message Converter 로 Jackson 을 사용합니다.

([Http Message Converters with the Spring Framework - Baeldung](https://www.baeldung.com/spring-httpmessageconverter-rest) 참고)

위에서 제시한 문제의 원인은 Lombok 이었지만 Jackson 의 JsonMessageConverter 의 동작에도 원인이 숨겨져 있습니다.

이를 확인하기 위해서는 Jackson 의 `DTO <-> Json` 과정이 어떻게 이루어지는 지 먼저 파악이 필요합니다.

<br>

## 1.1. Jackson 은 Getter 의 이름을 기반으로 Json Key 값을 만든다

Jackson 에는 한 가지 재미있는 사실이 있습니다.

`Object -> Json` 으로 변환 하면 해당 `Object` 의 필드명을 기준으로 될거라고 생각했는데 사실 **`Getter`의 이름 기준**으로 바뀝니다.

<br>

```java
public class JacksonDto {
    private String name;

    public String getNameChange() {
        return name;
    }
}
```

- 필드명은 `name` 이지만 Getter 이름은 `getNameChange()` 입니다.

<br>

```java
public class DtoTest {

    private static final ObjectMapper objectMapper = new ObjectMapper();

    @Test
    void test_jackson_dto() throws Exception {
        JacksonDto jacksonDto = new JacksonDto("my name");
        String content = objectMapper.writeValueAsString(jacksonDto);

        // 출력 = Jackson : {"nameChange":"my name"}
        System.out.println("Jackson : " + content);
    }
}
```

- 필드명 대신 Getter 의 이름으로 Json Key 값이 설정되었습니다.
- Getter 의 이름은 당연하게도 필드명과 동일하게 지어와서 지금까지 눈치 채지 못했습니다.

<br>

## 1.2. Jackson 이 Json Key 이름을 변환하는 데는 일정한 규칙이 있다

Object 의 필드명을 Getter 로 바꿀 때 일반적으로 맨 앞 글자를 대문자로 바꿔줍니다.

ex) `name` -> `getName()`

Jackson 은 Getter 를 기준으로 변환시키기 때문에 Jackson 내부적으로도 나름의 기준을 갖고 변홥합니다.

기본적으로는 JavaBeans 규약을 따르지만 다른 부분이 있었습니다.

먼저 JavaBeans 규약을 먼저 알아봅니다.

<br>

# 2. JavaBeans 규약

JavaBeans 는 메서드 이름에서 필드명을 추출할 때 일정한 규칙이 존재합니다.

[stack overflow 의 Naming convention for getters/setters in Java](https://stackoverflow.com/questions/2948083/naming-convention-for-getters-setters-in-java) 의 답변을 보면 Java Bean 규약을 첨부한 답변이 있습니다.

여기서 `8.8 Capitalization of inferred names` 챕터를 보면 아래와 같습니다.

<br>

> When we use design patterns to infer a property or event name, we need to decide what rules to follow for capitalizing the inferred name. 
> If we extract the name from the middle of a normal mixedCase style Java name then the name will, by default, begin with a capital letter. 
> Java programmers are accustomed to having normal identifiers start with lower case letters. 
> Vigorous reviewer input has convinced us that we should follow this same conventional rule for property and event names.
> 
> Thus when we extract a property or event name from the middle of an existing Java name, we normally convert the first character to lower case.
> However to support the occasional use of all upper-case names, we check if the first two characters of the name are both upper case and if
> so leave it alone. So for example,
> 
> “FooBah” becomes “fooBah”
> “Z” becomes “z”
> “URL” becomes “URL”
> 
> We provide a method Introspector.decapitalize which implements this conversion rule.

<br>

간단히 요약하면 클래스의 이름은 일반적으로 대문자로 시작하지만, 개발자들은 식별자가 소문자로 시작하는 것에 익숙하기 때문에 첫 번째 글자를 소문자로 변환한다는 겁니다.

다만, 모든 문자를 대문자로 사용하는 경우도 있기 때문에 이런 경우는 예외로 둔다고 합니다.

그리고 예외 케이스를 판별하기 위해 첫 두 문자가 모두 대문자인지를 확인합니다.

그리고 `java.beans` 패키지에 있는 `Introspector` 클래스를 확인해보면 실제로 어떤 로직이 들어가있는 지 알 수 있습니다.

<br>

```java
public class Introspector {
    // ...

    public static String decapitalize(String name) {
        if (name == null || name.length() == 0) {
            return name;
        }
        if (name.length() > 1 && Character.isUpperCase(name.charAt(1)) &&
                        Character.isUpperCase(name.charAt(0))){
            return name;
        }
        char chars[] = name.toCharArray();
        chars[0] = Character.toLowerCase(chars[0]);
        return new String(chars);
    }

    // ...
}
```

- 맨 앞 두개가 전부 대문자라면 그대로 리턴하고 아니라면 맨 앞 문자 하나만 소문자로 바꿔서 리턴합니다.

<br>

# 3. 그렇다면 Jackson 에서는?

Jackson 도 JavaBeans 규약을 따르지만 다른 점이 하나 있습니다.

테스트로 알아본 Jackson 의 규칙은 다음과 같습니다.

1. 맨 앞 두 글자가 모두 대문자인 경우 이어진 대문자를 모두 소문자로 변경한다.
2. 나머지 모든 케이스에서는 맨 앞 글자만 소문자로 바꿔준다.

<br>

JavaBeans 규약과 다른 부분은 1 번입니다.

JavaBeans 규약에서는 앞 두 글자가 대문자인 경우 그대로 사용한다고 했으나 Jackson 은 맨 앞부터 이어진 대문자를 모두 소문자로 변경합니다.

예제를 통해서 확인해보겠습니다.

<br>

## 3.1. 맨 앞 두 글자가 모두 대문자인 경우 이어진 대문자를 모두 소문자로 변경한다.

사실 JavaBeans 규약과 다른 게 이 부분입니다.

Jackson 에서는 맨 앞 두글자가 대문자라면 이어진 모든 대문자를 소문자로 변경합니다.

- `AAaa -> aaaa` : 앞 두 글자가 대문자라서 소문자로 변경
- `BBBb -> bbbb` : 앞 두 글자가 대문자라서 이어진 세번째 문자까지 소문자로 변경
- `CCcC -> cccC` : 앞 두 글자를 소문자로 변경하지만 맨 뒤의 대문자는 이어져 있지 않아서 그대로 사용
- `DDDD -> dddd` : 앞 두 글자부터 이어진 대문자를 모두 소문자로 변경 

<br>

### 3.1.1. DTO 정의

```java
@ToString
@NoArgsConstructor
public class OneDto {
    private String AAaa;
    private String BBBb;
    private String CCcC;
    private String DDDD;

    public String getAAaa() {
        return AAaa;
    }

    public String getBBBb() {
        return BBBb;
    }

    public String getCCcC() {
        return CCcC;
    }

    public String getDDDD() {
        return DDDD;
    }
}
```

<br>

### 3.1.2. Controller 작성

```java
@RestController
public class HelloController {

    @PostMapping("/one")
    public ResponseEntity<OneDto> postOne(@RequestBody OneDto dto) {
        System.out.println("----- Request POST /one ------");
        System.out.println(dto);
        
        return ResponseEntity.ok(dto);
    }
}
```

- 실제로 요청이 왔을 때 값이 어떻게 들어오는 지 확인합니다.
- 받은 `@RequestBody` 값을 그대로 다시 Response 로 내려줍니다.

<br>

### 3.1.3. Request

```json
POST http://localhost:8080/one
Content-Type: application/json

{
  "AAaa": "a",
  "BBBb": "b",
  "CCcC": "c",
  "DDDD": "d"
}
```

- IntelliJ 에서 제공하는 http request tool 을 사용했습니다.

<br>

### 3.1.4. Log

```java
----- Request POST /one ------
OneDto(AAaa=null, BBBb=null, CCcC=null, DDDD=null)
```

- Controller 에서 찍어둔 print 입니다.
- 값이 전부 null 로 들어옵니다.

<br>

### 3.1.5. Response

```json
{
  "aaaa": null,
  "bbbb": null,
  "cccC": null,
  "dddd": null
}
```

- 예측한 대로 나오는 걸 확인할 수 있습니다.
- 요청으로 들어온 `OneDto` 값을 그대로 리턴했을 뿐인데 Message Converter 에 의해 요청값과 응답값의 Json Key 값이 바꼈습니다.

<br>

## 3.2. 맨 앞 두글자가 대문자가 아니면 맨 앞 글자만 소문자로 바꿔준다

이거는 그냥 단순하게 1 번을 제외한 모든 케이스에서는 맨 앞글자만 소문자로 바꿔줍니다.

뒤에 오는 대문자나 소문자는 신경쓰지 않습니다.

<br>

### 3.2.1. DTO 정의

```java
@NoArgsConstructor
public class TwoDto {
    private String aaaa;
    private String bbbB;

    private String Cccc;
    private String DddD;

    private String eEee;
    private String fFfF;

    public String getAaaa() {
        return aaaa;
    }

    public String getBbbB() {
        return bbbB;
    }

    public String getCccc() {
        return Cccc;
    }

    public String getDddD() {
        return DddD;
    }

    public String geteEee() {
        return eEee;
    }

    public String getfFfF() {
        return fFfF;
    }
}
```

- DTO 를 정의하구 Controller 코드는 `OneDto` 와 동일하게 실행합니다.

<br>

### 3.2.2. Request

```json
POST http://localhost:8080/two
Content-Type: application/json

{
  "aaaa": "a",
  "bbbB": "b",
  "Cccc": "c",
  "DddD": "d",
  "eEee": "e",
  "fFfF": "f"
}
```

<br>

### 3.2.3. Log

```java
----- Request POST /two ------
TwoDto(aaaa=a, bbbB=b, Cccc=null, DddD=null, eEee=e, fFfF=f)
```

- `Cccc`, `DddD` 를 제외한 나머지는 전부 값이 제대로 들어옵니다.

<br>

### 3.2.4. Response

```json
{
  "aaaa": "a",
  "bbbB": "b",
  "cccc": null,
  "dddD": null,
  "eEee": "e",
  "fFfF": "f"
}
```

- 예측한 대로 잘 나옵니다.
- 맨 앞 글자가 대문자였던 `Cccc` 와 `DddD` 만 바뀌고 나머지는 그대로입니다.
- 중요하게 볼 점은 `TwoDto` 의 필드명과 달라진 애들은 값이 제대로 들어오지 않는다는 사실입니다.

<br>

## 3.3. Jackson 결론

우리는 지금까지의 테스트를 통해서 한 가지 사실을 알았습니다.

**DTO 의 필드명이 대문자로 시작하면 Request 요청 시 값이 제대로 들어오지 않습니다.**

필드명이 대문자로 시작하면 Getter 도 대문자로 시작하는 수밖에 없습니다.

**Jackson 의 규칙에 따라서 `get` 이후가 대문자로 시작하면 최소한 첫 글자는 항상 소문자로 바뀝니다.**

따라서 필드명과 일치하지 않아 데이터가 들어가지 않는 현상입니다.

필드명을 대문자로 시작하는 경우는 많이 없지만 `URL` 처럼 모두 대문자로 사용했다가 안될 가능성도 있습니다.

<br>

# 4. Lombok 은 무슨 관계일까?

Lombok 은 개발자들이 일일히 만들어야 하는 반복적인 코드를 줄일수 있게 도와주는 라이브러리입니다.

그 중에서도 `@Getter` 어노테이션은 거의 모든 Object 에 필수적으로 사용됩니다.

제가 이슈를 겪었던 DTO 오브젝트도 롬복을 사용했습니다.

그렇다면 롬복의 문제점은 무엇일까요?

<br>

## 4.1. Lombok 의 Getter 생성 규칙

Lomobk 의 `@Getter` 어노테이션을 붙이면 클래스의 Getter 메소드를 자동으로 생성해줍니다.

그런데 `@Getter` 의 생성 규칙은 굉장히 단순합니다.

`get` 다음에 무조건 필드명의 맨 앞 글자를 대문자로 바꿔서 만들어줍니다.

[lombok 의 Github Issue](https://github.com/rzwitserloot/lombok/issues/2693) 에도 이 내용에 대한 문의가 있습니다.

제가 문제를 겪었던 필드명도 `aCount` 였습니다.

Lombok 이 `getACount` 로 생성해주고 Jackson 을 거치니 `acount` 가 되어서 필드명이 일치하지 않아 문제가 발생했었습니다.

반면 `aaCount` 는 `getAaCount` 가 되고 Jackson 을 거쳐도 `aaCount` 가 되어서 정상적으로 값이 들어오죠.

<br>

## 4.2. 인텔리제이 Generator 의 Getter 생성 규칙

```java
public class CountDto {
    private int aCount;

    public int getaCount() {
        return aCount;
    }
}
```

Lombok 대신 인텔리제이에서 제공하는 제네레이터로 Getter 를 만들면 위 이슈를 회피할 수 있습니다.

`getACount` 대신에 `getaCount` 로 만들어주기 때문에 Jackson 을 거쳐도 `aCount` 라는 필드명과 일치합니다.

<br>

# Conclusion

지금까지 정리한 내용을 요약하면 아래와 같습니다.

1. Spring 의 Json Message Converter 는 Jackson 라이브러리를 사용
2. lombok 의 Getter 는 필드명 맨 앞을 항상 대문자로 만듬
3. Jackson 라이브러리는 Getter 의 맨 앞 두글자가 전부 대문자인 경우 필드명과 Json key 값이 달라짐
4. `aCount` 라는 필드명을 lombok 을 사용해서 Getter 를 만들면 `getACount()` 가 되기 때문에 이슈가 발생

<br>

위 문제를 해결하려면 필드명을 작성할 때 첫 번째는 소문자, 두 번째는 대문자인 케이스로 만들지 않으면 됩니다.

그래도 꼭 사용해야 한다면 lombok 의 `@Getter` 대신 직접 Getter 를 만들거나, `@JsonProperty` 를 사용하면 됩니다.

<br>

# Reference

- [lombok github issue](https://github.com/rzwitserloot/lombok/issues/2693)
- [stack overflow - Why does Jackson 2 not recognize the first capital letter if the leading camel case word is only a single letter long?](https://stackoverflow.com/questions/30205006/why-does-jackson-2-not-recognize-the-first-capital-letter-if-the-leading-camel-c)
- [Http Message Converters with the Spring Framework - Baeldung](https://www.baeldung.com/spring-httpmessageconverter-rest)
- [stack overflow - Naming convention for getters/setters in Java](https://stackoverflow.com/questions/2948083/naming-convention-for-getters-setters-in-java) 
