# MacOS 에서 H2 database 설치 및 Spring Boot 에 연결

# 1. H2 Database 홈페이지에서 다운로드

다운로드 링크 : https://www.h2database.com/html/main.html

<img src = "https://github.com/ParkJiwoon/PrivateStudy/raw/master/spring/images/h2-1.png" width="50%">

<br>

최신 버전보다는 안정화된 버전이 괜찮습니다.

<img src = "https://github.com/ParkJiwoon/PrivateStudy/raw/master/spring/images/h2-2.png" width="50%">

<br>

# 2. 압축 풀고 실행

다운 받은 파일의 압축을 풀면 다음과 같은 구성으로 되어 있습니다.

여기서 `./bin/h2.sh` 를 입력하면 h2 console 을 실행합니다.

만약 권한이 없으면 `chmod 755 ./bin/h2.sh` 로 권한을 부여합니다.

<img src = "https://github.com/ParkJiwoon/PrivateStudy/raw/master/spring/images/h2-3.png" width="50%">

<br>

# 3. 한번 연결해서 `~~~.mv.db` 파일 생성 후 실행

처음 진입하면 아래와 같은 화면이 나옵니다.

다른 칸은 전부 그대로 두고 JDBC URL 부분만 표시한 것처럼 바꿔줍니다.

그리고 연결 버튼을 눌러서 진입합니다.

<img src = "https://github.com/ParkJiwoon/PrivateStudy/raw/master/spring/images/h2-4.png" width="50%">

<br>

그럼 다음과 같이 내 Root 폴더에 `my-db-test.mv.db` 파일이 생깁니다.

이후에 다시 h2 console 에서 연결 끊기 후 `jdbc:h2:tcp://localhost/~/my-db-test` 로 접속해서 사용하면 됩니다.

<img src = "https://github.com/ParkJiwoon/PrivateStudy/raw/master/spring/images/h2-5.png" width="70%">

<br>

# 4. Spring Boot 에 연결

`application.yml` 에서 DB 설정할 때 `spring.datasource.url` 에 JDBC URL 과 동일하게 세팅합니다.

```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/my-db-test
    username: sa
    password:
    driver-class-name: org.h2.Driver
```