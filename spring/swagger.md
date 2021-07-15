# Spring Boot Swagger 3.x 적용

# 1. Swagger 란?

Swagger 는 OAS(Open Api Specification)를 위한 프레임워크입니다.

개발자들의 필수 과제인 API 문서화를 쉽게 할 수 있도록 도와주며, 파라미터를 넣어서 실제로 어떤 응답이 오는지 테스트도 할 수 있습니다.

또한, 협업하는 클라이언트 개발자들에게도 Swagger 만 전달해주면 API Path 와 Request, Response 값 및 제약 등을 한번에 알려줄 수 있습니다.

<br>

## 1.1. OpenAPI 와의 관계

[Swagger 공식 블로그 포스팅](https://swagger.io/blog/api-strategy/difference-between-swagger-and-openapi/)을 보면 Swagger 와 OpenAPI 의 차이가 나와있습니다.

**OpenAPI는 RESTful API 설계를 위한 업계 표준 사양을 나타내고 Swagger는 SmartBear 도구 세트를 나타냅니다**

간단히 말하면 Swagger 는 OpenAPI 사양을 구현하는 도구들로 Swagger Editor, Swagger UI, SwaggerHub 등이 존재하고 단순히 브랜드명만 변경하지 않은거라고 합니다.

<br>

# 2. 적용

Swagger 3.x 버전을 적용합니다.

<br>

## 2.1. 의존성 추가

```java
// build.gradle
dependencies {
    // ..
    implementation 'io.springfox:springfox-boot-starter:3.0.0'
    // ..
}
```

- Gradle 을 사용하는 경우엔 `build.gradle` 에 추가합니다.
- 사용할 버전은 https://mvnrepository.com/artifact/io.springfox/springfox-boot-starter 여기에서 확인 할 수 있습니다.
- Swagger 2.x 버전과 다르게 `springfox-boot-starter` 하나만 추가하면 하위에 필요한 모든 라이브러리가 포함되어 있습니다.

<br>

## 2.2. Config 추가

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.OAS_30)
                .useDefaultResponseMessages(false)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.login.controller"))
                .paths(PathSelectors.any())
                .build()
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Practice Login")
                .description("practice login api")
                .version("1.0")
                .build();
    }
}
```

- `Docket`
  - Swagger 설정의 핵심이 되는 Bean
- `useDefaultResponseMessages`
  - Swagger 에서 제공해주는 기본 응답 코드 (200, 401, 403, 404)
  - `false` 로 설정하면 기본 응답 코드를 노출하지 않음
- `apis`
  - api 스펙이 작성되어 있는 패키지 (Controller) 를 지정
- `paths`
  - `apis` 에 있는 API 중 특정 path 를 선택
- `apiInfo`
  - Swagger UI 로 노출할 정보

<br>

## 2.3. 실행 후 접속

로컬 실행 후 URL 접속하면 되는데 Swagger 2.x 버전과 조금 다릅니다.

- Swagger2: http://localhost:8080/swagger-ui.html
- Swagger3: http://localhost:8080/swagger-ui/index.html
