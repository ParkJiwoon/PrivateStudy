# Spring Boot 에서 Kakao, Naver 로그인하기 1편 (OAuth 2.0) - 앱 등록

# 1. Overview

서비스를 개발하다 보면 ID, Password 를 사용하는 기본 로그인 외에 SNS 를 활용한 소셜 로그인을 제공하는 경우도 있습니다.

구글 로그인이 대표적이며 국내에서는 네이버, 카카오 로그인을 많이 사용합니다.

이번 시리즈에서는 Spring Boot 환경에서 네이버, 카카오 로그인을 지원하는 기능을 만들어봅니다.

우선 1편에서는 간단한 OAuth 2.0 에 대한 설명과 앱 등록을 알아보고 [2편에서 실제 구현 코드](https://bcp0109.tistory.com/380)를 다룰 예정입니다.

<br>

# 2. OAuth 2.0 이란?

OAuth 2.0 을 간단하게 설명하면 어떤 서비스를 만들 때 **사용자 개인정보와 인증에 대한 책임을 지지 않고 신뢰할 만한 타사 플랫폼에 위임**하는 겁니다.

개인정보 관련 데이터를 직접 보관하는 것은 리스크가 큽니다.

보안적으로 문제되지 않도록 안전하게 관리해야 하고 ID/PW 에 관련된 지속적인 해킹 공격 등 여러 가지 신경써야 할 부분이 많습니다.

하지만 OAuth 2.0 을 사용해 신뢰할 수 있는 플랫폼 (구글, 페이스북, 네이버, 카카오 등) 에 개인정보, 인증 기능을 맡기면 서비스는 인증 기능에 대한 부담을 줄일 수 있습니다.

<br>

## 2.1. OAuth 2.0 Sequence Diagram

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_11_06_07_24.png?raw=true">

OAuth 2.0 은 일반적으로 위와 같은 플로우를 많이 사용합니다.

그래서 클라이언트와 서버측 모두 OAuth 2.0 플로우에 대해 숙지하고 있어야 합니다.

<br>

# 3. 카카오 앱 등록

카카오 로그인을 사용하기 위해선 우선 카카오에 내 서비스를 등록해야 합니다.

<br>

## 3.1. 앱 등록

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_06_06_30_42.png?raw=true" width="85%">

`https://developers.kakao.com/console/app` 에 접속해서 앱을 추가합니다.

<br>

## 3.2. 플랫폼 등록

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_12_20_17_14.png?raw=true" width="85%">

여기서는 테스트용으로 localhost 만 사용하기 때문에 따로 등록은 안했지만 도메인을 등록해야 사용 가능한 API 들도 존재합니다.

확인 후 필요한 경우 자신의 서비스 도메인을 등록해줍니다.

<br>

## 3.3. 로그인 API 활성화

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_06_23_53_30.png?raw=true" width="85%">

로그인 API 를 활성화 하고 Redirect URI 를 등록합니다.

Redirect URI 를 등록하지 않으면 인증에 필요한 code 를 받을 수 없으므로 필수로 등록해야 합니다.

Redirect URI 로 code 를 받는 부분은 클라이언트의 영역이므로 같이 협업하는 웹, 앱 개발자가 있다면 등록을 부탁하면 됩니다.

여기서는 서버 개발만 해서 테스트할 예정이므로 `http://localhost:8080/kakao/callback` 으로 등록했습니다.

<br>

## 3.4. 동의항목 활성화

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_06_23_55_16.png?raw=true" width="85%">

이제 동의항목으로 이동해서 필요한 데이터의 동의 항목을 활성화 합니다.

여기서는 테스트를 위해 닉네임, 이메일을 활성화 했습니다.

이메일 필수 동의를 받으려면 앱 검수가 필요하기 때문에 임시로 "선택 동의" 상태로 만듭니다.

<br>

## 3.5. 인가 코드 받기 테스트

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_07_00_00_03.png?raw=true" width="85%">

이제 앱 화면에서 REST API 키 (Client ID) 값을 확인합니다.

`client_id`, `redirect_uri`, `response_type` 파라미터를 사용해서 카카오 로그인 페이지 URL 을 만듭니다.

다른 여러가지 파라미터 값들은 [Kakao Developers - 인가 코드 받기](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#request-code) 를 참고합니다.

```text
https://kauth.kakao.com/oauth/authorize
?client_id=160cd4f66fc928d2b279d78999d6d018
&redirect_uri=http://localhost:8080/kakao/callback
&response_type=code
```

위 URL 에 접속하면 카카오 로그인 화면이 보입니다.

원래는 클라이언트에서 "카카오로 로그인하기" 버튼을 추가하고 사용자가 눌렀을 때 연결되어야 하지만 우리는 UI 가 없으므로 직접 URL 에 접속합니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_06_23_59_08.png?raw=true" width="40%" height="60%">

전체 동의 후 계속하기를 누르면 우리가 등록한 Redirct URI 로 페이지가 이동합니다.

그 페이지 URL 의 파라미터를 확인하면 code 를 찾을 수 있습니다.

```text
http://localhost:8080/kakao/callback
?code=IY6uM7PIvuWa5D1LYXYrnfvMcd1-0U36AwkwrHm_aUwud8-neFISILn6KzXuJesy0p4GOwopyV4AAAGGt3E46Q
```

이제 클라이언트에서 얻은 Authorization Code 로 우리가 만든 백엔드 API 를 호출해서 로그인을 진행할수 있습니다.

<br>

# 4. 네이버 앱 등록

카카오 앱 등록과 비슷하지만 네이버는 설정해야 하는 게 조금 더 적습니다.

<br>

## 4.1. 앱 등록

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_11_21_32_20.png?raw=true" height="90%">

카카오 앱 등록과 마찬가지로 애플리케이션 이름, 사용할 API 를 선택 후 동의 항목은 이메일과 별명을 설정합니다.

서비스 URL 에는 본인이 만들고 있는 서비스의 URL 을 넣습니다.

Callback URL 은 네이버 로그인 후 이동할 URL 이며 Authorization Code 을 파라미터로 받는 URL 입니다.

<br>

## 4.2. 코드 받기 테스트

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_11_21_53_26.png?raw=true" width="85%">

앱을 등록하고 나면 이렇게 정보를 얻을 수 있습니다.

네이버는 카카오와 달리 Client Secret 값을 추가로 제공하는데, 나중에 인가 코드를 요청할 때 필요합니다.

`client_id`, `redirect_uri`, `response_type`, `state` 파라미터를 사용해서 네이버 로그인 페이지 URL 을 만듭니다.

파라미터 값에 대한 상세한 설명은 [Naver Developers - 네이버 로그인 요청 변수](https://developers.naver.com/docs/login/api/api.md#3--%EC%9A%94%EC%B2%AD-%EB%B3%80%EC%88%98) 를 참고하시면 됩니다.

```text
https://nid.naver.com/oauth2.0/authorize
?response_type=code
&client_id=Y2i4SlApP7A1KZsUoott
&state=hLiDdL2uhPtsftcU
&redirect_uri=http://localhost:8080/naver/callback
```

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/spring/images/screen_2023_03_11_22_21_02.png?raw=true" width="50%" height="50%">

로그인을 완료하면 우리가 등록해둔 Callback URL 로 이동합니다.

URL 의 파라미터를 확인하면 `code`, `state` 값을 볼 수 있습니다.

```text
http://localhost:8080/naver/callback
?code=Sl6x32RuDGpdIXCjmV
&state=hLiDdL2uhPtsftcU
```

<br>

# 5. Conclusion

다음 포스팅에서는 Spring Boot 코드를 직접 짜보겠습니다.