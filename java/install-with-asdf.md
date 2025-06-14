# MacOS Java (Corretto) 설치 (with. asdf)

# History

- 2022.11.26
  - 첫 작성
- 2025.06.15
  - asdf 버전업으로 인한 명령어 변경 반영
  - Java 버전 corretto 21 로 변경

# Overview

과거 포스트에서 이미 [MacOS OpenJDK 설치 및 버전 관리](https://bcp0109.tistory.com/302)에 대해 다뤄본 적이 있으나 asdf 를 이용해서 설치하는 방법을 안내하려고 합니다.

원래 Java 를 설치하려면 brew 를 사용하거나 직접 홈페이지에 들어가 JDK 파일을 다운받아야 합니다.

하지만 한 PC 에서 여러 Java 버전을 사용한다면 터미널에서 빌드할 때마다 Java 버전을 바꿔야하고 관리하기도 까다롭습니다.

jenv 라는 Java 버전 관리 툴이 존재하지만 jenv 는 Java 를 직접 설치할 수는 없습니다.

하지만 [asdf](https://asdf-vm.com/) 라는 툴을 사용하면 Java 의 설치/삭제를 간단하게 하고 버전 관리도 편하게 할 수 있습니다.

뿐만 아니라 asdf 는 Java 외의 여러 언어, 오픈소스 등의 버전도 관리할 수 있습니다.

<br>

# 1. asdf 설치

```sh
# install asdf
$ brew install asdf

# add to shell
$ echo . /opt/homebrew/opt/asdf/libexec/asdf.sh >> ~/.zshrc
```

homebrew 를 사용해서 asdf 를 설치합니다.

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

# 3. 설치 가능한 Java 버전 리스트 확인

```sh
$ asdf list all java | grep corretto-21
corretto-21.0.0.34.1
corretto-21.0.0.35.1
corretto-21.0.1.12.1
corretto-21.0.2.13.1
corretto-21.0.3.9.1
corretto-21.0.4.7.1
corretto-21.0.5.11.1
corretto-21.0.6.7.1
corretto-21.0.7.6.1
```

설치할 수 있는 Java 버전을 확인합니다.

여러가지 버전이 있으나 AWS 에서 사용되는 corretto 버전을 사용하겠습니다.

JVM 버전은 21 을 사용했습니다.

<br>

# 4. Java 설치

```sh
# 설치
$ asdf install java corretto-21.0.7.6.1

# 설치된 확인
$ asdf list java
 *corretto-21.0.7.6.1
```

설치 후에는 `asdf list <언어>` 명령어로 설치된 버전을 확인할 수 있으며 `asdf list` 만 입력하면 설치된 모든 오픈 소스의 모든 버전을 볼 수 있습니다.

<br>

# 5. 사용할 버전 지정

```sh
$ asdf set java corretto-21.0.7.6.1
```

설치되었다고 끝난게 아니라 사용할 Java 버전을 지정해야 합니다.

프로젝트 별로 Java 버전을 다르게 사용한다면 맞춰서 설정할 수 있습니다.

<br>

# 6. JAVA_HOME 설정하기

```sh
$ . ~/.asdf/plugins/java/set-java-home.zsh
```

[halcyon/asdf-java - JAVA_HOME](https://github.com/halcyon/asdf-java#java_home) 를 참고해서 본인이 쓰는 shell 에 맞게 입력합니다.

저는 ~/.zshrc 에 추가했습니다.

<br>

# 7. Java 설치 완료

```sh
$ java -version
openjdk version "21.0.7" 2025-04-15 LTS
OpenJDK Runtime Environment Corretto-21.0.7.6.1 (build 21.0.7+6-LTS)
OpenJDK 64-Bit Server VM Corretto-21.0.7.6.1 (build 21.0.7+6-LTS, mixed mode, sharing)
```

터미널에서 자바 버전을 확인해서 제대로 나온다면 설치 완료입니다.

<br>

# Reference

- [asdf](https://asdf-vm.com/)
