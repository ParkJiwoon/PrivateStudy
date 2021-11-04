# Java (OpenJDK) 설치 및 버전 변경

# Overview

Java 설치 방법과 여러 개의 버전을 사용할 때 어떤 식으로 변경하는지 알아봅시다.

여러 Java (JDK) 버전을 사용하는 경우 원하는 버전을 기본으로 설정할 수 있습니다.

<br>

# 1. Java (OpenJDK) 설치

## 1.1. adoptopenjdk/openjdk 저장소 추가

```sh
$ brew tap adoptopenjdk/openjdk
```

<br>

## 1.2. cask 가 없다면 설치

```sh
$ brew install cask
```

<br>

## 1.3. OpenJDK 8 과 11 을 설치

```sh
$ brew install --cask adoptopenjdk8
$ brew install --cask adoptopenjdk11
```

- 하나의 버전만 사용한다면 하나만 설치하면 됩니다.

<br>

## 1.4. 설치 여부 확인

```sh
$ java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_292-b10)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.292-b10, mixed mode)
```

- Java 버전 확인을 확인해서 잘 나오면 설치가 완료된 겁니다.

<br>

# 2. Java (JDK) 버전 변경

## 2.1. 설치된 JDK 버전 확인

```sh
$ /usr/libexec/java_home -V

Matching Java Virtual Machines (2):
    11.0.11 (x86_64) "AdoptOpenJDK" - "AdoptOpenJDK 11" /Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home
    1.8.0_292 (x86_64) "AdoptOpenJDK" - "AdoptOpenJDK 8" /Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home
```

- 저는 현재 JDK 11 과 JDK 8 버전이 설치되어 있습니다.

<br>

## 2.2. JDK 버전 변경

```sh
# 1.8 버전으로 변경
$ export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)

# 11 버전으로 변경
$ export JAVA_HOME=$(/usr/libexec/java_home -v 11)
```


<br>

## 2.3. JDK 변경 확인

```sh
$ java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_292-b10)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.292-b10, mixed mode)

$ echo $JAVA_HOME
/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
```

- `$JAVA_HOME` 도 변경된 것을 확인 할 수 있습니다.

<br>

# 3. 기본 Java 버전 적용

프로젝트에 따라서 여러 개의 Java 버전을 사용해야 하는 경우가 있습니다.

자주 사용하는 Java 버전을 기본으로 세팅하고 싶다면 `bash` 를 사용하는 경우 `~/.bash_profile`, `zsh` 를 사용하는 경우 `~/.zshrc` 파일 가장 하단에 아래 코드를 한줄 추가해주면 됩니다.

JDK 버전 변경 때 사용했던 명령어와 동일합니다.

```sh
# 1.8 버전을 기본으로
$ export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)

# 11 버전을 기본으로
$ export JAVA_HOME=$(/usr/libexec/java_home -v 11)
```