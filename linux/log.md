# 여러 원격 서버 (SSH) 로그를 실시간으로 모아서 보기

# Shell Script 전체 코드

```sh
#! /usr/bin/env bash

hosts = (a.example.com b.example.com)
LOG = "access.log"
pattern = "ERROR|WARN"

for host in $hosts
do
    if [ -z $pattern ]; then
        ssh deploy@$host "tail -F $LOG" | awk -v server=$host '{print "["server"]", $0}' &
    else
        ssh deploy@$host "tail -F $LOG" | awk -v server=$host -v pattern=$pattern '$0~pattern {print "["server"]", $0}' &
    fi
done

trap "ps | grep ssh | grep $LOG | awk '{print \$1}' | xargs kill -9" INT
wait
```

<br>

# 명령어

각 명령어에 대한 자세한 설명은 생략하고 간단하게 나열합니다.

<br>

## tail

```bash
ssh deploy@$host "tail -F $LOG" &
```

원격 서버에 명령어를 전달하려면 `ssh deploy@$host "{command}"` 형태로 사용합니다.

명령어의 끝에 `&` 을 붙여서 모든 `tail` 명령어를 백그라운드에서 실행시키고 출력값만 현재 쉘에서 모아서 봅니다.

<br>

## awk

```bash
awk -v server=$host '{print "["server"]", $0}'
```

여기서 `awk` 명령어는 `tail` 로 출력되는 로그들을 특정 포맷으로 감싸서 재출력하는 기능을 합니다.

`-v` 옵션은 `awk` action 에 특정 변수를 넘겨주고 싶을 때 사용하고 `$0` 은 레코드 전체를 의미합니다.

즉 `awk -v server=$host '{print "["server"]", $0}'` 은 로그들을 `[a.example.com] 로그내용~` 으로 변환해서 출력해줍니다.

로그들이 여러개 쌓이면 어디 서버에서 발생한 건지 알수 없기 때문이죠

<br>

## awk (filter)

```bash
awk -v server=$host -v pattern=$pattern '$0~pattern {print "["server"]", $0}'
```

ERROR 또는 WARN 로그만 보고 싶을 수도 있습니다.

정규표현식 `~` 오퍼레이터를 사용하여 레코드 전체 (`$0`) 에서 정규표현식 `$pattern` 이 포함되어 있는 경우에만 출력하도록 필터를 걸었습니다.

처음에는 `grep` 명령어를 사용했으나 속도가 너무 느려서 실시간 전달이 안되고 로그가 뚝뚝 끊겨서 전달되어 보기 굉장히 불편했습니다.

<br>

## trap & wait

```bash
trap "ps | grep ssh | grep $LOG | awk '{print \$1}' | xargs kill -9" INT
wait
```

`&` 기호를 붙여 백그라운드에서 실행했기 때문에 로그를 그만 볼때 모든 프로세스를 한번에 종료시켜야 합니다.

`trap <handler> <signal>` 은 특정 시그널을 받았을 때 handler 를 실행시키는 명령어입니다.

`wait` 은 자식 프로세스가 종료될 때까지 대기합니다.

즉, 위 코드는 실행한 쉘 스크립트를 대기 시켰다가 `Ctrl + C` 시그널을 받았을 때 백그라운드 프로세스를 모두 `kill` 합니다.

`wait` 명령어를 넣지 않으면 스크립트를 실행한 후에 전달할 시그널이 애매합니다.

그리고 로그 보고 있을 거니까 부모 프로세스를 굳이 조작할 필요도 없습니다.

<br>

# Reference

- [StackOverflow - AWK variable issue](https://stackoverflow.com/questions/16283018/awk-variable-issue)