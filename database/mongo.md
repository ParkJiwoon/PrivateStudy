# 설치법

```sh
# 이미지 다운 (docker images 로 확인 가능)
$ docker pull mongo

# 컨테이너로 mongoDB 실행 (--name: 컨테이너 이름 설정, -p: 포트 포워딩, -d: 백그라운드에서 실행)
# (docker ps 로 확인 가능)
$ docker run --name mongo-container -p 27017:27017 -d mongo

# MongoDB cli 접속하려면 컨테이너 bash 실행 후 "mongo" 입력
$ docker exec -it mongo-container bash
```