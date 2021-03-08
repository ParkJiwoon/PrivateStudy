여러 대의 JAVA (JDK) 를 사용하는 경우 원하는 버전을 기본으로 설정할 수 있습니다.

<br>

## 1. 현재 버전 확인

```java
❯ java -version
openjdk version "11.0.10" 2021-01-19
OpenJDK Runtime Environment AdoptOpenJDK (build 11.0.10+9)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 11.0.10+9, mixed mode)
```

- 저는 현재 JDK 11 버전이 기본 설정입니다.

<br>

## 2. 설치되어 있는 JDK 리스트 확인

```java
❯ /usr/libexec/java_home -V
Matching Java Virtual Machines (2):
    11.0.10, x86_64:	"AdoptOpenJDK 11"	/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home
    1.8.0_252, x86_64:	"AdoptOpenJDK 8"	/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home
```

- 저는 JDK 11 과 JDK 8 버전이 설치되어 있습니다.

<br>

## 3. JDK 버전 변경

```java
// 1.8 버전으로 변경
❯ export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)

// 11 버전으로 변경
❯ export JAVA_HOME=$(/usr/libexec/java_home -v 11)
```

- JDK 버전을 변경해줍니다.

<br>

## 4. 변경 여부 확인

```java
❯ java -version
openjdk version "1.8.0_252"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_252-b09)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.252-b09, mixed mode)

❯ echo $JAVA_HOME
/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
```

- 다시 Java Version 을 확인해보면 변경된 것을 알 수 있습니다.
- `$JAVA_HOME` 도 변경된 것을 확인 할 수 있습니다.
