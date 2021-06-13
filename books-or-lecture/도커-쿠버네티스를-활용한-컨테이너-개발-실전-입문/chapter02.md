# 도커 컨테이너 배포

도커의 기본 조작 방법과 애플리케이션을 배포하는 과정까지 알아본다

- [도커 컨테이너 배포](#도커-컨테이너-배포)
- [1. 컨테이너로 애플리케이션 실행하기](#1-컨테이너로-애플리케이션-실행하기)
  - [1.1. 도커 이미지와 도커 컨테이너](#11-도커-이미지와-도커-컨테이너)
  - [1.2. 간단한 애플리케이션과 도커 이미지 만들기](#12-간단한-애플리케이션과-도커-이미지-만들기)
  - [1.3. 도커 이미지 빌드하기](#13-도커-이미지-빌드하기)
  - [1.4. 도커 컨테이너 실행](#14-도커-컨테이너-실행)
  - [1.5. 포트 포워딩](#15-포트-포워딩)
- [2. 도커 이미지 다루기](#2-도커-이미지-다루기)
  - [2.1. docker image build - 이미지 빌드](#21-docker-image-build---이미지-빌드)
  - [2.2. docker search - 이미지 검색](#22-docker-search---이미지-검색)
  - [2.3. docker image pull - 이미지 내려받기](#23-docker-image-pull---이미지-내려받기)
  - [2.4. docker image ls - 보유한 도커 이미지 목록 보기](#24-docker-image-ls---보유한-도커-이미지-목록-보기)
  - [2.5. docker image tag - 이미지에 태그 붙이기](#25-docker-image-tag---이미지에-태그-붙이기)
  - [2.6. docker image push - 이미지를 외부에 공개하기](#26-docker-image-push---이미지를-외부에-공개하기)
- [3. 도커 컨테이너 다루기](#3-도커-컨테이너-다루기)
  - [3.1. 도커 컨테이너의 생명주기](#31-도커-컨테이너의-생명주기)
  - [3.2. docker container run - 컨테이너 생성 및 실행](#32-docker-container-run---컨테이너-생성-및-실행)
  - [3.3. docker container ls - 도커 컨테이너 목록 보기](#33-docker-container-ls---도커-컨테이너-목록-보기)
  - [3.4. docker container stop - 컨테이너 정지하기](#34-docker-container-stop---컨테이너-정지하기)
  - [3.5. docker container restart - 컨테이너 재시작하기](#35-docker-container-restart---컨테이너-재시작하기)
  - [3.6. docker container rm - 컨테이너 파기하기](#36-docker-container-rm---컨테이너-파기하기)
  - [3.7. docker container logs - 표준 출력 연결하기](#37-docker-container-logs---표준-출력-연결하기)
  - [3.8. docker container exec - 실행 중인 컨테이너에서 명령 실행하기](#38-docker-container-exec---실행-중인-컨테이너에서-명령-실행하기)
  - [3.9. docker container cp - 파일 복사하기](#39-docker-container-cp---파일-복사하기)
- [4. 운영과 관리를 위한 명령](#4-운영과-관리를-위한-명령)
  - [4.1. prune - 컨테이너 및 이미지 파기](#41-prune---컨테이너-및-이미지-파기)
  - [4.2. docker container stats - 사용 현황 확인하기](#42-docker-container-stats---사용-현황-확인하기)
- [5. 도커 컴포즈](#5-도커-컴포즈)
  - [5.1. 도커 컴포즈 확인](#51-도커-컴포즈-확인)
  - [5.2. 도커 컴포즈로 실행하기](#52-도커-컴포즈로-실행하기)
- [6. 컴포즈로 여러 컨테이너 실행하기](#6-컴포즈로-여러-컨테이너-실행하기)
  - [6.1. 젠킨스 컨테이너 실행하기](#61-젠킨스-컨테이너-실행하기)
  - [6.2. 슬레이브 젠킨스 컨테이너 실행](#62-슬레이브-젠킨스-컨테이너-실행)

<br>

# 1. 컨테이너로 애플리케이션 실행하기

도커 이미지 하나로 여러 개의 컨테이너 생성 가능

- **도커 이미지**: 도커 컨테이너를 구성하는 파일 시스템과 실행할 애플리케이션 설정을 하나로 합친 것으로, 컨테이너를 생성하는 템플릿 역할을 함
- **도커 컨테이너**: 도커 이미지를 기반으로 생성되며, 파일 시스템과 애플리케이션이 구체화돼 실행되는 상태

<br>

## 1.1. 도커 이미지와 도커 컨테이너

도커 이미지로 도커 컨테이너를 만드는 과정을 한번 알아보자

<br>

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

<br>

**2) 도커 이미지 실행**

```sh
$ docker container run -t -p 9000:8080 gihyodocker/echo:latest
2021/06/10 17:29:03 start server
```

- 지금 만든 컨테이너는 옵션을 통해 포트 포워딩이 적용되어 있음
- 도커 실행 환경의 포트 9000 을 거쳐 HTTP 요청을 전달 받음

<br>

**3) 애플리케이션 실행 확인**

```sh
$ curl http://localhost:9000/
Hello Docker!!
```

- curl 명령어로 호출하면 정상적으로 동작되는 것을 확인 가능

<br>

**4) 도커 컨테이너 종료**

```sh
$ docker stop $(docker container ls -q)
dfc9aa5b2646
```

<br>

## 1.2. 간단한 애플리케이션과 도커 이미지 만들기

Go 언어로 만든 간단한 웹 서버를 도커 컨테이너에서 실행해보자

<br>

**1) 애플리케이션 코드 작성**

```go
// main.go

package main
 
import (
    "fmt"
    "log"
    "net/http"
)
 
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        log.Println("received request")
        fmt.Fprintf(w, "Hello Docker!!")
    })
 
    log.Println("start server")
    server := &http.Server{Addr: ":8080"}
    if err := server.ListenAndServe(); err != nil {
        log.Println(err)
    }
}
```

- 모든 HTTP 요청에 대해 'Hello Docker!!'라는 응답을 보냄
- 포트 8080 으로 요청을 받는 서버 애플리케이션
- 클라이언트로부터 요청을 받으면 `received request` 라는 메시지를 출력

<br>

**2) Dockerfile 작성**

`main.go` 파일과 같은 디렉터리에 Dockerfile 을 작성

```docker
FROM golang:1.9
 
RUN mkdir /echo
COPY main.go /echo
 
CMD ["go", "run", "/echo/main.go"]
```

- FROM
  - 도커 이미지의 바탕이 될 베이스 이미지 지정
  - Dockerfile 로 이미지를 빌드할 때 먼저 `FROM` 인스트럭션에 지정된 이미지를 내려받음
  - `FROM` 에서 받아오는 도커 이미지는 도커 허브 (Docker Hub) 라는 레지스트리에 공개된 것
  - 도커 이미지는 고유의 해시값을 갖는데, 해시만으론 해당 이미지가 무엇인지 특정하기가 어려워 특정 언어와 태그를 같이 사용하는 경우가 많음

- RUN
  - 도커 이미지를 실행할 때 컨테이너 안에서 실행할 명령

- COPY
  - 도커가 동작 중인 호스트 머신의 파일이나 디렉터리를 도커 컨테이너 안으로 복사
  - 비슷한 명령어로 ADD 가 존재

- CMD
  - 도커 컨테이너를 실행할 때 컨테이너 안에서 실행할 프로세스
  - `RUN` 은 이미지를 빌드할때 실행되고 `CMD` 는 컨테이너를 시작할 때 한번 실행

<br>

## 1.3. 도커 이미지 빌드하기

`docker image build` 명령으로 빌드

`-t` 옵션으로 이미지명 지정, 태그명도 지정 가능ㅎ며 생략 시 latest 태그가 붙음

```sh
# 도커 이미지 빌드 문법: docker image build -t <이미지명[:태그명]> <Dockerfile 경로>
$ docker image build -t example/echo:latest .

# 빌드된 도커 이미지 확인
$ docker image ls
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
example/echo       latest    c301cf8b8d7c   2 minutes ago   750MB
```

- `example` 은 네임스페이스며 이미지 이름 충돌 방지를 위해 붙이는 것이 좋음

<br>

## 1.4. 도커 컨테이너 실행

```sh
$ docker container run example/echo:latest
2021/06/12 15:09:12 start server
```

- `docker container run` 명령어로 실행 가능
- 포어그라운드에서 계속 동작함
- `docker stop $(docker container ls -q)` 명령어로 종료 가능

<br>

```sh
$ docker container run -d example/echo:latest
d13792d7cc6dae6c7ddebf47a17efc92b54bfdc989d2a9c4cac4f0a3699363b4
```

- `-d` 옵션을 붙여서 백그라운드로 컨테이너실행
- 백그라운드로 실행하면 도커 컨테이너의 ID (해시값) 출력

<br>

```sh
$ docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS     NAMES
d13792d7cc6d   example/echo:latest   "go run /echo/main.go"   45 seconds ago   Up 44 seconds             bold_blackburn
```

- 실행중인 컨테이너 목록 조회

<br>

## 1.5. 포트 포워딩

```sh
$ curl http://localhost:8080/
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

- 위에서 작성한 Go 언어 애플리케이션은 포트 8080 을 사용
- 그러나 8080 으로 GET 요청을 보내도 `Connection refused` 발생

<br>

도커 컨테이너에서 실행한 애플리케이션의 포트는 컨테이너 포트라고 함

curl 을 컨테이너 안에서 실행하면 올바르게 동작하지만 컨테이너 밖에서는 컨테이너 안의 포트를 바로 사용 불가능

**컨테이너 밖에서 온 요청을 컨테이너 안에 있는 애플리케이션에 전달하는 게 도커의 포트 포워딩**

호스트 머신의 포트를 컨테이너 포트와 연결해서 전달하여 컨테이너 포트를 컨테이너 외부에서 이용 가능하게 만듬

<br>

```sh
# -p 옵션으로 포트 포워딩 지정
$ docker container run -d -p 9000:8080 example/echo:latest
c498e3816dbd220f1777d6cd9b8577d4b0cfe53de145e7e4b2ddf2570a680fb9

# 포트 포워딩으로 호스트 포트에 요청하면 컨테이터 포트에 전달됨
$ curl http://localhost:9000/
Hello Docker!!

# 할당된 포트 확인 가능
$ docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                       NAMES
c498e3816dbd   example/echo:latest   "go run /echo/main.go"   2 minutes ago   Up 2 minutes   0.0.0.0:9000->8080/tcp, :::9000->8080/tcp   elastic_burnell
```

- `-p <호스트 포트:컨테이너 포트>` 옵션을 사용해서 포트 포워딩 지정
- 위 명령어는 호스트 포트 9000 을 컨테이너 포트 8080 에 연결
- 어떤 포트가 할당되었는지는 `ls` 명령어로 확인 가능

<br>

# 2. 도커 이미지 다루기

도커 이미지란? 도커 컨테이너를 만들기 위한 템플릿

- 도커 이미지 빌드
- 이미지를 다루는 기본 명령
- 도커 허브에 이미지 등록 과정

Docker 이미지 명령어를 보려면 `docker image --help` 를 사용할 수 있음

```html
$ docker image --help

Usage:  docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
```

<br>

## 2.1. docker image build - 이미지 빌드

```sh
# docker image build -t <이미지명[:태그명]> <Dockerfile 경로>

# -t 옵션 : 이미지명과 태그명 붙이기. 거의 필수적으로 사용
$ docker image build -t example/echo:latest .

# -f 옵션 : Dockerfile 파일명 (기본값: Dockerfile)
$ docker image build -f Dockerfile-test -t example/echo:latest .

# --pull 옵션 : 매번 base 이미지를 강제로 새로 받기
$ docker image build --pull=true -t example/echo:latest .
```

- 일반적으로 `FROM` 명령어로 지정된 이미지는 레지스트리에서 다운받은 Base 이미지를 기반으로 새로운 이미지를 빌드함
- 이렇게 레지스트리에서 받아온 도커 이미지는 일부러 삭제하지 않는 한 호스트 OS 에 저장됨
- 그러나 `--pull` 옵션을 상요하면 매번 Base 이미지를 새로 받음

<br>

## 2.2. docker search - 이미지 검색

```sh
# docker search [options] <검색 키워드>

$ docker search --limit 5 mysql

NAME                  DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                 MySQL is a widely used, open-source relation…   10990     [OK]
mysql/mysql-server    Optimized MySQL Server Docker images. Create…   814                  [OK]
mysql/mysql-cluster   Experimental MySQL Cluster Docker images. Cr…   85
bitnami/mysql         Bitnami MySQL Docker Image                      52                   [OK]
circleci/mysql        MySQL is a widely used, open-source relation…   20
```

- Docker Hub 에 등록된 레포지토리 검색
- 검색 결과 처음에 나오는 네임스페이스가 생략된 레포지토리는 mysql 공식 레포지토리
- 검색 결과는 Stars 순

<br>

## 2.3. docker image pull - 이미지 내려받기

```sh
# docker image pull [options] <레포지토리명[:태그명]>

$ docker image pull jenkins:2.60.3

2.60.3: Pulling from library/jenkins
55cbf04beb70: Already exists
1607093a898c: Already exists
9a8ea045c926: Already exists
d4eee24d4dac: Already exists
c58988e753d7: Pull complete
794a04897db9: Pull complete
70fcfa476f73: Pull complete
0539c80a02be: Pull complete
54fefc6dcf80: Pull complete
911bc90e47a8: Pull complete
38430d93efed: Pull complete
7e46ccda148a: Pull complete
c0cbcb5ac747: Pull complete
35ade7a86a8e: Pull complete
aa433a6a56b1: Pull complete
841c1dd38d62: Pull complete
b865dcb08714: Pull complete
5a3779030005: Pull complete
12b47c68955c: Pull complete
1322ea3e7bfd: Pull complete
Digest: sha256:eeb4850eb65f2d92500e421b430ed1ec58a7ac909e91f518926e02473904f668
Status: Downloaded newer image for jenkins:2.60.3
docker.io/library/jenkins:2.60.3
```

- 레포지토리명과 태그명은 도커 허브에 존재해야 함
- 태그명을 생략하면 기본값으로 지정된 태그가 적용 (보통 latest)

<br>

## 2.4. docker image ls - 보유한 도커 이미지 목록 보기

```sh
# docker image ls [options] <레포지토리[:태그]>

$ docker image ls

REPOSITORY         TAG       IMAGE ID       CREATED       SIZE
example/echo       latest    c301cf8b8d7c   2 hours ago   750MB
jenkins            2.60.3    cd14cecfdb3a   2 years ago   696MB
gihyodocker/echo   latest    3dbbae6eb30d   3 years ago   733MB
```

- 호스트 운영 체제에 저장된 도커 이미지 목록을 보여줌
- `pull` 로 원격에서 받은 이미지 뿐만 아니라 `build` 로 내려받은 이미지도 호스트 운영 체제에 저장됨

<br>

## 2.5. docker image tag - 이미지에 태그 붙이기

```sh
# docker image tag <기반이미지명[:태그]> <새 이미지명[:태그]>

# 0.1.0 태그 부여
$ docker image tag example/echo:latest example/echo:0.1.0

# IMAGE ID 가 같으므로 같은 이미지를 가리킨다는 것을 알 수 있음
$ docker image ls

REPOSITORY         TAG       IMAGE ID       CREATED       SIZE
example/echo       0.1.0     c301cf8b8d7c   2 hours ago   750MB
example/echo       latest    c301cf8b8d7c   2 hours ago   750MB
```

<br>

## 2.6. docker image push - 이미지를 외부에 공개하기

```sh
# docker image push [options] <레포지토리명[:태그]>

# 실제로 호출하면 access denied 뜸
$ docker image push example/echo:latest
```

- 저장된 도커 이미지를 도커 허브 등의 레지스트리에 등록
- 도커 허브는 자신 혹은 소속 기관이 소유한 레포지토리에만 이미지 등록 가능하기 때문에 네임스페이스 변경 필요
- 도커 허브에 푸시하려면 로그인 필요

<br>

# 3. 도커 컨테이너 다루기

도커 컨테이너는 가상 환경

파일 시스템과 애플리케이션이 함께 담겨 있는 박스라고 보면 됨

<br>

## 3.1. 도커 컨테이너의 생명주기

도커 컨테이너는 3 가지 상태를 가짐

각 컨테이너는 같은 이미지로 생성했따고 해도 별개의 상태를 가짐

이 점이 상태를 갖지 않는 도커 이미지와 컨테이너의 큰 차이

- 실행 중
  - `docker container run` 명령어로 실행시킨 상태
  - 실행이 끝나면 정지 상태가 됨
  - 애플리케이션 보통 종료하지 않는 이상 계속 실행 상태고, 단순히 출력만 하고 끝나버리는 코드도 있음
- 정지
  - 실행 중인 컨테이너를 사용자가 명시적으로 정지 (`docker container stop`) 하거나 컨테이너에서 실행된 애플리케이션의 정상/오류 여부를 막론하고 종료된 상태
  - 컨테이너를 정지해도 디스크에 종료 시점의 상태가 남기 때문에 정지 시킨 컨테이너를 다시 실행할 수 있음
  - `docker container ls -a` 명령어로 정지시킨 걸 포함한 모든 컨테이너를 확인 가능
- 파기
  - 정지 상태의 컨테이너는 명시적으로 파기하지 않는 이상 디스크에 그대로 남아 있음
  - 불필요한 컨테이너는 삭제 가능하며 한번 파기한 컨테이너는 다시 실행 불가능

<br>

## 3.2. docker container run - 컨테이너 생성 및 실행

```sh
# docker container run [options] <이미지명[:태그]> [<명령>] [<명령인자>..]
# docker container run [options] <이미지 ID> [<명령>] [<명령인자>..]

# example/echo:latest 이미지로 컨테이너를 백그라운드에서 실행
$ docker container run -d -p 9000:8080 example/echo:latest
2dd1db7bf713f962fd9cf29b5bd6ba91776c850300e8337f8cabe16a0938820f

## 컨테이너에 이름 붙여서 실행하기
# docker container run --name [<컨테이너명>] [<이미지명>]:[<태그>]

$ docker container run -d --name gihyo-echo example/echo:latest

$ docker container ls

CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                       NAMES
18d3c0feddca   example/echo:latest   "go run /echo/main.go"   4 seconds ago   Up 3 seconds                                               gihyo-echo
```

- `-d`
  - 백그라운드로 실행
- `-p`
  - 포트 포워딩 (호스트 포트: 9000 -> 컨테이너 포트: 8080)
- `--name`
  - 실행할 컨테이너에 이름을 붙임 (이 옵션을 사용하지 않으면 랜덤값으로 부여됨)
  - 같은 이름의 컨테이너를 새로 실행하려면 기존 컨테이너를 삭제해야 하기 때문에 개발용으로는 자주 사용되지만 운영 환경에서는 거의 사용되지 않음
- `--rm`
  - 컨테이너를 종료할 때 파기
  - 한번 실행한 후에 더이상 유지할 필요가 없는 명령행 도구 컨테이너를 실행할 때 유용
- `-v`
  - 컨테이너 간에 디렉터리나 파일을 공유하기 위해 사용

<br>

## 3.3. docker container ls - 도커 컨테이너 목록 보기

실행 중이거나 종료된 컨테이너의 목록을 보여줌

```sh
# docker container ls [optinos]

# --filter 옵션 사용
$ docker container ls --filter "name=gihyo-echo" # 컨테이너명
$ docker container ls --filter "ancestor=example/echo" # 이미지명
```

- `-q`
  - 컨테이너 ID 만 추출
- `-a`
  - 종료된 컨테이너를 포함한 모든 컨테이너 목록 조회
- `--filter`
  - 특정 조건을 만족하는 컨테이너 목록 조회

<br>

## 3.4. docker container stop - 컨테이너 정지하기

```sh
# docker container stop <컨테이너 ID 또는 컨테이너명>

$ docker container run -d -p 9000:8080 example/echo:latest
76860e335dfa643979fb740bc0faf12941c49bfff719ae8eda420780102b3c38

$ docker container stop 76860e335dfa643979fb740bc0faf12941c49bfff719ae8eda420780102b3c38
76860e335dfa643979fb740bc0faf12941c49bfff719ae8eda420780102b3c38
```

- 실행 중인 컨테이너 종료

<br>

## 3.5. docker container restart - 컨테이너 재시작하기

```sh
# docker container restart <컨테이너 ID 또는 컨테이너명>

$ docker container restart gihyo-echo
gihyo-echo

$ docker container ls

CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS         PORTS     NAMES
18d3c0feddca   example/echo:latest   "go run /echo/main.go"   12 minutes ago   Up 4 seconds             gihyo-echo
```

- 파기되지 않은 정지 상태 컨테이너 재시작

<br>

## 3.6. docker container rm - 컨테이너 파기하기

```sh
# docker container rm [options] <컨테이너 ID 또는 컨테이너명>

# 1. 정지된 컨테이너 확인
$ docker container ls -a

CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                     PORTS     NAMES
76860e335dfa   example/echo:latest       "go run /echo/main.go"   2 minutes ago    Exited (2) 2 minutes ago             heuristic_sutherland

# 2. 파기
$ docker container rm heuristic_sutherland
heuristic_sutherland

# 3. 파기 여부 확인
$ docker container ls -a

CONTAINER ID   IMAGE                     COMMAND                  CREATED          STATUS                     PORTS     NAMES
```

- 정지 상태의 컨테이너를 완전히 파기
- 정지 상태의 컨테이너가 쌓이면 디스크 용량을 차지하기 때문에 정리 필요
- 같은 이름을 가진 컨테이너가 존재하는 경우 새로운 컨테이너를 생성하려면 기존 컨테이너 파기 필요
- `-f`
  - 실행중인 컨테이너 강제로 파기

<br>

## 3.7. docker container logs - 표준 출력 연결하기

```sh
# docker container logs [options] <컨테이너 ID 또는 컨테이너명>
```

- 실행 중인 특정 컨테이너의 로그를 확인 가능
- 실제로 운영할 때는 다른 툴로 로그를 볼 수 있기 때문에 실제로 사용하는 경우는 많지 않음
- `-f`
  - 새로 출력되는 로그를 계속 보여줌 (테일링)

<br>

## 3.8. docker container exec - 실행 중인 컨테이너에서 명령 실행하기

```sh
# docker container exec [options] <컨테이너 Id 또는 컨테이너명> <컨테이너에서 실행할 명령>

# 컨테이너에 pwd 명령어 실행
$ docker container exec gihyo-echo pwd
/go

# 해당 컨테이너에 접속한 것처럼 사용하고 싶을 때
$ docker container exec -it gihyo-echo sh
# pwd
/go
# ls
bin  src
```

- 컨테이너에 ssh 로 접속한 것처럼 컨테이너 내부 조작 가능
- 컨테이너 내부의 파일을 수정하는 건 부작용을 초래할 수 있기 때문에 운영 환경에서는 하면 안됨

<br>

## 3.9. docker container cp - 파일 복사하기

```sh
# docker container cp [options] <컨테이너 ID 또는 컨테이너명>:<원본파일> <대상파일>
# docker container cp [options] <호스트 원본파일> <컨테이너 ID 또는 컨테이너명>:<대상파일>

# 컨테이너 안의 파일을 호스트로 복사
$ docker container cp echo:/echo/main.go .

# 호스트의 파일을 컨테이너로 복사
$ docker container cp dummy.txt echo:/tmp
```

- 컨테이너끼리 또는 컨테이너와 호스트 간의 파일 복사
- 아직 파기되지 않은 정지 상태 컨테이너에도 실행 가능

<br>

# 4. 운영과 관리를 위한 명령

지금까지는 이미지와 컨테이너를 다루는 명령어를 살펴봄

도커를 운영하고 관리하기 위한 명령어도 있음

<br>

## 4.1. prune - 컨테이너 및 이미지 파기

- 컨테이너 파기
  - 명령어: `docker container prune [options]`
  - 실행 중이 아닌 모든 컨테이너 삭제
- 이미지 파기
  - 명령어: `docker image prune [options]`
  - 태그가 붙지 않은 모든 이미지 삭제
  - 실행중인 컨테이너의 이미지는 삭제되지 않음
- 모든 리소스 삭제
  - 명령어: `docker system prune`
  - 사용하지 않는 도커 이미지, 컨테이너, 볼륨, 네트워크 등 일괄 삭제

<br>

## 4.2. docker container stats - 사용 현황 확인하기

```sh
# docker container starts [options] [<대상 컨테이너 ID> ...]

$ docker container stats

CONTAINER ID   NAME                 CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O     PIDS
7520a39f0f3b   charming_lederberg   0.00%     9.738MiB / 1.941GiB   0.49%     946B / 0B   0B / 8.19kB   15
```

- 시스템 리소스 사용 현황 확인
- 유닉스 계열 운영 체제의 `top` 명령어와 같은 역할

<br>

# 5. 도커 컴포즈

실용적인 수준의 시스템을 구축하려면 여러 개의 애플리케이션이 통신하며 의존 관계를 형성해야한다.

이 말은 여러 개의 컨테이너가 서로 의존 관계를 갖게 되기 때문에 단일 컨테이너와 달리 신경 써야 하는 부분이 많다.

이럴 때 필요한게 **도커 컴포즈 (Docker Compose)** 다.

도커 컴포즈는 yaml 포맷으로 기술된 설정 파일로, 여러 컨테이너의 실행을 한번에 관리할 수 있게 해준다.

<br>

## 5.1. 도커 컴포즈 확인

```sh
# 도커가 설치되어 있다면 바로 사용 가능
$ docker-compose version

docker-compose version 1.29.2, build 5becea4c
docker-py version: 5.0.0
CPython version: 3.9.0
OpenSSL version: OpenSSL 1.1.1h  22 Sep 2020
```

<br>

## 5.2. 도커 컴포즈로 실행하기

**1) docker-compose.yml 파일 작성**

```yaml
version: "3"
services:
  echo:
    image: example/echo:latest
    ports:
      - 9000:8080
```

- `version: "3"`: 도커 컴포즈 파일을 해석 하는데 필요한 문법 버전
- `echo`: 컨테이너 이름
- `image`: 도커 이미지
- `ports`: 포트 포워딩 설정

<br>

**2) docker-compose.yml 파일이 위치한 디렉터리에서 명령어 입력**

```sh
# 도커 컴포즈로 컨테이너 실행
$ docker-compose up -d
Starting practice-docker_echo_1 ... done

# 도커 컨테이너 실행 확인
$ docker container ls
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                       NAMES
9650541e5c0a   example/echo:latest   "go run /echo/main.go"   39 seconds ago   Up 20 seconds   0.0.0.0:9000->8080/tcp, :::9000->8080/tcp   practice-docker_echo_1

# 도커 컴포즈 파일에 정의된 모든 컨테이너 정지
$ docker-compose down
Stopping practice-docker_echo_1 ... done
Removing practice-docker_echo_1 ... done
Removing network practice-docker_default
```

<br>

**3) image 속성 대신 build 속성으로 변경**

```yaml
# docker-compose.yml 파일 수정

version: "3"
services:
  echo:
    build: .
    ports:
      - 9000:8080
```

- Dockerfile 이 위치한 경로를 지정

<br>

**4) 도커 컨테이너 실행**

```sh
❯ docker-compose up -d --build

Creating network "practice-docker_default" with the default driver
Building echo
[+] Building 2.4s (8/8) FINISHED
 => [internal] load build definition from Dockerfile                                                                                            0.0s
 => => transferring dockerfile: 132B                                                                                                            0.0s
 => [internal] load .dockerignore                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                 0.0s
 => [internal] load metadata for docker.io/library/golang:1.9                                                                                   2.2s
 => [1/3] FROM docker.io/library/golang:1.9@sha256:8b5968585131604a92af02f5690713efadf029cc8dad53f79280b87a80eb1354                             0.0s
 => [internal] load build context                                                                                                               0.0s
 => => transferring context: 434B                                                                                                               0.0s
 => CACHED [2/3] RUN mkdir /echo                                                                                                                0.0s
 => CACHED [3/3] COPY main.go /echo                                                                                                             0.0s
 => exporting to image                                                                                                                          0.0s
 => => exporting layers                                                                                                                         0.0s
 => => writing image sha256:c301cf8b8d7c6ea23d669d4419e8aea1a1f6690010f05ea782f2b37f79cbacfd                                                    0.0s
 => => naming to docker.io/library/practice-docker_echo                                                                                         0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them
Creating practice-docker_echo_1 ... done
```

- 기본적으로는 빌드한 이미지가 없다면 빌드를 먼저 하지만 `--build` 옵션을 사용하면 강제로 리빌드 가능
- 이미지가 자주 수정되는 경우에 유용

<br>

# 6. 컴포즈로 여러 컨테이너 실행하기

도커 컴포즈의 진가는 여러 컨테이너를 실행할 때 발휘된다.

젠킨스를 예로 들어서 컴포즈로 실행해보자.

<br>

## 6.1. 젠킨스 컨테이너 실행하기

**1) docker-compose.yml 파일 작성**

```yaml
version: "3"
services:
  master:
    container_name: master
    image: jenkins/jenkins
    ports:
      - 8080:8080
    volumes:
      - ./jenkins_home:/var/jenkins_home
```

- 젠킨스 이미지는 도커 허브에 있는걸 사용
- `volumes` 는 호스트와 컨테이너 사이에 파일을 공유할 수 있음
- 호스트의 `./jenkins_home` 경로와 컨테이너의 `/var/jenkins_home` 경로를 연결

<br>

**2) 포어그라운드로 컴포즈 실행**

```sh
$ docker-compose up

Creating network "example-jenkins_default" with the default driver
Pulling master (jenkins/jenkins:)...
latest: Pulling from jenkins/jenkins
d960726af2be: Pull complete
971efeb01290: Pull complete
...
...
master    | *************************************************************
master    | *************************************************************
master    | *************************************************************
master    |
master    | Jenkins initial setup is required. An admin user has been created and a password generated.
master    | Please use the following password to proceed to installation:
master    |
master    | 6042768596664e6aa004f8297fee7ef1 # password
master    |
master    | This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
master    |
master    | *************************************************************
master    | *************************************************************
master    | *************************************************************
master    |
master    | 2021-06-13 14:12:22.533+0000 [id=32]	INFO	jenkins.InitReactorRunner$1#onAttained: Completed initialization
master    | 2021-06-13 14:12:22.585+0000 [id=21]	INFO	hudson.WebAppMain$3#run: Jenkins is fully up and running
master    | 2021-06-13 14:12:23.159+0000 [id=51]	INFO	h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
master    | 2021-06-13 14:12:23.160+0000 [id=51]	INFO	hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1
master    | 2021-06-13 14:12:23.182+0000 [id=51]	INFO	hudson.model.AsyncPeriodicWork#lambda$doRun$0: Finished Download metadata. 13,590 ms
```

- 초기 설정에서 패스워드가 생성되는데 복사해두기 (`6042768596664e6aa004f8297fee7ef1`)

<br>

**3) 젠킨스 접속**

- `http://localhost:8080/` 접속해서 위에서 복사해둔 비밀번호 입력
- 'Install suggested plugins' 클릭하면 젠킨스 홈 화면에 접속 가능
- 젠킨스 공식 이미지에서는 `/var/jenkins_home` 아래에 데이터가 저장되기 때문에 컴포즈로 실행한 젠킨스를 종료했다가 재시작해도 초기 설정 유지됨

<br>

## 6.2. 슬레이브 젠킨스 컨테이너 실행

실제로 젠킨스 운영 할 때 단일 서버로 하는 경우는 많지 않음

관리/작업지시 등은 마스터가 맡고 작업 진행은 슬레이브가 담당

<br>

**1) 마스터 젠킨스용 SSH 키 생성**

```sh
$ docker container exec -it master ssh-keygen -t rsa -C ""

Generating public/private rsa key pair.
Enter file in which to save the key (/var/jenkins_home/.ssh/id_rsa):
Created directory '/var/jenkins_home/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/jenkins_home/.ssh/id_rsa.
Your public key has been saved in /var/jenkins_home/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:FGzft9KIF0/87EJFjAX9M3y93RpsQvV/GhWXWR2sSoo
The key's randomart image is:
+---[RSA 2048]----+
|       ..     oBO|
|        o.    o=*|
|       ... . ooo=|
|       .  ..+.++B|
|        S oo.O *O|
|       E ...= X.B|
|           . = *.|
|              + .|
|               . |
+----[SHA256]-----+
```

- 마스터가 슬레이브에 접속할 수 있도록 마스터 컨테이너에서 SSH 키 생성
- 만들어진 `/var/jenkins_home/.ssh/id_rsa.pub` 파일은 마스터 젠킨스가 슬레이브 젠킨스에 접속할 때 사용할 키

<br>

**2) 슬레이브 젠킨스 컨테이너 생성**

```yml
version: "3"
services:
  master:
    container_name: master
    image: jenkins/jenkins
    ports:
      - 8080:8080
    volumes:
      - ./jenkins_home:/var/jenkins_home
    links:
      - slave01

  slave01:
    container_name: slave01
    image: jenkinsci/ssh-slave
    environment:
      - JENKINS_SLAVE_SSH_PUBKEY=ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCqX2Ppf6OGPwjVwBPh9crF+v6WrvSCxlmOQMT4lysNfxmSDZEVcxRmMKe8j7MiJzwE5ZU8JBVqnUnRW8LaMAVxpH8xTSRK4XvMTe/wxr72fMwaHqgXnS8vzhQHawUq+JLlI7rI5rbfbAkw1f/1lEu8rw69rhGA0SVsJqAsutRKer8/1TmbjpmPeddQVzv/rQ/FOT7CBb1Ac2cobiftqxr9/Cihvs06ogZYiHFu/mOUrfXovGylo8Ii81U6PcE3bzUSlwBCK2uh9X1OWHxdKIfPy1Z5CMBXQySF3IZar78GecOl8gVe5hFTc2/bpK71H3kG1SRwbvi69yX2e4uZXgzL
```

- docker-compose.yml 설정 변경
- `jenkinsci/ssh-slave` 이미지에 환경 변수 `JENKINS_SLAVE_SSH_PUBKEY` 를 설정해서 SSH 로 접속하는 상대가 이 키를 보고 마스터 젠킨스 여부를 식별
- `JENKINS_SLAVE_SSH_PUBKEY` 값은 호스트의 `./jenkins_home/.ssh/id_rsa.pub` 값을 그대로 사용
- `links` 
  - 다른 services 그룹에 해당하는 컨테이너와 통신 가능
  - 여기선 마스터 컨테이너가 슬레이브 컨테이너를 찾을 수 있게 추가

<br>

**3) 마스터/슬레이브 컨테이너 실행**

```sh
# 도커 컴포즈 실행
$ docker-compose up -d

# 마스터/슬레이브 컨테이너 동시에 실행됨
$ docker-compose ps

 Name                Command               State                          Ports
------------------------------------------------------------------------------------------------------
master    /sbin/tini -- /usr/local/b ...   Up      50000/tcp, 0.0.0.0:8080->8080/tcp,:::8080->8080/tcp
slave01   setup-sshd                       Up      22/tcp
```
