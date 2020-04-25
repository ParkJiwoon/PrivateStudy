# Git Directory 이름 변경

[Algorithm repo](https://github.com/ParkJiwoon/Algorithm) 에서 디렉토리 이름을 변경할 필요가 생겼습니다.

별거 아닌 이유지만 `boj -> BOJ` 로 `leetcode -> LeetCode` 로 변경하고 싶었습니다.

우선 Github 에서 디렉토리 이름을 변경하는 방법을 검색했습니다.

1. 이름을 변경하려는 디렉토리의 상위로 이동
2. `git mv oldName newName` 입력
3. `git add` `git commit` 하면 반영

<br>

그런데 막상 사용하려니 문제점이 하나 있었습니다.

바로 **대소문자를 구분하지 않는다** 는 점

```shell
$ git mv boj BOJ
fatal: bad source, source=boj, destination=BOJ
```

<br>

해결법은 단순했습니다.

다른 이름으로 변경 후 다시 변경하면 됩니다.

```shell
$ git mv boj baekjoon
$ git mv baekjoon BOJ
$ git add .
$ git commit -m "Rename boj to BOJ"

[master 9f7487f] Rename boj to BOJ
 68 files changed, 0 insertions(+), 0 deletions(-)
 rename {boj => BOJ}/1005.java (100%)
 rename {boj => BOJ}/1057.java (100%)
 rename {boj => BOJ}/1074.java (100%)
 .
 .
 .
 rename {boj => BOJ}/image/5397_4.PNG (100%)
```
