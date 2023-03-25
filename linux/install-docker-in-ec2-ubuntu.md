# EC2 Ubuntu 에 Docker 설치

# Overview

Docker 공식 홈페이지에 있는 [Ubuntu 설치](https://docs.docker.com/engine/install/ubuntu/)를 보고 따라하면 됩니다

<br>

## 1. Docker 설치

```sh
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg
```

<br>

## 2. Docker 공식 GPG 키 추가

```sh
$ sudo mkdir -m 0755 -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

<br>

## 3. Docker Repository 설치

```sh
$ echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

<br>

## 4. Docker 설치

```sh
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

<br>

## 5. Docker 실행 테스트

```sh
$ sudo docker run hello-world

# 실행된 도커 컨테이너 확인
$ sudo docker ps

# 이미지 확인
$ sudo docker images
```

테스트 용으로 `hello-world` 라는 이미지를 실행합니다.

docker image 를 따로 받지 않아도 없으면 자동으로 pull 을 먼저 땡깁니다.

