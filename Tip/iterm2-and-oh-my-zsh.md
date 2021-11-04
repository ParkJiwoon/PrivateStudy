# Mac OS 터미널을 이쁘게 꾸며보자 (Iterm2 + Oh My Zsh)

# 1. Hack Font 설치

개인적으로 가장 선호하는 폰트입니다.

https://sourcefoundry.org/hack/#download 에서 Mac OS Zip 을 다운받고 풀어서 설치하면 됩니다.

<br>

# 2. Iterm2 설치

Iterm2 은 기본 터미널 애플리케이션보다 다양한 기능과 테마가 탑재된 보조 터미널 애플리케이션입니다.

[공식 홈페이지](https://iterm2.com/) 에서 설치하거나 [Homebrew 로 설치](https://formulae.brew.sh/cask/iterm2) 가능합니다.

```sh
$ brew install --cask iterm2
```

설치 후에는 `Preference > Profiles > Text` 로 이동해서 폰트를 변경할 수 있습니다.

<img src="https://user-images.githubusercontent.com/28972341/140167044-54c375dc-7f73-43fb-94f6-a7890a2cec07.png" width="80%">

<br><br>

## 2.1. Iterm2 테마 변경 (Dracula)

```sh
$ git clone https://github.com/dracula/iterm.git
```

위 명령어로 Dracula 테마를 아무 위치에나 다운 받습니다.

다운 받은 폴더에 들어가 `Dracula.itermcolors` 파일 더블클릭 하면 자동으로 테마가 추가됩니다.

`Iterm2 > Perferences > Profiles > Colors > Color Presets...` 로 가서 Dracula 를 선택합니다.

<img src="https://user-images.githubusercontent.com/28972341/139925235-f6d6d73c-dfe1-49e7-9e1a-aa3ad78bb634.png" width="80%">

<br><br>

# 3. Oh My ZSH 설치

Oh My Zsh 는 기본 쉘보다 더 다양한 기능을 제공하는 Z Shell 의 플러그인입니다.

Catalina OS 부터 zsh 가 기본 터미널이 되었기 때문에 따로 설치할 필요는 없고 플러그인만 설치하면 됩니다.

https://github.com/ohmyzsh/ohmyzsh 를 참고해서 설치 가능합니다.

```sh
# ohmyzsh curl 설치
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

<img src="https://user-images.githubusercontent.com/28972341/140358804-bfe5c82a-5e93-4ddc-8bc9-1de7f8ef13ab.png" width="80%">

<br><br>

## 3.1. zsh 테마 적용

`agnoster` 가 국룰인것 같은데 저는 `powerlevel10k` 테마를 적용했습니다.

```sh
# zsh 테마에 powerlevel10k 추가
$ git clone https://github.com/romkatv/powerlevel10k.git ~/.oh-my-zsh/themes/powerlevel10k

# .zshrc 파일 수정
$ vi ~/.zshrc

ZSH_THEME="powerlevel10k/powerlevel10k"

# 변경한 .zshrc 적용하면 여러 가지 설정 Step 이 나오고 다 선택하면 최종 적용됨
$ source ~/.zshrc
```

<br>

## 3.2. Syntax Highlighting 플러그인 적용

<img src="https://user-images.githubusercontent.com/28972341/140348276-09c2c1b8-37e0-449c-a826-54034d746b77.png" width="80%">

존재하지 않은 명령어를 입력하면 빨간색으로 알려주는 편리한 플러그인입니다.

존재하는 명령어는 녹색으로 표시됩니다.

```sh
# 설치
$ brew install zsh-syntax-highlighting

# 적용 (.zshrc 맨 밑에 추가해야 터미널을 껐다 켜도 적용됨)
$ source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

<br>

## 3.3. 자동완성제안 (Auto Suggestions) 플러그인 적용

<img src="https://user-images.githubusercontent.com/28972341/140347740-a376aed6-8f4e-43b9-ad06-662e9f72a653.png" width="80%">

과거에 입력한 명령어를 자동으로 만들어주는 기능입니다.

이미 입력해본 적이 있는 명령어라면 조금만 입력해도 위 사진처럼 가장 최근에 입력한 명령어를 보여줍니다.

이 상태에서 방향키 오른쪽 버튼을 누르면 전체 명령어를 완성해줍니다.

```sh
# 설치
$ brew install zsh-autosuggestions

# 적용 (.zshrc 맨 밑에 추가해야 터미널을 껐다 켜도 적용됨)
$ source /usr/local/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

<br>

## 3.4. autojump 플러그인 추가

<img src="https://user-images.githubusercontent.com/28972341/140350043-bd357de8-2ed4-4581-9b99-fa248bfee683.png" width="80%">

```sh
# 설치
brew install autojump

# .zshrc 수정
plugins=(git autojump)
```

과거에 방문했던 위치를 적당히 힌트만 주면 알아서 찾아줍니다.

예를 들어 위 사진 처럼 `/Downloads` 에서 `j study` 를 입력하면 study 라는 이름을 가진 디렉토리 중 가장 많이 방문한 곳으로 이동합니다.

`j -s` 명령어로 방문한 디렉토리 기록도 확인 가능합니다.
