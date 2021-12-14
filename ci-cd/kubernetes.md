# kubectl 명령어

# 1. gcloud (구글 클라우드)

클러스터는 Node 의 묶음입니다.

클러스터를 전환하면 `kubectl get nodes` 명령어의 정보도 달라집니다.

```sh
# 구글 클라우드 클러스터 리스트 조회
$ gcloud container clusters list

# 구글 클라우드의 클러스터를 kubeconfig 에 등록
$ gcloud container clusters get-credentials <클러스터 이름>

# 클러스터 생성 (노드 3개와 함께)
$ gcloud container clusters create <클러스터 이름> --num-nodes 3

# 구글 클라우드 클러스터 삭제
$ gcloud container clusters delete <클러스터 이름>
```

<br>

# 2. 노드 (Node)

```sh
$ kubectl get nodes
```

<br>

# 3. 파드 (Pod)

```sh
# 파드 조회 (-o wide 붙이면 좀더 자세한 정보 나옴)
$ kubectl get pods

# 파드 삭제
$ kubectl delete pod kubia

# 파드 상세한 설명 보기
$ kubectl describe pod <파드 이름>
```

<br>

# 4. 서비스 (Service)

```sh
# 서비스 생성
$ kubectl expose deployment kubia --type=LoadBalancer --port=8080 --name=kubia-http
```

<br>

# 4. kubectl config

`kubectl` 명령어에 클러스터 정보, 인증 정보를 세팅해두고 편하게 사용할 수 있습니다.

<br>

## 4.1. Credential

```sh
# 인증 정보 조회
$ kubectl config get-users

# 인증 정보 생성
$ kubectl config set-credentials <이름> --옵션들

# 인증 정보 삭제
$ kubectl config unset users.<이름>
```

<br>

## 4.2. Cluster

```sh
# 클러스터 리스트 조회
$ kubectl config get-clusters

# 클러스터 생성
$ kubectl config set-cluster <클러스터 이름> --server <Host:Port>

# 클러스터 삭제
$ kubectl config unset clusters.<클러스터 이름>
```

<br>

## 4.3. Context

컨텍스트는 클러스터 정보와 인증 정보를 매핑해서 하나로 관리할 수 있게 해줍니다.

특정 클러스터를 사용하기 위해선 사용자 정보도 필요합니다.

만약 A 유저 정보를 사용하는 A 클러스터와 B 유저 정보를 사용하는 B 클러스터가 존재한다고 생각해 봅시다.

각 클러스터를 스위칭하기 위해선 유저 정보를 또 스위칭하는 과정을 매번 해야 합니다.

Context 는 클러스터와 유저 정보를 하나로 묶어서 컨텍스트를 스위칭 하는 것만으로도 클러스터 접근을 쉽게 할 수 있습니다.

```sh
# 컨텍스트 리스트 조회
$ kubectl config get-contexts

# 현재 컨텍스트 조회
$ kubectl config current-context

# 새로운 컨텍스트 생성
$ kubectl config set-context <컨텍스트 이름>

# 특정 컨텍스트 사용 (전환)
$ kubectl config use-context <컨텍스트 이름>

# 특정 컨텍스트 삭제
$ kubectl config unset contexts.<컨텍스트 이름>
```

<br>

## 4.4. 컨텍스트에 인증 정보와 클러스터를 매핑

```sh
# 1. username, password 로 인증 정보 생성
$ kubectl config set-credentials <Credential 이름> --username=<아이디> --password=<비밀번호>

# 2. token 으로 인증 정보 생성
$ kubectl config set-credentials <인증 정보 이름> --token=<토큰>

# 클러스터 정보 생성
$ kubectl config set-cluster <클러스터 이름> --server <Host:Port>

# 클러스터 정보와 인증 정보로 새로운 컨텍스트 생성
$ kubectl config set-context <Context 이름> --cluster=<Cluster 이름>  --user=<Credential 이름>

# 컨텍스트 사용
$ kubectl config use-context account-webapp-dev-context
```
