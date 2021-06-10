# 도커 컨테이너 배포

도커의 기본 조작 방법과 애플리케이션을 배포하는 과정까지 알아본다

<br>

# 1. 컨테이너로 애플리케이션 실행하기

도커 이미지 하나로 여러 개의 컨테이너 생성 가능

- **도커 이미지**: 도커 컨테이너를 구성하는 파일 시스템과 실행할 애플리케이션 설정을 하나로 합친 것으로, 컨테이너를 생성하는 템플릿 역할을 함
- **도커 컨테이너**: 도커 이미지를 기반으로 생성되며, 파일 시스템과 애플리케이션이 구체화돼 실행되는 상태

<br>

## 1.1. 도커 이미지와 도커 컨테이너

도커 이미지로 도커 컨테이너를 만드는 과정을 한번 알아보자

**1) 도커 이미지 다운**

```sh
$ docker image pull gihyodocker/echo:latest
latest: Pulling from gihyodocker/echo
723254a2c089: Pull complete
abe15a44e12f: Pull complete
409a28e3cc3d: Pull complete
503166935590: Pull complete
abe52c89597f: Pull complete
ce145c5cf4da: Pull complete
96e333289084: Pull complete
39cd5f38ffb8: Pull complete
22860d04f4f1: Pull complete
7528760e0a03: Pull complete
Digest: sha256:4520b6a66d2659dea2f8be5245eafd5434c954485c6c1ac882c56927fe4cec84
Status: Downloaded newer image for gihyodocker/echo:latest
docker.io/gihyodocker/echo:latest
```

- `gihyodocker/echo:latest` 라는 도커 이미지를 받아옴
- 이 이미지는 누구나 받을 수 있도록 공개되어 있으며 `docker image pull` 명령어로 다운 가능

**2) 도커 이미지 실행**

```sh
$ docker container run -t -p 9000:8080 gihyodocker/echo:latest
2021/06/10 17:29:03 start server
```

- 지금 만든 컨테이너는 옵션을 통해 포트 포워딩이 적용되어 있음
- 도커 실행 환경의 포트 9000 을 거쳐 HTTP 요청을 전달 받음

**3) 애플리케이션 실행 확인**

```sh
$ curl http://localhost:9000/
Hello Docker!!
```

- curl 명령어로 호출하면 정상적으로 동작되는 것을 확인 가능

**4) 도커 컨테이너 종료**

```sh
$ docker stop $(docker container ls -q)
dfc9aa5b2646
```

<br>

## 1.2. 간단한 애플리케이션과 도커 이미지 만들기
