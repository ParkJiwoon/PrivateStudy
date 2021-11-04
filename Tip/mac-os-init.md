# Mac OS 초기 설정과 유용한 앱

# Overview

최근에 Mac OS 를 BigSur 로 업그레이드 하면서 데이터를 초기화 했습니다.

거의 2년 넘게 사용하던 설정들이 다 초기화 돼서 하나씩 설정했는데 다음에 또 세팅할 일이 생기면 볼 수 있게 기록해둡니다.

<br>

# 1. 환경설정

## 1.1. 트랙패드 탭해서 클릭

<img src="https://user-images.githubusercontent.com/28972341/139861950-2d8b0417-b974-406c-a99b-7cf238c51d1c.png" width="70%">

맥북은 기본적으로 꾹 눌러서 클릭하는데 위 설정을 키면 손가락 한번 탭한 것으로 클릭 가능합니다.

더블 클릭도 마찬가지로 두번 탭하면 됩니다.

<br>

## 1.2. 트랙패드 세손가락으로 드래그

<img src="https://user-images.githubusercontent.com/28972341/139862421-8ac4ff66-12a5-4890-acb9-e4050f23780d.png" width="70%">

`환경설정 > 손쉬운 사용 > 포인터 제어기 > 트랙패드 옵션` 에 들어가서 활성화 가능합니다.

아이콘, 이미지 등을 드래그 할 때 꾹 누르는 대신 세 손가락으로 쉽게 드래그 가능합니다.

<br>

## 1.3. 마우스 가속도 끄기

```sh
# 현재 감도 확인
defaults read .GlobalPreferences com.apple.mouse.scaling

# 새로운 감도 지정 (-1 로 설정하면 끄기)
defaults write .GlobalPreferences com.apple.mouse.scaling -1
```

기본적으로 마우스 가속도가 켜져 있는데 저는 불편해서 끄는 편입니다.

인텔 맥북에서만 가능하고 M1 맥북에서는 별도의 툴을 다운받아야 한다고 하네요.


<br>

## 1.4. ₩ 를 ` 로 입력하게 바꾸기

BigSur OS 로 업그레이드 하고 나니 한글 상태에서 1 왼쪽에 있는 키가 \` (백틱) 대신 `₩` 로 입력되어 마크다운이나 코드블럭을 작성할 때 불편했습니다.

https://ani2life.com/wp/?p=1753 포스트를 참고하여 키 설정 변경 후 각 애플리케이션들을 새로 시작하면 각각 적용됩니다.

```sh
# KeyBindings 디렉토리 생성
$ mkdir ~/Library/KeyBindings

# DefaultkeyBinding.dict 생성 및 편집
$ vi ~/Library/KeyBindings/DefaultkeyBinding.dict

# 아래 내용 추가
{
    "₩" = ("insertText:", "`");
}
```

<br>

## 1.5. Mac 에서 한글 입력이 잘 안되는 현상

<img src="https://user-images.githubusercontent.com/28972341/140368046-1535a725-347c-4f91-b230-656f77e0ebe5.png" width="70%">

개인적인 이슈였을 수도 있는데, 맥북 에디터에서 한글을 입력하면 자음이나 모음 한두개가 입력되지 않고 씹히는 현상을 겪었습니다.

`환경설정 > 키보드 > 입력 소스` 로 이동해서 "삭제 방식" 을 "글자" 로 변경하면 해결됩니다.

변경 후에는 백스페이스로 글자를 지울 때 자음, 모음 단위로 안 지워지고 무조건 문자 단위로 지워지는데 크게 불편한 점은 못느끼고 있습니다.

<br>

# 2. 유용한 앱들

다음은 제가 맥북에서 사용하는 앱들입니다.

<br>

## 2.1. iShot

[App Store 에서 다운](https://apps.apple.com/kr/app/ishot-%E4%BC%98%E7%A7%80%E7%9A%84%E6%88%AA%E5%9B%BE%E5%BD%95%E5%B1%8F%E5%B7%A5%E5%85%B7/id1485844094) 받을 수 있는 화면 캡쳐 프로그램입니다.

실행만 하면 `option + A` 조합으로 별다른 설정 없이 바로 사용할 수 있습니다.

전체화면, 특정 창, 드래그, 스크롤 캡쳐 등 다양한 기능을 제공하고 캡쳐한 사진을 바로 편집해서 복사, 저장할 수 있습니다.

<br>

## 2.2. itsycal

<img src="https://user-images.githubusercontent.com/28972341/140375421-d0d6dc82-082a-49c2-b3e4-9b8bd9ecd58f.png" width="30%">

https://www.mowglii.com/itsycal/ 에서 다운로드 하거나 `brew install --cask itsycal` 명령어로 설치할 수 있습니다.

맥북은 달력 보기가 굉장히 불편해서 하나 설치해서 보면 좋습니다.

맥 캘린더랑 연동도 되는 것 같은데 달력 보는 용도 외에는 사용해본 적 없는거 같네요.

<br>

## 2.3. Rectangle

<img src="https://user-images.githubusercontent.com/28972341/139869750-071d2003-5eab-4acb-ad5a-44766965749d.png" width="30%">

https://rectangleapp.com/ 에서 직접 다운받아 사용 가능합니다.

윈도우에서는 기본으로 제공되는 화면 분할이 맥에서는 없기 때문에 별도의 앱이 필요합니다.

`control + option + @` 의 조합으로 사용할 수 있으며 생각한것보다 굉장히 유용합니다.

더블 모니터를 사용하는 경우에는 서로 이동도 돼서 마우스 사용 빈도가 낮아졌습니다.

<br>

## 2.5. AppCleaner

앱 지울 때 사용하기 위해 [다운](https://freemacsoft.net/appcleaner/) 받습니다.

<br>

## 2.6. Notion

https://www.notion.so/desktop 에서 다운가능합니다.

업무용, 개인용으로도 사용하기 굉장히 좋은 메모 앱입니다.

Mac OS, Window, Mobile 전부 호환 돼서 편리합니다.

<br>

## 2.7. Spark

<img src="https://user-images.githubusercontent.com/28972341/139893436-9a9beb91-3e72-4e66-9770-9a906f012c85.png" width="50%">

맥북 기본 메일앱이 마음에 안들어서 [App Store 에서 다운](https://apps.apple.com/kr/app/spark-email-app-by-readdle/id1176895641) 받았습니다.

기본 앱은 메일을 받아도 노티가 오래 걸렸는데 스파크는 속도가 빨라서 유용하게 사용하고 있습니다.

<br>

## 2.8. Amphetamine

[App Store 에서 다운](https://apps.apple.com/kr/app/amphetamine/id937984704) 가능한 잠자기 방지 앱입니다.

원래는 환경 설정에서 화면 보호기 없는걸로 세팅했었는데 이 앱을 사용하면 좀더 여러 가지 설정이 가능합니다.

저는 그냥 항상 깨움 상태로 사용합니다.

<br>

## 2.9. Visual Studio Code

유용한 에디터이며 [공식 홈페이지에서 다운](https://code.visualstudio.com/) 가능합니다.

개발용으로도 많이 쓰지만 저는 JetBrain 제품을 사용하기 때문에 마크다운 에디터로만 사용하고 있습니다.

설치한 확장팩
- Markdown All in One : Markdown 작성 도와주는 확장팩
- Markdown Preview Enhanced : Markdown Github 스타일로 미리보기 가능
- Material Icon Theme : 그냥 아이콘이 이뻐보여서..
- One Dark Pro : 그냥 테마
