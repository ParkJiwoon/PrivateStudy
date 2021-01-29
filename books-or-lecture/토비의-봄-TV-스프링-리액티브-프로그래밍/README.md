토비님의 스프링 리액티브 프로그래밍 유튜브 라이브를 보고 간단히 내용을 기록합니다.

16년 12월 영상으로 글 작성 시점에서도 4년이나 지났지만 리액티브와 웹플럭스의 기초를 이해하는데 큰 도움이 됩니다.

<br>

# 4. 자바와 스프링의 비동기 기술

- Yotube: [https://www.youtube.com/watch?v=aSTuQiPB4Ns](https://www.youtube.com/watch?v=aSTuQiPB4Ns)

<br>

# Java 의 비동기 처리

과거의 Java 에서는 비동기 작업의 결과를 처리하는 방법으로 Callback 과 Future 라는 두가지가 존재합니다. 

Callback 은 JS 를 다룬 사람들에게는 익숙한 방법입니다.

비동기 작업을 할 때 결과값을 다루는 메소드까지 통째로 넘겨서 처리하게 하는 방법이죠.

영상에서는 Callback 으로 비동기 작업 성공, 실패 코드까지 직접 짜셨는데 여기서는 `Future` 에 대해서만 간략히 기록합니다.

<br>

# Future

- Java 1.5 에서 나옴
- 비동기적인 작업을 수행하고 난 결과를 갖고 있는 것
- 비동기 작업을 한다는 건 현재 쓰레드가 아닌 새로운 쓰레드를 열어서 작업을 수행하는 것
- 다른 쓰레드에서 마친 작업의 결과를 일반적인 방법으로 가져올 수 없는데 그걸 가능하게 하는게 `Future`
- `Future.get()` 메소드로 결과값을 가져올 수 있는 대신 비동기 작업이 끝날 때까지 해당 쓰레드를 블록
- `Future.isDone()` 메소드로 작업의 완료 여부를 확인할 수 있음

<br>

## (1) 블로킹 단일 쓰레드에서의 실행

```java
Thread.sleep(2000);
log.info("Hello");

log.info("Exit");
```

읿반적인 단일 블로킹 쓰레드에서 위 코드를 실행하면 2 초 뒤 Hello, Exit 가 순차적으로 찍힙니다.

<br>

```java
// 같은 main 쓰레드에 찍힘

22:05:30.087 [main] INFO FutureEx - Hello
22:05:30.090 [main] INFO FutureEx - Exit
```

<br>

## (2) ExecutorService.execute 를 사용한 비동기 실행

```java
ExecutorService es = Executors.newCachedThreadPool();

es.execute(() -> {
    try {
        Thread.sleep(2000);
    } catch (InterruptedException ignored) { }

    log.info("Hello");
});
es.shutdown();

log.info("Exit");
```

`ExecutorService` 를 사용해서 비동기로 호출하면 Exit 가 먼저 찍히고 2 초 뒤에 Hello 가 다른 쓰레드에 찍힙니다.

<br>

```java
// Exit 가 먼저 찍히고 2초 뒤 다른 쓰레드에 Hello 가 찍힘

22:07:34.640 [main] INFO FutureEx - Exit
22:07:36.640 [pool-1-thread-1] INFO FutureEx - Hello
```

<br>

## (3) ExecutorService.submit 과 Future 를 사용해서 비동기 결과 가져오기

```java
ExecutorService es = Executors.newCachedThreadPool();

Future<String> f = es.submit(() -> {
    Thread.sleep(2000);
    log.info("Hello");
    return "Result";
});
es.shutdown();

log.info("Running");
log.info(f.get());
log.info("Exit");
```

`ExecutorService.submit` 은 파라미터로 `Callable` 을 받기 때문에 내부에 Exception 처리를 빼도 됩니다.

위 코드를 실행하면 어떻게 될까요?

`Future.get()` 메소드는 비동기 작업의 결과가 완료될 때까지 쓰레드를 블록시킵니다.

따라서 쓰레드가 블록 되기 전의 로그인 Running 은 바로 찍히지만, Exit 는 비동기 작업이 완료될 때까지 기다려야 하기 때문에 가장 마지막에 찍힙니다.

<br>

```java
// Future.get 메소드가 비동기 처리가 완료될 때까지 쓰레드를 블록시킴

22:39:21.076 [main] INFO FutureEx - Running
22:39:23.075 [pool-1-thread-1] FutureEx - Hello
22:39:23.075 [main] INFO FutureEx - Result
22:39:23.075 [main] INFO FutureEx - Exit
```

<br>

# Spring 의 비동기 처리

## @Async 사용

```java
public String hello() throws InterruptedException {
	Thread.sleep(2000);
	return "Hello";
}
```

위 코드와 같은 `hello()` 메소드가 있다고 가정합니다.

이 메소드는 보다싶이 2 초 동안 쓰레드를 블록 시킨 후에 값을 리턴합니다.

해당 메소드를 비동기로 만드려면 `@Async` 메소드만 붙여주고 값을 `Future` 로 감싸서 리턴해주면 됩니다.

(`AsyncResult` 는 `Future` 를 상속받는 구현체입니다)

그리고 컨테이너 위에 `@EnableAsync` 를 추가하면 비동기로 동작합니다.

<br>

```java
@Async
public Future<String> hello() throws InterruptedException {
	Thread.sleep(2000);
	return new AsyncResult<>("hello");
}

...

@SpringBootApplication
@EnableAsync // 이 어노테이션을 추가해야 @Async 가 제대로 동작함
public class PracticeWebfluxApplication {
	public static void main(String[] args) {
		SpringApplication.run(PracticeWebfluxApplication.class, args);
	}
}
```

위 코드를 실행해서 로그를 찍어보면 다른 쓰레드에 찍히는 걸 확인할 수 있습니다.

<br>

아무런 설정없이 `@Async` 를 사용하면 `SimpleAsyncTaskExecutor` 를 사용하는데 별로 좋지 않습니다.

비동기 요청이 들어온 만큼 쓰레드를 생성하는데 캐싱하지도 않고 따로 관리하지도 않아 메모리 낭비가 극심합니다.

<br>

따라서 ThreadPool 에 관한 설정을 추가해서 사용하는 것이 좋습니다.

아무런 설정이 없다면 위에 언급한 `SimpleAsyncTaskExecutor` 을 사용하고, `ExecutorService` 또는 `ThreadPoolTaskExecutor` 를 구현한 Bean 이 존재하면 해당 Executor 을 쓰게 되어있습니다.

만약 비동기 작업이 여러개고 쓰레드 풀을 분리해서 사용하고 싶다면 어노테이션 뒤에 명시적으로 지정할 수 있습니다.

```java
@Async("tp")
public Future<String> hello() throws InterruptedException {
	Thread.sleep(2000);
	return new AsyncResult<>("hello");
}

@Bean
ThreadPoolTaskExecutor tp() {
	ThreadPoolTaskExecutor te = new ThreadPoolTaskExecutor();
	te.setCorePoolSize(10);		  // 기본적으로 만들어 두는 쓰레드 갯수. 무조건 만드는 건 아니고 첫번째 쓰레드 요청이 오면 만듬
	te.setQueueCapacity(200); 	// CorePoolSize 가 꽉 찼을 때 요청이 들어오면 큐에 넣어둠
	te.setMaxPoolSize(100);		  // QueueCapacity 사이즈가 꽉 차면 쓰레드 MaxPoolSize 만큼 늘려줌
	te.setKeepAliveSeconds(60);	// CorePoolSize 를 초과해서 만들어졌다가 쓰레드 반환 후에 일정시간 이상 재할당이 안되면 제거하기 시작함. 불필요한 메모리 점유를 막음
	te.setTaskDecorator();		  // 쓰레드를 새로 만들거나 반환하는 시점 앞뒤에 콜백을 걸 수 있음 (로그용으로 사용 가능)
	te.setThreadNamePrefix("mythread");
	te.initialize();

	return te;
}
```

<br>

## Future 로 받아서 Callback 등록

Spring 4.0 부터는 `ListenableFuture` 을 지원합니다.

이 클래스는 `addCallback()` 메소드에 onSuccess 와 onError 를 넘겨줄 수 있습니다.

<br>

# 비동기 Servlet

Servlet 3.0 : 비동기 서블릿

- HTTP connectoin 은 이미 논블로킹 IO
- 서블릿 요청 읽기, 응답, 쓰기는 블로킹
- 비동기 작업 시작 즉시 서블릿 쓰레드 반납
- 비동기 작업이 완료되면 서블릿 쓰레드 재할당
- 비동기 서블릿 컨텍스트 이용 (AsyncContext)

Servlet 3.1 : 논블로킹 IO

- 논블로킹 서블릿 요청, 응답 처리
- Callback

<br>

쓰레드가 블록되는 상황은 컨텍스트 스위칭 때문에 CPU 가 메모리 자원을 많이 먹게 됩니다.

사용되지 않는 쓰레드를 대기 상태로 변환하고 다른 쓰레드를 하나 끌어와서 작업을 하는데 이걸 컨텍스트 스위칭 이라고 합니다.

<br>

## 비동기 서블릿을 활용한 요청/응답 처리

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/books-or-lecture/%ED%86%A0%EB%B9%84%EC%9D%98-%EB%B4%84-TV-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/images/toby-reactive-1.png?raw=true)

요청이 오면 응답을 줄 때까지 쓰레드를 점유하고 있던 기존 서블릿과 달리 비동기 서블릿은 요청이 오면 작업 쓰레드에 바로 넘기고 쓰레드를 반납했다가 작업 완료 시점에 다시 쓰레드를 할당 받아 응답을 줍니다.

이 구조는 서블릿 쓰레드를 오랫동안 점유하지 않기 때문에 적은 쓰레드로도 많은 요청을 처리할 수 있지만, 사실 함정이 있습니다.

결국 실제 로직을 처리하는 작업 쓰레드는 요청수만큼 할당 받아야 하기 때문에 메모리가 효율적이라고 할 수 없습니다.
