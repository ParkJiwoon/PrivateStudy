# docker push 에러 (denied: requested access to the resource is denied)

# Issue

```sh
Using default tag: latest
The push refers to repository [docker.io/asdfaasdf/kubia]
6bb2a0932f1d: Preparing 
ab90d83fa34a: Preparing 
8ee318e54723: Preparing 
e6695624484e: Preparing 
da59b99bbd3b: Preparing 
5616a6292c16: Waiting 
f3ed6cb59ab0: Waiting 
654f45ecb7e3: Waiting 
2c40c66f7667: Waiting 
denied: requested access to the resource is denied
```

[Docker Hub](https://hub.docker.com/) 에 이미지를 푸시하려려고 하는 데 아래와 같은 에러 로그가 나타났습니다.

로그를 보면 알 수 있듯이 권한 관련된 이슈인데 이유는 크게 두 가지가 있습니다.

1. Docker Hub 로그인을 하지 않음
2. Docker Hub 아이디와 태그된 이미지의 이름이 일치하지 않음

<br>

# Solution

우선 Docker Hub 에 로그인합니다.

```sh
$ docker login           
Login with your Docker ID to push and pull images from Docker Hub. If you dont have a Docker ID, head over to https://hub.docker.com to create one.
Username: bcp0109
Password: 
Login Succeeded
```

<br>

Docker Tag 를 Docker Hub ID 와 동일하게 생성합니다.

```sh
$ docker tag kubia bcp0109/kubia
```

<br>

이미지를 푸시합니다.

```sh
$ docker push bcp0109/kubia     
Using default tag: latest
The push refers to repository [docker.io/bcp0109/kubia]
6bb2a0932f1d: Pushed 
ab90d83fa34a: Pushed 
8ee318e54723: Pushed 
e6695624484e: Pushed 
da59b99bbd3b: Pushed 
5616a6292c16: Pushed 
f3ed6cb59ab0: Pushed 
654f45ecb7e3: Pushed 
2c40c66f7667: Pushed 
```

<br>

Docker Hub 에 이미지가 올라간 걸 확인할 수 있습니다.

![](https://github.com/ParkJiwoon/PrivateStudy/raw/master/trouble-shooting/images/screen_2021_12_05_02_15_38.png)