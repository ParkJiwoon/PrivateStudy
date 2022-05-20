# IntelliJ 유용한 설정 및 플러그인

# 1. Overview

IntelliJ IDEA 를 처음 설치했을때 할만한 세팅과 유용한 플러그인을 알아봅니다.

Ultimate 을 기준으로 합니다.

- [IntelliJ 유용한 설정 및 플러그인](#intellij-유용한-설정-및-플러그인)
- [1. Overview](#1-overview)
- [2. Configuration](#2-configuration)
  - [2.1. SDK 설정](#21-sdk-설정)
  - [2.2. Auto Import 체크](#22-auto-import-체크)
  - [2.3. 대소문자 구분 체크 해제](#23-대소문자-구분-체크-해제)
  - [2.4. Build Memory 늘리기](#24-build-memory-늘리기)
  - [2.5. Memory Indicator](#25-memory-indicator)
  - [2.6. Always Select Opened File](#26-always-select-opened-file)
  - [2.7. Gradle Build 를 IntelliJ IDEA 로 변경](#27-gradle-build-를-intellij-idea-로-변경)
  - [2.8. Annotation Processor](#28-annotation-processor)
  - [2.9. Inlay Hints](#29-inlay-hints)
- [3. Plugin](#3-plugin)
  - [3.1. Key Promoter X](#31-key-promoter-x)
  - [3.2. Rainbow Brackets](#32-rainbow-brackets)
  - [3.3. CodeGlance](#33-codeglance)
  - [3.4. GitToolBox](#34-gittoolbox)
- [Reference](#reference)

<br>

# 2. Configuration

필수 설정도 있고 단순한 편의 용도도 있습니다.

<br>

## 2.1. SDK 설정

`File > Project Structure... > Project SDK` 에서 사용 중인 JDK 를 지정합니다.

<img src="https://user-images.githubusercontent.com/28972341/144170909-12d45fa9-fa69-4edd-8e8d-c6ee847d3d0e.png" width=80%>

<br><br>

## 2.2. Auto Import 체크

<img src="https://user-images.githubusercontent.com/28972341/144170975-03bff7f0-8625-4d0a-aa14-956e96611baa.png" width=80%>

<br><br>

## 2.3. 대소문자 구분 체크 해제

`system` 을 검색하면 대소문자가 구별되어서 `System` 이 안나오기 때문에 체크 해제합니다.

대소문자 구분이 필요하면 검색창에서 필터를 추가할 수 있습니다.

<img src="https://user-images.githubusercontent.com/28972341/144171108-febac5c3-69d4-4b0b-8260-3179a0b06f95.png" width=80%>

<br><br>

## 2.4. Build Memory 늘리기

빌드할 때 메모리 때문에 실패할 수 있습니다.

Heap Size 를 늘려줍니다.

<img src="https://user-images.githubusercontent.com/28972341/144173534-fe50a274-f028-4e8b-8516-531be522f860.png" width=80%>

<br>

`Help > Edit Custom VM Options..` 에서 추가로 아래 설정을 해주면 좀더 쾌적하게 이용 가능합니다.

([IntelliJ Memory Option 최적화](https://snow-line.tistory.com/34) 참고)

```text
-Xmx4096m
-Xms4096m
```

<br>

## 2.5. Memory Indicator

메모리 정보를 실시간으로 확인하고 싶다면 인텔리제이 우측 하단을 우클릭하고 `Memory Indicator` 를 체크하면됩니다.

<img src="https://user-images.githubusercontent.com/28972341/144173752-c2e5f51a-26e0-409c-8130-7b3e3b486631.png" width=80%>

<br><br>

## 2.6. Always Select Opened File

파일 위치를 검색해서 들어가는 경우 왼쪽 파일 리스트에서 위치를 찾지 못할 때가 있습니다.

`Project > Show Options Menu (톱니바퀴) > Always Select Opened File` 을 활성화하면 현재 열려있는 파일 위치로 이동시켜줍니다.

<img src="https://user-images.githubusercontent.com/28972341/144173879-f518952e-31e3-4be1-98d4-b6867fa83f19.png" width=80%>

<br><br>

## 2.7. Gradle Build 를 IntelliJ IDEA 로 변경

Gradle 을 사용할 때만 Build 속도를 향상시킬 수 있습니다.

<img src="https://user-images.githubusercontent.com/28972341/144173983-13e2738c-03fa-4c2b-a2b3-d95c387f70d7.png" width=80%>

<br><br>

## 2.8. Annotation Processor

<img src="https://user-images.githubusercontent.com/28972341/144175333-951e6d68-d829-4e24-9ee3-3a24dd2072b9.png" width=80%>

<br><br>

## 2.9. Inlay Hints

Kotlin 을 사용하는 경우 `val`, `var` 를 사용하여 변수를 선언하는데, 타입을 명시하지 않는 경우도 있습니다.

타입을 생략하는 경우 어떤 타입인지 한눈에 안들어올 수가 있는데 Inlay Hints 를 켜면 타입을 알려줍니다.

언어별로 설정할 수도 있으며 저는 그냥 다 켜두는 편입니다.

<img src="https://github.com/ParkJiwoon/PrivateStudy/blob/master/intellij/images/screen_2022_05_20_22_57_50.png?raw=true" width="80%">

<br><br>

# 3. Plugin

플러그인은 필수는 아니지만 설치하면 개발 생산성 향상에 도움을 줍니다.

<br>

## 3.1. Key Promoter X

마우스 클릭으로 어떤 액션을 하면 단축키를 알려줍니다.

인텔리제이 단축키를 잘 모르거나 헷갈릴때 익히는 데 도움을 줍니다.

<img src="https://user-images.githubusercontent.com/28972341/144173126-93586cda-f896-415e-ab33-fd488ee85e37.png" width=80%>

<br><br>

## 3.2. Rainbow Brackets

여러 개의 괄호가 중첩될 때 색으로 구분해줍니다.

<img src="https://user-images.githubusercontent.com/28972341/144173254-80584051-1d31-4b5e-99f5-a4a14318179f.png" width=80%>

<br><br>

## 3.3. CodeGlance

코드 우측에 미니맵을 보여줍니다.

파일 크기가 크면 스크롤 할 때 편리하지만 분할해서 볼 때 공간을 차지하기 때문에 호불호가 좀 갈릴 거 같네요.

<img src="https://user-images.githubusercontent.com/28972341/144173400-7f442b61-f7b5-4a04-9af4-7ac8065edef3.png" width=80%>

<br><br>

## 3.4. GitToolBox

Git 에 관한 지원을 해줍니다.

Inline Blame 이 특히 유용합니다.

<img src="https://user-images.githubusercontent.com/28972341/144173448-68ce5467-a4e2-405a-9d2a-c5bd3962630a.png" width=80%>

<br><br>

# Reference

- [IntelliJ Memory Option 최적화](https://snow-line.tistory.com/34)