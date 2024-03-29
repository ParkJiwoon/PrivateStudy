# AWS 1편: EC2 생성 후 Spring Boot 띄우기

# Overview

AWS EC2 인스턴스를 생성하고 Spring Boot 서버를 띄워보는 것까지 진행합니다.

주 목표는 서버를 외부에 제공하는 거라서 따로 배포 시스템을 구축하지 않고 단순히 빌드 파일을 복사해서 수동으로 띄울 겁니다.

글은 다음과 같은 순서로 진행합니다.

- EC2 인스턴스 생성
- 탄력적 IP (Elastic IP) 추가
- 터미널에서 SSH 클라이언트로 EC2 접속
- 보안 그룹 설정
- Spring Boot 서버 띄우기

<br>

# 1. EC2 인스턴스 생성

우선 [AWS 홈페이지](https://aws.amazon.com) 에 접속해서 계정을 생성 후 콘솔에 로그인 된 상태여야 합니다.

2022년 1월 16일 기준으로 작성되었으며 이후에 홈페이지 인터페이스가 변경될 수 있습니다.

**2022년 4월 29일 기준으로 AWS EC2 인스턴스 생성 UI 가 바뀌어서 이부분을 수정합니다.**

<br>

## 1.1. AWS Region 설정

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_18_38_44.png" width="50%">

우선 위치를 서울로 설정합니다.

리전에 따라서 인스턴스 위치가 결정되기 때문에 외국으로 하면 속도가 낮을 수도 있습니다.

만약 대한민국이 아닌 다른 나라에 서비스 하려면 그 도시를 선택해도 됩니다.

<br>

## 1.2. EC2 메뉴로 이동

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_18_41_13.png">

처음에 대시보드가 나올텐데 만약 EC2 서비스를 찾기 힘들다면 검색해서 들어갑니다.

<br>

## 1.3. 새 인스턴스 생성

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_18_34_30.png">

인스턴스 메뉴로 들어가 인스턴스 시작 버튼을 클릭합니다.

<br>

## 1.4. Amazon Machine Image(AMI) 및 인스턴스 유형 선택

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_04_30_03_12_44.png" width="70%">

AMI 는 어떤 서버로 구성할지 선택하는 겁니다.

여러 종류가 있어서 원하는 걸 선택하면 되고, 저는 프리 티어에 Ubuntu LTS 버전을 선택했습니다.

인스턴스 유형은 프리 티어를 사용한다면 다른 선택권이 없습니다.

스펙이 좋을수록 과금이 더 많이 되기 때문에 처음부터 좋은걸 고르기보다는 작게 시작했다가 스케일업 해나가는 걸 추천드립니다.

<br>

## 1.5. 키 페어 생성

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_04_30_03_29_30.png" width="70%">

키 페어는 EC2 서버에 SSH 접속을 하기 위해 필수라서 생성해야 합니다.

"새 키 페어 생성" 을 눌러 원하는 이름을 적고 생성합니다.

위 그림처럼 설정 후 생성하면 자동으로 `my-key.pem` 파일이 다운되며, SSH 환경에 접속하기 위해서는 해당 키 파일이 존재하는 위치로 가서 `ssh` 명령어를 실행하면 됩니다.

한번 다운받은 후에는 재다운 받을 수 없기 때문에 안전한 곳에 저장해둡니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_04_30_03_34_29.png" width="70%">

생성한 후에는 이렇게 키 페어를 선택하면 됩니다.

<br>

## 1.6. 네트워크 및 스토리지 선택

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_04_30_03_39_54.png" width="70%">

네트워크 설정은 EC2 에 접속을 허용하는 ACL 을 생각하면 됩니다.

나중에 "보안 그룹" 이라는 걸로 별도 설정을 할 예정이기 때문에 SSH 트래픽만 허용해줍니다.

위 설정대로 하면 SSH 트래픽 접속 가능한 IP 가 내 IP 로 자동 설정 됩니다.

<br>

프리티어는 최대 30 까지 지원하기 때문에 해당 부분만 변경해줍니다.

볼륨 유형은 범용 SSD 로 선택해야 합니다.

만약 Provisioned IOPS SSD (프로비저닝된 IOPS SSD) 를 선택한다면 사용하지 않아도 활성화한 기간만큼 계속 비용이 발생하게 됩니다.

<br>

## 1.7. 인스턴스 설정 요약

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_04_30_03_53_57.png" width="70%">

다 설정하면 우측에 간단하게 지금까지 설정한 인스턴스 요약이 뜹니다.

이상한 부분이 없다면 인스턴스 시작을 눌러서 생성하면 됩니다.

<br>

## 1.8. 인스턴스 생성 완료

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_19_19_54.png">

처음 화면으로 돌아오면 이렇게 인스턴스가 생성된 것을 확인할 수 있습니다.

이제 **탄력적 IP 와 보안 그룹**을 추가하기 위해 인스턴스 ID 를 클릭합니다.

<br>

# 2. 탄력적 IP (Elastic IP)

AWS EC2 인스턴스는 서버를 중지하고 다시 실행시키면 퍼블릭 IP 가 변경되기 때문에 클라이언트가 사용할 수 있는 변하지 않는 IP 가 필요합니다.

탄력적 IP (Elastic IP) 란 외부에서 인스턴스에 접근 가능한 고정 IP 입니다.

탄력적 IP 는 만들어놓고 사용하지 않더라도 과금이 되기 때문에 필요한 만큼만 생성하는 것이 중요합니다.

자세한 설명은 [AWS Docs - 탄력적 IP](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) 에서 확인하실 수 있습니다.

<br>

우선, 진짜 퍼블릭 IP 가 변경하는지 한번 확인해봅시다.

현재 생성된 인스턴스의 정보입니다.

퍼블릭 IP 로 `3.35.238.69` 가 할당된 것을 확인할 수 있습니다.

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_19_54_10.png">

<br><br>

인스턴스를 중지 시켰다가 다시 실행시켜보았습니다.

인스턴스 ID, 프라이빗 IP 와 같은 정보는 동일한데 퍼블릭 IP 가 `13.125.3.218` 로 변경된 것을 볼 수 있습니다.

이렇게 퍼블릭 IP 는 변경될 가능성이 있기 때문에 변하지 않는 탄력적 IP 를 할당해주어야 합니다.

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_20_51_35.png">

<br><br>

## 2.1. 탄력적 IP 메뉴 접근

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_19_55_46.png">

메뉴에서 탄력적 IP 를 찾아서 들어갑니다.

아직 아무것도 할당받은 게 없기 때문에 새로운 IP 를 추가합니다.

<br>

## 2.2. 새로운 탄력적 IP 할당

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_20_19_06.png" width="60%">

딱히 변경할 건 없고 바로 할당을 선택해서 만들어줍시다.

<br>

## 2.3. 탄력적 IP 주소 선택

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_22_04_35.png">

방금 생성한 탄력적 IP 를 선택해서 연결을 시도합니다.

<br>

## 2.4. 인스턴스 선택 및 연결

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_22_05_33.png" width="70%">

설정 화면에 들어가면 현재 내 인스턴스 목록을 선택할 수 있고 연결된 프라이빗 IP 까지 선택 가능합니다.

<br>

## 2.5. 인스턴스 정보 확인

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_22_07_58.png">

탄력적 IP 를 연결하고 다시 인스턴스 정보를 확인해보면 IP 가 할당된 것을 볼 수 있습니다.

퍼블릭 IP 주소도 기존 값에서 탄력적 IP 주소로 자동으로 변경되었습니다.

<br>

# 3. SSH 클라이언트로 서버 접속

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_22_08_51.png">

이제 내가 만든 EC2 인스턴스에 접속해봅니다.

인스턴스 정보에서 "연결" 버튼을 클릭하면 인스턴스에 연결 가능한 여러가지 방법을 알려줍니다.

여기서 "SSH 클라이언트" 로 접속하는 방법을 알아봅니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_21_30_20.png" width="70%">

사실 너무 친절하게 알려주고 명령어도 복사할수 있게 되어있어서 따로 할건 없습니다.

저 방법대로 서버에 한번 접속해보고, 호스트를 등록해서 간편하게 등록하는 방법을 알아봅시다.

<br>

## 3.1. 터미널 실행 후 키 페어 파일 위치로 이동

터미널을 실행해서 이전에 다운 받은 키 페어 파일 위치로 이동합니다.

저는 다운로드 받은 후 따로 옮기지 않았기 때문에 Downloads 디렉토리에 들어갔습니다.

키 파일이 없으면 접속할 수 없고 다시 다운받을 수도 없기 때문에 잘 관리해야 합니다.

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_22_12_44.png" width="70%">

<br>

## 3.2. 키 파일 권한 변경

```sh
$ chmod 400 my-key.pem
```

키 파일의 권한을 변경해줍니다.

<br>

## 3.3. SSH 접속

가이드에 있는 대로 **퍼블릭 DNS 또는 퍼블릭 IP 를 사용**해서 인스턴스에 접속할 수 있습니다.

```sh
# 퍼블릭 DNS 로 접속
$ ssh -i "my-key.pem" ubuntu@ec2-3-37-206-248.ap-northeast-2.compute.amazonaws.com

# 퍼블릭 IP 로 접속
$ ssh -i "my-key.pem" ubuntu@3.37.206.248
```

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_22_27_44.png" width="70%">

<br><br>

## 3.4. 호스트 등록해서 간편하게 접속

이제 우리는 SSH 로 EC2 인스턴스 서버에 접속할 수 있습니다.

하지만 위에서 본 것처럼 매번 `ssh -i {키 페어 파일} {ubuntu}@{탄력적 IP}` 를 입력해야 합니다.

일일히 기억하기도 귀찮고 타이핑도 번거롭기 때문에 호스트로 등록해서 쉽게 접속할 수 있도록 변경해봅시다.

<br>

### 3.4.1. ~/.ssh 디렉터리로 키 페어 파일 복사

우선 키 페어 파일을 `~/.ssh/` 로 복사합니다.

```sh
$ cp my-key.pem ~/.ssh/
```

<br>

### 3.4.2. 키 페어 파일 권한 변경

키 페어 파일의 권한을 변경합니다.

```sh
$ chmod 600 my-key.pem
```

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_00_52_45.png" width="50%">

<br><br>

### 3.4.3. ~/.ssh/config 파일 생성

`~/.ssh/` 디렉터리에 `config` 파일을 생성해서 다음과 같이 입력합니다.

이미 파일이 존재한다면 맨 아래에 입력하면 됩니다.

User 는 우분투를 선택했다면 ubuntu 고 그 외에는 전부 ec2-user 일겁니다.

```sh
$ vi ~/.ssh/config

# 아래는 파일 내용
# ssh -i {키 페어 파일} {유저 이름}@{탄력적 IP}
Host {원하는 호스트 이름}
User {유저 이름}
HostName {탄력적 IP}
IdentityFile {키 페어 파일 위치}
```

<br>

위 형식에 따라 저는 다음과 같이 작성했습니다.

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_01_35_11.png" width="40%">

<br><br>

### 3.4.4. 설정한 Host 이름으로 접속

이제 키 페어 파일이 없는 곳에서도 간단한 별칭으로 SSH 접속이 가능합니다.

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_01_36_55.png" width="70%">

<br><br>

# 4. 보안 그룹 설정

보안 그룹은 AWS 에서 제공하는 방화벽 모음입니다.

서비스를 제공하는 애플리케이션이라면 상관 없지만 RDS 처럼 외부에서 함부로 접근하면 안되는 인스턴스는 허용된 IP 에서만 접근하도록 설정이 필요합니다.

- 인바운드 (Inbound): 외부 -> EC2 인스턴스 내부 허용
- 아웃바운드 (Outbound): EC2 인스턴스 내부 -> 외부 허용

<br>

## 4.1. 현재 보안 그룹 확인

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_16_22_51_44.png">

인스턴스 정보의 보안 탭에서 현재 설정된 보안 그룹을 확인할 수 있습니다.

처음 인스턴스를 생성할 때 아무런 보안 그룹을 설정하지 않았기 때문에 default 값만 들어가있습니다.

인바운드 규칙 해석해보면 22번 포트의 모든 IP 에 대해서 TCP 연결을 허용한다는 의미입니다.

SSH 접속을 위해 22번 포트는 기본으로 설정해준 것 같네요.

이제 새로운 보안 그룹을 만들어 봅시다.

<br>

## 4.2. 보안 그룹 ID 리스트 확인

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_00_02_06.png">

인스턴스에서 보안 그룹 ID 를 보고 들어갈 수도 있지만 메뉴에서 직접 들어가면 이렇게 보안 그룹 ID 리스트를 볼 수 있습니다.

보안 그룹은 인스턴스와 별개로 존재하기 때문에 한번 만들어두면 새 인스턴스를 생성해도 한번에 적용할 수 있습니다.

우측 상단의 "보안 그룹 생성" 버튼을 눌러서 새 보안 그룹을 만들어봅시다.

<br>

## 4.3. 보안 그룹 생성

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_00_28_46.png">

먼저 보안 그룹의 이름과 설명을 추가하고 인바운드 규칙과 아웃바운드 규칙을 편집합니다.

<br>

## 4.4. 인바운드 규칙

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_01_54_03.png">

인바운드는 외부에서 EC2 로 요청할 때 허용할 IP 대역을 설정할 수 있습니다.

원하는 규칙을 추가하려면 "규칙 추가" 버튼을 누릅니다.

유형을 먼저 선택하면 자동으로 그에 맞는 프로토콜과 포트 범위가 고정됩니다.

웬만한 유형들이 이미 존재하기 때문에 편하게 설정할 수 있습니다.

우선 로컬 PC 에서 서버에 접속할 수 있게 SSH 를 추가하고 소스를 내 IP 로 추가합니다.

"소스" 를 선택하면 우측에 알아서 IP 가 추가되며, 특정 IP 나 대역을 넣으려면 "사용자 지정" 으로 추가할 수 있습니다.

만약 여러 사람이 함께 작업하는 프로젝트라면 각각의 로컬 PC IP 를 전부 추가해줘야 합니다.

SSH, HTTP, HTTPS 와 같은 기본 포트들을 열고 스프링 부트 프로젝트트까지 사용자 지정으로 추가해줍니다.

<br>

## 4.5. 아웃바운드 규칙

아웃바운드 규칙은 딱히 설정할 필요 없기 때문에 "모든 트래픽" 그대로 둡니다.

<br>

## 4.6. 인스턴스에서 보안 그룹 변경

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_02_04_12.png">

다시 인스턴스로 돌아와서 "보안 그룹 변경" 버튼을 눌러줍니다.

<br>

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_02_06_30.png" width="70%">

방금 생성한 MySecurityGroup 을 추가해줍니다.

보안 그룹은 여러 개를 동시에 설정할 수 있기 때문에 기존에 설정된 launch-wizard-1 보안 그룹은 제거해줍니다.

<br>

## 4.7. 변경된 보안 그룹 확인

<img src="https://github.com/ParkJiwoon/PrivateStudy/raw/master/ci-cd/images/screen_2022_01_17_02_09_06.png" width="70%">

다시 인스턴스 설정을 보면 보안 그룹이 적용된 것을 볼 수 있습니다.

만약 제대로 보이지 않는다면 새로고침을 한번 해주세요.

<br>


# 5. EC2 인스턴스에 Spring Boot 서버 띄우기

배포 시스템 이런거 생략하고 진짜 단순하게 서버 띄우는 것만 확인해봅니다.

가장 간단한 방법은 두가지가 있는데 1번으로 진행하겠습니다.

1. jar 파일을 빌드하여 EC2 복사 후 실행
2. EC2 에서 프로젝트 git clone 후 실행

<br>

## 5.1. JDK 설치

```sh
# EC2 인스턴스
$ sudo apt-get update
$ sudo apt-get install openjdk-11-jdk
```

`java -version` 으로 명령어로 설치 여부를 확인할 수 있습니다.

<br>

## 5.2. Spring Boot 프로젝트 빌드

```sh
# 프로젝트 파일로 이동 후
$ gradle clean build

BUILD SUCCESSFUL in 5s
17 actionable tasks: 10 executed, 7 up-to-date


# 빌드 파일 복사
$ scp ./build/libs/api-0.0.1-SNAPSHOT.jar {호스트 이름}:/home/ubuntu
```

프로젝트를 빌드하면 `./build/libs` 디렉토리에 jar 파일이 생성됩니다.

해당 파일을 EC2 서버로 복사합니다.

호스트 이름에는 `ubuntu@{퍼블릭 IP}` 또는 `ubuntu@{퍼블릭 DNS}` 가 들어가야 하는데 만약 `~/.ssh/config` 에 호스트 이름을 등록해두었다면 간소화된 이름을 사용할 수 있습니다.

퍼블릭 IP (탄력적 IP) 또는 퍼블릭 DNS 를 그대로 사용한다면 키 페어 파일 (.pem) 이 명령어를 사용하는 위치에 존재해야 합니다.

<br>

## 5.3. EC2 인스턴스에서 실행

```sh
# EC2 인스턴스
$ nohup java -jar api-0.0.1-SNAPSHOT.jar &
```

`nohup` 명령어를 사용하면 로그를 `nohup.out` 파일에 남길 수 있습니다.

<br>

## 5.4. 퍼블릭 IP 또는 DNS 로 접근 확인

`http://{탄력적 IP}`로 접속하면 정상적으로 서버에 연결이 되는 걸 볼 수 있습니다.

<br>

# Conclusion

지금까지 AWS EC2 인스턴스를 생성하고, 변하지 않는 탄력적 IP 를 할당해주고, 보안 그룹으로 방화벽을 설정한 후에 서버를 띄우는 것까지 진행해봤습니다.

다음에는 데이터베이스를 관리하는 RDS 인스턴스를 만들어보겠습니다.