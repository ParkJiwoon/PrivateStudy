# **Github 에 새로운 저장소 추가**

## **git init**

Git 저장소로 만들고자 하는 디렉토리로 이동한 다음 아래 명령어를 타이핑합니다.

`.git` 과 `.gitignore` 폴더가 생긴다면 성공입니다.

`.git` g폴더는 숨겨져있습니다. (`ls -a` 명령어를 사용하면 숨겨진 파일까지 전부 볼 수 있습니다)

```powershell
$ git init
```

<br>

## **git remote**

그리고 Github 로 이동해서 repository 를 생성한 다음 URL 을 복사합니다.

- `v` 옵션으로 제대로 추가되었는지 확인할 수 있습니다.

지울땐 `remove` 명령어를 사용합니다.

```powershell
git remote add origin [URL]     # 추가
git remote -v                   # 버전 확인
git remote remove origin        # 삭제
```

<br>

## **git add / commit / push**

```powershell
git add .
git commit -m "first commit"
git push -u origin master
```

<br><br>

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

<br><br>

# 소스트리에서 삭제된 브랜치 갱신

Fetch 할 때 다음 항목에 체크하면 된다.

`원격 서버들에서 제거된 브랜치 정리하기 (Prune tracking branches no longer present on remotes)`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a34b34f8-df3b-44ec-812e-0682261e2eb0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a34b34f8-df3b-44ec-812e-0682261e2eb0/Untitled.png)
