## 62. Node Affinity

Node에 Label을 적용하는 명령어

`kubectl label node node01 color=blue`

Node에 Taints가 있는지 확인하는 명령어

`kubectl describe node node01 | grep Taints`

vi 환경에서 한번에 여러 줄을 shift하기 위해서 'V + 여러줄 마우스로 드래그'하면 된다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/66237694/229594223-c12adab6-315a-490a-867e-3e2ea2d0ab2e.png">


- How many Labels exist on node node01?

`kubectl describe node node01`

- Create a new deployment named blue with the nginx image and 3 replicas.
`kubectl create deployment blue --image=nginx --replicas=3`

- Which nodes can the pods for the blue deployment be placed on?
 - Make sure to check taints on both nodes!

```
Check if controlplane and node01 have any taints on them that will prevent the pods to be scheduled on them. If there are no taints, the pods can be scheduled on either node.

So run the following command to check the taints on both nodes.
```

`kubectl describe node controlplane | grep -i taints`
`kubectl describe node node01 | grep -i taints`

- Set Node Affinity to the deployment to place the pods on node01 only.
```
- Name: blue
- Replicas: 3
- Image: nginx
- NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
- Key: color
- value: blue
```

`kubectl edit deployment blue`

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2023-04-03T18:27:53Z"
  generation: 1
  labels:
    app: blue
  name: blue
  namespace: default
  resourceVersion: "2302"
  uid: f7a5654c-c64d-4d3c-bf7f-97e8cc812fd3
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: blue
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blue
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 3
  conditions:
  - lastTransitionTime: "2023-04-03T18:28:03Z"
    lastUpdateTime: "2023-04-03T18:28:03Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2023-04-03T18:27:53Z"
    lastUpdateTime: "2023-04-03T18:28:03Z"
    message: ReplicaSet "blue-7698dfcb9f" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 3
  replicas: 3
  updatedReplicas: 3
                                                                       66,7          Bot
37,7          Top
```                           

- Which nodes are the pods placed on now?

`kubectl get pods -o wide` and see the Node column.

- Create a new deployment named red with the nginx image and 2 replicas, and ensure it gets placed on the controlplane node only. Use the label key - node-role.kubernetes.io/control-plane - which is already set on the controlplane node.


- Name: red
- Replicas: 2
- Image: nginx
- NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
- Key: node-role.kubernetes.io/control-plane
- Use the right operator


Create the file reddeploy.yaml file as follows:

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists
```

Then run `kubectl create -f reddeploy.yaml`

## 65. Taints and Toleration vs Node Affinity

빨강, 파랑, 초록, 기타 노드들이 있고 이에 맞춰 빨강, 파랑, 초록, 기타 파드들이 존재한다.
각 노드에 동일한 색의 파드들이 배치되어야 한다.(빨강 노드 - 빨강 파드, 파란 노드 - 파란 파드, 초록 노드 - 초록 파드)

<img width="558" alt="image" src="https://user-images.githubusercontent.com/66237694/229604820-afe41367-793b-4cf8-b0b6-b0ed394769bc.png">

이를 Taints와 Toleration으로 해결하기 위해서는 Node에 Taint를, Pod에 Toleration을 정의하면 된다.
그러나 Taints가 없는 기타 파드들이 존재하기 때문에 Pod들이 동일한 색을 가진 Node에만 배포된다는 보장이 없다.

<img width="601" alt="image" src="https://user-images.githubusercontent.com/66237694/229605081-6e8a9b49-1636-401f-bb43-ca32c5d08552.png">

이번에는 Node Affinity를 통해 동일한 문제를 해결해보자.
Node Affinity를 이용하기 위해 먼저 각각의 노드들에 라벨을 붙인 다음 파드의 nodeAffinity를 설정해서 원하는 라벨의 노드를 선택하도록 한다.
하지만 이 방법도 기타 파드가 색상 라벨이 지정되어있는 노드에 배치될 가능성이 있다.

<img width="629" alt="image" src="https://user-images.githubusercontent.com/66237694/229605241-d3c6d16c-efc9-400a-b8eb-a9d9f638f47a.png">

Taints/Toleration과 Node Affinity를 함께 사용하면 특정 파드에 노드를 완전히 고정할 수 있다.
처음에는 Taints와 Toleration을 사용해 다른 파드가 우리 노드에 놓이는 것을 막는다.
그리고 Node Affinity를 이용해 파드가 각 라벨을 가진 노드에 배치되도록 한다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/66237694/229605409-86365029-da81-4dac-b872-5bca188b9d39.png">

## 66. Resource Requirements and Limits

다음은 3개의 노드로 이루어진 쿠버네티스 클러스터이다.
각 노드엔 CPU, 메모리, 디스크가 사용 가능한 리소스만큼 할당되어 있다.

<img width="768" alt="image" src="https://user-images.githubusercontent.com/66237694/229605595-8f2cd11e-3db4-4b23-bd18-19597d2d7d04.png">

파드가 노드에 놓일 때마다 그 노드에 사용 가능한 리소스를 소비한다.
쿠버네티스 스케줄러는 파드가 어느 노드로 갈지 결정할 때 파드가 요구하는 리소스의 양과 노드에서 사용 가능한 리소스의 양을 고려한다.
노드에 충분한 리소스가 없으면 스케줄러는 해당 노드에 파드를 놓는 것을 피한다.
만약 사용 가능한 노드가 없으면 아래 사진과 같이 파드의 배포를 보류한다.

<img width="730" alt="image" src="https://user-images.githubusercontent.com/66237694/229605768-7d7dfffb-afac-47e1-883a-ece8b39ee048.png">

파드를 정의할 때 resources 영역을 추가하고 필요한 리소스에 대해 요청할 수 있다.

<img width="730" alt="image" src="https://user-images.githubusercontent.com/66237694/229605898-2252c96a-c009-45b2-91f6-b5f1eb8c8d60.png">

위의 파드 정의 파일의 resources 영역에 작성한 1개의 cpu는 아래 사진과 같은 하나의 CPU를 말한다.

<img width="522" alt="image" src="https://user-images.githubusercontent.com/66237694/229606060-98116a13-b691-48e4-9e13-930130053bee.png">

메모리는 다음과 같은 단위를 사용해서 표현할 수 있다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/66237694/229606242-71c4372f-b94b-492b-acc4-66c27986c446.png">

도커 세계에서 도커 컨테이너는 노드에서 소비할 수 있는 리소스에 한계가 없다.
그래서 하나의 컨테이너에서 다른 컨테이너에 영향을 줄만큼 리소스를 소비할 수 있다.

하지만 파드의 리소스 사용량에는 제한을 둘 수 있다.
만약 특별히 파드의 리소스 사용량을 제한하지 않으면 하나의 파드는 1 vCPU, 512 Mi의 기본 한도를 갖게 된다.
만약 기본 한도를 바꾸고 싶다면 파드 정의 파일의 resources 영역에 한도(limits)를 지정하면 된다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/66237694/229606529-76a06f64-85a3-40a1-a691-48b8ad7109b8.png">

파드에 있는 컨테이너마다 requests와 limits이 있다는 것을 명시해야 한다.

파드가 지정된 한도를 초과하여 자원을 소비하게 되면 어떻게 될까?
CPU의 경우, 쿠버네티스가 CPU를 조절하여 지정 한도를 넘지 않도록 한다. 따라서, 컨테이너는
한도를 초과하는 CPU 리소스를 사용할 수 없다.
그러나 메모리의 경우, 파드가 자신의 한도보다 더 많은 메모리를 소모하려고 하면 그 파드는 즉시 종료된다.

## 67. Note on default resource requirements and limits

네임스페이스를 생성해서 다른 리소스들로부터 분리한 다음 LimitRange를 정의하는 manifest file을 작성해서 default limits과 default requests를 정의할 수 있다.

- 메모리의 default limits과 default requests를 정의하는 LimitRange manifest file

<img width="331" alt="image" src="https://user-images.githubusercontent.com/66237694/229606764-3341d86a-b1e8-4c89-a54e-1a3b2e0eecd6.png">

- CPU의 default limits과 default requests를 정의하는 LimitRange manifest file

<img width="336" alt="image" src="https://user-images.githubusercontent.com/66237694/229606873-72b5bd97-ba15-4524-bda8-a6aabd450ac2.png">

## 67. A quick note on editing PODs and Deployments

아래 이외의 사항들은 edit pod가 불가능하다.

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations

따라서, 작동 중인 pod의 환경 변수, 리소스 제한 등을 수정하고 싶으면 kubectl edit 명령어를 실행해서 편집창을 열고 수정한 다음, 파일의 모든 내용을 복사해서 임시 파일에 붙여 넣고 기존의 pod를 삭제하고 임시 파일의 내용을 파드를 재생성하면 된다.

또는 `kubectl get pod webapp -o yaml > my-new-pod.yaml` 명령어로 파드 정의 파일을 추출하고 변경사항을 수정한 다음, 기존의 파드를 삭제하고 새로운 파드 정의 파일로 파드를 재생성하면 된다.


파드와 달리, Deployments는 정의 파일의 파드 템플릿 영역에서 아무 필드나 수정 가능하다.
따라서, `kubectl edit deployment my-deployment` 명령어를 사용하면 된다.
해당 명령어를 실행하면 자동으로 기존의 파드가 삭제되고 새로운 파드가 재생성된다.


## 70. Solution: Resource Limits

파드의 현재 상황에 대한 원인을 알기 위해서는 `kubectl describe` 명령어를 통해 pod의 마지막 상태와 그에 대한 이유를 확인하면 된다.
OOMKilled는 메모리에 문제가 있다는 의미이다.

<img width="258" alt="image" src="https://user-images.githubusercontent.com/66237694/229607636-9ab3a2da-aa29-41ec-9afa-9521d555abf1.png">

`kubectl replace --force` 명령어를 사용하면 삭제와 재생성을 한번에 해준다.

<img width="600" alt="image" src="https://user-images.githubusercontent.com/66237694/229607766-a4f962f4-62ce-48a0-825f-683a00e5a753.png">

## 71. DaemonSets

데몬셋은 ReplicaSet 같은 것이다.
여러 개의 인스턴스 파드를 배포하도록 도와준다.
하지만 ReplicaSet과 달리, 클러스터의 노드마다 파드를 하나씩 실행한다.
클러스터에 새 노드가 추가될 때마다 파드 복제본이 자동으로 해당 노드에 추가된다.
노드가 제거되면 파드는 자동으로 제거된다.

즉, 데몬셋은 파드의 복사본을 클러스터 내 모든 노드에 항상 존재하게 한다.

<img width="765" alt="image" src="https://user-images.githubusercontent.com/66237694/229608083-e8649cc4-11a7-4ab3-936e-b997dd303969.png">

데몬셋은 Monoitoring Solution이나 Logs Viewer를 파드의 형태로 배포하기에 최적이다.
데몬셋이 알아서 동작하기 때문에 클러스터에 변화가 있을 때 노드에서 해당 파드를 추가하거나 제거할 필요가 없다.

<img width="761" alt="image" src="https://user-images.githubusercontent.com/66237694/229608266-34d985f2-4c49-46c1-a313-fb99a81aba56.png">

클러스터 내 모든 노드에 필요한 kube-proxy와 wave-net과 같은 네트워킹 솔루션은 데몬셋의 좋은 예시이다.

데몬셋 정의 파일을 통해 데몬셋을 생성할 수 있다.
데몬셋 정의 파일은 레플리카셋 정의 파일과 몹시 흡사하다.

<img width="764" alt="image" src="https://user-images.githubusercontent.com/66237694/229608403-efdb5d3c-ecad-4ce4-b311-9fb008a22b15.png">

데몬셋을 조회하기 위해서는 다음 명령어를 사용하면 된다.

<img width="760" alt="image" src="https://user-images.githubusercontent.com/66237694/229608527-90e34e46-837e-47f6-95c1-df05e51568bb.png">

쿠버네티스 버전 1.2 이전에는 nodeName을 지정하는 방식으로 데몬셋이 동작했다.
쿠버네티스 버전 1.2 부터는 nodeAffinity와 default scheduler를 사용하여 데몬셋을 각 노드에 스케줄링한다.

<img width="763" alt="image" src="https://user-images.githubusercontent.com/66237694/229608626-ee972556-50e0-4baa-be8c-1acdffb4365d.png">

## 73. Solution - DaemonSets

DaemonSet 정의 파일을 만들기 위해서 Deployment 정의 파일 템플릿을 사용할 수 있다.
Deployment 정의 파일을 생성한 다음, kind 필드를 DaemonSet을 변경하고 replicas 등 불필요한 필드를 제거하면 된다.

<img width="791" alt="image" src="https://user-images.githubusercontent.com/66237694/229608852-7e181580-c9c1-41ed-b971-1edfe6ab0e68.png">

## 74. Static Pods

강의 초반에 이야기했듯이 kubelet은 kubeAPI 서버에 의존해 노드에 파드를 배포한다.
이는 ETCD 클러스터에 저장된 kube-scheduler의 결정에 기초한다.

<img width="766" alt="image" src="https://user-images.githubusercontent.com/66237694/229609100-f1271fd8-7b23-40d1-946b-e890c91d6461.png">

API 서버나 쿠버네티스 클러스터의 구성 요소의 개입 없이 kubelet은 미리 저장해둔 파드 정의 파일을 읽어서 스스로 Static Pod를 생성할 수 있다.
kubelet은 주기적으로 디렉터리를 확인하고 파드 정의 파일을 읽고 호스트에 파드를 만든다. 파드를 만들 뿐 아니라 파드의 생존을 보장한다. kubelet은 Static pod가 다운되면 Static pod을 재시작하고 파드 정의 파일이 수정되면 파드를 재창조하며 파드 정의 파일이 삭제되면 파드는 자동으로 삭제된다.
다른 쿠버네티스 리소스들은 kubelet을 포함한 다른 구성 요소의 도움이 있어야 생성할 수 있다. 따라서, kubelet은 레플리카셋, 디플로이먼트, 서비스 등은 만들 수 없고 Static Pod만 만들 수 있다.

<img width="771" alt="image" src="https://user-images.githubusercontent.com/66237694/229609462-e311e3c6-17cd-4f4d-ac7f-3ff49cce261f.png">

Static Pod를 만들기 위한 지정된 디렉터리의 위치는 kubelet.service의 옵션으로 전달된다.

<img width="765" alt="image" src="https://user-images.githubusercontent.com/66237694/229609579-51dac430-e480-49de-962f-a8fc13cde01b.png">

또다른 방법은 kubelet.service의 옵션으로 직접 디렉터리 위치를 지정하는 대신 디렉터리 위치를 저장한 구성 파일을 만들고 구성 파일을 전달하는 것이다.
kubeadm 툴은 후자의 방법을 사용한다.

<img width="764" alt="image" src="https://user-images.githubusercontent.com/66237694/229609707-edcc6780-2d38-418c-abd4-fe629ea62063.png">

Static Pod가 생성되면 `docker ps` 명령어를 통해 Static Pod를 조회할 수 있다.
`kubectl` 커맨드를 사용할 수 없는 이유는 나머지 쿠버네티스 클러스터를 찾을 수 없기 때문이다.
`kubectl` 커맨드 유틸리티는 kube API 서버와 작동한다.

<img width="766" alt="image" src="https://user-images.githubusercontent.com/66237694/229609818-c40774d8-f269-46f2-84df-c213e46ccea1.png">

kubelet은 Static Pod 정의 파일로부터 파드를 생성할 수 있으며 kube-api server로부터 입력값을 제공받아 파드를 생성할 수 있다.
kube-api server는 kubelet이 만든 static pod를 인식한다.
그래서 kubectl 명령어를 통해 파드를 조회하면 다른 파드들과 동일하게 static pod가 조회된다.
이것이 가능한 이유는 kubelet이 static pod를 만들 때 쿠버네티스 클러스터에 포함되어 있다면 kube-api server에 미러 개체를 만들기 때문이다.
kube-api sever에서 보이는 미러 개체는 읽기 전용이다. 파드에 대한 세부사항은 볼 수 있지만 다른 파드처럼 편집하거나 삭제할 수 없다.

<img width="768" alt="image" src="https://user-images.githubusercontent.com/66237694/229610121-e7b412d3-bff3-4008-baac-0934e3cc7490.png">

그렇다면 Static Pod를 사용하는 이유는 무엇일까?
Static Pod는 쿠버네티스 control plane에 의존하지 않기 때문에 Static Pod를 이용해서 control plane에 구성요소를 배포할 수 있다.
이렇게 하면 서비스를 구성하는 바이너리를 다운로드하거나 서비스 충돌을 걱정할 필요가 없다. 만약 어떤 서비스가 고장나면 kubelet이 자동적으로 재가동해준다.
이 방법이 kubeadm 툴이 쿠버네티스 클러스터를 설정하는 방법이다.
그래서 kube-system 네임스페이스에 파드를 조회할 때 control plane의 구성요소를 확인할 수 있는 것이다.

<img width="752" alt="image" src="https://user-images.githubusercontent.com/66237694/229610486-65b8d52f-9ea7-40ed-8d8a-7dce093e93ae.png">

데몬셋은 응용 프로그램의 인스턴스 하나가 클러스터 내 모든 노드에서 사용 가능하도록 하는 데 사용된다. 데몬셋은 kube-api server를 통해 설정된다.

반면에 Static Pod는 kube-api server나 control plane의 방해 없이 kubelet이 직접 생성한다. 또한 Static Pod은 control plane의 구성요소 자체를 배포하는데 사용될 수 있다.

kube-scheduler는 데몬셋과 Static Pod에게 아무런 영향을 주지 않는다.

<img width="747" alt="image" src="https://user-images.githubusercontent.com/66237694/229610568-a45831e7-5fd1-48d8-b573-2f34e20ddf77.png">

## 75. Solution - Static Pods

Static Pod를 구분하는 방법은 파드의 이름을 보는 것이다. 파드의 이름 끝에 노드 이름이 들어 있으면 Static Pod이다.
또 다른 방법은 `kubectl get 파드명 -o yaml` 명령어를 통해 파드의 ownerRefrences 영역의 kind와 name 필드를 보는 것이다. 소유주가 노드이면 Static Pod이다.

<img width="365" alt="image" src="https://user-images.githubusercontent.com/66237694/229610906-cf29f3c3-4774-4fc8-a9d0-49e681240c4a.png">

Static Pod 정의 파일을 찾기 위해서는 다음 경로를 확인하면 된다.

`cat var/lib/kubelet/config.yaml`
`staticPodPath: /etc/kubernetes/manifests`
`ls /etc/kubernetes/manifests`
`etcd.yaml kube-apiserver.yaml kube-controller-manager.yaml kube-scheduler.yaml`

Static Pod를 생성하기 위해서는 Static Pod 정의 파일을 생성하고 디렉터리에 Static Pod 정의 파일을 옮기면 된다.

<img width="765" alt="image" src="https://user-images.githubusercontent.com/66237694/229611333-aa4fad00-b5cb-4b46-9f04-d7d0162e5949.png">

<img width="692" alt="image" src="https://user-images.githubusercontent.com/66237694/229611419-e5b87352-e7c7-46ca-ad88-ae16b357350c.png">

Static Pod를 수정하기 위해서는 Static Pod 정의 파일의 값을 수정하면 된다.

<img width="744" alt="image" src="https://user-images.githubusercontent.com/66237694/229611564-dcf4c7ea-b38b-427a-9410-242a89d9a5b3.png">

<img width="755" alt="image" src="https://user-images.githubusercontent.com/66237694/229611668-b7fa10f8-ba83-4558-a4bf-236233f6fb06.png">

`kubectl delete pod` 명령으로 Static Pod를 삭제하면 Static Pod가 곧바로 재생성된다.
'static-greenbox-node01' Static Pod를 삭제하기 위해서는 디렉터리의 Static Pod 정의 파일을 삭제해야 한다.
아래의 경우, Static Pod가 node01에 생성되어 있기 때문에 `ssh {IP}` 명령어를 사용하여 node01에 접속하고 디렉터리의 Static Pod 정의 파일을 삭제한다.

<img width="783" alt="image" src="https://user-images.githubusercontent.com/66237694/229611802-08274fec-53c1-400b-86ba-acdb836f4044.png">

<img width="786" alt="image" src="https://user-images.githubusercontent.com/66237694/229611963-c8a7bb24-24bb-4a03-bc60-40082cec4d59.png">
