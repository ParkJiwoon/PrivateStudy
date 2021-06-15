# 컨테이너 실전 구축 및 배포

- [컨테이너 실전 구축 및 배포](#컨테이너-실전-구축-및-배포)
- [1. 애플리케이션과 시스템 내 단일 컨테이너의 적정 비중](#1-애플리케이션과-시스템-내-단일-컨테이너의-적정-비중)
  - [1.1. 컨테이너 1개 = 프로세스 1개 ?](#11-컨테이너-1개--프로세스-1개-)
  - [1.2. 컨테이너 하나로 cron 과 작업 프로세스 모두 실행하기](#12-컨테이너-하나로-cron-과-작업-프로세스-모두-실행하기)
  - [1.3. 컨테이너 1개에 하나의 관심사](#13-컨테이너-1개에-하나의-관심사)
- [2. 컨테이너의 이식성](#2-컨테이너의-이식성)
  - [2.1. 커널 및 아키텍쳐의 차이](#21-커널-및-아키텍쳐의-차이)
  - [2.2. 라이브러리와 동적 링크 문제](#22-라이브러리와-동적-링크-문제)
- [3. 도커 친화적인 애플리케이션](#3-도커-친화적인-애플리케이션)
  - [3.1. 환경 변수 활용](#31-환경-변수-활용)
- [4. 퍼시스턴스 데이터를 다루는 방법](#4-퍼시스턴스-데이터를-다루는-방법)
  - [4.1. 데이터 볼륨](#41-데이터-볼륨)
  - [4.2. 데이터 볼륨 컨테이너](#42-데이터-볼륨-컨테이너)
  - [4.3. Example: 데이터 볼륨에 MySQL 데이터 저장하기](#43-example-데이터-볼륨에-mysql-데이터-저장하기)

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

컨테이너 하나가 한 가지 역할이나 문제 영역 (도메인) 에만 집중해야 한다.

앞에서 살펴본 cronjob 을 예로 들면

1. 1분에 1번씩 작업 실행 트리거를 동작시키는 cron 작업
2. 'Hello!' 문자열을 출력하는 작업

위 두개를 하나로 합쳐서 '1분에 1번씩 Hello! 문자열을 출력하는 작업' 으로 컨테이너를 구성했다.

각 컨테이너가 맡은 역할을 적절히 나누는 것이 핵심

<br>

# 2. 컨테이너의 이식성

도커의 큰 장점 중 하나는 이식성 (portability)

도커를 사용하면 애플리케이션과 인프라를 컨테이너라는 단위로 분리 가능

도커가 설치된 환경이라면 어떤 호스트 운영 체제와 플랫폼, 클라우드 환경에서 그대로 동작

하지만 도커의 이식성은 완벽하지 않음..

<br>

## 2.1. 커널 및 아키텍쳐의 차이

도커의 컨테이너형 가상화 기술은 호스트 운영체제와 커널 리소스를 공유

사실상 도커 컨테이너를 실행하려면 호스트가 특정 CPU 아키텍쳐 혹은 운영 체제를 사용해야 한다는 의미

**모든 환경에서 동작하는 컨테이너는 없다**

그래도 도커는 지원 플랫폼의 범위를 넓혀가고 있음

<br>

## 2.2. 라이브러리와 동적 링크 문제

애플리케이션이 **어떤 라이브러리를 사용하느냐에 따라** 이식성을 해치는 경우가 있음 (네이티브 라이브러리를 동적 링크해 사용하는 경우)

- 정적 링크
  - 애플리케이션에 사용된 라이브러리를 내부에 포함하는 형태
  - 애플리케이션의 크기가 비대해짐
  - 이식성은 뛰어남
- 동적 링크
  - 애플리케이션을 실행할 때 라이브러리가 링크됨
  - 애플리케이션 크기가 작아짐
  - 애플리케이션을 실행할 호스트에서 라이브러리를 갖춰야 하기 때문에 이식성이 나쁨

도커를 사용할 때 Dockerfile 에 `ADD` 나 `COPY` 는 호스트에서 파일을 복사해옴 (컨테이너 외부에서 주입)

정적 링크로 인해 애플리케이션 크기가 커지는 걸 방지하기 위해 빌드 프로세스 전용 컨테이너를 따로 만들면 경량화 가능

어쨌든 **도커의 이식성은 절대적인 것이 아님**

<br>

# 3. 도커 친화적인 애플리케이션

이식성 높은 애플리케이션을 구축하기 위한 컨테이너의 제어와 설정

<br>

## 3.1. 환경 변수 활용

도커 컨테이너 형태로 실행되는 애플리케이션의 동작을 제어하는 방법

- **실행 시 인자**
  - 실행 시 인자 형태로 값을 전달
  - 장점
    - 외부에서 값을 주입받을 수 있음
  - 단점
    - 실행 시 인자가 너무 많아지면 인자를 내부 변수로 매핑해주는 처리가 복잡해짐
- **설정 파일**
  - 실행할 애플리케이션에 dev / prod 같은 환경 이름을 부여하고 그에 따라 설정 파일을 바꿔가며 사용
  - ROR, Maven, Gradle 등에서 흔히 볼 수 있음
  - 애플리케이션의 도커 이미지를 각 설정 파일 별로 만들어둠
  - 그러나 환경이 변경될 때마다 설정 파일을 변경해야 하고 이미지를 새로 빌드해야 함
- **애플리케이션 동작을 환경 변수로 제어 (추천)**
  - 앞의 예제에서 슬레이브 쪽에 `JENKINS_SLAVE_SSH_PUBKEY` 라는 환경 변수에 값을 설정했었음
  - 애플리케이션 외부에서 주입하는 형태라 별도 레포지토리를 통해 관리하는 것이 일반적
  - 컴포즈를 사용하는 경우 `docker-compose.yml` 파일의 `env` 속성에 기술
  - 장점
    - 매번 이미지를 다시 빌드하지 않아도 됨
    - 환경 변수 값을 바꾼 후 컨테이너를 다시 시작하면 끝
  - 단점
    - 환경 변수는 특성상 Key-Value 형태라서 계층 구조를 가지기 어려움
- **설정 파일에 환경 변수를 포함**
  - 설정 파일의 장점과 환경 변수의 장점을 모두 취할 수 있는 방법
  - 환경별 설정 파일을 애플리케이션에 포함하는 대신, 설정 파일 템플릿에 환경 변수를 포함

<br>

**설정 파일에 환경 변수 포함 예시 (Spring)**

모든 애플리케이션 프레임워크가 지원하지는 않음

```properties
db.driverClass=${DB_DRIVER_CLASS:com.mysql.jdbc.Driver}
db.jdbcUrl=${DB_JDBC_URL}
db.user=${DB_USER}
db.password=${DB_PASSWORD}
db.initialSize=${DB_INITIAL_SIZE:10}
db.maxActive=${DB_MAX_ACTIVE:50}
db.maxIdle=${DB_MAX_IDLE:20}
db.minIdle=${DB_MIN_IDLE:10}
```

<br>

# 4. 퍼시스턴스 데이터를 다루는 방법

도커 컨테이너가 실행 중에 작성 혹은 수정된 파일은 컨테이너가 파기될 때 삭제됨

컨테이너를 사용해서 운영하려면 새로운 컨테이너가 배포돼어도 이전 컨테이너에서 사용하던 파일 및 디렉토리를 그대로 사용 가능해야 함

이런 경우에 사용되는 것이 **데이터 볼륨 (Data Volume)**

<br>

## 4.1. 데이터 볼륨

도커 컨테이너 안의 디렉토리를 디스크에 퍼시스턴스 데이터로 남기기 위한 메커니즘

호스트와 컨테이너 사이의 디렉토리 공유 및 재사용 기능 제공

컨테이너를 파기해도 디스크에 그대로 남음

<br>

**사용법**

```sh
# $ docker container run [options] -v <호스트 디렉토리>:<컨테이너 디렉토리> <레포지토리명>[:<태그>] [<명령>] [<명령인자>]

# 컨테이너 안에서 이미지 생성 후 종료
$ docker container run -v ${PWD}:/workspace gihyodocker/imagemagick:latest convert -size 100x100 xc:#000000 /workspace/gihyo.jpg

# 데이터 볼륨 설정으로 인해 컨테이너 안의 /workspace 와 현재 작업 디렉토리 ($PWD) 가 공유됨
$ ls
gihyo.jpg
```

- 컨테이너 실행할 때 `-v` 옵션 붙이면 됨
- 호스트에서 편집한 파일을 데이터 볼륨을 통해 컨테이너와 공유 가능
  
데이터 볼륨은 컨테이너 안의 설정 파일을 쉽게 수정할 수 있지만, 호스트의 특정 경로에 의존성이 생기기 때문에 호스트 쪽 데이터 볼륨을 잘못 다루면 애플리케이션에 부정적 영향을 미칠 수 있다.

이 때문에 이식성 면에서는 아직 개선의 여지가 있는 기법

<br>

## 4.2. 데이터 볼륨 컨테이너

앞에서 설명한 데이터 볼륨은 컨테이너-호스트 사이의 공유라면, 데이터 볼륨 컨테이너는 컨테이너-컨테이너 사이의 공유

이름 그대로 **데이터를 저장하는 것만이 목적인 컨테이너**

디스크에 퍼시스턴스 데이터를 볼륨으로 만들어 다른 컨테이너에 공유

- 호스트-컨테이너 데이터 볼륨: 호스트의 특정 디렉토리에 의존성을 가짐
- 데이터 볼륨 컨테이너: 도커에서 관리하는 영역인 호스트의 `/var/lib/docker/volumes/` 아래에 위치

<br>

**장점**

- 호스트 영역에 미치는 영향을 최소화
- 데이터 볼륨 컨테이너가 직접 볼륨을 다루기 때문에 볼륨을 사용하는 컨테이너는 호스트 디렉토리를 알 필요가 없음
- 컨테이너 안에 든 애플리케이션 데이터와 결합이 느슨해져서 애플리케이션 컨테이너와 데이터 볼륨 교체 가능

<br>

## 4.3. Example: 데이터 볼륨에 MySQL 데이터 저장하기

**1) Dockerfile 작성**
   
```dockerfile
FROM busybox
 
VOLUME /var/lib/mysql
 
CMD ["bin/true"]
```

- busybox 는 운영 체제 기능만 제공하는 경량 운영 체제로, 도커 이미지의 기반 이미지로 많이 사용됨
- 데이터 볼륨 컨테이너는 데이터 저장만을 목적으로 하기 때문에 되도록 작은 이미지를 사용하는 것이 효과적

<br>

**2) 이미지 빌드**

```sh
$ docker image build -t example/mysql-data:latest .

[+] Building 3.7s (5/5) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                                                             0.0s
 => => transferring dockerfile: 98B                                                                                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                                                                0.0s
 => => transferring context: 2B                                                                                                                                                                  0.0s
 => [internal] load metadata for docker.io/library/busybox:latest                                                                                                                                3.1s
 => [1/1] FROM docker.io/library/busybox@sha256:930490f97e5b921535c153e0e7110d251134cc4b72bbb8133c6a5065cc68580d                                                                                 0.4s
 => => resolve docker.io/library/busybox@sha256:930490f97e5b921535c153e0e7110d251134cc4b72bbb8133c6a5065cc68580d                                                                                 0.0s
 => => sha256:dca71257cd2e72840a21f0323234bb2e33fea6d949fa0f21c5102146f583486b 527B / 527B                                                                                                       0.0s
 => => sha256:69593048aa3acfee0f75f20b77acb549de2472063053f6730c4091b53f2dfb02 1.46kB / 1.46kB                                                                                                   0.0s
 => => sha256:b71f96345d44b237decc0c2d6c2f9ad0d17fde83dad7579608f1f0764d9686f2 766.61kB / 766.61kB                                                                                               0.3s
 => => sha256:930490f97e5b921535c153e0e7110d251134cc4b72bbb8133c6a5065cc68580d 2.08kB / 2.08kB                                                                                                   0.0s
 => => extracting sha256:b71f96345d44b237decc0c2d6c2f9ad0d17fde83dad7579608f1f0764d9686f2                                                                                                        0.1s
 => exporting to image                                                                                                                                                                           0.0s
 => => exporting layers                                                                                                                                                                          0.0s
 => => writing image sha256:db889de8cbf48ade3aa96bf12ff812943708eb72b62ff34bc5cf00d65a3d3336                                                                                                     0.0s
 => => naming to docker.io/example/mysql-data:latest
```

<br>

**3) 데이터 볼륨 컨테이너 실행**

```sh
$ docker container run -d --name mysql-data example/mysql-data:latest
655f2b7301ef2964c434b0b2188b68aeee7288c05a6efeee5ead694ee438d611
```

- `mysql-data` 라는 이름으로 컨테이너 실행
- `CMD` 에서 셸을 실행하는 게 전부라서 실행이 끝나면 바로 컨테이너 종료

<br>

**4) MySQL 동작시킬 컨테이너 실행**

```sh
$ docker container run -d --rm --name mysql \
-e "MYSQL_ALLOW_EMPTY_PASSWORD=yes" \
-e "MYSQL_DATABASE=volume_test" \
-e "MYSQL_USER=example" \
-e "MYSQL_PASSWORD=example" \
--volumes-from mysql-data \
mysql:5.7

Unable to find image 'mysql:5.7' locally
5.7: Pulling from library/mysql
69692152171a: Pull complete
1651b0be3df3: Pull complete
951da7386bc8: Pull complete
0f86c95aa242: Pull complete
37ba2d8bd4fe: Pull complete
6d278bb05e94: Pull complete
497efbd93a3e: Pull complete
a023ae82eef5: Pull complete
e76c35f20ee7: Pull complete
e887524d2ef9: Pull complete
ccb65627e1c3: Pull complete
Digest: sha256:a682e3c78fc5bd941e9db080b4796c75f69a28a8cad65677c23f7a9f18ba21fa
Status: Downloaded newer image for mysql:5.7
161d68ffb294e4ded37b0aac43e526485026a58439150260a33c6d5181729ada
```

- 환경 변수로 데이터베이스 이름, 사용자명, 패스워드 설정
- `--volumes-from` 옵션을 사용해서 데이터 볼륨 컨테이너 `mysql-data` 를 MySQL 컨테이너에 마운트
- 이제 MySQL 컨테이너의 `/var/lib/mysql` 에는 데이터가 저장되지 않음

<br>

**5) 데이터 저장 쿼리 실행**

```sh
$ docker container exec -it mysql mysql -u root -p volume_test
Enter password: # 그냥 엔터 누르면됨

# CREATE 쿼리로 테이블 생성
mysql> CREATE TABLE user(
    ->         id int PRIMARY KEY AUTO_INCREMENT,
    ->         name VARCHAR(255)
    -> ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE utf8mb4_unicode_ci;
Query OK, 0 rows affected (0.01 sec)

# INSERT 쿼리로 데이터 저장
mysql> INSERT INTO user (name) VALUES ('gihyo'), ('docker'), ('alice');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

<br>

**6) 컨테이너 정지 및 파기 후 새로운 컨테이너 실행해서 데이터 확인**

```sh
# 컨테이너 정지 (--rm 옵션으로 실행했기 때문에 자동 파기)
$ docker container stop mysql
mysql

# MySQL 컨테이너 재실행
$ docker container run -d --rm --name mysql \
-e "MYSQL_ALLOW_EMPTY_PASSWORD=yes" \
-e "MYSQL_DATABASE=volume_test" \
-e "MYSQL_USER=example" \
-e "MYSQL_PASSWORD=example" \
--volumes-from mysql-data \
mysql:5.7
fcd544eab38e22df1f4b62f90e196484c252aab7055ae5e81f3217f9a9d9f69e

# SELECT 쿼리로 데이터 남아있는 지 확인
$ docker container exec -it mysql mysql -u root -p volume_test
Enter password:

mysql> SELECT * FROM user;
+----+--------+
| id | name   |
+----+--------+
|  1 | gihyo  |
|  2 | docker |
|  3 | alice  |
+----+--------+
3 rows in set (0.00 sec)
```

- 컨테이너 실행 시 `--rm` 옵션을 붙였기 때문에 정지하면 바로 파기
- 이렇게 애플리케이션 컨테이너와 데이터 볼륨 컨테이너를 분리해서 쉽게 데이터와 컨테이너 교체 가능