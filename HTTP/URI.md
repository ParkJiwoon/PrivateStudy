# URI 란?

URI (Uniform Resource Identifier) 는 인터넷에 있는 자원을 나타내는 유일한 주소이다.

URI 에는  URL 과 URN 두 종류가 있는데 일반적으로 URL 을 많이 사용한다.

<br>

## 문법

``` html
scheme:[//[user[:password]@]host[:port]][/path][?query][#fragment]
```

RFC 3986 에 정의 되어 있는 URL 은 다음과 같은 형태를 나타낸다.

여러 구성 요소 중에 가장 중요한 세 가지는 scheme, host, path 이다.

<br>

## 요소

1. scheme
   - 스킴이라고 하며 보통 프로토콜을 의미한다.
   - URL 나머지 부분들과 콜론(:) 으로 구분된다.
   - 대소문자를 구분하지 않는다.
   - 리소스에 어떻게 요청, 접근할 것인지 명시하는 how 를 담당한다.
   - ex) http, ftp, rtsp 등등..
2. host
   - 접근하려고 하는 리소스를 가지고 있는 인터넷 상의 호스트 장비
   - 도메인명 (localhost) 또는 IP 주소 (127.0.0.1) 로 제공한다.
3. port
   - 하나의 호스트에서 여러 개의 통신을 할 때 구분되는 값
   - 서버가 열어놓을 수 있다.
   - 포트 번호를 명시하지 않으면 default 로 80 이 사용된다.
4. path
   - 리소스가 서버의 어디에 있는지 알려준다.
