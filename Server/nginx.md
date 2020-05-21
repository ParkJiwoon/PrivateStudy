# Nginx 란?

Nginx 란 러시아의 이고르 시쇼브란 개발자가 Apache 의 C10K Problem (하나의 웹서버에 10,000개의 클라이언트의 접속을 동시에 다룰 수 있는 기술적인 문제) 을 해결하기 위해 Event-Driven 구조로 만든 오픈 소스 소프트웨어다.

<br>

# Apache 와의 차이점

## Apache

Client가 HTTP 요청을 보낼 때, Apache는 MPM (Multi Processing Module) 을 사용하여 처리한다. 

<br>

### 1. Prefork 방식 (멀티 프로세스 방식)

자식 프로세스를 미리 생성해두고 클라이언트 요청에 하나에 대해 한 프로세스가 담당한다.

따라서 한 자식 프로세스가 알 수 없는 원인으로 정지하더라도 다른 자식 프로세스에 영향을 주지 않는다.

자식 프로세스의 수는 최대 1024 개다.

프로세스 당 한 개의 스레드만 존재하기 때문에 스레드간 메모리 공유를 하지 않아서 안정적인 대신에 메모리 사용량이 많다.

시작 시 생성한 프로세스의 수보다 요청이 많아지면 실행 중인 프로세스를 복제하여 실행한다. 이 때, 메모리 영역까지 같이 복제된다.

<br>

### 2. Worker 방식 (멀티 프로세스 & 멀티 스레드 방식)

자식 프로세스마다 멀티 스레드로 실행하며 각 클라이언트의 요청을 스레드가 처리한다.

하나의 프로세스가 여러 요청을 담당하며 Prefork 와 비교해서 시작 프로세스 수를 줄일 수 있다.

스레드 간 메모리를 공유하기 때문에 메모리 사용량이 적다.

한 프로세스 당 최대 64 개의 스레드 처리가 가능하다.

<br>

## Apache 한계

Apache는 접속마다 Process 또는 Thread를 생성하는 구조이다. 

동시 접속 요청이 10,000 개라면 그 만큼 Process or Thread 생성 비용이 들 것이고 대용량 요청을 처리할 수 있는 웹서버로서의 한계를 드러내게 된다.

<br>

# Nginx

Nginx 는 Event-Driven 방식으로 동작한다. 

한 개 또는 고정된 프로세스만 생성 하고, 그 프로세스 내부에서 비 동기 방식으로 효율적으로 작업들을 처리한다.

따라서 동시 접속 요청이 많아도 Process 또는 Thread 생성 비용이 존재하지 않는다.

Event-Driven 방식에선 작업을 하다 I/O, socket read/write 등 CPU가 관여하지 않는 작업이 시작되면 기다리지 않고 바로 다른 작업을 수행한다.

<br>

# 명령어

nginx 디렉토리로 가서 명령어 타이핑 (권한이 필요하면 `sudo`)

```sh
# nginx 시작
$ ./nginx

# nginx 중지
$ ./nginx -s stop

# nginx 재시작
$ ./nginx -s reload
```

<br>

# Reference

- [넌 뭐니 NGINX? - 서정국님 Medium](https://medium.com/sjk5766/%EB%84%8C-%EB%AD%90%EB%8B%88-nginx-9a8cae25e964)
- [opentutorials](https://opentutorials.org/module/384/4526)