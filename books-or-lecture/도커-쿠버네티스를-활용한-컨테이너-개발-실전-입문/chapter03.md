# 컨테이너 실전 구축 및 배포

- [컨테이너 실전 구축 및 배포](#컨테이너-실전-구축-및-배포)
- [1. 애플리케이션과 시스템 내 단일 컨테이너의 적정 비중](#1-애플리케이션과-시스템-내-단일-컨테이너의-적정-비중)
  - [1.1. 컨테이너 1개 = 프로세스 1개 ?](#11-컨테이너-1개--프로세스-1개-)
  - [1.2. 컨테이너 하나로 cron 과 작업 프로세스 모두 실행하기](#12-컨테이너-하나로-cron-과-작업-프로세스-모두-실행하기)
  - [1.3. 컨테이너 1개에 하나의 관심사](#13-컨테이너-1개에-하나의-관심사)

# 1. 애플리케이션과 시스템 내 단일 컨테이너의 적정 비중

도커를 사용한 시스템 구성은 애플리케이션이나 미들웨어 이미지로 만든 컨테이너들이 서로 협력하는 스택을 구축하는 것이다.

실제 운영 환경에서는 애플리케이션을 컨테이너 안에 어떻게 배치하는지가 매우 중요하다.

컨테이너 하나가 맡을 수 있는 적정 수준의 책임은 어느정도 일까? 비중은 어느 정도로 해야할까?

<br>

## 1.1. 컨테이너 1개 = 프로세스 1개 ?

도커는 애플리케이션 배포에 특화된 가상화 기술. 애플리케이션과 인프라를 도커 컨테이너라는 단위로 분리한 것.

컨테이너 1개에 프로세스 1개가 적당할까?

<br>

정기적으로 어떤 작업을 실행하는 컨테이너가 있다고 가정.

- 스케줄러와 작업이 합쳐진 애플리케이션을 만든다면 컨테이너 1개 = 프로세스 1개 원칙을 지킬 수 있음
- 그러나 모든 애플리케이션이 스케줄러 기능을 지원하진 않음
- 여기서의 예제도 스케줄러 기능이 없다고 가정
- 정기 작업을 실행하려면 cron 을 사용해야함

<br>

cron 을 사용한다면..

- cron 은 1개의 상주 프로세스 형태로 동작
- 스케줄러가 실행하는 작업 역시 하나의 프로세스
- 컨테이너 1개 = 프로세스 1개 방식을 택한다면 cron 이 1개 컨테이너고 실행되는 작업이 또 1개의 컨테이너
- 지나치게 복잡해서 비효율적

<br>

## 1.2. 컨테이너 하나로 cron 과 작업 프로세스 모두 실행하기

컨테이너 하나로 cron 과 작업 프로세스를 모두 실행하는 방법을 시도해보자

<br>

**1) 필요한 파일 만들기**

**task.sh**

```sh
#!/bin/sh
echo "[`date`] Hello" >> /var/log/cron.log
```

<br>

**cron-example**

```text
* * * * * root sh /usr/local/bin/task.sh
```

- `task.sh` 파일을 1분에 한번씩 실행

<br>

**Dockerfile**

```docker
FROM ubuntu:16.04
 
RUN apt update
RUN apt install -y cron
 
COPY task.sh /usr/local/bin/
COPY cron-example /etc/cron.d/
RUN chmod 0644 /etc/cron.d/cron-example
 
CMD ["cron", "-f"]
```

- `ubuntu:16.04` 이미지 사용
- `apt` 로 cron 설치
- `task.sh` 와 `cron-example` 파일을 컨테이너로 복사하고 권한 조정
- `CMD` 로 cron 실행
  - cron 은 기본적으로 백그라운드에서 동작하기 때문에 실행하면 바로 종료됨
  - `-f` 옵션을 붙여 포어그라운드로 실행

<br>

**디렉터리 구조**

```sh
$ tree -a -L 2
.
├── Dockerfile
├── cron-example
└── task.sh

0 directories, 3 files
```

<br>

**2) 도커 이미지 빌드**

```sh
$ docker image build -t example/cronjob:latest .

[+] Building 22.0s (11/11) FINISHED
 => [internal] load build definition from Dockerfile                                                                                            0.0s
 => => transferring dockerfile: 226B                                                                                                            0.0s
 => [internal] load .dockerignore                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                 0.0s
 => [internal] load metadata for docker.io/library/ubuntu:16.04                                                                                 3.1s
 => [1/6] FROM docker.io/library/ubuntu:16.04@sha256:9775877f420d453ef790e6832d77630a49b32a92b7bedf330adf4d8669f6600e                           7.2s
 => => resolve docker.io/library/ubuntu:16.04@sha256:9775877f420d453ef790e6832d77630a49b32a92b7bedf330adf4d8669f6600e                           0.0s
 => => sha256:9ff95a467e458bb9e8653b1df439e02e07fc0be5b362cc3d9aeb0d04039d5925 3.36kB / 3.36kB                                                  0.0s
 => => sha256:80bce60046fa9e5ccbe54c9bd4bfa3f379ce7bc43bed493ae92389050de04024 46.46MB / 46.46MB                                                4.6s
 => => sha256:55a738a1554069bc9050c0a60b57fc93e98069e59822677a483cc74cafaf2bf7 852B / 852B                                                      0.6s
 => => sha256:e19cf0706c6229033d11dbf952b3eb96ad70e1f32527960aeb3c83ad86f16551 526B / 526B                                                      0.7s
 => => sha256:9775877f420d453ef790e6832d77630a49b32a92b7bedf330adf4d8669f6600e 1.42kB / 1.42kB                                                  0.0s
 => => sha256:d7bb0589725587f2f67d0340edb81fd1fcba6c5f38166639cf2a252c939aa30c 1.15kB / 1.15kB                                                  0.0s
 => => sha256:de4cdd6c27d1f17cf5ff350e76b7efe80aceff4dc99fd518065bf048abd6494a 169B / 169B                                                      0.9s
 => => extracting sha256:80bce60046fa9e5ccbe54c9bd4bfa3f379ce7bc43bed493ae92389050de04024                                                       1.9s
 => => extracting sha256:55a738a1554069bc9050c0a60b57fc93e98069e59822677a483cc74cafaf2bf7                                                       0.0s
 => => extracting sha256:e19cf0706c6229033d11dbf952b3eb96ad70e1f32527960aeb3c83ad86f16551                                                       0.0s
 => => extracting sha256:de4cdd6c27d1f17cf5ff350e76b7efe80aceff4dc99fd518065bf048abd6494a                                                       0.0s
 => [internal] load build context                                                                                                               0.0s
 => => transferring context: 169B                                                                                                               0.0s
 => [2/6] RUN apt update                                                                                                                        7.8s
 => [3/6] RUN apt install -y cron                                                                                                               3.3s
 => [4/6] COPY task.sh /usr/local/bin/                                                                                                          0.0s
 => [5/6] COPY cron-example /etc/cron.d/                                                                                                        0.0s
 => [6/6] RUN chmod 0644 /etc/cron.d/cron-example                                                                                               0.3s
 => exporting to image                                                                                                                          0.1s
 => => exporting layers                                                                                                                         0.1s
 => => writing image sha256:dbf585acbc41ce96f00025ddf666aa9452c097cecd00274dbbd2395483b296e9                                                    0.0s
 => => naming to docker.io/example/cronjob:latest
```

<br>

**3) 컨테이너 실행**

```sh
$ docker container run -d --rm --name cronjob example/cronjob:latest
d957e550c939843f12cc9db174ae1950f62aff21b258eca0fb3bfa025bfe36b2
```

<br>

**4) 프로세스 실행 확인**

```sh
$ docker container exec -it cronjob tail -f /var/log/cron.log

[Sun Jun 13 16:01:01 UTC 2021] Hello
[Sun Jun 13 16:02:01 UTC 2021] Hello
[Sun Jun 13 16:03:01 UTC 2021] Hello
```

- 컨테이너 하나에서 cron 과 작업까지 2 개 프로세스를 모두 실행
- 이런 경우에는 컨테이너 1개 = 프로세스 1 개를 억지로 고수하는 것보다 나음

<br>

## 1.3. 컨테이너 1개에 하나의 관심사

애플리케이션을 구축하며 컨테이너 1 개 = 프로세스 1 개 원칙을 고수하는 것은 무리임을 알았다.

<br>

**도커의 공식 문서 'Best Practices for writing Dockerfiles'**

Each container should have only one concern.

컨테이너는 하나의 관심사에만 집중해야 한다.

<br>

**하나의 관심사란?**

컨테이너 하나가 한 가지 역할이나 문제 영역 (도메인) 에만 집중해냐 한다.

<br>

