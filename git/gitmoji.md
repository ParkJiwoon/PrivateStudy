# Gitmoji (for Git Commit Convention)

# Overview

Gitmoji 란 Git + Emoji 입니다.

여러 사람이 커밋 메시지를 작성하다보면 일관성이 없고 나중에는 히스토리를 알아보기 힘들어집니다.

Gitmoji 는 이모지를 사용하여 커밋 메시지를 일정하게 작성하도록 도와주는 툴입니다.

커밋 메시지 타이틀 앞에 특정 이모지를 넣으면서 이모지만 보고도 어떤 목적으로 한 커밋인지 알아볼 수 있습니다.

gitmoji 에 대한 설명은 [gitmoji 공식](https://github.com/carloscuesta/gitmoji)에서 볼 수 있지만 사용하기 위해서는 [gitmoji-cli](https://github.com/carloscuesta/gitmoji-cli) 를 참고해야 합니다.

<br>

# 1. Install

```sh
# use brew
$ brew install gitmoji

# use npm
$ npm i -g gitmoji-cli
```

`brew` 또는 `npm` 을 사용해서 설치할 수 있습니다.

<br>

# 2. Configuration

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_18_04_36_16.png?raw=true" width="80%">

gitmoji 를 사용하기 전에 미리 설정을 해두는게 좋습니다.

`gitmoji -g` 명령어를 사용해서 여러가지 옵션을 설정할 수 있습니다.

저는 대부분 기본값을 사용하는데 `Select how emojis should be used in commits` 옵션만 text 대신 emoji 를 선택했습니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_18_04_38_39.png?raw=true" width="80%">

Github 에서 볼 때는 별 문제 없으나 이렇게 이모지를 지원하지 않는 외부 툴에서는 Text 그대로 노출됩니다.

그래서 이모지 자체를 커밋 메시지에 넣을 수 있도록 설정했습니다.

<br>

# 3. Usage

gitmoji 의 사용법은 크게 어렵지 않습니다.

그냥 평범한 사용법에서 `git commit` 대신 `gitmoji -c` 를 입력해주면 됩니다.

<br>

## 3.1. Choose Gitmoji

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_18_04_24_15.png?raw=true" width="80%">

`gitmoji -c` 를 입력하면 위 그림처럼 이모지 선택이 나옵니다.

위아래 방향키를 사용해서 다른 이모지들을 찾아볼 수도 있고, 직접 원하는 기능을 검색할 수도 있습니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_18_04_25_57.png?raw=true" width="80%">

위 그림처럼 refactor 를 검색하면 그에 맞는 이모지를 띄워줍니다.

<br>

## 3.2. Input Commit Message

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_18_04_28_04.png?raw=true" width="80%">

원하는 이모지를 선택했다면 Commit Title, Message 를 입력합니다.

Message 는 생략해도 됩니다.

전부 입력 후 엔터를 누르면 커밋이 완료됩니다.

이후에는 똑같이 `git push` 를 사용해서 원격 저장소에 반영할 수 있습니다.

<br>

## 3.3. Repository 확인

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/git/images/screen_2022_06_18_04_31_15.png?raw=true" width="80%">

이제 커밋 로그를 확인하면 이렇게 이모지가 잘 들어간 것을 볼 수 있습니다.

제가 실제로 두번 커밋한거고 중복해서 로그가 쌓인건 아닙니다.

<br>

# 4. Plugin 및 GUI

IntelliJ 는 Plugin, VSCode 는 Extension 으로 지원하기도 합니다.

IDE 에서 직접 커밋 메시지를 작성하는 스타일이었다면 이용해보는 게 좋습니다.

Source Tree, GitKraken, Git Fork 등등 별도 GUI 를 사용할 때는 아쉽게도 지원되지 않는 것 같아요.

개인적으로 CLI 대신 GUI 를 많이 사용하기 때문에 이 부분이 참 아쉬웠습니다.

그래서 그냥 `add`, `push`, `pull` 등은 전부 GUI 를 이용하고 오로지 커밋만 `gitmoji -c` 를 사용합니다.

만약 이게 싫다면 이모지를 직접 복사해서 GUI 커밋 메시지에 붙여넣는 방법도 가능합니다.

[Gitmoji Dev](https://gitmoji.dev/) 사이트에서 원하는 이모지를 검색할 수 있고 그림을 누르면 자동으로 복사되기 때문에 GUI 에서 작성하는 경우 편리하게 이용할 수 있습니다.

<br>

# 5. 장단점

가장 큰 **장점은 커밋 로그를 시각적으로 확인할 수 있다** 입니다.

**단점은 이모지가 뭘 의미하는지 알고 있어야 하고 GUI 와 같이 쓰기 번거롭다**는 점인것 같아요.

Gitmoji 를 처음 봤을 때 생각난건 `feat:`, `fix:`, `refactor:` 등을 사용하는 [Angular Git Commit Guidelines](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines) 였는데요.

시각적으로는 그림인 이모지가 더 잘들어오긴 하지만 가독성은 개인에 따라 다르기 때문에 뭐가 더 낫다고 단언할 수는 없을 것 같아요.

Angular Convention 과 비교했을 때 가장 큰 장점을 뽑자면 Search 기능이라고 생각합니다.

파일을 수정하고 어떤 prefix 를 붙여야할 까 고민될 때 gitmoji 는 대충 이런 목적이다~ 검색을 하면 그에 맞는 이모지를 추천해줍니다.

<br>

# Conclusion

사실 Gitmoji 가 뭔지 잘 모르다가 회사에서 사용하면서 알게 되었는데요.

처음에는 CLI 와 GUI 를 왔다갔다 하는게 불편했지만 적응해보니 생각보다 쓸만한 것 같습니다.

특히 Search 기능이 강력해서 적당히 입력해도 알아서 추천해주니 큰 고민 없이 넣을 수 있다는게 좋았습니다.

<br>

# Reference

- [Gitmoji Dev](https://gitmoji.dev/)
- [gitmoji Github](https://github.com/carloscuesta/gitmoji)
- [gitmoji-cli Github](https://github.com/carloscuesta/gitmoji-cli)
- [Angluar Git Commit Guidelines](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)