- [Youtube 링크](https://www.youtube.com/watch?v=PzxV-bmLSFY)

<br>

7 장에서는 Java 8 부터 제공하는 `CompletableFuture` 에 대한 사용법을 간단히 알아보고 비동기 처리를 이쁘게 하는 방법을 알아봅니다.

<br>

# CompletableFuture

CompletableFuture 는 Java 8 에 추가된 클래스입니다.

Future 는 비동기 작업의 결과를 담고 있는 오브젝트라고 보면 됩니다.

비동기라는 건 결국 새로운 쓰레드를 만들어서 백그라운드에서 수행시키는 건데 이에 대한 결과를 가져오는 방법 중 하나로 Future 로 감싸주는 방법이 있습니다.

그 다음에 나온 ListenableFuture 는 콜백 구조로 비동기 작업이 완료되는 시점에 수행될 FutureTask 를 넘겨줍니다.

좀 더 간단하지만 콜백 헬을 유발한다는 단점이 있었습니다.

Java 8 에 나온 CompletableFuture 는 이런 단점들을 보완하고 비동기 작업을 위한 여러가지 설정을 제공합니다.

그리고 람다 표현식과 파이프라이닝을 활용하여 콜백헬과 달리 이쁜 코드 구조를 만들 수 있습니다.

<br>

CompletableFuture 는 비동기 작업의 결과를 간단하게 만들 수 있습니다.

```java
CompletableFuture<Integer> f = CompletableFuture.completedFuture(1);
System.out.println(f.get());  // 1 출력
```

<br>

## complete, completeExceptionally

`complete` 와 `completeExceptionally` 을 사용하면 Future 에 대한 결과값을 명시적으로 쓸 수 있습니다.

```java
// complete 가 없으면 수행할 비동기 작업이 없어서 f.get() 호출시 무한대기 상태가 됨
CompletableFuture<Integer> f = new CompletableFuture<>();
f.complete(2);
System.out.println(f.get());  // 2 출력

// 실제로 f.get() 을 호출하기 전까지는 에러가 발생하지 않음
CompletableFuture<Integer> f = new CompletableFuture<>();
f.completeExceptionally(new RuntimeException());
```

<br>

# CompletionStage

CompletableFuture 는 Future 뿐만 아니라 CompletionStage 인터페이스도 구현하고 있습니다.

CompletionStage 는 하나의 비동기 작업을 수행하고 완료되었을 때 의존적으로 또다른 작업을 수행할 수 있도록 하는 명령들을 갖고 있습니다.

예를 들어 `CompletableFuture.runAsync()` 을 호출하면 CompletableFuture 을 리턴하고 이 클래스는 CompletionStage 인터페이스를 구현하고 있기 때문에 체이닝으로 다음에 할 작업들을 이어서 쓸 수 있습니다.

<br>

## (1) runAsync, thenRun

```java
CompletableFuture
        .runAsync(() -> log.info("runAsync"))
        .thenRun(() -> log.info("thenRun 1"))
        .thenRun(() -> log.info("thenRun 2"));

log.info("exit");

ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
```

- runAsync
    - `Runnable` 을 넘겨받아서 수행
- thenRun
    - `Runnable` 을 넘겨받아서 수행

<br>

```java
01:47:44.408 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - runAsync
01:47:44.408 [main] INFO FutureEx - exit
01:47:44.410 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - thenRun 1
01:47:44.411 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - thenRun 2
```

로그를 보면 동일한 시각에 runAsync 와 exit 가 찍히고 thenRun 이 순차적으로 찍힙니다.

비동기로 수행한 작업은 main 이 아닌 다른 쓰레드에서 실행되는 것을 볼 수 있습니다.

<br>

## (2) supplyAsync, thenApply, thenAccept

```java
CompletableFuture
        .supplyAsync(() -> {
            log.info("supplyAsync");
            return 1;
        })
        .thenCompose(s -> {
            log.info("thenCompose {}", s);
            return CompletableFuture.completedFuture(s + 1);
        })
        .thenApply(s -> {
            log.info("thenApply {}", s);
            return s * 3;
        })
        .thenAccept(s -> log.info("thenAccept {}", s));

log.info("exit");

ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
```

- supplyAsync
    - `Supplier` 를 넘겨받아서 수행하고 임의의 값을 리턴
- thenCompose
    - `Function<? super T, ? extends CompletionStage<U>>` 을 넘겨받아서 수행
    - 임의의 파라미터를 받은 후 `CompletionStage` 타입으로 감싸서 리턴
    - 스트림의 `flatMap` 과 유사
- thenApply
    - `Function<? super T,? extends U>` 을 넘겨받아서 수행
    - 임의의 파라미터를 받은 후 임의의 값을 리턴
    - 스트림의 `map` 과 유사
- thenAccept
    - `Consumer` 을 넘겨받아서 수행
    - 임의의 값을 넘겨받아서 수행하고 끝냄

<br>

```java
02:22:10.934 [main] INFO FutureEx - exit
02:22:10.934 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - supplyAsync
02:22:10.937 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - thenCompose 1
02:22:10.939 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - thenApply 2
02:22:10.939 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - thenAccept 6
```

역시 main 에는 exit 가 찍히고 다른 쓰레드에 supplyAsync 부터 순차적으로 찍힙니다.

이전 작업에서 넘겨 받은 `s` 값이 연산을 통해 계속해서 아래로 전달되는 것을 볼 수 있습니다.

<br>

## (3) exceptionally

```java
CompletableFuture
        .supplyAsync(() -> {
            log.info("supplyAsync");
            if (1 == 1) throw new RuntimeException();
            return 1;
        })
        .thenCompose(s -> {
            log.info("thenCompose {}", s);
            return CompletableFuture.completedFuture(s + 1);
        })
        .exceptionally(e -> {
            log.info("exceptionally " + e);
            return -10;
        })
        .thenApply(s -> {
            log.info("thenApply {}", s);
            return s * 3;
        })
        .thenAccept(s -> log.info("thenAccept {}", s));

log.info("exit");

ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
```

- exceptionally
    - `Function<Throwable, ? extends T>` 을 받음
    - 위 단계에서 어느 곳이든 예외가 발생하면 `exceptionally` 를 수행함
    - 임의의 값을 다음 단계로 넘겨주고 계속해서 수행

<br>

```java
02:42:01.310 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - supplyAsync
02:42:01.310 [main] INFO FutureEx - exit
02:42:01.312 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - exceptionally java.util.concurrent.CompletionException: java.lang.RuntimeException
02:42:01.312 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - thenApply -10
02:42:01.315 [ForkJoinPool.commonPool-worker-9] INFO FutureEx - thenAccept -30
```

첫 단계인 `supplyAsync` 수행 시에 `RuntimeException` 발생했기 때문에 `thenCompose` 는 수행되지 않습니다.

발생된 에러는 `exceptionally` 에서 받아서 처리되고, 단계를 벗어나는게 아니라 다음 단계로 값을 넘겨줍니다.

그래서 `thenApply` 는 -10 을 받아서 작업을 수행하고 `thenAccept` 까지 넘겨서 모든 작업을 완료합니다.

<br>

## (4) then~~Async

위에서 보여줬던 작업들은 `runAsync` 와 `supplyAsync` 를 수행했던 쓰레드에서 다음 작업들을 계속 이어나갑니다.

만약 특정 작업들을 다른 쓰레드에서 하고 싶다면 `thenApply` 대신 `thenApplyAsync` 을 사용하면 됩니다.

<br>

```java
ExecutorService es = Executors.newFixedThreadPool(10);

CompletableFuture
        .supplyAsync(() -> {
            log.info("supplyAsync");
            return 1;
        }, es)
        .thenCompose(s -> {
            log.info("thenCompose {}", s);
            return CompletableFuture.completedFuture(s + 1);
        })
        .exceptionally(e -> {
            log.info("exceptionally " + e);
            return -10;
        })
        .thenApplyAsync(s -> {
            log.info("thenApplyAsync {}", s);
            return s * 3;
        }, es)
        .thenAcceptAsync(s -> log.info("thenAcceptAsync {}", s), es);

log.info("exit");

ForkJoinPool.commonPool().awaitTermination(10, TimeUnit.SECONDS);
```

`then~~Async` 를 제대로 사용하기 위해선 직접 쓰레드풀 설정을 해줘야 합니다.

`Executor` 에 쓰레드 풀을 설정해서 두번째 인자로 넘겨주면 해당 쓰레드로 적용이 됩니다.

<br>

```java
05:59:14.129 [pool-1-thread-1] INFO FutureEx - supplyAsync
05:59:14.129 [main] INFO FutureEx - exit
05:59:14.132 [pool-1-thread-1] INFO FutureEx - thenCompose 1
05:59:14.133 [pool-1-thread-2] INFO FutureEx - thenApplyAsync 2
05:59:14.133 [pool-1-thread-3] INFO FutureEx - thenAcceptAsync 6
```

`supplyAsync` 는 1 번 쓰레드에서 실행되고 `thenCompose` 도 같은 쓰레드에서 실행됩니다.

그리고 `thenApplyAsync` 와 `thenAcceptAsync` 는 다른 쓰레드에서 실행되는 것을 확인할 수 있습니다.

쓰레드 설정이나 갯수에 따라서 조금씩 동작이 다른데 여기서는 자세히 다루지 않습니다.

<br>

# ListenableFuture 를 CompletableFuture 로 개선

5 장 마지막 부분에서 보았던 콜백헬을 기억하실 겁니다.

`CompletableFuture` 을 사용해서 가독성 있고 깔끔한 코드로 바꿔봅니다.

로직은 간단하게 첫번째 API 호출하고 결과로 두번쨰 API 호출 후 결과로 비동기 작업 수행 후 응답을 주는겁니다.

비동기 작업이 총 세번 이루어지는데 코드를 직접 보면 어떻게 바뀌었는지 차이점이 확 드러날겁니다.

<br>

## 기존 코드

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

## Refactoring

가장 먼저 해야할 일은 `ListenableFuture` 을 `CompletableFuture` 로 바꾸는 일입니다.

공통적으로 사용할 메소드를 하나 추가합니다.

<br>

```java
<T> CompletableFuture<T> toCF(ListenableFuture<T> lf) {
    CompletableFuture<T> cf = new CompletableFuture<>();
    lf.addCallback(cf::complete, cf::completeExceptionally);
    return cf;
}
```

`ListenableFuture` 을 받아서 `addCallback` 으로 성공, 실패 함수를 등록해줍니다.

<br>

```java
toCF(rt.getForEntity(URL_1, String.class, "hello" + idx))
    .thenCompose(s -> toCF(rt.getForEntity(URL_2, String.class, s.getBody())))
    .thenCompose(s -> toCF(asyncWork(s.getBody())))
    .thenAccept(s -> dr.setResult(s + "/work"))
    .exceptionally(e -> {
        dr.setErrorResult(e.getMessage());
        return null;
    });
```

`CompletableFuture` 로 바꾸는 것만 알고있으면 지금까지 학습한 내용들로 간단히 바꿀 수 있습니다.
