## 18. Kubelet

kubelet은 배의 선장과 같다

선내의 모든 활동을 지휘한다

Worker Nodes 들은 Scheduler 에게 선박과 컨테이너 상태를 일정 간격으로 보고한다

kubelet은 Pod의 상태와 컨테이너를 계속 모니터링하고 동시에 kube API server에 보고한다.

클러스터를 배포하기 위해 kubeadm을 사용한 경우, kubelet은 자동으로 배포되지 않는다.

작업자 노드에 반드시 수동으로 kubelet을 설치해야 한다.

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet
```

작업자 노드에서 실행 중인 프로세스 중 kubelet을 검색함으로써 실행 중인 kubelet 프로세스의 옵션을 확인할 수 있다.

```
ps -aux | grep kubelet
```



## 19. Kube Proxy

쿠버네티스에서 모든 pod는 다른 모든 pod에 end-to-end 통신이 가능하다.

이건 pod 네트워킹 솔루션을 클러스터에 배포함으로써 이뤄진다.

pod 네트워크는 내부 가상 네트워크로 모든 파드가 연결되는 클러스터 내 모든 노드에 걸쳐 있다.

그렇기 때문에 이 네트워크를 통해 모두 서로 통신 가능하다.

항상 상대방 Pod의 IP가 같을 것이라는 보장이 없다.

상대방 IP에 access를 하기위한 더 나은방법은 서비스를 이용하는 것이다.

우선 클러스터에 걸쳐 어떠한 앱을 노출할 서비스를 생성한다

서비스 IP 주소를 할당한다.

서비스 이름을 통해 상대 Pod에 접근할 수 있다.

서비스는 파드 네트워크에 포함되어 있지 않다.

서비스는 실제 것이 아니기 때문이다.

서비스는 Pod 같은 컨테이너가 아니라서 인터페이스도 없고 쿠버네티스 메모리에만 존재하는 가상 구성 요소이다.

그럼에도 이러한 서비스가 클러스터를 가로질러 어떤 노드에서도 접근 가능한 이유는 Kube-Proxy 덕분이다.

Kube-Proxy는 쿠버네티스 클러스터의 각 노드에서 실행되는 프로세스이다.

Kube-Proxy는 새 서비스가 생성될 때마다 각 노드에 적절한 규칙을 만들어 그 서비스로 트래픽을 전달한다. iptable 규칙을 이용하기도 한다.

쿠버네티스 배포 페이지에서 Kube-Proxy 바이너리를 다운로드해서 서비스로 실행하면 Kube-Proxy를 설치할 수 있다.

command line
```
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy
```
## View kube-proxy - kubeadm

kubeadm 툴은 kube-proxy 를 배포한다 각각의 노드의 pod로
```
kubectl get pods -n kube-system
```
DaemonSet도 배포한다.

단일 pod는 각각의 클러스터의 노드에 항상 배포된다

```
kubectl get daemonset -n kube-system
```


## 20. Recap - PODs

Pod 를 이해하기 이전에 deploying이 setup 되어있다고 가정해보자

응용 프로그램이 이미 개발돼 Docker 이미지에 내장되었으며 Docker Hub와 같은 Docker Repository에서 사용 가능하다고 가정하자.

또한 쿠버네티스 클러스터도 이미 설정돼 작동중이라고 가정하자 단일 노드든 다중노드는 노상관!

최종 목표는 컨테이너 형태의 응용 프로그램을 작업자 노드로 구성된 컴퓨터 세트에 배포하는 것이다.

컨테이너는 Pod 형태로 포장된다. Pod는 응용 프로그램의 단일 인스턴스이다.

Pod는 쿠버네티스에서 만들 수 있는 가장 작은 물체이다.

Pod는 보통 앱을 실행하는 컨테이너와 1:1 관계를 갖는다.

따라서, 규모를 키우려면 새 파드를 만들어야 하고 규모를 줄이기 위해서는 기존 파드를 삭제해야 한다.

응용 프로그램 규모를 키우기 위해 기존 파드에 추가 컨테이너를 추가하지 않는다.

만약 현재 노드가 충분한 용량을 갖추지 못한다면 클러스터의 새 노드에 추가 파드를 배포한다.

그렇다면 하나의 파드에 하나의 컨테이너만 들어갈 수 있을까?

하나의 파드에 여러 개의 컨테이너가 들어갈 수 있다.

하지만 보통은 같은 종류의 컨테이너가 여러 개 있지 않다.

두 컨테이너는 같은 네트워크를 공유하기 때문에 로컬호스트라는 이름으로 서로 통신할 수 있다.

게다가 같은 저장소 공간을 쉽게 공유할 수도 있다.

Pod로 구성된 컨테이너만 정의하면 Pod에 있는 컨테이너는 기본값으로 같은 저장소와 같은 네트워크 네임스페이스에 접근할 수 있다. Pod의 컨테이너들은 함께 생성되고 함께 파괴된다.

컨테이너가 하나만 필요하더라도 쿠버네티스는 파드를 만들어야 한다.

다중 파드 컨테이너는 드문 사례이기 때문에 강의에서는 파드당 하나의 컨테이너를 사용할 것이다.

kubectl 명령어를 통해 파드를 배포할 수 있다.

파드를 생성해서 컨테이너를 배포한다.

이미지 매개변수를 통해 이미지의 이름을 얻고 Docker Hub 레포지토리에서 이미지를 다운로드 받는다.

### kubectl
```
kubectl run nginx --image nginx
```

## 21. PODs with YAML

쿠버네티스는 YAML 파일을 Pod, ReplicaSet, Deployment, Service 등 개체 생성을 위한 입력 양식으로 사용한다.

```
kubectl api-resources -o wide
```

명령어를 통해 kubernetes resources list를 확인할 수 있다.

쿠버네티스 정의 파일은 항상 4개의 상위 레벨 필드(apiVersion, kind, metadata, spec)를 포함한다.

이것들은 루트 레벨의 속성들이다.


### YAML in Kubernetes

pod-definition.yaml

```
appVersion:
kind:
metadata:
```
|kind|version|
|---|---|
|POD|v1|
|Service||v1|
|ReplicaSet|apps/v1|
|Deployment|apps/v1|


- apiVersion: 쿠버네티스 api 버전이다. 만들려는 것이 무엇이냐에 따라 올바른 API 버전을 사용해야 한다.
- kind: 만드려는 개체 유형을 나타낸다.
- metadata: 이름이나 라벨처럼 개체 위의 데이터이다. 라벨은 키와 값 쌍을 가질 수 있다.
- spec: 사양을 나타낸다. 생성하려는 개체에 따라 쿠버네티스에게 개체와 관련된 추가 정보를 제공한다.



```
appVersion:v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-cluster
  image: nginx
```

```
kubectl create -f pod-definition.yml
```

### Commands
```
kubectl get pods
```

```
kubectl describe pod myapp-pod
```

### multicontainer pod의 경우 다음과 같이 명시한다.

```
appVersion:v1
kind: Pod
metadata:
  name: multipod
spec:
  containers:
  - name: nginx-cluster
    image: nginx:1.14
    ports:
    - containerPort: 80
  - name: centos-cluster
    image: centos:7
    command:
    - sleep
    - "10000"
```


## 22. Demo - PODs with YAML

pod를 생성하는 실습 과정이다

주의사항!
- 대소문자를 구분해야한다
- 하위요소들의 인덴트는 2칸이다(space)

터미널 open - yaml 생성할 폴더로 이동

```
vim pod.yaml
```

```
appVersion:v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    type: frontend
spec:
  containers:
  - name: nginx
  image: nginx
```

```
kubectl apply -f pod.yaml
```

```
kubectl get pods
```
