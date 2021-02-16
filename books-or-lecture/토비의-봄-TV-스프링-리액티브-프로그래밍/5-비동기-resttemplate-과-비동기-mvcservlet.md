- [Youtube 링크](https://www.youtube.com/watch?v=ExUfZkh7Puk)

<br>

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
