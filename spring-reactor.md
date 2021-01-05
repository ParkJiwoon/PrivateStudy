# Mono

## create

직접적으로 데이터를 생성 하거나 에러 신호를 내보낼 수 있음

 `subscribe` 하지 않으면 데이터를 방출하지 않음

Cold Publisher

<br>

## just

값을 즉시 방출 (`subscribe` 하지 않아도)

처음 방출된 값을 캐싱 해놓고 다음 구독자 에게는 캐싱된 값을 방출

Hot Publisher

<br>

## defer

데이터의 방출을 구독 전까지 지연시킴

<br>

## fromCallable

Blocking Call 이 필요할 때 별도의 쓰레드로 실행할 수 있게 해줌

```java
Mono.fromCallable(() -> {
	/* blocking call ex java */
}).subscribeOn(Schedulers.elastic());
```

<br><br>

# Schedulers

Schedulers 는 Project Reactor 의 핵심 패키지 중 하나입니다.

`reactor.core.scheduler` 에는 `Schedulers` 라는 추상 클래스가 존재하고 비동기 코드에서 Thread 를 다룰 수 있게 합니다.

- `Schedulers.immediate()`
    - 현재 쓰레드에서 실행
- `Schedulers.single()`
    - 쓰레드가 한 개인 쓰레드풀을 이용해서 실행
    - 즉 한 쓰레드를 공유
- `Schedulers.elastic()`
    - 무한정 증가하는 쓰레드 풀을 이용해서 실행
    - Blocking I/O 를 리액터로 처리할 때 적합
    - 쓰레드가 필요하면 새로 생성하고 일정 시간(기본 60초) 이상 유휴 상태인 쓰레드는 제거
    - 수행시간이 오래걸리는 블로킹 작업에 대한 대안으로 사용할 수 있게 최적화 되어 있음
- `Schedulers.boundedElastic()`
    - elastic() 과 동일하지만 쓰레드 갯수가 정해져 있음
- `Schedulers.parallel()`
    - 고정 크기 쓰레드 풀을 이용해서 실행
    - 병렬 작업에 적합

<br>

`single()`, `elastic()`, `parallel()` 은 매번 새로운 쓰레드 풀을 만들지 않고 동일한 쓰레드 풀을 공유합니다.

위 함수들이 생성한 쓰레드는 main 쓰레드가 종료되면 함께 종료됩니다.

만약 새로운 쓰레드풀을 사용하고 싶다면 `newXXX()` 를 사용하면 됩니다.

- `newSingle(String name, boolean daemon)`
- `newElastic(String name, int ttlSeconds, boolean daemon)`
- `newParallel(String name, int parallelism, boolean daemon)`

<br>

각 파라미터들은 다음과 같습니다.

- `name` : 쓰레드 이름으로 사용할 접두사이다.
- `daemon` (optional) : 데몬 쓰레드 여부를 지정한다. 지정하지 않으면 false이다. 데몬 쓰레드가 아닌 경우 JVM 종료시에 생성한 스케줄러의 dispose()를 호출해서 풀에 있는 쓰레드를 종료해야 한다.
- `ttlSeconds` (optional) : elastic 쓰레드 풀의 쓰레드 유휴 시간을 지정한다. 지정하지 않으면 60(초)이다.
- `parallelism` (optional) : 작업 쓰레드 개수를 지정한다. 지정하지 않으면 `Runtime.getRuntime().availableProcessors()`이 리턴한 값을 사용한다.

<br>

`newXXX()` 로 생성하는 쓰레드 풀은 기본으로 데몬 쓰레드가 아니기 때문에 어플리케이션 종료시에는 다음과 같이 `dispose()` 메서드를 호출해서 쓰레드를 종료시켜 주어야 합니다.

그렇지 않으면 어플리케이션이 종료되지 않는 문제가 발생할 수 있습니다.

```java
// 비데몬 스케줄러 초기화
Scheduler scheduler = Schedulers.newElastic("SUB", 60, false);

// 비데몬 스케줄러 사용
someFlux.publishOn(scheduler)
            .map(...)
            .subscribe(...)

// 어플리케이션 종료시에 스케줄러 종료 처리
scheduler.dispose();
```
