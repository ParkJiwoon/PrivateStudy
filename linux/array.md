# Shell Script 배열 사용법

# 1. 배열 생성

```sh
array = (1 2 3)
```

괄호 안에 입력하며 공백 한칸으로 구분합니다.

<br>

# 2. 배열 출력

```sh
echo $array
```

`$` 기호로 변수값을 출력할 수 있습니다.

<br>

# 3. 배열 순회

```sh
for i in $array
do
echo $i
done

# 세미콜론(;) 사용해서 한줄 줄일 수 있음
for i in $array; do
  echo $i
done
```
