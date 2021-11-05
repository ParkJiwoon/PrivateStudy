# Mac OS IntelliJ 및 초기 개발 환경 설정

# Overview

Mac OS 개발환경 초기 세팅과 Java 설치, IntelliJ 세팅 등을 알아봅니다.

<br>

# 1. Command Line Tools 설치

```sh
# 커맨드 입력 후 팝업창 뜨면 동의 필요
$ xcode-select --install

# 설치 확인
$ xcode-select --version
```

<br>

# 2. Homebrew 설치

```sh
# https://brew.sh/index_ko 참고
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

<br>

# 3. Java (OpenJDK) 설치

[Mac OS Java (OpenJDK) 설치 및 버전 변경](../java/mac-os-install-and-change-jdk.md) 참고

<br>

# 4. IntelliJ 환경 설정

Spring 개발을 할 때 꼭 필요한 JetBrain 사의 IntelliJ 입니다.

그대로 사용해도 좋지만 몇가지 설정과 플러그인을 추가하면 좀더 쾌적한 개발이 가능합니다.

<br>

## 4.1. SDK 설정


