# AWS 2편: RDS 생성 후 EC2 와 연동

# Overview

[지난 포스팅](aws-01-ec2.md)에서는 AWS 에서 EC2 인스턴스를 생성하고 Spring Boot 서버를 띄워 외부에서 요청하는 것까지 해봤습니다.

이번에는 데이터베이스 연동을 위해 RDS 인스턴스를 생성하고 이전에 만든 EC2 와 연동하는 것까지 진행해봅니다.

다음과 같은 순서로 진행됩니다.

- RDS 인스턴스 생성
- 보안 그룹 설정
- RDS 접속 테스트
- 파라미터 그룹 설정

<br>

# 1. RDS 인스턴스 생성

EC2 서버에 DB 연동을 하기 위한 RDS 인스턴스를 생성해봅니다.

<br>

## 1.1. RDS 메뉴로 이동

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_02_17_10.png" width="90%">

EC2 와 마찬가지로 검색하면 쉽게 찾을 수 있습니다.

<br>

## 1.2. 데이터베이스 생성

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_02_18_39.png" width="90%">

대시보드에서 선택해도 되고 아니면 이렇게 메뉴에 진입해서 직접 데이터베이스 생성을 눌러도 됩니다.

<br>

## 1.3. DB 종류 선택

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_02_27_03.png" width="90%">

저는 MySQL 을 선택했습니다.

<br>

## 1.4. DB 설정 입력

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_15_00.png" width="90%">

데이터베이스 이름, 마스터 이름, 비밀번호를 입력합니다.

실제 DB 에 접근할 때 사용할 정보이므로 신중하게 입력해야 합니다.

<br>

## 1.5. 스토리지 설정

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_02_31_46.png" width="90%">

사실 프리 티어일 때는 선택권이 거의 없습니다.

그냥 대부분 기본 설정을 사용하면 되는데 "스토리지 자동 조정" 저 부분 체크만 해제해주시면 됩니다.

안그러면 개발을 진행하다 임계값이 초과되면 자동으로 스토리지가 늘어나서 과금될 가능성이 있습니다.

<br>

## 1.6. 보안 그룹 설정

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_12_35.png" width="90%">

퍼블릭 액세스는 "예" 로 지정해줍니다.

"아니요"를 선택하면 퍼블릭 IP 주소가 할당되지 않기 때문에 외부에서 접속할 수 없습니다.

그리고 EC2 와 마찬가지로 보안 그룹을 설정하거나 새로 생성할 수 있습니다.

기존에 사용 중인게 있다면 "기존 항목 선택" 을 누르고 보안 그룹을 추가하면 됩니다.

여기서는 "새로 생성" 으로 해보겠습니다.

<br>

## 1.7. 추가 구성

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_02_37_46.png" width="90%">

추가 구성에서 데이터베이스 이름을 적고 자동 백업을 비활성화 합니다.

어차피 개발용이라 데이터가 중요하지 않아서 자동 백업을 비활성화 했지만, 만약 지워져도 복구해야 하는 데이터라면 당연히 백업을 활성화 해야 합니다.

여기까지 진행했으면 "데이터베이스 생성" 을 눌러서 생성을 완료합니다.

생성 완료까지는 시간이 좀 걸립니다.

<br>

# 2. RDS 보안 그룹 설정

RDS 인스턴스를 생성할 때 보안 그룹을 새로 생성한 걸 기억하실 겁니다.

데이터베이스는 서버에서 접근 가능해야 하기 때문에 보안 그룹 설정이 추가로 필요합니다.

여기서 서버란 곧 "EC2 인스턴스의 탄력적 IP" 를 의미합니다.

탄력적 IP 를 직접 넣는 대신 손쉽게 설정할 수 있는 방법이 있습니다.

<br>

## 2.1. 현재 보안 그룹 확인

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_24_02.png" width="90%">

RDS 인스턴스 정보로 들어가면 하단에서 보안 그룹 규칙을 확인할 수 있습니다.

친절하게 제가 접속할 수 있게 로컬 IP 만 인바운드 규칙에 추가해두었네요.

아웃바운드는 EC2 와 마찬가지로 모든 트래픽에 대해 열어두었습니다.

RDS 보안 그룹은 EC2 보안 그룹이랑 별도로 관리하지 않고 같은 곳에서 관리합니다.

따라서, 보안 그룹을 편집하려면 EC2 대시보드로 이동하거나 저 보안 그룹 이름을 클릭해서 페이지를 열어야 합니다.

<br>

## 2.2. 보안 그룹 리스트에서 EC2 보안 그룹 ID 복사

EC2 대시보드의 보안 그룹 메뉴로 이동하면 지금까지 만들었던 보안 그룹 리스트를 확인할 수 있습니다.

이전에 만들었던 MySecurityGroup 도 있을 텐데 보안 그룹 ID 를 복사해줍니다.

<br>

## 2.3. RDS 인바운드 규칙에 EC2 보안 그룹 ID 입력

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_25_10.png" width="90%">

유형을 MYSQL/Aurora 로 선택하고 사용자 지정으로 EC2 보안 그룹의 ID 를 추가하고 저장합니다.

<br>

# 3. RDS 접속 테스트

보안 그룹 설정까지 했다면 실제로 연결이 잘 되는지 다음 두 가지를 테스트 해봅시다.

- 로컬 PC 에서 접속
- EC2 인스턴스 서버에서 접속

<br>

## 3.1. 엔드포인트와 포트 확인

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_31_52.png" width="90%">

RDS 인스턴스 정보에서 엔드포인트와 포트를 확인합니다.

<br>

## 3.2. 로컬 PC (Sequel Ace) 에서 접속

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_35_09.png" width="70%">

Database GUI 툴은 여러 가지가 있으므로 각자 편한걸로 접근하시면 됩니다.

저는 Sequel Ace 에서 접속을 시도해보겠습니다.

아래 세가지만 입력 후 Connect 를 누르면 접속 가능합니다.

- Host: RDS 엔드포인트
- Username: RDS 생성 시 입력했던 정보
- Password: RDS 생성 시 입력했던 정보

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_36_53.png" width="30%">

RDS 생성 시에 지정했던 초기 데이터베이스 이름도 확인할 수 있네요.

<br>

## 3.3. EC2 에서 접속

먼저 EC2 에 접속해줍니다.

우선 MySQL 을 먼저 설치해줘야 합니다.

```sh
# Ubuntu 에서 MySQL 설치
$ sudo apt-get update
$ sudo apt-get install mysql-server
```

<br>

그리고 mysql 명령어로 접속을 시도합니다.

권한 문제가 있으면 sudo 로 재시도 합니다.

```sh
# mysql -u {유저이름} -p --host {엔드포인트}
$ mysql -u admin -p --host my-rds-instance.ciweuig9oiko.ap-northeast-2.rds.amazonaws.com
```

<br>

접속이 잘 된것을 확인할 수 있습니다.

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_47_55.png" width="90%">

<br>

# 4. RDS 파라미터 그룹 설정

이제 추가적으로 파라미터 그룹 설정을 해줍시다.

RDS 는 다음 세가지 설정을 필수로 해줘야 합니다.

- Time Zone
- Character Set
- Max Connection

<br>

## 4.1. 파라미터 그룹 페이지로 이동

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_03_58_46.png" width="90%">

먼저 파라미터 그룹 메뉴를 찾아 이동합니다.

<br>

## 4.2. 파라미터 그룹 생성

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_04_02_07.png" width="90%">

파라미터 그룹 패밀리는 RDS DB 와 맞춰서 선택하고 이름과 설명만 입력해서 생성합니다.

생성한 파라미터 그룹을 클릭해서 파라미터 편집을 누릅니다.

<br>

## 4.3. Time Zone

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_04_08_33.png" width="90%">

타임존을 Asia/Seoul 로 변경합니다.

<br>

## 4.4. Character Set

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_04_10_29.png" width="90%">

character_set 으로 검색해서 나온 6 개의 값을 전부 `utf8mb4` 로 변경해줍니다.

원래는 `utf8` 을 많이 사용했으나 `utf8mb4` 가 이모지까지 지원하기 때문에 더 많이 사용되는 추세입니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_04_13_08.png" width="90%">

collation 으로 검색해서 나온 값들도 전부 `utf8mb4_general_ci` 로 변경해줍니다.

<br>

## 4.5. Max Connection

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_04_14_59.png" width="90%">

마지막으로 max_connections 을 수정해줍니다.

이 값은 원래 RDS 인스턴스 사양에 의해 결정됩니다.

<br>

## 4.6. 최종 변경사항 정리

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_04_16_06.png" width="90%">

저장하기 전에 미리보기를 하면 마지막으로 변경 사항들을 확인할 수 있습니다.

<br>

## 4.7. RDS 파라미터 그룹 변경

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_04_19_15.png" width="90%">

RDS 인스턴스로 이동해서 "수정" 버튼을 클릭합니다.

그리고 "추가 구성" 탭으로 이동해서 DB 파라미터 그룹을 변경해줍니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_04_20_08.png" width="90%">

RDS 는 인스턴스를 수정할 때 예약 적용과 즉시 적용을 선택할 수 있는데 초기 설정이므로 "즉시 적용" 을 선택해줍시다.

<br>

# Conclusion

이렇게 해서 EC2 에 인스턴스 연동까지 진행해봤습니다.

Spring Boot 에서 DB 에 연동하고 싶다면 로컬에서 접속한 것처럼 세팅해주고 진행하면 됩니다.

이제 DB 연동한 서버를 외부에 노출하는 것까지 가능합니다.