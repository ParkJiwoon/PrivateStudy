# asdf 사용해서 Ruby 설치

```sh
# 1. asdf 설치 (rbenv 같은 버전 관리 툴)
$ brew install asdf

# 2. ruby 설치 (XCode12 버전부터 install 하면 에러가 발생해서 컴파일 옵션 변경 필요)
$ asdf plugin add ruby
$ export CFLAGS="-Wno-error=implicit-function-declaration"
$ asdf install ruby 2.6.2

# 설치 확인
$ asdf list
$ asdf global ruby 2.6.2
$ asdf current

# 3. Ruby 프로젝트 루트로 이동
$ asdf exec bundle install

# bundle 시 gem 실패하면
$ asdf exec gem install mysql2 -v '0.5.4'

# mysql database 가 없다고 뜨면 만들어줘야함
$ mysql -uroot
$ mysql> create database {database_name}

# 서버 실행 후 127.0.0.1:3000 접속
$ asdf exec rails s
```

`rvm` 을 지나 `rbenv` 를 지나 이제 `asdf` 라는 걸 사용해서 Ruby 를 설치할 수 있습니다.

`asdf` 는 Java, Python 같은 다른 언어도 설치할 수 있으므로 여러 언어를 관리할 때 편합니다.

<br>

# 접속 성공한 모습

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ruby/images/screen_2022_06_15_22_56_29.png">