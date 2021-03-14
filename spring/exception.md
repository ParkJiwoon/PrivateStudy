# 1. Java Exception

Java 에는 `Checked Exception` 과 `Unchecked Exception` 이 존재합니다.

이 둘은 헷갈리기 쉽지만 사실 큰 차이가 존재합니다.

<br>

## 1.1. 예외 처리 필수

checked 와 unchecked 를 나누는 가장 큰 기준입니다.

`Checked Exception` 은 직접 예외 처리를 하던지 상위 메소드로 넘기던지 **반드시 예외를 처리**해줘야 합니다.

`Checked Exception` 의 한 종류인 `IOException` 를 예로 들어보겠습니다.

```java
public void needTryCatch() {
  try {
    // ...
  } catch (IOException e) {
    // ..
  }
}
```

- `try catch` 같은걸로 반드시 예외를 잡아서 처리를 해줘야 함

<br>

```java
public void needThrow() throws IOException {
  // ...
}
```

- 만약 직접 처리하기 싫다면 메소드를 정의할 때 뒤에 `thorws Exception` 으로 상위 메소드에 넘겨서 처리하게 만듬

<br>

## 1.2. Transaction Rollback

DB 처리 도중 `RuntimeException` 이 발생하면 **롤백이 진행**되지만 `Checked Exception` 은 발생해도 데이터가 **롤백되지 않고 커밋까지 완료**됩니다.

만약 예외 발생 시 롤백을 진행하고 싶다면 `try catch` 로 잡아서 `Uncheked Exception` 을 던져줘야 합니다.

<br>

## 1.3. 검증 단계

checked 는 **컴파일 단계**에서 Exception 체크가 가능합니다.

모든 unchecked exception 은 `RuntimeException` 을 상속받습니다.

unchecked 는 `RuntimeException` 이라는 이름에서도 알 수 있듯이 **런타임 단계**에서 발견되며, 어떤 예외가 발생할지 개발자가 미리 예측하기 힘듭니다.

<br>

# 2. Spring Exception 의 HTTP Status

Spring 에는 HTTP Status 응답 처리를 위한 여러가지 방법이 있습니다.

<br>

## 2.1. @ResponseStatus

```java
@ResponseStatus(code = HttpStatus.NOT_FOUND, reason = "Data Not Found")
public class DataNotFoundException extends RuntimeException {
}
```

Spring 3 부터는 HTTP Status 와 Response 를 제공하는 `@ResponseStatus` 어노테이션이 생겼습니다.

개발자가 정의한 Exception 이 발생하면 해당 Status 와 Message 를 전달합니다.

테스트를 위해 Controller 에서 강제로 Exception 을 발생 시켜보겠습니다.

<br>

### 2.1.1. @ResponseStatus 없이 그냥 NotFoundException("No Data")

```json
{
  "timestamp": "2021-03-13T07:23:00.732+00:00",
  "status": 500,
  "error": "Internal Server Error",
  "message": "No Data",
  "path": "/auth/signup"
}
```

<br>

### 2.1.2. @ResponseStatus 정의된 Custom Exception 사용

```json
{
  "timestamp": "2021-03-13T07:30:38.299+00:00",
  "status": 404,
  "error": "Not Found",
  "message": "Data Not Found",
  "path": "/auth/signup"
}
```

<br>

### 2.1.3. 한계점

`@ResponseStatus` 에 정의한 대로 잘 나오는 걸 확인할 수 있습니다.

이 방법은 별다른 설정 없이 어노테이션 추가만으로 간단하게 Custom Exception 을 만들 수 있지만, 한 가지 단점이 있습니다.

위에서 예시로 데이터가 없는 경우 `DataNotFoundException` 를 리턴하게 했는데, 이 Exception 은 항상 동일한 HTTP Status 와 Message 를 리턴합니다.

같은 Exception 이 발생하는 상황이더라도 다른 Message 를 보내는게 불가능합니다.

<br>

## 2.2. ResponseStatusException

`ResponseStatusException` 은 `@ResponseStatus` 의 대체제로 Spring 5 에 등장했습니다.

RuntimeException 을 상속하며 마찬가지로 HTTP Status 와 Message 를 설정할 수 있습니다.

```java
public ResponseStatusException(HttpStatus status, @Nullable String reason, @Nullable Throwable cause) {
}
```

- status: HTTP Status
- reason: HTTP response Message
- cause: ResponseStatusException 을 발생시킨 Exception

<br>

스프링에서는 `HandlerExceptionResolver` 가 모든 exception 을 가로채서 처리합니다.

이 중에서 `ResponseStatusExceptionResolver` 라는 클래스가 `ResponseStatusException` 또는 `@ResponseStatus` 어노테이션이 붙은 Exception 을 찾아서 처리해줍니다.

<br>

### 2.2.1. 장점

- 비슷한 유형의 예외를 별도로 처리할 수 있고, 응답마다 다른 상태 코드를 세팅 가능합니다.
- 불필요한 Exception 클래스 생성을 피할 수 있습니다.
- Exception 처리를 추가적인 어노테이션 없이 코드 단에서 자연스럽게 처리할 수 있습니다.

<br>

# 3. Spring 전역으로 공통 Exception 처리하기

2 번에서 Exception 에 따라 HTTP Status 제공하는 여러 가지 방법을 알아봤는데, 토이 프로젝트를 하면서 만들어보니 위 설정대로 쓸 일이 없었습니다.

<br>

```java
throw new ResponseStatusException(HttpStatus.NOT_FOUND, "No Data");
```

- Spring 5 부터 제공하는 `ResponseStatusException` 의 일반적인 사용입니다.
- 생성자로 HTTP Status 와 String 만을 받습니다.
- 만약 없는 데이터의 종류가 다르다면 ?? `No Member Data`, `No Profile Data` 등등.. 이렇게 표현해야 합니다.
- 하지만 이렇게 명확한 규격 없이 String 으로만 받으면 여러 사람들이 작업할 때 중복된 응답을 주거나 이미 있는 응답을 새로 만들거나, 오타로 인해 실수할 가능성도 있습니다.

<br>

위와 같은 이유로 Spring Exception 처리는 다른 방법으로 하게 되었습니다.

앞으로 사용할 클래스들입니다.

- `ErrorCode` : 핵심. 모든 예외 케이스를 이곳에서 관리함
- `CustomException` : 기본적으로 제공되는 Exception 외에 사용
- `ErrorResponse` : 사용자에게 JSON 형식으로 보여주기 위해 에러 응답 형식 지정
- `GlobalExceptionHandler` : Custom Exception Handler
    - `@ControllerAdvice` : 프로젝트 전역에서 발생하는 Exception 을 잡기 위한 클래스
    - `@ExceptionHandler` : 특정 Exception 을 지정해서 별도로 처리해줌

<br>

## 3.1. Error 관련 properties 설정

단순한 설정입니다.

작성하지 않아도 상관 없으며 필요한 경우에 참고해서 설정하시면 됩니다.

```yaml
# application.yml
server:
  error:
    include-exception: false      # Response 에 Exception 을 표시할지
    include-message: always       # Response 에 Exception Message 를 표시할지 (never | always | on_param)
    include-stacktrace: on_param  # Response 에 Stack Trace 를 표시할지 (never | always | on_param) on_trace_params 은 deprecated
    whitelabel.enabled: true      # 에러 발생 시 Spring 기본 에러 페이지 노출 여부 
```

<br>

## 3.2. ErrorCode

```java
@Getter
@AllArgsConstructor
public enum ErrorCode {

    /* 400 BAD_REQUEST : 잘못된 요청 */
    INVALID_REFRESH_TOKEN(BAD_REQUEST, "리프레시 토큰이 유효하지 않습니다"),
    MISMATCH_REFRESH_TOKEN(BAD_REQUEST, "리프레시 토큰의 유저 정보가 일치하지 않습니다"),
    CANNOT_FOLLOW_MYSELF(BAD_REQUEST, "자기 자신은 팔로우 할 수 없습니다"),

    /* 401 UNAUTHORIZED : 인증되지 않은 사용자 */
    INVALID_AUTH_TOKEN(UNAUTHORIZED, "권한 정보가 없는 토큰입니다"),
    UNAUTHORIZED_MEMBER(UNAUTHORIZED, "현재 내 계정 정보가 존재하지 않습니다"),

    /* 404 NOT_FOUND : Resource 를 찾을 수 없음 */
    MEMBER_NOT_FOUND(NOT_FOUND, "해당 유저 정보를 찾을 수 없습니다"),
    REFRESH_TOKEN_NOT_FOUND(NOT_FOUND, "로그아웃 된 사용자입니다"),
    NOT_FOLLOW(NOT_FOUND, "팔로우 중이지 않습니다"),

    /* 409 CONFLICT : Resource 의 현재 상태와 충돌. 보통 중복된 데이터 존재 */
    DUPLICATE_RESOURCE(CONFLICT, "데이터가 이미 존재합니다"),

    ;

    private final HttpStatus httpStatus;
    private final String detail;
}
```

- 에러 형식을 Enum 클래스로 정의합니다.
- 응답으로 내보낼 HttpStatus 와 에러 메세지로 사용할 String 을 갖고 있습니다.
- `ResponseStatusException` 과 비슷해 보입니다. 하지만 가장 큰 차이점은 개발자가 정의한 새로운 Exception 을 모두 한 곳에서 관리하고 재사용 할 수 있다는 점입니다.

<br>

## 3.3. CustomException

```java
@Getter
@AllArgsConstructor
public class CustomException extends RuntimeException {
    private final ErrorCode errorCode;
}
```

- 전역으로 사용할 `CustomException` 입니다.
- `RuntimeException` 을 상속받아서 Unchecked Exception 으로 활용합니다.
- 생성자로 `ErrorCode` 를 받습니다.

<br>

## 3.4. ErrorResponse

```java
@Getter
@Builder
public class ErrorResponse {
    private final LocalDateTime timestamp = LocalDateTime.now();
    private final int status;
    private final String error;
    private final String code;
    private final String message;

    public static ResponseEntity<ErrorResponse> toResponseEntity(ErrorCode errorCode) {
        return ResponseEntity
                .status(errorCode.getHttpStatus())
                .body(ErrorResponse.builder()
                        .status(errorCode.getHttpStatus().value())
                        .error(errorCode.getHttpStatus().name())
                        .code(errorCode.name())
                        .message(errorCode.getDetail())
                        .build()
                );
    }
}
```

- 실제로 유저에게 보낼 응답 Format 입니다.
- 일부러 500 에러 났을 때랑 형식을 맞췄습니다. `status`, `code` 값은 사실 없어도 됩니다.
- `ErrorCode` 를 받아서 `ResponseEntity<ErrorResponse>` 로 변환해줍니다.

<br>

## 3.5. @ControllerAdvice and @ExceptionHandler

`@ControllerAdvice` 는 프로젝트 전역에서 발생하는 모든 예외를 잡아줍니다.

`@ExceptionHandler` 는 발생한 특정 예외를 잡아서 하나의 메소드에서 공통 처리해줄 수 있게 해줍니다.

따라서 둘을 같이 사용하면 모든 예외를 잡은 후에 Exception 종류별로 메소드를 공통 처리할 수 있습니다.

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value = { ConstraintViolationException.class, DataIntegrityViolationException.class})
    protected ResponseEntity<ErrorResponse> handleDataException() {
        log.error("handleDataException throw Exception : {}", DUPLICATE_RESOURCE);
        return ErrorResponse.toResponseEntity(DUPLICATE_RESOURCE);
    }

    @ExceptionHandler(value = { CustomException.class })
    protected ResponseEntity<ErrorResponse> handleCustomException(CustomException e) {
        log.error("handleCustomException throw CustomException : {}", e.getErrorCode());
        return ErrorResponse.toResponseEntity(e.getErrorCode());
    }
}
```

- View 를 사용하지 않고 Rest API 로만 사용할 때 쓸 수 있는 `@RestControllerAdvice` 를 사용합니다.
- `handleDataException` 메소드에서는 hibernate 관련 에러를 처리합니다.
- `handleCustomException` 메소드는 직접 정의한 `CustomException` 을 사용합니다.
- Exception 발생 시 넘겨받은 `ErrorCode` 를 사용해서 사용자에게 보여주는 에러 메세지를 정의합니다.

<br>

## 3.6. Exception 사용

```java
@RequiredArgsConstructor
@Service
public class MemberService {
    private final MemberRepository memberRepository;

    @Transactional
    public boolean follow(Long memberId) {
        Member currentMember = getCurrentMember();

        // 팔로우할 상대방 정보가 없는 경우
        Member targetMember = memberRepository.findById(memberId)
                .orElseThrow(() -> new CustomException(MEMBER_NOT_FOUND));

        // 자기 자신을 팔로우 하려는 경우
        if (currentMember.equals(targetMember))  {
            throw new CustomException(CANNOT_FOLLOW_MYSELF);
        }
				
				// code...
    }
}
```

- 토이 프로젝트의 일부 코드입니다.
- 상대방의 Member ID 를 입력 받아서 팔로우 하는 기능입니다.
- Exception 에 담겨지는 `ErrorCode` 만 보고도 어떤 종류의 문제가 발생한 건지 알 수 있습니다.
- 또한 불필요하게 여러 Exception 을 만들지 않고 `ErrorCode` 만 새로 추가하면 사용 가능합니다.

<br>

### 3.6.1. MEMBER_NOT_FOUND 실제 응답

```json
{
  "timestamp": "2021-03-14T03:29:01.878659",
  "status": 404,
  "error": "NOT_FOUND",
  "code": "MEMBER_NOT_FOUND",
  "message": "해당 유저 정보를 찾을 수 없습니다"
}
```

<br>

### 3.6.2. CANNOT_FOLLOW_MYSELF 실제 응답

```json
{
  "timestamp": "2021-03-14T03:16:25.98361",
  "status": 400,
  "error": "BAD_REQUEST",
  "code": "CANNOT_FOLLOW_MYSELF",
  "message": "자기 자신은 팔로우 할 수 없습니다"
}
```

<br>

## 3.7. 결론

Spring 에는 프로젝트 전역에서 발생하는 Exception 을 한 곳에서 처리할 수 있다.

Enum 클래스로 `ErrorCode` 를 정의하면 Exception 클래스를 매번 생성하지 않아도 된다.

실제 클라에게 날라가는 응답에서 `code` 부분만 확인하면 어떤 에러가 발생했는지 쉽게 파악 가능하다.

<br>

# Reference

- [Error Handling for REST with Spring (Baeldung)](https://www.baeldung.com/exception-handling-for-rest-with-spring)
- [Spring ResponseStatusException (Baeldung)](https://www.baeldung.com/spring-response-status-exception)
- [Error handling for a Spring-based REST API (Microflash)](https://mflash.dev/blog/2020/07/26/error-handling-for-a-spring-based-rest-api/)
