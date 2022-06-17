# Git Directory 이름 변경

`boj -> BOJ` 로 `leetcode -> LeetCode` 로 변경

1. 이름을 변경하려는 디렉토리의 상위로 이동
2. `git mv oldName newName` 입력
3. `git add` `git commit` 하면 반영

<br>

주의할 점은 대소문자를 구분하지 않는 다는 점입니다.

```powershell
$ git mv boj BOJ
fatal: bad source, source=boj, destination=BOJ
```

<br>

만약 같은 이름으로 대소문자만 변경하길 원한다면 다른 이름으로 한번 변경해야 합니다.

```powershell
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
