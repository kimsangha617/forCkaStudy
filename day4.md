쿠버네티스 서비스는 어플리케이션 안팎의 다양한 구성요소간의 통신을 가능하게 한다

쿠버네티스 서비스는 애플리케이션을 다른 어플리케이션 또는 사용자와 연결하는데 도움을 준다

예를들어 어플리케이션에는 다양한 섹션을 실행하는 그룹이 있다

사용자에게 프론트엔드 로드를 실행하는 그룹과 백엔드 프로세스를 실행하는 다른 그룹 외부 데이터 소스에 연결하는 제 3그룹이있다

이런 포드 그룹 간에 연결을 가능하게 하는게 서비스다

서비스는 프론트엔드 응용프로그램을 최종 사용자가 사용할수 있도록 해준다

이것은 백엔드 포드와 프론트엔드 포드의 통신을 돕고 외부 데이터소스와의 연결을 설정하는데 도움을 준다

따라서 서비스들은  마이크로서비스 사이에서 느슨한 결합을 가능하게 한다



우리가 웹 애플리케이션이 동작하고 있는 pod를 배포하고 있다고 가정해보자

외부 사용자는 어떻게 웹페이지에 이용할 수 있을까?

쿠버 노드에는 IP주소가 있는데 192.168.1.2 가 있다고 가정해보자

제 노트북도 같은 네트워크에 있어서 IP주소가 192.168.1.10 이다

내부 pod 네트워크는 10.244.0.0 범위 내에 있다

pod IP는 10.244.0.2

명확히 cannot ping or access the pod at address 10.244.0.2 이것은 분리된 네트워크에 있다

웹페이지를 볼 수 있는 방법은 ?

first, 만약 ssh로 쿠버 노드로 들어간다면  curl http://192.168.1.2 방법으로...

curl 함으로써 접근할 순 있겠다

하지만 이건 쿠버네티스 노드 내부에서 왔고 우리가 원하는 방법이 아니다

노드에 ssh를 사용하지 않고 내 노트북(192.168.1.10)에서 웹서버에 엑세스 하길 원한다.

쿠버네티스는 노드의 IP를 엑세스 함으로써 말이다.

중간에 도움이 될만한게 필요하다. 

노트북 노드에서 노드를 통해 웹 컨테이너를 실행 중인 포드로 요청을 맵핑하는데 도움이 될만한 것들 말이다

이럴 때에 사용하는것이 쿠버네티스 서비스이다

Kubernetes Service 는 전에 작업했던 Pod나 ReplicaSet 혹은 배포와 같은 객체이다

Node Port :  서비스가 노드의 port를 listen 해서 pod 에 요청한다(forward request) 

Service Types

1. NodePort

서비스가 노드의 포트에 내부 포트를 엑세스하게 하는것이다. Cluster IP로만 접근이 가능한것이 아니라, 모든 노드의 IP와 포트를 통해서도 접근이 가능하게 된다.

2. ClusterIP

서비스는 cluster 안에 Virtual IP를 만들어 다양한 서비스 간의 통신을 가능하게 한다

프런트엔드 서버 세트와 백엔드 서버 세트 간의 통신을 돕는다

3. Load Balancer

클라우드 제공자의 부하 분산 장치를 사용하여 서비스를 외부에 노출한다.


* Service - NodePort

Node (port:30008)

Node 안에 Service(port:80) / simply referred to as the port (just called 'port')

Node 안에 Pod(port:80) / target port

서비스관점에서 불리는거임 (타겟포트, 포트)

NodePort 외부에서 웹 서버에 access 할 때 사용하는 port(30008)

NodePort의 유효범위는 30000 ~ 32767

 - 서비스 생성 방법 - 
 
 * service-definition.yml

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec: 
  type: NodePort
  ports:
   - targetPort: 80
     port: 80
     nodePort: 30008
  selector:
     app: myapp
     type: front-end
```


 * pod-definition.yml
 
 ```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels: 
    app: myapp
    type: front-end
spec: 
  containers:
  - name: nginx-container
    image: nginx
```

pod-definition.yml 에서
labels를 꺼내서 
service-definition의 selector 에 추가해주게 되면 service와 pod를 연결한다


완료되면 kubectl을 이용해 서비스를 생성하고 서비스 정의 파일을 입력해라

> kubectl create -f service-definition.yml

console
service "myapp-service" created

이렇게되면 서비스가 생성되게 된다 

생성한 서비스를 보려면

> kubectl get services

> curl http://192.168.1.2.:30008



 - 쿠버네티스가 launch 될 때 서비스가 실행된다
 - 쿠버네티스 service의 default type은 무엇인가 ?
    - ClusterIP
 - 쿠버네티스 서비스의 설정된 targetPort는 몇번인가?
    - 6443
      - 쿠버네티스의 targetPort를 따로 설정해주지 않으면 default로 port의 값이 설정된다
      - 쿠버네티스의 default port 값은 6443 이다
 - kubectl describe service
 service의 설정값을 볼 수 있다
 
 
