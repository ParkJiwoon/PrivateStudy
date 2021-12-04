# Docker 명령어

## 1. docker run

```sh
$ docker run --name my-container -p 8080:8080 -d my-container-image
```

- `--name <name>`: 도커 컨테이너 이름을 설정
- `-d`: 백그라운드로 실행
- `-p <local port>:<container port>`
  - 특정 포트로 연결
  - 로컬 머신의 8080 포트를 컨테이너 내부의 8080 포트와 매핑시킴
  - `http://localhost:8080` 으로 애플리케이션에 접근 가능

컨테이너 이미지로 컨테이너를 실행합니다.

<br>

## 2. docker exec

```sh
# my_continaer 컨테이너에 echo 명령어 실행
$ docker exec my_container echo "hello world!"

# my_container 컨테이너의 bash 접속
$ docker exec -it my_container bash
```

- `-i`: 표준 입력(STDIN) 을 오픈 상태로 유지합니다. 셸에 명령어를 입력하기 위해 필요합니다.
- `-t`: 의사 (PSEUDO) 터미널 (TTY) 을 할당합니다.

도커 컨테이너에 명령어를 전달하여 실행합니다.

`i` 옵션을 빼면 명령어를 입력할 수 없고, `t` 옵션을 빼면 명령어 프롬포트가 화면에 표시되지 않습니다.

<br>

## 3. docker images

```sh
$ docker images
```

도커 이미지 목록을 조회합니다.

<br>

## 4. docker rmi

```sh
$ docker rmi my-docker-image
```

도커 이미지를 삭제합니다.

<br>

# Reference

- [Docker docs](https://docs.docker.com/engine/reference/commandline/docker/)