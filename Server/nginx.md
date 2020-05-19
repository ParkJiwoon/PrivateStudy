# Nginx 란?

Nginx 란 러시아의 이고르 시쇼브란 개발자가 Apache 의 C10K Problem (하나의 웹서버에 10,000개의 클라이언트의 접속을 동시에 다룰 수 있는 기술적인 문제) 을 해결하기 위해 Event-Driven 구조로 만든 오픈 소스 소프트웨어다.

<br>

# Apache 와의 차이점

## Apache

Client가 HTTP 요청을 보낼 때, Apache는 MPM (Multi Processing Module) 을 사용하여 처리한다. MPM 에는 크게 두 가지 방식이 있다.

<br>

### 1. Prefork 방식 (멀티 프로세스 방식)

Client 요청에 대해 default 개수만큼 apache 자식 프로세스를 생성하여 처리하고, 그 이상으로 요청이 많을 경우 Process 를 생성하여 처리하는 방식이다. 

Apache 설치 시, default 로 설정되어 있다.

<br>

### 2. Worker 방식 (멀티 프로세스 & 멀티 스레드 방식)

Prefork 와 같이 Default Apache 자식 프로세스를 생성하고 요청이 많아지면 각 프로세스의 스레드를 생성해 처리하는 구조이다.

두 방식의 특징은 우리가 흔히 알고 있는 프로세스와 쓰레드 사용의 장/단점과 동일하다.

<br>

## Apache 한계

Apache는 접속마다 Process 또는 Thread를 생성하는 구조이다. 

동시 접속 요청이 10,000 개라면 그 만큼 Process or Thread 생성 비용이 들 것이고 대용량 요청을 처리할 수 있는 웹서버로서의 한계를 드러내게 된다.

<br>

# Nginx

NGINX는 Event-Driven 방식으로 동작한다. 

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