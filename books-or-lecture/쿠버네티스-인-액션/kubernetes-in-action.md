# 쿠버네티스 인 액션

- [책 정보](http://www.yes24.com/Product/Goods/89607047)

<br>

# 1장 쿠버네티스 소개

![](images/screen_2021_12_04_20_29_55.png)

- 모놀리식 애플리케이션은 구축이 쉬운 반면에 유지보수가 어렵고 확장성이 떨어짐
- MSA 는 각 요소를 분리해서 유지보수와 확장성에 유리하지만 하나의 시스템으로 구성하는 게 어려움
- 리눅스 컨테이너는 가상머신에 비해 훨씬 가볍고 하드웨어 활용도를 높일 수 있음
- 개발자는 쿠버네티스를 통해 별도의 시스템 관리자 없이도 애플리케이션을 배포, 유지할 수 있음

<br>

# 2장 도커와 쿠버네티스 첫걸음

## 2.1. Docker 명령어 수행 시 발생하는 일

![](images/screen_2021_12_04_23_33_51.png)

- `docker run busybox echo "Hello world"` 를 실행했을 때 발생하는 일을 그림으로 나타냄
- `busybox` 뿐만 아니라 다른 이미지를 실행하는 것도 동일
- 다른 이미지를 실행할 때는 `echo` 명령어 같은것도 없기 때문에 오히려 더 간단
  - 실행할 명령어를 이미지 생성할 때 내부에 넣어서 같이 패키징 (오버라이드 가능)

<br>

## 2.2. 도커 이미지 버전

```sh
$ docker run <image>:<tag>
```

도커는 동일한 이미지와 이름에 여러 개의 버전을 가질 수 있습니다.

명시적으로 태그를 지정하지 않으면 `latest` 태그를 참조합니다.

<br>

## 2.3. 도커로 애플리케이션 실행

**2.3.1. 애플리케이션 파일 생성**

우선 실행하기 위한 간단한 애플리케이션 `app.js` 를 만듭니다.

```js
const http = require('http');
const os = require('os');

console.log("Kubia server starting...");

var handler = function(request, response) {
    console.log("Received request from " + request.connection.remoteAddress);
    response.writeHead(200);
    response.end("You've hit " + os.hostname() + "\n");
}

var www = http.createServer(handler);
www.listen(8080);
```

<br>

**2.3.2. Dockerfile 생성**

애플리케이션을 이미지로 패키징하기 위한 `Dockerfile` 파일을 생성합니다.

`Dockerfile` 은 `app.js` 와 동일한 디렉터리에 있어야 합니다.

```dockerfile
FROM node:7
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```

- `FROM`
  - 시작점이라고 하며 이미지 생성의 기반이 되는 기본 이미지입니다.
  - 여기서는 `node` 컨테이너 이미지의 태그 7 을 사용합니다.
  - 애플리케이션을 실행하기 위한 `node` 가 없으면 설치해줍니다
- `ADD`
  - 로컬 디렉터리의 `app.js` 파일을 이미지의 루트 디렉터리에 동일한 이름으로 추가합니다.
- `ENTRYPOINT`
  - 이미지를 실행했을 때 수행해야 할 명령어를 정의합니ㅏㄷ.

<br>

**2.3.3. 도커 이미지 빌드하여 컨테이너 이미지 생성**

![](images/screen_2021_12_05_00_24_06.png)

- `docker build -t kubia .` 명령어를 실행하면 현재 디렉터리 (`./`) 에 있는 `Dockerfile` 을 사용하여 `kubia` 라는 이름의 컨테이너 이미지를 생성합니다.
- 빌드 디렉터리의 모든 파일이 데몬에 업로드 되기 때문에 쓸데없는 파일은 같은 위치에 두지 않는게 좋습니다.

<br>

**2.3.4. 이미지 레이어란?**

![](images/screen_2021_12_05_00_36_05.png)

- 컨테이너 이미지는 하나의 큰 이미지로 되어 있는게 아니라 여러 개의 레이어로 구성되어 있습니다.
- 서로 다른 이미지가 여러 개의 레이어를 공유할 수 있기 때문에 이미지의 저장과 전송에 효과적입니다.
- 도커에서 이미지를 다운로드 할 때도 로컬에 없는 레이어만 받을 수 있습니다.

<br>

**2.3.5. 컨테이너 이미지 실행**

```sh
$ docker run --name kubia-container -p 8080:8080 -d kubia
```

- `--name <name>`
  - 컨테이너의 이름을 설정
  - 없으면 랜덤값으로 세팅
- `-p <local port>:<container port>`
  - 특정 포트로 연결
  - 로컬 머신의 8080 포트를 컨테이너 내부의 8080 포트와 매핑시킴
  - `http://localhost:8080` 으로 애플리케이션에 접근 가능
- `-d`
  - 백그라운드로 실행

<br>

**2.3.6. 실행중인 컨테이너 조회**

```sh
$ docker ps                                              
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                    NAMES
298367027b87   kubia     "node app.js"            10 minutes ago   Up 10 minutes   0.0.0.0:8080->8080/tcp   kubia-container
```

- `-a` 옵션을 추가하면 실행중인 컨테이너 뿐만 아니라 종료된 컨테이너도 모두 조회할 수 있습니다.

<br>

**2.3.7. 컨테이너에 관한 자세한 정보 얻기**

```sh
# docker inspect <container-name>
$ docker inspect kubia-container
```

<br>

**2.3.8. 실행중인 컨테이너 내부 탐색하기**

```sh
$ docker exec -it kubia-container bash
```

- 이 명령어를 사용하면 실행중인 `kubia-container` 컨테이너 내부에서 `bash` 를 실행합니다.
- `-i`: 표준 입력(STDIN) 을 오픈 상태로 유지합니다. 셸에 명령어를 입력하기 위해 필요합니다.
- `t`: 의사 (PSEUDO) 터미널 (TTY) 을 할당합니다.
- `i` 옵션을 빼면 명령어를 입력할 수 없고, `t` 옵션을 빼면 명령어 프롬포트가 화면에 표시되지 않습니다.

<br>

**2.3.9. 컨테이너 중지와 삭제**

```sh
# 실행중인 컨테이너 중지
$ docker stop kubia-container

# 컨테이너 완전히 삭제
$ docker rm kubia-container
```

- `docker stop` 명령어로 컨테이너 실행을 중지하면 `docker ps -a` 명령어에서는 종료된 컨테이너로 출력됩니다.
- 중지된 컨테이너는 `docker start` 명령어로 다시 실행할 수 있습니다.
- 만약 컨테이너를 완전히 삭제하려면 `docker rm` 명령어를 사용해야 합니다.

<br>

**2.3.10. 이미지 레지스트리에 이미지 푸시**

지금까지 빌드한 이미지는 로컬 컴퓨터에서만 사용 가능합니다.

다른 곳에서도 사용하려면 외부 이미지 저장소에 이미지를 푸시해야 합니다.

대표적으로 [도커 허브](https://hub.docker.com/)가 있습니다.

도커 허브에 이미지를 푸시하기 위해서는 먼저 가입 후 로그인을 진행해야 합니다.

이미지를 푸시하기 전에 도커 허브의 규칙에 따라 이미지 태그를 다시 지정해야 합니다.

이미지를 푸시하고 나면 다른 머신에서도 해당 이미지를 사용할 수 있습니다.

```sh
# 도커 허브 로그인
$ docker login

# kubia 이미지를 woody/kubia 라는 이름으로 복사 (같은 이미지 ID 가짐)
$ docker tag kubia bcp0109/kubia

# 도커 허브에 이미지 푸시
$ docker push bcp0109/kubia

# 도커 이미지 삭제
$ docker rmi bcp0109/kubia

# 푸시한 이미지로 컨테이너를 실행하면 로컬에 없기 때문에 다운 받은 후 실행됨
$ docker run -p 8080:8080 -d bcp0109/kubia
```

<br>

## 2.4. 쿠버네티스 클러스터 설치

지금까지 컨테이너 이미지에 애플리케이션을 패키징하고 도커 허브를 통해 사용할 수 있게 됐습니다.

이걸 도커에서 직접 실행하는 대신 쿠버네티스 클러스터에 배포할 수 있습니다.

그전에 쿠버네티스 클러스터를 설치해야 합니다.

**2.4.1. GKE 환경 설정**

1. [QuickStart 가이드](https://cloud.google.com/kubernetes-engine/docs/quickstart)를 참고해서 "가입 - 프로젝트 생성 - 빌링 생성 - 쿠버네티스 엔진 API 활성" 순으로 진행합니다.
2. [구글 클라우드 SDK](https://cloud.google.com/sdk/docs/install) 를 다운로드하고 설치합니다. (`gcloud` 를 포함하고 있음)
3. `gcloud components install kubectl` 명령어로 `kubectl` 설치

<br>

**2.4.2. 노드 세 개를 가진 쿠버네티스 클러스터 생성**

```sh
# 노드 3개 생성
$ gcloud container clusters create kubia --num-nodes 3

# 생성한 노드들 조회
$ kubectl get nodes
NAME                                   STATUS   ROLES    AGE   VERSION
gke-kubia-default-pool-90ff6c74-4n8b   Ready    <none>   36s   v1.21.5-gke.1302
gke-kubia-default-pool-90ff6c74-h08l   Ready    <none>   36s   v1.21.5-gke.1302
gke-kubia-default-pool-90ff6c74-k0k4   Ready    <none>   36s   v1.21.5-gke.1302
```

<br>

**2.4.3. 클러스터의 개념 이해하기**

![](images/screen_2021_12_05_04_09_05.png)

<br>

## 2.5. 쿠버네티스에 애플리케이션 실행하기

96p