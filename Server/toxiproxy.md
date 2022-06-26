# Toxiproxy 실행

```sh
# toxiproxy 서버 실행
$ toxiproxy-server
INFO[0000] Starting HTTP server on endpoint localhost:8474  host=localhost port=8474 version=2.4.0


# toxiproxy 생성/삭제
$ toxiproxy-cli create -l 0.0.0.0:21212 -u localhost:3306 mysql_dev
$ toxiproxy-cli delete mysql_dev


# toxiproxy 리스트
$ toxiproxy-cli list
Name			Listen		Upstream		Enabled		Toxics
======================================================================================
mysql_dev	0.0.0.0:21212	localhost:3306		enabled		None 


# toxiproxy 추가
$ toxiproxy-cli toxic add -t latency -a latency=20000 mysql_dev
$ toxiproxy-cli toxic add --toxicName timeout -t timeout -a timeout=20000000 mysql_dev
```

<br>

# 실험

## 1. 커넥션이 없을 때 DB 실패 

1. DB 블락
2. 레일즈 실행
3. DB 요청

<br>

### 1.1. config 추가하지 않았을 때

expect: ConnectionPool 에 커넥션이 존재하지 않아 `connect_timeout` 만큼 블락 된 후 실패 (default 120초??)

actual (O): 120초 후에 실패 `ActiveRecord::ConnectionNotEstablished (Lost connection to MySQL server at 'waiting for initial communication packet', system error: 60)`

<br>

### 1.2. config 추가 (connect_timeout: 5, read_timeout: 10)

expect: ConnectionPool 에 커넥션이 존재하지 않아 `connect_timeout` 만큼 블락 된 후 실패 (5초)

actual (O): 5초 후에 실패

<br>

## 2. 커넥션이 존재할 때 DB 실패

1. 레일즈 실행
2. DB 요청 (ConnectionPool 에 커넥션 생성됨)
3. DB 블락
4. DB 요청

<br>

### 2.1. config 추가하지 않았을 때

expect: Connection 은 존재하기 때문에 바로 DB 요청 날라감. `read_timeout` 만큼 대기 후 커넥션 실패를 깨닫고 `connect_timeout` 만큼 추가 대기 (default 30분 + 120초??)

actual (X): 멈추지 않음..

<br>

### 2.2. config 추가 (connect_timeout: 5, read_timeout: 10)

expect: Connection 은 존재하기 때문에 바로 DB 요청 날라감. `read_timeout` 만큼 대기 후 커넥션 실패를 깨닫고 `connect_timeout` 만큼 추가 대기 (10초 + 5초)

actual: 10초만에 실패.. `ActiveRecord::AdapterTimeout (Mysql2::Error::TimeoutError: Timeout waiting for a response from the last query. (waited 10 seconds))`

그리고 이후 요청은 `ActiveRecord::ConnectionNotEstablished (MySQL client is not connected)` 으로 실패

`read_timeout: 15` 로 변경하고 나서 테스트해보니 15초만에 실패하는 걸 보니 설정에 따라 바뀐다는 걸 확인할 수 있음

<br>

# Reference

- [Shopify/toxiproxy Github](https://github.com/Shopify/toxiproxy)
- [Grap Tech Blog - Deep Dive into Database Timeouts in Rails](https://engineering.grab.com/deep-dive-into-database-timeouts-in-rails)