# Java Future

## 개념
- 비동기 처리를 위해서 사용

## Method
- ```V get()```
    - Call 작업의 실행이 완료될 때까지 블로킹 되며 완료되면 그 결과값을 리턴
    - CancellationException: 작업이 취소되는 경우
    - ExecutionException: 작업 도중 예외가 발생한 경우
    - InterruptedException: 현재 쓰레드가 인터럽트된 경우

- ```V get(long timeout, TimeUnit unit)```
    - 지정된 시간동안 작업의 실행 결과를 기다림
    - TimeoutException: 대기시간이 초과한 경우
    - ```get()``` Exception 은 전부 같음

- ```boolean cancel(boolean mayInterruptIfRunning)```
    - 작업 취소 시도
    - 작업이 이미 완료 또는 취소되었거나 취소할 수 없는 경우 실패
    - 작업이 시작되기 전이었다면 실행하지 않음
    - ```cancel``` 메소드가 리턴된 후 ```isDone()``` 메소드는 항상 true
    - ```cancel``` 메소드가 true를 리턴하면 ```isCancelled()``` 메소드는 항상 true

- ```boolean isCancelled()```
    - 작업이 완료되기 전에 최소된 경우 true

- ```boolean isDone()```
    - 작업이 완료된 경우 true
    - 작업의 완료란 정상 종료, 예외, 취소 등을 전부 말함
