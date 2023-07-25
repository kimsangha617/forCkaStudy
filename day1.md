k8s 의 목적은 응용프로그램을 컨테이너 형식으로 호스트 하는것이다 자동화 방식으로
프로그램의 많은 인스턴스를 배포할수 있고 응용 프로그램 간의 통신도 쉽게 가능하게 할 수 있다

두 개의 배를 볼 수 있다
컨테이너를 싣고 바다를 항해하는 화물선과

화물선을 통제하는 관제선은 화물선을 감시하고 관리한다

쿠버네티스 클러스터는 노드 세트로 구성되는데  물리적, 가상, 온프레미스 또는 클라우드 일 수도 있고 컨테이너 형태의 응용프로그램 일 수 있다

클러스터의 작업자 노드는 컨테이너를 로딩할 수 있는 배 이다

누군가는 선박에 컨테이너를 적재해야 한다.

적재만 하는게 아니라 어떻게 적재할지 계획 하고 선박을 식별하고 그에 대한 정보를 저장하고 선박에 실린 컨테이너의 위치를 감시하고

추적하고 전적인 적재 과정을 관리하는 등등을 해야 한다

여러 사무실과 부서를 관리하는 관제선과 모니터링 장비 통신 장비 선박사이에 컨테이너를 옮기는 크레인 등으로 이뤄져있다

관제선(컨트롤쉽) 은 쿠버네티스 클러스터의 마스터 노드와 연결돼 있다.

마스터 노드는 클러스터를 관리하고 서로 다른 노드에 대한 정보를 저장하고 어떤 컨테이너가 어디로 갈지 계획하고 노드와 컨테이너를 모니터링하는 등등을 책임진다

매일 많은 컨테이너가 배에서 하역되고 있다. 그래서 각 선박에 대한 정보를 유지해야 한다. 어떤 컨테이너에가 어느 배에 있고

몇시에 적재됐는지 등등을.. 이 모든게 `ETCD` 라 불리는 고가용성 키-밸류 저장소에 저장 된다

`ETCD`는 키-밸류 형식으로 정보를 저장하는 데이터베이스 이다


---
배가 도착하면 크레인으로 컨테이너를 싣는다 

크레인이 선박에 실어야할 컨테이너를 식별한다

선박의 크기와 적재 용량 선박에 실린 컨테이너의 수와 기타 조건들을 기준으로 선박의 목적지와 실을 수 있는 컨테이너의 종류등을 결정한다

쿠버네티스 클러스터에 있는 스케쥴러들이다 (kube-shceduler)

kube-shceduler 는 컨테이너를 설치하기 위해 올바른 노드를 식별한다. 컨테이너 리소스 요구 사항이나 worker-node 용량 혹은 다른

정책이나 제약 조건들, 등등

부두에는 여러 사무실이 있고 각각 특별 업무나 부서로 배정된다.

예를 들어 운영팀은 선박 관리나 교통 관제 등을 담당한다 

파손 문제나 선박의 항로 등을 처리한다

컨테이너는 화물팀이 관리한다

컨테이너가 손상되거나 파손되면 새 컨테이너를 준비해둔다 

서비스 사무실이 있어서 IT와 함선간의 통신을 관리한다

쿠버네티스에도 컨트롤러가 있어서 다양한 영역을 관리할 수 있다

Node-Controller는 Node 를 관리한다

새 노드를 클러스터에 온보딩 하고 노드가 사용불가되거나 파괴되는 상황을 처리한다.

Replication-Controller는 원하는 컨테이너 수가 복제 그룹에서 항상 실행되도록 보장한다

이렇게 Controller-manager 로 구성된다

각각의 사무실들이 어떻게 연결되고 누가 높은수준에서 관리할까?

Kube API 서버는 쿠버네티스의 주요 관리 구성요소 이다

Kube API 서버는 클러스터 내에서 모든 작업을 오케스트레이션 한다

어플리케이션은 컨테이너 형태이다

마스터 노드에 전체관리 시스템을 형성하는 다양한 구성요소는 컨테이너 형태로 호스트 될 수 있다.

DNS 서비스 네트워킹 솔루션은 컨테이너 형태로 배포될 수 있다.

그래서 컨테이너를 실행할 이런 소프트웨어가 필요하다

그게 Container Runtime Engine 이다 

인기 있는건 Docker 이다

따라서 Docker가 필요하다, 클러스터 내 모든 노드에 설치되어 있다. 마스터 노드를 포함해서 말이다

컨트롤 구성 요소를 컨테이너로 호스트 하고 싶다면 말이다

다시 화물선으로 가보자

모든 배들엔 선장이 있다. 선장은 모든 활동을 관리할 책임이 있다. 선장은 주요 선박들과 연락을 주고 받는 일을 맡는다.

선박에 실을 컨테이너에 대한 정보를 서로 주고 받고 필요한 만큼 적재하고 해당 선박의 컨테이너 상태와 선박에 실린 컨테이너

상태 등을 주 선장으로부터 보고 했다.

이 배의 선장은 kubelet 이다 

kubelet은 클러스터의 각 노드에서 실행되는 에이전트 이다. Kube API 서버의 지시를 듣고 필요한대로 노드에서 컨테이너를 배포하거나

파괴한다.

Kube API 서버는 주기적으로 kubulet으로부터 상태 보고서를 가져옵니다 노드와 컨테이너의 상태를 모니터링하기 위함이다

kubelet은 선장에 가깝다. 컨테이너를 관리하는 선장이지만 작업자 노드에서 실행되는 응용 프로그램은 서로 통신할 수 있어야 했다

가령 한 노드의 한 컨테이너에서 웹 서버가 실행되고 다른 노드의 다른 컨테이너에서 데이터베이스 서버가 실행된다 보자

웹서버가 어떻게 다른 데이터베이스 서버에 도달할까 ? 

작업자 노드간의 통신은 또 다른 구성 요소로 가능하다. Worker-Node 에서 실행되는 것으로 kube-proxy 서비스이다.

<img width="1075" alt="image" src="https://github.com/kimsangha617/forCkaStudy/assets/66237694/c4e4b9fa-bc80-447d-a83a-ca0e2439f4db">


## 13. ETCD for beginners

목차
- what is ETCD?
- what is a Key-Value Store?
- How to get started quickly?
- How to operate ETCD?

Later
- what is a distributed system?
- How ETCD Operates
- RAFT Protocol
- Best practices on number of nodes


ETCD란 무엇인가
분산되고 신뢰할 수 있는 키-밸류 스토어로 안전하고 신속하다

`Install ETCD
1. Download Binaries
curl -L https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcd-v3.3.11-linux-amd64.tar.gz -o etcd-v3.3.11-linux-amd64.tar.gz
2. Extract
tar xzvf etcd-v3.3.11-linux-amd64.tar.gz
3. Run ETCD Service
./etcd


etcd 를 실행하면 port 2379 가 default인 서비스가 시작된다

./etcdctl set key1 value1

저장된 데이터를 검색하려면

./etcdctl get key1

더 많은 옵션을 보려면 인수 없이 etcdctl 명령을 실행하면 된다

./etcdctl

위에는 버전 2에서 쓰이는 명령어

etcd에는 다양한 버젼이 있는데 v2.0 과 v3.0에는 꽤나 큰 차이가 생겼다

etcd의 버젼 확인 명령어

./etcdctl --version

etcdctl을 API 버전 3로 작업하려면 ETCDCTL_API라는 환경 변수를 각 명령에 3으로 설정해야한다

```
ETCDCTL_API=3 ./etcdctl version
```

ETCD version2 에서 실행하는 버전 확인 명령어는

```
./etcdctl --version
```

ETCD version3 용

버전 3에 값을 설정할 명령어는
```
export ETCDCTL_API=3
./etcdctl version
```

```
./etcdctl put key1 value1
./etcdctl get key1
```


ETCD의 역할에 대해 알아보자

이 데이터 저장소는 클러스터에 관한 정보를 저장한다 예를들어

- Nodes
- PODs
- Configs
- Secrets
- Accounts
- Roles
- Bindings
- Others

kubectl을 실행할 때 얻게 되는 모든 정보는 ETCD Cluster 에서 온다

클러스터에 변화를 줄때마다,, 가령 Node 를 추가한다거나 POD를 배포한다거나 일련의 것들이 진행 되면 ETCD 서버에 업데이트가 된다

클러스터를 어떻게 설정하냐에 따라 ETCD 는 다양하게 배포가 된다

이 섹션에서 쿠버네티스 배포의 두가지 유형에 대해 다룰것이다

하나는 처음부터 배포하고 다른건 kubeadm tool 을 이용해서 이다

클러스터를 스크래치에서 셋업하는 경우 기타 바이너리들을 직접 다운로드해 배포한다

마스터 노드에 바이너리를 직접 설치하고 서비스로서 구성한다

Setup - Manual

```
wget -q --https-only \
  "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"

etcd를 cluster로 구성해보자

k8s에서 high-availability을 설정할 때 옵션을 보자

--advertise-client-urls https://${INTERNAL_IP}:2379 \\

유일한 옵션이다 ,, ETCD 가 listen 하는 기본 주소이다

이건 kube API 서버에서 구성돼야할 URL 이다


Setup - kubeadm

kubeadm 클러스터를 이용해 설정을 하면

kubeadm이 다양한 서버를 배포한다. kube 시스템 네임스페이스에 있는 포드로서도 말이다

```kubectl get pods -n kube-system```

kubernetes가 저장한 모든 키를 열거하려면 get 명령어를 실행해라

```
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```










