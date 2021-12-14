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

<br>

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

가장 간단한단 방법을 사용해서 쿠버네티스에 애플리케이션을 실행해봅니다.

보통은 배포 구성 요소를 기술한 JSON 이나 YAML 매니페스트를 준비해야합니다.

<br>

**2.3.1. Node.js 애플리케이션 구동하기**

이전에 도커 허브에 푸시한 이미지를 실행해본다.

```sh
# 도커 이미지를 실행하는 파드를 시작
$ kubectl create deployment --image=bcp0109/kubia kubia
deployment.apps/kubia created
```

- `--image`: 실행하고자 하는 컨테이너 이미지를 명시

<br>

**2.3.2. 파드 소개**

![](images/screen_2021_12_07_01_26_19.png)

쿠버네티스는 개별 컨테이너를 직접 다루지 않고 함께 배치된 다수의 컨테이너라는 개념을 사용합니다.

이 컨테이너의 그룹을 **파드 (Pod)** 라고 합니다.

각 파드는 자체 IP, 호스트 이름, 프로세스 등이 있는 논리적으로 분리된 머신입니다.

애플리케이션은 단일 컨테이너로 실행되는 단일 프로세스일 수도 있고, 주 애플리케이션 프로세스 (서버) 와 부가적으로 도와주는 프로세스 (DB) 로 이루어질 수 있습니다.

같은 워커 노드에서 실행중이라 할지라도 다른 파드에 실행 중인 컨테이너는 다른 머신에서 실행 중인 것으로 나타납니다.

<br>

**2.3.3. 파드 조회하기**

컨테이너는 독립적인 쿠버네티스 오브젝트가 아니기 때문에 **개별 컨테이너를 조회할 수 없고 대신 파드를 조회**해야 합니다.

```sh
$ kubectl get pods
```

<br>

**2.3.4. 백그라운드에서 일어난 동작**

![](images/screen_2021_12_07_03_52_23.png)

1. 우선 로컬에서 도커 허브에 이미지를 푸시하여 노드에서도 접근할 수 있게 해줍니다.
2. `kubectl` 명령어를 실행하면 쿠버네티스의 API 서버로 HTTP 요청을 전달하고 클러스터에 새로운 레플리케이션컨트롤러 오브젝트를 생성합니다.
3. 레플리케이션컨트롤러는 새 파드를 생성하고 워커 노드 중 하나에 스케줄링(할당) 됩니다.
4. 해당 워커 노드의 Kubelet 은 파드가 스케줄링 된 것을 보고 로컬에 없는 이미지를 Pull 하도록 도커에게 지시합니다.
5. 이미지를 다운로드 한 후 도커는 컨테이너를 생성 후 실행합니다.

<br>

**2.3.5. 웹 애플리케이션에 접근하기**

각 파드는 자체 IP 주소를 갖고 있지만 이 주소는 클러스터 내부에 있으며 외부에서 접근 불가능합니다.

외부에서 파드를 접근하기 위해선 서비스 오브젝트를 통해 노출해야 합니다.

```sh
# 서비스를 통해 포트를 노출
$ kubectl expose deployment kubia --type=LoadBalancer --port=8080 --name=kubia-http

service/kubia-http exposed
```

<br>

생성된 서비스는 명령어로 조회할 수 있습니다.

```sh
$ kubectl get svc
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1     <none>        443/TCP          2d
kubia-http   LoadBalancer   10.96.13.36   <pending>     8080:32294/TCP   12s


$ kubectl get svc
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1     <none>         443/TCP          2d
kubia-http   LoadBalancer   10.96.13.36   34.97.67.145   8080:32294/TCP   67s
```

처음에는 로드 밸런서를 생성하기 때문에 pending 상태였지만 나중에는 External IP 가 표시되는 걸 볼 수 있습니다.

이제 http://34.97.67.145:8080/ 주소로 어디서든 애플리케이션에 접근 가능합니다.

![](images/screen_2021_12_07_04_20_56.png)

<br>

## 2.6. 파드, 서비스의 이해

**2.6.1. 파드의 중요성**

시스템의 가장 중요한 구성 요소는 파드입니다.

컨테이너 내부에는 Node.js 프로세스가 있고 포트 8080 에 바인딩 돼 HTTP 요청을 기다리고 있습니다.

파드는 자체의 고유한 사설 IP 와 호스트 이름을 갖습니다.

<br>

**2.6.2. 서비스의 필요성**

파드는 일시적입니다.

파드가 실행 중인 노드가 실패할 수도 있고, 누군가 파드를 삭제할 수도 있고, 비정상 노드에서 파드가 제거될 수도 있습니다.

이러면 기존 파드가 사라지고 새로운 IP 주소가 할당된 파드로 대체되는데, 이렇게 항상 변경되는 파드에 접근하기 위해 서비스가 그 역할을 대신해줍니다.

서비스는 정적 IP 를 할당 받고 변경되지 않기 때문에 내부에 존재하는 파드의 IP 를 알 필요 없이 외부에서는 서비스를 통해 통신할 수 있습니다.

<br>

## 2.7. 요약

컨테이너는 애플리케이션의 실행 단위입니다.

컨테이너가 모여서 파드가 됩니다.

파드가 모여서 노드가 됩니다.

노드가 모여서 클러스터가 됩니다.

파드에는 서비스를 사용해서 접근합니다.

<br>

# 3장 파드: 쿠버네티스에서 컨테이너 실행

- 파드의 생성, 실행, 정지
- 파드와 다른 리소스를 레이블로 조직화
- 특정 레이블을 가진 모든 파드에서 작업 수행
- 네임스페이스를 이용해 파드를 겹치지 않는 그룹으로 나누기
- 특정한 형식을 가진 워커 노드에 파드 배치

<br>

## 3.1. 파드 소개

