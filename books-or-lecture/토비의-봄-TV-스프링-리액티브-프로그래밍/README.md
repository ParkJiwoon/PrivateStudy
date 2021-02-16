토비님의 스프링 리액티브 프로그래밍 유튜브 라이브를 보고 간단히 내용을 기록합니다.

16년 12월 영상으로 글 작성일 기준으로 4년 전 영상이지만 리액티브와 웹플럭스의 기초를 이해하는데 큰 도움이 됩니다.

<br>

- [1. Reactive Streams](./1-reactive-streams.md)
- [4. 자바와 스프링의 비동기 기술](./4-자바와-스프링의-비동기-기술.md)
- [5. 비동기 RestTemplate 과 비동기 MVC/Servlet](./5-비동기-resttemplate-과-비동기-mvcservlet.md)
- [6. AsyncRestTemplate 의 콜백 헬과 중복 작업 문제](./6-asyncresttemplate-의-콜백-헬과-중복-작업-문제.md)
- [7. CompletableFuture](./7-completablefuture.md)
- [8. WebFlux](#8-webflux)


# 8. WebFlux

- [Youtube 링크](https://www.youtube.com/watch?v=ScH7NZU_zvk)

<br>

Spring 5 부터는 기존의 서블릿 API 기반과 다르게 리액티브 프로그래밍을 활용할 수 있는 WebFlux 라는 환경을 지원합니다.

WebFlux 는 Tomcat 과 같은 서블릿 컨테이너에 의존적이지 않습니다.

물론 Tomcat 에서도 띄울수 있긴 합니다.

Spring 에는 디폴트 컨테이너가 세팅이 안되어 있어서 명시적으로 설정해줘야 하지만 Spring Boot 에서는 별다른 설정이 없다면 내장 컨테이너로 Netty 가 들어갑니다.

WebFlux 의 모든 응답은 Mono 또는 Flux 로 감싼걸 기본으로 합니다.

<br>

# WebFlux Example

아래는 Spring Boot WebFlux 를 활용해서 만들어본 컨트롤러 예제입니다.

```java
@RestController
public static class MyController {
	static final String URL = "http://localhost:8080/rest?idx={idx}";

	WebClient client = WebClient.create();

	@GetMapping("/rest")
	public Mono<String> rest(int idx) {
		return Mono.just("Rest Get " + idx);
	}

	@GetMapping("/")
	public Mono<String> index(int idx) {
		Mono<ClientResponse> res = client.get().uri(URL, idx).exchange();
		return Mono.just("Hello");
	}
}
```

- WebClient
    - RestTemplate 을 대신 Spring 5 부터 지원하는 외부 API 호출 라이브러리입니다.
    - `get()` 은 HTTP GET 요청을 한다는 뜻입니다. POST, PUT 등 다른 메소드도 존재합니다.
    - `uri()` 에 요청 URL 과 파라미터를 넣습니다.
    - `exchange()` 로 실제 요청을 보냅니다. 최신 버전에서는 Deprecated 되어 `exchangeToMono()` 등을 사용해야 합니다.

<br>

위 예제를 실행하고 브라우저에서 `http://localhost:8080?idx=3` 으로 접속하면 Hello 라는 메세지가 뜹니다.

`/` path 에 접속하면 `/rest` 를 호출하고 결과값으로는 Hello 를 리턴하도록 코드를 짰습니다.

그렇다면 정말  `/rest` 에 HTTP 호출이 갈까요?

<br>

`/rest` 는 호출되지 않습니다.

위 코드는 호출한다는 사실만을 `Mono<ClientResponse>` 에 담아 두는 것이고 실제로 수행되지는 않습니다.

`Mono` 가 상속받는 `Publisher` 는 기본적으로 실제로 `Subscriber` 가 `subscribe` 하지 않으면 수행되지 않습니다.

<br>

그럼 응답값으로 리턴하는 `Mono<String>` 은 어떻게 Hello 를 전달하는 걸까요?

Spring WebFlux 가 subscribe 를 해주기 때문에 신경쓸 필요가 없습니다.
