# 파드
- 쿠버네티스의 기본 빌드 단위이며, 쿠버네티스에서 가장 중요한 개념이다
- 파드는 컨테이너 하나가 아니라 여러 컨테이너의 그룹이다
    - 하지만 일반적으로 파드는 하나의 컨테이너만 포함한다
    - 파드에 속한 컨테이너들은 항상 하나의 워커노드에서 실행된다(여러 노드에 분산되서 실행되지 않는다)
- 컨테이너를 직접 사용하지 않고 파드를 사용하는 이유는 아래와 같다
    - 여러 프로세스를 같이 실행해야하는 상황을 충족시키지 못하기 떄문이다
        > 컨테이너 자체가 **단일 프로세스를 실행하는 것을 목적으로 설계된 기술**이기 때문이다
        > 컨테이너에 여러 프로세스가 들어간다면, 개별 프로세스에 대한 제어 메커니즘이 필요해지고, 여러 프로세스에서 동일한 표준출력으로 찍는 로그를 파악하기 어려워진다
    - 파드를 사용하면 단일 컨테이너 안에서 프로세스가 같이 실행되는 것 처럼 환경을 구성할 수 있다

- 특별한 이유가 없다면 하나의 파드에 하나의 컨테이너(어플리케이션)만 넣어 구성하는 것이 좋다
    - 아래의 사항들을 체크해보면 좋다
        - 컨테이너를 함꼐 실행해야 하는가(IPC?)? 혹은 다른 호스트에서 실행해도 되는가?
        - 여러 컨테이너가 모여서 하나의 구성요소가 되는가? 혹은 개별적인 구성 요소인가?
        - 컨테이너가 함께, 혹은 개별적으로 스케일링 되어야 하는가?
    - 컨테이너를 개별적으로 스케일링 해야한다면, 하나의 파드에 배포해야한다 
        - 쿠버네티스 수평확장의 최소단위가 파드이기 때문에(컨테이너 개별 확장 불가능)
    - 여러 컨테이너를 하나의 파드에 넣는 주된 이유는, 어플리케이션이 하나의 주요 프로세스와 하나 이상의 보완 프로세스로 구성된 경우이다
        > e.g. 주 컨테이너는 특정 디렉터리에서 파일을 제공하는 웹 서버이고, 추가 컨테이너는 외부 소스에서 주기적으로 콘텐츠를 받아 디렉터리에 저장함
        > e.g. 로그 로테이터와 수집기, 데이터 프로세서 등

- 같은 파드내 컨테이너들은 부분적으로 격리된다
    - 여러 프로세스가 실행되는 것과 동일한 환경을 만들어줘야 하기 때문이다
    - 파드내 모든 컨테이너가 동일한 네임스페이스(네트워크, UTS, IPC)를 사용하도록 한다
    - 그러나 파일시스템은 컨테이너의 특성 때문에 여전히 분리되므로, 볼륨이라는 개념을 사용해야 한다

## kubectl 로 pod 컨트롤하기
- yaml 파일을 통해 pod 만들기
```shell
$ kubectl create -f {pod-yaml-name}.yaml
```
- 실행중 pod 들 보기
```shell
$ kubectl get pods
```
- 파드의 전체 정의보기
```shell
$ kubectl get pods -o yaml
```
- 파드 로그 보기
```shell
$ kubectl logs {pod-name}
```
- 파드에서 컨테이너를 지정해 로그 보기
```shell
$ kubectl log {pod-name} -c {container-name}
```
- 로컬 네트워크 포트를 pod 의 포트로 포워딩
```shell
$ kubectl port-forward {pod-name} {local-port}:{pod-port}
```
- 이름으로 파드 삭제
    - SIGTERM 신호를 보내고 지정된 시간동안 기다린 뒤, 시간내에 종료되지 않으면 SIGKILL 신호를 통해 종료한다
```shell
# 하나 삭제
$ kubectl delete pods {pod-name}

# 여러개 삭제
$ kubectl delete pods {pod-name} {pod-name} {pod-name}
```

# 레이블
- 어플리케이션의 숫자가 많아지다 보면 자연스럽게 pod 의 수도 많아지게되고, 이를 위해 pod 를 부분 집합으로 분류할 필요가 생기게 된다  
- 레이블은 이를 해결하기 위해 만들어진 기능으로써, 쿠버네티스의 모든 리소스를 조직화 할 수 있는 단순하면서 강력한 기능이다  
- ![파드-레이블](img/pod-label.jpg)
    - 레이블을 추가해서 위처럼 2차원적으로 구성할 수 있다  
- label 을 지정하고 싶다면 yaml 파일의 `metadata` 필드에 label 을 명시해주면 된다
- 기존 파드에 레이블을 추가하고 싶다면 아래와 같이 입력하면 된다
```shell
$ kubectl label pods {pod-name} {label_key=label_value}
```

- 레이블은 레이블 셀렉터와 함께 사용할 때 강력한 기능을 발휘한다
- 아래의 기준에 따라 리소스들을 그룹핑하여 작업을 수행할 수 있기 떄문이다
    - 특정한 키를 포함하거나 포함하지 않는 레이블
    - 특정한 키와 값을 가진 레이블
    - 특정한 키를 갖고 있지만, 다른 값을 가진 레이블
- 레이블 셀렉터 사용법은 아래와 같다
```shell
# env 레이블의 값이 prod 인 pod 들만 출력한다
$ kubectl get pods -l env=prod

# env 레이블을 갖고 있지만, 값이 무엇이든 상관없는 pod 들을 출력한다
$ kubectl get pods -l env

# env 레이블을 갖고 있지 않은 pod 들을 출력한다
$ kubectl get pods -l '!env'

# env 레이블의 값이 prod 가 아닌 pod 들을 출력한다
$ kubectl get pods -l 'env!=prod'

# env 레이블의 값이 dev 또는 prod 로 설정되어 있는 pod 들을 출력한다
$ kubectl get pods -l env in (prod, dev)

# env 레이블의 값이 dev, prod 가 아닌 pod 들
$ kubectl get pods -l env notin (prod, dev)

# 쉼표로 연결해 여러 조건을 사용할 수 있다(and 로 연결됨)
$ kubectl get pods -l app=pc, rel=beta
```
- 레이블 셀렉터는 레이블로 셀렉트한 리소스들에 특정 작업을 수행할 떄 진가를 발휘한다

- 레이블 셀렉터를 이용해 pod 를 특정 노드에만 배포하기
```shell
# gpu 가 달려있는 노드들에 label 추가
$ kubectl label node gke-kube-test-default-pool-c469142f-pscl gpu=true

# nodeSelector 가 gpu=true 로 작성된 yaml 파일로 pod 생성
$ kubectl create -f kube-gpu.yaml

# 제대로 배포되었는지 확인
$ kubectl get pods -o wide
NAME              READY   STATUS     RESTARTS   AGE    IP           NODE                                       NOMINATED NODE   READINESS GATES
kubia-gpu         1/1     Running    0          2s     <none>       gke-kube-test-default-pool-c469142f-pscl   <none>           <none>
kubia-manual      1/1     Running    0          4h9m   10.56.1.6    gke-kube-test-default-pool-c469142f-tbfd   <none>           <none>
```

- 레이블 셀렉터를 이용한 파드 벌크 삭제
```shell
$ kubectl delete pods -l release=canary
```

# 네임스페이스
- 쿠버네티스 클러스터에 환경별로 리소스들이 올라가있거나, 여러 사용자가 클러스터에 리소스들을 올리는 등의 상황에서는 label 만으로는 리소스들을 식별하기가 힘들어진다
- 이를 위해 쿠버네티스에서는 네임스페이스 개념을 제공한다
- 리소스 이름, 레이블 등은 모두 네임스페이스 안에서만 고유하면 된다
    - 참고로 노드 리소스는 전역이라 네임스페이스에 얽매이지 않는다
- 네임스페이스는 아래와 같이 조회할 수 있다
```shell
# namespace 조회
$ kubectl get namespaces 
NAME              STATUS   AGE
default           Active   5h59m # 지정안할 시 기본으로 사용됨
kube-node-lease   Active   5h59m
kube-public       Active   5h59m
kube-system       Active   5h59

# namespace 지정해서 리소스 조회
$ kubectl get pods --namespace kube-system
NAME                                                       READY   STATUS    RESTARTS   AGE
kube-dns-5995c95f64-kqrfs                                  4/4     Running   0          5h46m
kube-dns-autoscaler-8687c64fc-vgj2m                        1/1     Running   0          5h46m
kube-proxy-gke-kube-test-default-pool-c469142f-pscl        1/1     Running   0          5h46m
...
```
- 네임스페이스는 아래와 같이 생성할 수 있다
```shell
$ kubectl create namespace custom-namespace
```
- 쿠버네티스 리소스에 네임스페이스를 지정하는 법은 yaml 파일 metadata 부분에 namespace 항목을 넣거나 kubectl create 명령을 사용할 때 네임스페이스를 지정하는 방법이 있다
```shell
$ kubectl create -f kubia-manual.yaml -n custom-namespace
```
- 네임스페이스는 실행중인 오브젝트에 대해서는 격리를 제공하지 않는다
    - foo 네임스페이스에 속한 pod 가 bar 네임스페이스에 속한 pod 와 통신할 수 없다고 생각할 수 있지만, 이는 네트워킹 솔루션에 따라 다르다
    - 네임스페이스에 대한 별도 네트워킹 솔루션이 없다면 foo 의 pod 가 bar 의 pod 로 통신하는 것이 가능하다