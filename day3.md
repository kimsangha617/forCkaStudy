## 28. ReplicaSets

이번 강의에서는 Replication 컨트롤러에 대해 얘기할 것이다

어떤 이유로든 앱이 다운되고 고장난다면
사용자들은 더이상 앱에 접속할 수 없게된다

사용자가 앱에 대한 액세스를 잃지 않도록하려면 한개 이상의 인스턴스나 Pod가 동시에 실행되어야 한다

Replication controller Kubernetes에 있는 단일 포드의 다중 인스턴스를 실행하도록 도와준다. High-Availability 를 제공한다


### Load Balancing & Scaling

Replication controller 필요한 또 다른 이유는

여러개의 포드를 만들어 로드를 공유하기 위해서이다

예를 들어 단일 pod가 사용자들에게 제공 할 때

사용자 수가 증가하면 pod를 하나 더 설치해 두 부분의 하중의 균형을 잡는다(Load-Balancing)

수요가 증가하고 첫 번쨰 노드에 리소스가 바닥나면 클러스터 내 다른 노드에 추가 
pod 를 배포할 수 있다

Replication controller는 클러스터내 여러 노드로 뻗어있다

서로 다른 노드의 여러 포드에 걸쳐 부하를 분산하는데 도움이 되고 수요가 증가하면 앱의 스케일도 조정할 수 있다.

Replication Controller

Replica Set

은 다르다

#### 복제 컨트롤러를 생성해보자

rc-definition.yml
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  - template:

```
쿠버의 정의 파일에서 스펙 섹션은 우리가 만드는 개체 안에 뭐가있는지 정의한다

이 경우 복제 컨트롤러는 포드에 여러 개의 인스턴스를 만든다

근데 어떤 포드이지..?

`spec:` 아래에 템플릿 섹션을 생성한다 Replcation Controller가

복제본을 만들기 위해 사용할 포드 템플릿을 제공하기 위해서 이다

그럼 포드 템플릿을 어떻게 정의할까??

이전에 우리가 만들어봤던 Pod 정의 파일의 모든 컨텐츠를 Replication Controller(이하 RC) 섹션으로 이동한다

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:        // Replication Controller
  - template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:    // POD
      containers:
      - name: nginx-container
        image: nginx
```

하나 빠진게 있다

복제컨트롤러에 복제본이 얼마나 필요한지 언급을 안했다

하나의 property 가 더 추가된다

필요한 복제본의 수를 입력하는 부분이다

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:        // Replication Controller
  - template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:    // POD
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
```

#### RC 생성

```
kubectl create -f rc.definition.yml
```