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

Go 언어로 만든 간단한 웹 서버를 도커 컨테이너에서 실행해보자

**1) 코드 작성**

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

**2) 만든 코드를 도커 컨테이너에 배치**

`main.go` 파일과 같은 디렉터리에 Dockerfile 을 작성

```docker
FROM golang:1.9
 
RUN mkdir /echo
COPY main.go /echo
 
CMD ["go", "run", "/echo/main.go"]
```

**3) 도커 이미지 빌드하기**




<br>

## 1.3. Dockerfile 인스트럭션

**1) FROM**

- 도커 이미지의 바탕이 될 베이스 이미지 지정
- Dockerfile 로 이미지를 빌드할 때 먼저 `FROM` 인스트럭션에 지정된 이미지를 내려받음
- `FROM` 에서 받아오는 도커 이미지는 도커 허브 (Docker Hub) 라는 레지스트리에 공개된 것
- 도커 이미지는 고유의 해시값을 갖는데, 해시만으론 해당 이미지가 무엇인지 특정하기가 어려워 특정 언어와 태그를 같이 사용하는 경우가 많음

**2) RUN**

- 도커 이미지를 실행할 때 컨테이너 안에서 실행할 명령

**3) COPY**

- 도커가 동작 중인 호스트 머신의 파일이나 디렉터리를 도커 컨테이너 안으로 복사
- 비슷한 명령어로 ADD 가 존재

**4) CMD**

- 도커 컨테이너를 실행할 때 컨테이너 안에서 실행할 프로세스
- `RUN` 은 이미지를 빌드할때 실행되고 `CMD` 는 컨테이너를 시작할 때 한번 실행
  