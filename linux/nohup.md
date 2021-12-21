# nohup 으로 백그라운드에서 프로세스 실행

# Overview

보통 백그라운드에서 데이터를 실행할 때는 명령어 마지막에 `&` 를 붙여서 실행합니다.

```sh
$ java -jar my-app.jar &
```

<br>

하지만 위 명령어대로 하면 애플리케이션의 로그를 볼 수 없습니다.

따라서 `nohup` 을 사용합니다.

<br>

## 1. nohup 으로 백그라운드 실행

```sh
$ nohup java -jar my-app.jar &

[1] 97569
nohup: ignoring input and appending output to 'nohup.out'
```

<br>

## 2. nohup 로그 조회

이제 `nohup.out` 파일에 기록되는 로그를 확인할 수 있습니다.

```sh
# 로그 조회
$ cat nohup.out

# 로그 테일링
$ tail -f nohup.out
```

<br>

## 3. 백그라운드 Job 확인

백그라운드에서 실행되는 프로세스가 있는지 확인해볼 수 있습니다.

```sh
# 백그라운드에서 실행되는 프로세스 확인
$ bg
-bash: bg: job 1 already in background

# 출력
$ jobs
[1]+  Running                 nohup java -jar my-app.jar &
```