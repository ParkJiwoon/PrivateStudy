토비님의 스프링 리액티브 프로그래밍 유튜브 라이브를 보고 간단히 내용을 기록합니다.

16년 12월 영상으로 글 작성일 기준으로 4년 전 영상이지만 리액티브와 웹플럭스의 기초를 이해하는데 큰 도움이 됩니다.

<br>

# 4. 자바와 스프링의 비동기 기술

- [Yotube 링크](https://www.youtube.com/watch?v=aSTuQiPB4Ns)

<br>

이 장에서는 과거의 Java 와 Spring 이 어떤 방식으로 비동기 처리를 했는지 간단하게 내부 로직과 변화를 알아봅니다.

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

일반적인 단일 블로킹 쓰레드에서 위 코드를 실행하면 2 초 뒤 Hello, Exit 가 순차적으로 찍힙니다.

<br>

```java
22:05:30.087 [main] INFO FutureEx - Hello
22:05:30.090 [main] INFO FutureEx - Exit
```

로그가 같은 main 쓰레드에 찍히는 것을 확인할 수 있습니다.

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
22:07:34.640 [main] INFO FutureEx - Exit
22:07:36.640 [pool-1-thread-1] INFO FutureEx - Hello
```

Exit 가 먼저 찍히고 2초 뒤 다른 쓰레드에 Hello 가 찍힙니다.

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

비동기 작업의 실행 결과를 받으려면 `ExecutorService.submit` 을 사용하고 `Future` 로 받으면 됩니다.

`ExecutorService.submit` 은 파라미터로 `Callable` 을 받기 때문에 내부의 Exception 처리를 빼도 됩니다.

위 코드를 실행하면 어떻게 될까요?

<br>

```java
22:39:21.076 [main] INFO FutureEx - Running
22:39:23.075 [pool-1-thread-1] FutureEx - Hello
22:39:23.075 [main] INFO FutureEx - Result
22:39:23.075 [main] INFO FutureEx - Exit
```

`Future.get()` 메소드는 비동기 작업의 결과가 완료될 때까지 쓰레드를 블록시킵니다.

따라서 쓰레드가 블록 되기 전의 로그인 Running 은 바로 찍히지만, Exit 는 비동기 작업이 완료될 때까지 기다려야 하기 때문에 가장 마지막에 찍힙니다.

<br>

# Spring 의 비동기 처리

## (1) @Async 사용

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

아무런 설정없이 `@Async` 를 사용하면 `SimpleAsyncTaskExecutor` 를 사용하게 되기 때문에 실제 업무에서는 절대 하지 말아야 합니다.

비동기 요청이 들어온 만큼 쓰레드를 생성하는데 캐싱하지도 않고 따로 관리하지도 않아 메모리 낭비가 극심합니다.

<br>

따라서 항상 ThreadPool 에 관한 설정을 추가해서 사용하는 것이 좋습니다.

아무런 설정이 없다면 위에 언급한 `SimpleAsyncTaskExecutor` 을 사용하고, `ExecutorService` 또는 `ThreadPoolTaskExecutor` 를 구현한 Bean 이 존재하면 해당 Executor 을 쓰게 되어있습니다.

만약 비동기 작업이 여러개고 쓰레드 풀을 분리해서 사용하고 싶다면 어노테이션 뒤에 명시적으로 지정할 수 있습니다.

```java
@Async("tp")  // 명시적으로 tp Executor 를 사용
public Future<String> hello() throws InterruptedException {
    Thread.sleep(2000);
    return new AsyncResult<>("hello");
}

@Bean
ThreadPoolTaskExecutor tp() {
    ThreadPoolTaskExecutor te = new ThreadPoolTaskExecutor();
    te.setCorePoolSize(10);        // 기본적으로 만들어 두는 쓰레드 갯수. 무조건 만드는 건 아니고 첫번째 쓰레드 요청이 오면 만듬
    te.setQueueCapacity(200);      // CorePoolSize 가 꽉 찼을 때 요청이 들어오면 큐에 넣어둠
    te.setMaxPoolSize(100);        // QueueCapacity 사이즈가 꽉 차면 쓰레드 MaxPoolSize 만큼 늘려줌
    te.setKeepAliveSeconds(60);    // CorePoolSize 를 초과해서 만들어졌다가 쓰레드 반환 후에 일정시간 이상 재할당이 안되면 제거하기 시작함. 불필요한 메모리 점유를 막음
    te.setTaskDecorator();         // 쓰레드를 새로 만들거나 반환하는 시점 앞뒤에 콜백을 걸 수 있음 (로그용으로 사용 가능)
    te.setThreadNamePrefix("mythread");
    te.initialize();

    return te;
}
```

<br>

## (2) Future 로 받아서 Callback 등록

Spring 4.0 부터는 `ListenableFuture` 을 지원합니다.

이 클래스는 `addCallback()` 메소드에 onSuccess 와 onError 를 넘겨줄 수 있습니다.

`ListenableFuture` 사용법에 대해서는 5 장에서 다룰 예정입니다.

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

## 비동기 서블릿을 활용한 요청/응답 처리 - Callable

```java
@RestController
public static class MyController {

   @GetMapping("/callable")
   public Callable<String> callable() {
      return () -> {
         Thread.sleep(2000);
         return "hello";
      };
   }
}
```

일반적인 HTTP 응답 대신 `Callable` 을 사용하면 서블릿 쓰레드에서 직접 작업하지 않고 워커 쓰레드에게 넘깁니다.

그리고 `Callable` 이라는 작업 완료 신호를 받을 때 서블릿 쓰레드가 다시 할당되어 응답을 보내줍니다.

이 과정을 그림으로 표현해보면 아래와 같습니다.

<br>

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/books-or-lecture/%ED%86%A0%EB%B9%84%EC%9D%98-%EB%B4%84-TV-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/images/toby-reactive-1.png?raw=true)

요청이 오면 응답을 줄 때까지 쓰레드를 점유하고 있던 기존 서블릿과 달리 비동기 서블릿은 요청이 오면 작업 쓰레드에 바로 넘기고 쓰레드를 반납했다가 작업 완료 시점에 다시 쓰레드를 할당 받아 응답을 줍니다.

이 구조는 서블릿 쓰레드를 오랫동안 점유하지 않기 때문에 적은 쓰레드로도 많은 요청을 처리할 수 있지만, 사실 함정이 있습니다.

결국 실제 로직을 처리하는 작업 쓰레드를 요청수만큼 생성해야 하기 때문에 효율적인 메모리 활용이라고 할 수 없습니다.

<br>

## 비동기 서블릿을 활용한 요청/응답 처리 - DeferredResult

```java
@RestController
public static class MyController {

    Queue<DeferredResult<String>> queue = new ConcurrentLinkedQueue<>();

    @GetMapping("/dr")
    public DeferredResult<String> callable() {
        DeferredResult<String> dr = new DeferredResult<>(6000L);
        queue.add(dr);
        return dr;
    }

    @GetMapping("/dr/count")
    public String drCount() {
        return String.valueOf(queue.size());
    }

    @GetMapping("/dr/event")
    public String drEvent(String msg) {
        for (DeferredResult<String> dr : queue) {
            dr.setResult("Hello " + msg);
                queue.remove(dr);
        }
        return "OK";
    }
}
```

`/dr` 로 API 를 요청하면 클라이언트 (브라우저) 는 대기 상태가 되고 `queue` 에는 `DeferredResult<String>` 값이 들어갑니다.

`/dr/event` 로 외부 이벤트를 발생시켜서 `dr.setResult()` 메소드를 호출하면 `/dr` 을 호출했던 브라우저에 결과값이 전달되고 커넥션은 종료됩니다.

<br>

Long Polling 과 비슷한 개념입니다.

Client 에서 요청이 오면 `DeferredResult` 에 담아두고 서블릿은 바로 반납하되 Connection 을 유지하며 응답 대기하고 있습니다.

그러다가 다른 이벤트가 발생해서 `DeferredResult.setResult()` 메소드가 호출되면 대기 중이던 `DeferredResult` 에 값이 쓰여지고 바로 응답을 합니다.

가장 큰 특징은 워커 쓰레드가 따로 만들어지지 않고 메모리에 값이 저장되어 있기 때문에 이벤트 기반에서 서블릿 자원을 최소화 할 수 있습니다.

![](https://github.com/ParkJiwoon/PrivateStudy/blob/master/books-or-lecture/%ED%86%A0%EB%B9%84%EC%9D%98-%EB%B4%84-TV-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D/images/toby-reactive-2.png?raw=true)

<br>

# 5. 비동기 RestTemplate 과 비동기 MVC/Servlet

- [Youtube 링크](https://www.youtube.com/watch?v=ExUfZkh7Puk)

4장에서는 조금 원시적이지만 Java 와 Spring 에서 비동기를 활용하여 쓰레드를 점유하지 않고 요청을 처리하는 방법을 알아보았습니다.

그러나 시간이 지나고 트렌드가 변하면서 클라이언트-서버 통신 뿐만 아니라 서버-서버 통신을 활용한 마이크로 서비스가 유행하게 되었습니다.

쓰레드 자체는 논블로킹으로 동작하지만 외부 API 호출은 여전히 블로킹으로 동작하여 전체적인 응답 속도를 저하시킵니다.

일반적으로 Spring 은 요청 하나당 쓰레드 하나씩을 할당해서 작업합니다.

그런데 만약 굉장히 많은 수의 요청이 동시에 들어온다고 가정했을 때, 쓰레드를 무한정 생성하기는 어렵습니다.

가령 쓰레드를 무한정 생성할 수 있다고 해도 쓰레드를 생성하는건 굉장히 많은 메모리를 사용하고 컨텍스트 스위칭에 많은 비용이 소요됩니다.

5 장에서는 비동기 기법으로 외부 API 통신을 효율적으로 하는 방법을 알아봅니다.

<br>

# RestTemplate

가장 일반적인 API 요청입니다.

외부 API 를 요청하고 응답이 올 때까지 해당 쓰레드가 블록됩니다.

간단한 예제 코드를 만들어봅시다.

<br>

```java
@RestController
public static class RemoteController {

    @GetMapping("/service")
    public String service(String req) throws InterruptedException {
        Thread.sleep(2000);  // 강제로 2 초 소요되는 API 로 만듬
        return req + "/service";
    }
}
```

`Thread.sleep(2000)` 을 사용하여 2초가 소요되는 외부 API 를 만들었습니다.

파라미터를 받은 후 뒤에 `/service` 라는 String 을 함께 붙여서 리턴해줍니다.

<br>

```java
@RestController
public static class MyController {
    static final String URL = "http://localhost:8080/service?req={req}";
    
    RestTemplate rt = new RestTemplate();

    @GetMapping("/rest")
    public String rest(int idx) {
        return rt.getForObject(URL, String.class, "hello" + idx);
    }
}
```

`/rest` 로 요청을 받으면 `/service` 라는 외부 API 를 호출해서 받은 결과에 hello 라는 값을 받는 API 입니다.

<br>

위와 같은 애플리케이션에 100 명이 동시에 `/rest`  에 요청을 날렸다고 가정해봅니다.

`RestTemplate.getForObject` 는 Blocking 메소드기 때문에 쓰레드 하나당 한개의 요청만 처리할 수 있습니다.

만약 쓰레드가 100 개라면 각각 처리할수 있겠지만 쓰레드가 1 개인 상황이라면 요청 하나당 2 초씩 소모되어 100 개의 요청을 전부 처리하는데 굉장히 오랜 시간이 걸립니다.

여기서는 한 개의 API 만을 호출했지만 API 가 여러개인데 전부 2초씩 걸린다면 쓰레드가 작업을 처리하는 동안 나머지 요청들은 Tomcat 큐에서 대기해야만 합니다.

게다가 외부 API 에 요청하고 대기하는 일만 하기 때문에 블록된 쓰레드는 사실상 놀고 있는거나 다름이 없습니다.

<br>

이러한 문제점을 해결하기 위한 방법은 외부 API 를 비동기로 호출한 뒤 결과를 바로 받고 다른 작업을 시작하는 겁니다.

Spring 3.x 에서는 솔루션을 제공하지 못했지만 Spring 4.0 부터는 `AsyncRestTemplate` 이라는 비동기 버전을 제공했습니다.

<br>

# AsyncRestTemplate

Spring 5.x 부터는 WebClient 를 제공하면서 이 클래스는 Deprecated 되었습니다.

Spring 5.x 이전에는 어떤 방식으로 비동기 API 요청을 했는지 가벼운 맘으로 읽어보시면 좋을 것 같습니다. 

<br>

## AsyncRestTemplate 를 사용한 비동기 API 요청

```java
@RestController
public static class MyController {
    static final String URL = "http://localhost:8080/service?req={req}";
    
    AsyncRestTemplate rt = new AsyncRestTemplate();

    @GetMapping("/rest")
    public ListenableFuture<ResponseEntity<String>> rest(int idx) {
        return rt.getForEntity(URL, String.class, "hello" + idx);
    }
}
```

`getForObject` 대신 `getForEntity` 라는 메소드를 사용하는데 응답값이  `ListenableFuture<ResponseEntity<String>>` 타입입니다.

ListenableFuture 는 콜백을 등록해야 하는데 그냥 응답 타입으로 사용해도 괜찮을까요?

네, Spring WebMVC 가 알아서 콜백을 등록해주기 때문에 괜찮습니다.

<br>

하지만 이 과정에는 사실 함정이 있습니다.

비동기로 동작해서 Tomcat 쓰레드는 하나지만 실제로는 백그라운드 쓰레드를 만들어서 작업을 할당합니다.

하나의 쓰레드를 사용하기 때문에 효율적인 것처럼 보이지만 사실 서버 자원을 100 개 더 사용하는 겁니다.

일시적이긴 하지만 쓰레드를 새로 만드는건 굉장히 큰 비용이기 때문에 우리가 원하는 상황은 아닙니다.

<br>

## AsyncRestTemplate + Netty 를 활용한 향상된 비동기 API 요청

추가적인 쓰레드를 사용하지 않으려면 논블로킹 I/O 를 수행하는 라이브러리를 사용하는 겁니다.

여기서는 예를 들어서 Netty 를 사용할 수 있습니다.

<br>

```java
@RestController
public static class MyController {
    static final String URL = "http://localhost:8080/service?req={req}";
    AsyncRestTemplate rt = new AsyncRestTemplate(new Netty4ClientHttpRequestFactory(new NioEventLoopGroup(1)));

    @GetMapping("/rest")
    public ListenableFuture<ResponseEntity<String>> rest(int idx) {
        return rt.getForEntity(URL, String.class, "hello" + idx);
    }
}
```

`AsyncRestTemplate` 생성자로 `Netty4ClientHttpRequestFactory` 를 넘기면 됩니다.

그리고 추가적인 설정도 할 수 있는데 영상에서는 `NioEventLoopGroup` 를 사용하여 네티 쓰레드도 하나만 생성하게 설정했습니다.

디폴트로 생성되는 Netty 쓰레드 갯수는 (프로세스 갯수 * 2 개) 만큼 생성됩니다.

<br>

# 비동기 API 로 받은 응답을 논블로킹으로 처리

지금까지 본 예제들은 외부 API 를 호출해서 결과값을 그대로 리턴하기만 했습니다.

만약 외부 API 로 받은 결과물을 가공해서 응답해야 하는 상황을 생각해봅시다.

이 과정에서 쓰레드가 블록된다면 효율적이라고 할 수 없죠.

<br>

우선 API 로 받은 ListenableFuture 값에 callback 을 등록합니다.

callback 은 코드 내에서 수행하고 끝나는건데 이걸 어떻게 응답값으로 전달할 수 있을까요?

여기서 바로 지난 시간에 봤던 `DeferredResult` 를 사용할 수 있습니다.

<br>

```java
@RestController
public static class MyController {
    static final String URL = "http://localhost:8080/service?req={req}";
    
    AsyncRestTemplate rt = new AsyncRestTemplate(new Netty4ClientHttpRequestFactory(new NioEventLoopGroup(1)));

    @GetMapping("/rest")
    public DeferredResult<String> rest(int idx) throws InterruptedException {
        DeferredResult<String> dr = new DeferredResult<>();

        ListenableFuture<ResponseEntity<String>> f1 = rt.getForEntity(URL, String.class, "hello" + idx);

        f1.addCallback(s -> {
            dr.setResult(s.getBody() + "/work");
        }, e -> {
            dr.setErrorResult(e.getMessage());
        });

        Thread.sleep(3000);
        return dr;
    }
}
```

`DeferredResult` 를 응답 값으로 넘기면 클라이언트는 요청에 대한 응답을 메모리에 보관하다가 `setResult()` 가 호출되는 순간에 값을 넘겨줍니다.

그래서 ListenableFuture 의 Success Callback 안에 `setResult()` 를 추가하면 작업 완료 후에 값을 넘겨줄 수 있습니다.

<br>

# 외부 API 연쇄 호출

만약 A 외부 API 를 호출해서 받은 결과를 사용해서 다시 B 외부 API 를 호출하고 싶다면 어떻게 할 수 있을까요?

테스트를 위해 `RemoteController` 에 API 엔드포인트를 하나 더 추가합니다.

```java
@RestController
public static class RemoteController {

    @GetMapping("/service1")
    public String service1(String req) throws InterruptedException {
        Thread.sleep(2000);
        return req + "/service1";
    }

    @GetMapping("/service2")
    public String service2(String req) throws InterruptedException {
        Thread.sleep(2000);
        return req + "/service2";
    }
}
```

<br>

그리고 API 요청을 받아서 가공하는 `asyncWork` 메소드를 추가합니다.

```java
@Async
ListenableFuture<String> asyncWork(String req) {
    return new AsyncResult<>(req + "/asyncwork");
}
```

<br>

지금까지 공부한 것과 크게 다르지 않습니다.

단순히 ListenableFuture 의 callback 에 API 호출을 추가하면 됩니다.

연속으로 비동기 처리하는 부분만 간단하게 코드로 표현하면 다음과 같습니다.

```java
ListenableFuture<ResponseEntity<String>> f1 = rt.getForEntity(URL_1, String.class, "hello" + idx);

f1.addCallback(s -> {
    ListenableFuture<ResponseEntity<String>> f2 = rt.getForEntity(URL_2, String.class, s.getBody());

    f2.addCallback(s2 -> {
        ListenableFuture<String> f3 = asyncWork(s2.getBody());

        f3.addCallback(s3 -> {
            dr.setResult(s3 + "/work");
        }, e -> {
            dr.setErrorResult(e.getMessage());
        });
    }, e -> {
        dr.setErrorResult(e.getMessage());
    });
}, e -> {
    dr.setErrorResult(e.getMessage());
});
```

<br>

단순히 3 개의 비동기 작업만 처리하는데 코드량은 굉장히 길어졌습니다.

그리고 Error 처리와 같은 중복되는 코드도 보입니다.

쓰레드와 메모리 효율은 증가했지만 코드의 가독성이 굉장히 떨어졌습니다.

7 장에서는 `CompletableFuture` 을 사용해서 이 코드를 우아하게 개선하는 방법에 대해 알아봅니다.
