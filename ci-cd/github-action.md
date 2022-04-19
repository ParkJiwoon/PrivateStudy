# Github Actions 으로 AWS 에 Spring Boot 배포하기

# Overview

로컬 서버를 띄우면 내 컴퓨터에서만 접근 가능합니다.

그래서 외부에서도 접근 가능하도록 서버를 AWS 와 같은 클라우드 환경에 배포합니다.

가장 단순하게 배포하는 방법은 빌드한 파일을 AWS 환경에 전달해서 실행하는 겁니다.

하지만 만약 서버를 여러 개 구성했다면 일일히 파일을 복사해서 넘기는 일은 굉장히 번거롭고 몇 개 빼먹는 등 실수할 확률이 높습니다.

스크립트 파일을 따로 만들어서 사용하면 편해지지만 만약 여러 사람이 개발한다면 스크립트 파일을 서로 공유해야합니다.

그리고 스크립트 파일에 변경사항이 생길 때마다 (서버 추가) 모든 사람이 각자의 스크립트 파일을 수정해야 합니다.

이러한 번거로움을 막기 위해 CI/CD 라는 개념이 생겼습니다.

<br>

# CI/CD

- CI (Continuous Integration)
  - 해석하면 "지속적 통합" 으로 여러 개발자가 하나의 프로젝트를 같이 개발할 때 발생하는 불일치를 최소화 해주는 개념입니다.
  - CI 를 제대로 구현하면 애플리케이션 변경 사항 반영 시 자동으로 빌드 및 테스트 되어 잘못된 코드가 공유되는 걸 방지합니다.
- CD (Continuous Deployment)
  - "지속적 배포" 라는 뜻으로 프로젝트의 변경 사항을 가상 환경에 자동으로 배포하는 것을 의미합니다.
  - CD 를 구성해두면 변경 사항을 배포할 때 사용하는 파이프라인을 공유하여 번거로움을 없앨 수 있습니다.

좀더 자세한 내용은 [RedHat 공식문서](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)를 참고하세요.

<br>

# Github Actions

Github Actions 는 Github 에서 제공하는 CI/CD 툴입니다.

build, test, deploy 등 필요한 Workflow 를 등록해두면 Gihtub 의 특정 액션 (push, pull request) 이 발생했을 때 해당 워크 플로우를 수행합니다.

예를 들어 Pull Request 를 올리면 자동으로 해당 코드의 테스트를 수행하여 수행한다던지 master branch 에 코드를 push 하면 자동으로 코드를 배포하는 등 여러 가지 반복적인 작업을 자동으로 수행해줍니다.

Jenkins, Circle CI, Travis CI 등 다른 후보들도 있지만 Github Actions 은 별다른 툴을 설치하지 않아도 Github Repository 에서 바로 사용할 수 있다는 장점이 있습니다.

다만, 워크플로우를 실행하기 위한 [Github-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners) 의 스펙이 그렇게 좋은 편은 아니므로 아직은 소규모 프로젝트에 적합하다고 생각합니다.

<br>

# 배포 방법

Github Actions 는 단순한 Workflow 라서 배포하는 방법은 여러 가지가 존재합니다.

구글에 검색했을 때 가장 많이 나오는 방식은 다음과 같습니다.

1. 빌드한 jar 파일을 압축
2. AWS S3 에 업로드
3. AWS CodeDeploy 를 사용해서 EC2 에 배포

<br>

하지만 Github Actions 공식에서 가이드하는 AWS 배포 방식은 조금 다릅니다.

1. jar 파일을 Docker 이미지로 빌드
2. AWS 에서 제공하는 Docker Image Registry 인 ECR 에 업로드
3. 마찬가지로 AWS 에서 제공하는 ECS 에서 ECR 에 있는 도커 이미지를 끌어와서 실행

<br>

위 두 가지 방식 중 어느게 낫다 할 수는 없지만 개인적으로 AWS 에서 제공하는 ECR 을 활용하는 게 좋아보였습니다.

하지만 위 가이드에서 안내하는 방법은 AWS ECS 에 배포하는 건데요.

AWS ECS 는 AWS 에서 제공하는 도커 컨테이너 서비스라고 생각하면 됩니다.

사용자의 설정에 따라 스케일 인/아웃을 하며 여러 인스턴스를 관리하는 클러스터 개념입니다.

<br>

여기서 문제가 되었던 부분이, 제가 하고 싶었던 건 EC2 라는 기존 인스턴스에 배포하는 일이었습니다.

따라서 위 방법을 조금 수정하여 최종적으로 아래와 같은 순서로 배포를 자동화하였습니다.

1. jar 파일을 Docker 이미지로 빌드
2. AWS ECR 에 도커 이미지 업로드
3. AWS CodeDeploy 를 사용하여 EC2 에서 ECR 에 올라온 이미지를 가져와서 실행

<br>

# ECR

## 1. IAM 에서 권한은 사용자 그룹 단위로 줄 수 있기 때문에 사용자 그룹에 사용자를 추가해야함

## 2. Dockerfile 추가해야함

```dockerfile
FROM openjdk:11-jre-slim

# 빌드 결과 디렉토리 지정
ARG JAR_FILE=build/libs/*.jar

# app.jar 파일로 복사
COPY ${JAR_FILE} app.jar

# jar 파일 실행
ENTRYPOINT ["java","-jar","/app.jar"]
```

<br>

Spring Boot 2.5 버전부터는 빌드 시 `-plain.jar` 파일이 만들어지기 때문에 `build.gradle` 수정 필요

```gradle
jar {
    enabled = false
}
```

