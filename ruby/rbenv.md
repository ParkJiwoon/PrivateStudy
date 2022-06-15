# Mac OS X 에서 rbenv 설치

```sh
brew update
brew install rbenv
rbenv install 2.3.3 && rbenv rehash
```

<br>

# 루비 버전 변경

```sh
rbenv local 2.3.3   # 현재 디렉토리에 대해서만 설정
rbenv global 2.3.3  # 전역으로 설정
```

<br>

# 루비 버전이 바뀌지 않을 때

```bash
# ~/.bash_profile 에 세팅
if which rbenv > /dev/null; then
  export PATH="$HOME/.rbenv/shims:$PATH"
  eval "$(rbenv init -)";
fi
```

`sudo vi ~/.bash_profile` 로 설정 후 터미널 껐다 키거나 `source ~/.bash_profile`
