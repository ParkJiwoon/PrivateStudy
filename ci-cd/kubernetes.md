# kubectl 명령어

# 1. gcloud (구글 클라우드) Cluster

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

# 2. kubectl config

`kubectl` 명령어에 클러스터 정보, 인증 정보를 세팅해두고 편하게 사용할 수 있습니다.

<br>

## 2.1. Credential

```sh
# 인증 정보 조회
$ kubectl config get-users

# 인증 정보 생성
$ kubectl config set-credentials <이름> --옵션들

# 인증 정보 삭제
$ kubectl config unset users.<이름>
```

<br>

## 2.2. Cluster

```sh
# 클러스터 리스트 조회
$ kubectl config get-clusters

# 클러스터 생성
$ kubectl config set-cluster <클러스터 이름> --server <Host:Port>

# 클러스터 삭제
$ kubectl config unset clusters.<클러스터 이름>
```

<br>

## 2.3. Context

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

## 2.4. 컨텍스트에 인증 정보와 클러스터를 매핑

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

<br>

# 3. 노드 (Node)

노드는 여러 개의 파드를 갖는 단위입니다.

```sh
$ kubectl get nodes
```

<br>

# 4. 파드 (Pod)

파드는 쿠버네티스의 가장 기본적인 배포 단위입니다.

파드는 하나 이상의 컨테이너를 포함하기 때문에 각 컨테이너를 각각 배포하지 않고 한번에 배포 가능합니다.

파드는 고유의 IP 와 Port 를 가지며, 파드 내의 컨테이너들은 `localhost:<각 컨테이너 포트>` 형식으로 서로 통신 가능합니다.

```sh
# 파드 조회 (-o wide 붙이면 좀더 자세한 정보 나옴) (pods = po)
$ kubectl get pods

# 좀더 자세히 조회
$ kubectl get po -o wide

# 파드 삭제
$ kubectl delete pod kubia

# 파드 상세한 설명 보기
$ kubectl describe pod <파드 이름>

# 로그 보기
$ kubectl logs <파드 이름>

# 여러 컨테이너를 포함한 파드인 경우에 컨테이너 이름 지정
$ kubectl logs <파드 이름> -c  <컨테이너 이름>

# 이전 컨테이너의 로그 확인
$ kubectl logs <파드 이름> --previous
```

<br>

## 4.1. 새로운 파드 생성 (kubectl create vs apply)

- create: 새로운 Object 생성. 이미 존재하면 에러
- apply: 새로운 Obejct 생성. 이미 존재하면 변경된 부분만 반영

https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create

<br>

# 5. 서비스 (Service)

서비스는 외부에서 파드에 접근할 수 있게 해주는 로드 밸런서 역할을 합니다.

파드는 어떠한 이유에 의해 새로 생성되거나 지워질 수 있는데 이럴 때마다 파드의 IP 가 바뀌기 때문에 외부 클라이언트가 파드의 정확한 IP 를 추적하기 어렵습니다.

서비스는 여러 파드를 갖고 있으며 외부에서는 각 파드의 IP, Port 대신 서비스의 IP, Port 에 접근하면 서비스가 알아서 파드로 연결해줍니다.

```sh
# 서비스 생성
$ kubectl expose deployment kubia --type=LoadBalancer --port=8080 --name=kubia-http
```

<br>

# 6. 레이블 (Label)

이름 그대로 라벨링의 역할을 합니다. (발음은 책마다 다른 것 같네요)

각 파드에 `Key:Value` 레이블을 설정해서 나중에 "레이블 셀렉터" 라는 걸로 특정 파드들을 동시에 제어할 수 있습니다.

주로 서비스가 목적에 따라 구분된 파드들을 동시에 실행하거나, 삭제하거나, 조회할 때 사용합니다.

```sh
# 레이블까지 조회
$ kubectl get po --show-labels

# 레이블 추가 (--overwrite 옵션을 추가하면 기존 레이블 수정)
$ kubectl label po <파드 이름> <레이블>=<설정>

# 특정 레이블들까지 같이 조회
$ kubectl get po -L creation_method=manual,env

# creation_method=manual 에 해당하는 파드 조회
$ kubectl get po -l creation_method=manual

# 값에 상관 없이 env 레이블을 갖고 있는 파드 조회
$ kubectl get po -l 'env'

# 위와 반대로 env 레이블을 갖고 있지 않은 파드 조회
$ kubectl get po -l '!env'
```

<br>

노드에도 파드처럼 레이블 세팅이 가능합니다.

```sh
# gpu=true 의 레이블을 노드에 세팅
$ kubectl label node <노드이름> gpu=true
```

<br>

## 6.1. 레이블 셀렉터 (Label Selector)

등록된 레이블 기반으로 특정 파드들을 그룹화 할 때 사용합니다.

쿼리처럼 사용 가능합니다.

- `creation_method!=manual`: creation_method 레이블 갖고 있는 파드 중에 값이 manual 이 아닌 것
- `env in (prod,dev)`: env 레이블 값이 prod 또는 dev 로 되어 있는 파드
- `env notin (prod,dev)`: env 레이블 값이 prod, dev 가 아닌 파드

<br>

# 7. 어노테이션 (Annotation)

파드에 붙이는 주석 같은 개념입니다.

PR 번호, 작성자 등등.. 레이블처럼 키 값으로 사용할 수는 없지만 Description 역할을 합니다.

```sh
# 특정 파드에 어노테이션 추가
$ kubectl annotate pod <파드이름> mycompany.com/someannotation="foo bar"
```

<br>

# 8. 네임스페이스 (namespace)

클러스터를 논리적으로 구분하는 가상 클러스터라고 생각하면 됩니다.

Pod, Service 등을 네임스페이스 별로 생성하거나 관리할 수 있고 사용자의 권한 역시 별도로 부여할 수 있습니다.

처음 공부할 때는 레이블이랑 뭐가 다른건지 헷갈렸었는데..

레이블은 말 그대로 파드에 붙은 정보를 의미하는 거고 네임스페이스는 다음과 같은 일을 할 수 있습니다.

- 특정 네임스페이스에 별도의 권한을 두어 파드, 서비스 등 접근 분리
- dev, sandbox, prod 처럼 애플리케이션은 동일하지만 환경이 다른 경우 각 환경 별로 리소스를 다르게 할당 가능 (prod 환경은 좀더 많이 할당)

위에서 한번 언급했지만 네임스페이스는 논리적으로 구분하는 것이지 물리적인 구분은 아니라서 각 파드끼리 통신이 가능합니다.

```sh
# 클러스터에 있는 모든 네임스페이스를 나열
$ kubectl get ns

# 특정 네임스페이스의 파드 조회 (-n 으로 축약 가능)
$ kubectl get po --namespace <네임스페이스>

# 네임스페이스 생성 (yaml 파일로 만드는걸 추천)
$ kubectl create namespace <네임스페이스>

# 특정 네임스페이스에 파드 생성
$ kubectl apply -f kubia-manual.yaml -n <네임스페이스>
```
