# Docker 명령어

## docker exec

```sh
# Usage
$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

# Example
## my_continaer 컨테이너에 echo 명령어 실행
$ docker exec my_container echo "hello world!"

## my_container 컨테이너의 bash 접속
$ docker exec -it my_container bash
```

- 실행중인 컨테이너의 터미널에 명령을 내림

# Reference

- [Docker docs](https://docs.docker.com/engine/reference/commandline/exec/)