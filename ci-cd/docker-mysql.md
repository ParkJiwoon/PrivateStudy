# Apple Silicon 에서 MySQL Docker 실행

```sh
# Docker image pull
$ docker pull --platform linux/x86_64 mysql

# docker image 실행
$ docker run --name mysql-container -e MYSQL_ALLOW_EMPTY_PASSWORD=true -d -p 3306:3306 mysql:latest

# 떳는지 확인
$ docker ps

# MySQL Docker 컨테이너 중지
$ docker stop mysql-container

# MySQL Docker 컨테이너 시작
$ docker start mysql-container

# MySQL Docker 컨테이너 재시작
$ docker restart mysql-container
```

그냥 `docker pull mysql` 하면 에러 발생합니다.

[stackoverflow](https://stackoverflow.com/questions/65456814/docker-apple-silicon-m1-preview-mysql-no-matching-manifest-for-linux-arm64-v8) 참고하여 명령어를 수행하면 됩니다.
