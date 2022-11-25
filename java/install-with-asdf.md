# MacOS Java (OpenJDK) 설치 (with. asdf)

# Overview

과거 포스트에서 이미 [MacOS OpenJDK 설치 및 버전 관리](https://bcp0109.tistory.com/302)에 대해 다뤄본 적이 있으나 asdf 를 이용해서 설치하는 방법을 안내하려고 합니다.

원래 Java 를 설치하려면 brew 를 사용하거나 직접 홈페이지에 들어가 JDK 파일을 다운받아야 합니다.

하지만 한 PC 에서 여러 Java 버전을 사용한다면 터미널에서 빌드할 때마다 Java 버전을 바꿔야하고 관리하기도 까다롭습니다.

jenv 라는 Java 버전 관리 툴이 존재하지만 jenv 는 Java 를 직접 설치할 수는 없습니다.

하지만 [asdf](https://asdf-vm.com/) 라는 툴을 사용하면 Java 의 설치/삭제를 간단하게 하고 버전 관리도 편하게 할 수 있습니다.

뿐만 아니라 asdf 는 Java 외의 여러 언어, 오픈소스 등의 버전도 관리할 수 있습니다.

<br>

# 1. asdf 설치

[mysetting - asdf](https://mysetting.io/apps/asdf) 을 참고하면 설치 및 사용방법 등을 알 수 있습니다.

```sh
# install dependencies (필요시)
$ brew install coreutils curl git

# install asdf
$ brew install asdf

# add to shell
$ echo -e "\n. $(brew --prefix asdf)/asdf.sh" >> ~/.zshrc
```

우선 asdf 를 설치합니다.

마지막의 add to shell 은 사용자마다 다릅니다.

저는 zsh 를 사용하고 있기 때문에 `~/.zshrc` 에 추가했고 만약 bash 를 사용한다면 `~/.bash_profile` 에 추가하면 됩니다.

<br>

# 2. Java Plugin 설치

```sh
$ asdf plugin add java
$ asdf plugin update java
```

설치를 위해선 플러그인을 먼저 설치해야 합니다.

<br>

# 3. Java 버전 리스트 확인

```sh
$ asdf list-all java
adoptopenjdk-11.0.15+10
adoptopenjdk-11.0.16+8
adoptopenjdk-11.0.16+101
adoptopenjdk-11.0.17+8
adoptopenjdk-17.0.0+35
...
..
.
zulu-jre-javafx-19.30.11
```

설치할 수 있는 Java 버전을 확인합니다.

저는 원래 AdoptOpenJDK 를 사용하였으나 deprecated 되었기 때문에 Adoptimu 에서 권장하는 Temurin 버전을 사용합니다.

([AdoptOpenJDK Blog - Good-bye AdoptOpenJDK. Hello Adoptium!](https://blog.adoptopenjdk.net/2021/08/goodbye-adoptopenjdk-hello-adoptium/) 참고)


<br>

# 4. Java 설치

```sh
# 설치
$ asdf install java temurin-11.0.17+8

# 설치된 확인
$ asdf list java
  temurin-11.0.17+8
```

Temurin 의 Java 11 버전 중 가장 최신 버전을 설치합니다.

설치 후에는 `asdf list <언어>` 명령어로 설치된 버전을 확인할 수 있으며 `asdf list` 만 입력하면 설치된 모든 오픈 소스의 모든 버전을 볼 수 있습니다.

<br>

# 5. 사용할 버전 지정

```sh
# global
$ asdf global java temurin-11.0.17+8

# local
$ asdf local java temurin-11.0.17+8
```

프로젝트 별로 설정하고 싶다면 local, 전역으로 설정하고 싶다면 global 을 사용해 지정합니다.

<br>

# 6. JAVA_HOME 설정하기

```sh
$ . ~/.asdf/plugins/java/set-java-home.zsh
```

[halcyon/asdf-java - JAVA_HOME](https://github.com/halcyon/asdf-java#java_home) 를 참고해서 본인이 쓰는 shell 에 맞게 입력합니다.

<br>

# 7. Java 설치 완료

```sh
$ java -version
openjdk version "11.0.17" 2022-10-18
OpenJDK Runtime Environment Temurin-11.0.17+8 (build 11.0.17+8)
OpenJDK 64-Bit Server VM Temurin-11.0.17+8 (build 11.0.17+8, mixed mode)
```

터미널에서 자바 버전을 확인해서 제대로 나온다면 설치 완료입니다.

<br>

# Reference

- [asdf](https://asdf-vm.com/)
- [AdoptOpenJDK Blog - Good-bye AdoptOpenJDK. Hello Adoptium!](https://blog.adoptopenjdk.net/2021/08/goodbye-adoptopenjdk-hello-adoptium/)