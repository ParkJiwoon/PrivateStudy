# Git Merge (feat. Github)

# Overview

한 프로젝트를 여러 사람이 작업하면 각자의 `feature` 브랜치에서 수정한 내용을 하나의 통합 브랜치 (`main`) 에 합치는 방식으로 진행합니다.

이렇게 브랜치를 통합할 때 사용하는 명령어가 `git merge` 입니다.

이 merge 는 각 브랜치의 상황에 따라 다르게 동작하고 방법도 다양하기 때문에 어떤 방법들이 있는지 알아봅니다.

<br>

# 1. Git Merge

CLI 또는 GUI 에서 사용하는 경우입니다.

크게 Merge, Fast-Forward, Squash, Rebase 가 있습니다.

<br>

## 1.1. Merge

![](https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_20_03_49_52.png)

```sh
$ git switch main
$ git merge feature
```

`main` 브랜치에 추가 작업 내역이 있다면 새로운 Merge Commit 을 만들게 됩니다.

가장 일반적인 Merge 방법입니다.

`feature` 의 모든 커밋 로그와 하나로 합친 Merge Commit 로그가 전부 남습니다.

<br>

## 1.2. Fast-Forward

![](https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_20_03_50_22.png)

```sh
$ git switch main
$ git merge feature # main 에 추가 작업 내역 없음
```

`feature` 브랜치를 딴 이후로 `main` 브랜치에 아무런 커밋이 없다면 merge 할 때 Fast-Forward 방식으로 합쳐집니다.

Fast-Forward 를 그대로 직역하면 "빨리감기" 라는 뜻입니다.

이 말 그대로 별도의 Merge 기록 없이 원래 `main` 에서 작업한 것처럼 로그가 남습니다.

병합하려는 `main` 브랜치에 커밋이 존재한다면 Fast-Forward Merge 가 되지 않습니다.

<br>

## 1.3. No Fast-Forward (--no-ff)

![](https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_20_03_50_44.png)

```sh
$ git switch main
$ git merge --no-ff feature # main 에 추가 작업 내역 없지만 머지 커밋 생성
```

만약 `main` 에 추가 작업 내역이 없어도 새로운 Merge Commit 을 만들고 싶다면 `--no-ff` 옵션을 추가합니다.

`feature` 브랜치의 존재를 남기고 싶을 때 사용할 수 있습니다.

<br>

## 1.4. Squash (--squash)

![](https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_20_03_48_58.png)

```sh
$ git switch main
$ git merge --squash feature
$ git commit -m "Merge Squash feature/squash"
```

Squash Merge 는 `feature` 의 모든 커밋을 하나의 커밋으로 만들어 `main` 에 머지합니다.

`feature` 에서 리뷰 반영, 버그 수정 등으로 쓸데없는 커밋이 많아진 경우 이를 다 기록하지 않고 하나의 새로운 커밋으로 남길 수 있습니다.

대신 `feature` 브랜치의 수정사항이 큰 경우 하나의 커밋으로 전부 표현하기 보다 커밋을 잘개 쪼개는게 알아보기 더 편할 수 있기 때문에 신중히 사용해야 합니다.

<br>

## 1.5. Rebase

![](https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_20_03_51_43.png)

```sh
$ git switch main
$ git rebase feature
```

`main` 에 아무런 추가 커밋이 없다면 Fast-Forward 와 동일하게 HEAD 만 이동합니다.

하지만 다른 커밋이 있다면 이름 그대로 커밋을 재배치 합니다.

재배치 하고 나면 **현재 브랜치의 커밋이 rebase 하려는 브랜치의 뒤로 이동**합니다.

즉 `main` 브랜치에서 `git rebase feature` 를 했다면 `feature` 의 커밋 내역이 먼저 찍히고 이후 `main` 브랜치의 커밋이 찍힙니다.

만약 같은 범위를 수정해서 rebase 과정에서 충돌이 발생한다면 각 커밋 별로 충돌을 해결해야 합니다.

별도의 Merge Commit 이 남지 않는다는 점은 Fast-Forward 와 동일하지만 Rebase 는 각 브랜치에 다른 커밋이 있어도 하나의 줄기로 합쳐줄 수 있다는 장점이 있습니다.

<br>

![](https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_20_03_53_41.png)

위 사진과 같이 어디 브랜치에서 시작하냐에 따라 커밋의 순서가 바뀝니다.

현재 checkout 한 브랜치의 커밋이 가장 최신에 위치하는 걸 볼 수 있습니다.

<br>

# 2. Github Merge

Github 에서도 Merge 를 할 수 있습니다.

아마 Github 에서 Pull Request 로 코드 리뷰를 받은 후에 Merge 하는 경우가 더 많을 것 같습니다.

Github 에서 지원하는 Merge 는 크게 세종류입니다.

<br>

## 2.1. Create a merge commit

`Create a merge commit` 을 사용하면 `main` 브랜치에 커밋이 있건말건 **무조건 `--no-ff` 옵션으로 머지**됩니다.

커밋 로그가 전부 남으며 새로운 Merge Commit 이 함께 만들어집니다.

<br>

## 2.2. Squash and merge

Git 과 마찬가지로 `feature` 의 커밋 이력들 대신 새로운 Merge Commit 하나만 남깁니다.

저는 리뷰 수정사항이 많아서 기능에 비해 커밋 수가 너무 많을 때 사용하는 방법입니다.

<br>

## 2.3. Rebase and merge

Rebase 는 `feature` 의 커밋 로그를 `main` 브랜치 커밋 로그 뒤에 붙여줍니다.

위에서 설명할 때 rebase 를 하면 현재 브랜치의 커밋이 대상 브랜치의 커밋 뒤로 이동한다고 했습니다.

이 특성을 이용해서 우선 `feature` 브랜치에서 `main` 브랜치를 rebase 한 뒤, `feature -> main` 으로 Fast-Forward Merge 한 셈입니다.

Git 명령어로 나타내면 아래와 같습니다.

<br>

```sh
# feature 브랜치로 이동
$ git switch feature

# main 브랜치의 커밋내역을 feature 브랜치 이전으로 끼워넣기
# 이렇게 하면 최신 main 브랜치에서 feature 를 딴 뒤 수정한 것처럼 됨
$ git rebase main 

# 다시 main 브랜치로 이동
$ git switch main

# main 에 feature 브랜치 머지
# rebase 를 했기 때문에 HEAD 만 이동하는 fast-forward merge 실행
$ git merge feature
```

이렇게 하면 깔끔하게 `main` 브랜치 뒤에 `feature` 브랜치 커밋 로그를 남길 수 있습니다.

다만, `feature` 브랜치의 커밋이 엄청 많은 경우 전부 rebase merge 해버리면 트리만 봤을 때 작업의 큰 줄기가 잘 보이지 않는다는 단점이 있습니다.

<br>

# Conclusion

그래서 어떤걸 사용하는게 좋냐? 라고 한다면 정해진 답은 없습니다.

여러 개의 커밋을 남기고 싶다면 `--no-ff` 머지를 사용하고 하나만 깔끔하게 남긴다면 Squash 또는 Rebase 를 사용합니다.

현재 제가 사용하는 방식을 그림으로 나타내면 아래와 같습니다.

<br>

![](https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_20_03_59_53.png)

저는 대부분의 머지를 PR 에서 하기 때문에 PR 을 최대한 작은 단위로 나눈 후 전부 Squash Merge 하는 걸 선호했습니다.

코드리뷰를 받다보면 추가 수정 사항이 발생하고 자잘한 커밋들을 전부 남기면 지저분하게 느껴서 없애고 싶었어요.

회사에 와서 `git rebase main` 으로 `feature` 브랜치를 최신화 하고 `--no-ff` 옵션으로 Merge Commit 을 남기는 방법을 알게 되었는데 굉장히 깔끔한 것 같습니다.

여러 회사들의 기술 블로그를 보면 각 팀마다 사용하는 Merge 전략이 있는데 여러 가지를 찾아보고 본인이 느끼기에 가장 좋아보이는 전략을 선택하면 됩니다.