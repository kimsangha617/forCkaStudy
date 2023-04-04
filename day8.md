## Deployment
#### Updates and Rollback


<img width="773" alt="image" src="https://user-images.githubusercontent.com/66237694/229945072-da139470-3863-4bba-80bc-a7af5019e59c.png">

1. 첫 배포를 하면 롤아웃이 생성됌

1. 새로운 롤아웃은 새로운 배포 revisions를 생성함 - revision1 이라고 하자

1. 향후 앱이 업그레이드 되면 즉, 컨테이너 버전이 새것으로 업데이트 되면 새 롤아웃이 trigger 되고 새 배포 revisions가 생성됨 - revision2

1. 이렇게하면 배포에 일어난 변화를 추적할 수 있고 롤백도 가능함

명령을 실행해 롤아웃 상태를 볼 수 잇다

Rollout Command
----

<img width="746" alt="image" src="https://user-images.githubusercontent.com/66237694/229945201-633d8d6c-9a27-4646-81cf-aba9302a15ed.png">

`kubectl rollout status deployment/myapp-deployment`

revisions와 rollout history를 확인하려면
`kubectl rollout history deployment/myapp-deployment`

#### Deployment Strategy

<img width="711" alt="image" src="https://user-images.githubusercontent.com/66237694/229945274-9dfbf343-9951-4da9-91e9-b29a4aaf37b8.png">

5개의 replica applicationd이 있다고 가정하자

1. Java Web Application 1.5.0
2. Java Web Application 1.5.0
3. Java Web Application 1.5.0
4. Java Web Application 1.5.0
5. Java Web Application 1.5.0

Rolling Update 전략은

1번 앱을 다운 -> 업데이트버전 1.5.1 배포
2번 앱을 다운 -> 업데이트버전 1.5.1 배포
...

이렇게 하면 앱이 다운되지 않고 배포가 가능

Rolling Update 는 Default Deployment Strategy 이다


두 명령어는 각각 deployment를 업데이트하여 새로운 rollout을 생성하고 새로운 revision이 만들어진다.
후자의 방법은 deployment 정의 파일과 다른 구성을 갖게 되기 때문에 주의해야 한다.

<img width="785" alt="image" src="https://user-images.githubusercontent.com/66237694/229945334-e80e687f-3996-4b08-ab4c-f8ec5c3701d4.png">

업그레이드를 잘못해서 새 버전 빌드에 문제가 있다면 kubectl rollout undo 명령어를 사용해서 이전 리비전으로 롤백할 수 있다.
새 replicaset의 파드가 파괴되고 이전의 replicaset으로 돌아간다.

<img width="768" alt="image" src="https://user-images.githubusercontent.com/66237694/229945787-620fdfe6-bc7e-4194-b513-b2a5648fac45.png">

`kubectl get replicasets` 명령어를 통해
replica의 변형 상태를 볼 수 있다

<img width="802" alt="image" src="https://user-images.githubusercontent.com/66237694/229946045-0262531d-e150-4c86-99db-12a6f1bf4646.png">

--- 여기까지가 day7 수정 진행해야함




------day8


## 95. Commands

도커에서 컨테이너는 컨테이너 내부의 프로세스가 살아있어야 Running 상태가 된다.
컨테이너 안의 웹 서비스가 멈추거나 충돌하면 컨테이너는 종료된다.

컨테이너에서 실행되는 프로세스는 Dockerfile에 의해 정의된다.
Dockerfile의 CMD는 컨테이너 안에서 실행될 명령이다.

컨테이너 시작 명령을 어떻게 지정할까?

첫번째 방법은 docker run 명령어에 COMMAND를 추가하는 것이다.
그럼 이미지에 지정된 기본 명령을 재정의한다.
docker run ubuntu sleep 5를 실행하면 컨테이너가 작동할 때 sleep을 5초 동안 실행하고 종료된다.

<img width="601" alt="image" src="https://user-images.githubusercontent.com/66237694/229947245-4cc4b93d-adc9-4dd0-8172-a41a3ec409f2.png">

두 번째 방법은 CMD를 추가한 Dockerfile을 작성해서 이미지를 빌드하는 것이다.
일시적으로 명령을 실행하는 앞의 방법과 다르게 컨테이너가 영구적으로 명령을 실행하도록 할 수 있다.

명령을 지정하는 방법은 셸 양식, JSON이 있다.
JSON 배열 포맷에서 명령을 지정할 때 배열의 첫 번째 요소가 실행 가능해야 한다.
또한 명령과 매개 변수를 별도로 지정해야 한다.

<img width="768" alt="image" src="https://user-images.githubusercontent.com/66237694/229947314-14ccd74f-5466-4cf5-a7a4-359039fd88fa.png">

CMD는 docker run 명령의 [COMMAND] 옵션에 따라 CMD의 매개변수가 완전히 바뀐다.
반면에, ENTRYPOINT의 경우 전달받은 매개변수를 추가해서 명령을 실행한다.
만약, 매개변수를 전달 받지 못하면 'sleep' 명령이 실행되어 누락된 명령문이라는 오류가 발생한다.

<img width="763" alt="image" src="https://user-images.githubusercontent.com/66237694/229947394-720adb16-2d73-4a45-83e8-eff1c64b5d1d.png">

매개변수의 기본값을 설정하기 위해서 ENTRYPOINT와 CMD를 둘 다 사용하면 된다.
이 경우, 매개변수를 지정하지 않았다면 ENTRYPOINT 명령어에 CMD 명령이 추가된다.
만약 매개변수가 지정되었으면 CMD 명령은 무효화된다.
이러한 경우에는 ENTRYPOINT와 CMD를 항상 JSON 포맷으로 지정해야 한다.

마지막으로 ENTRYPOINT를 수정하고 싶다면 '--entrypoint' 옵션을 사용해 재정의하면 된다.

<img width="770" alt="image" src="https://user-images.githubusercontent.com/66237694/229947463-aad26b72-3bed-4d26-b726-ae25831992e6.png">


