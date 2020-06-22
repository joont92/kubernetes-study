# Pod 내 컨테이너 Network
Pod은 내부의 컨테이너들끼리 IP와 port를 공유한다는 특징이 있었는데, 어떻게 구성되어 있길래 그럴까?  
아래는 Pod 의 네트워크를 잘 설명해주는 하나의 그림이다  

![kubernetes-pod-network](https://joont92.github.io/temp/kubernetes-pod-network.png)  
> 출처 : <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>  

기본적으로 도커 컨테이너를 하나 생성하면, 해당 컨테이너를 namespace로 격리한 뒤 eth0 interface를 생성하고, 노드에 veth interface를 생성해서 연결시키는 구조로 동작한다  
하지만 위의 그림은 보다시피, 컨테이너별로 각기 다른 veth interface에 연결된 것이 아닌, 공통된 하나의 veth interface에 연결되어 있음을 볼 수 있다  

이는 쿠버네티스가 Pod 내에서 컨테이너를 생성할 때 도커 컨테이너를 bridge 모드(default)가 아닌 container 모드로 띄웠기 때문이다  
container 모드는 아래와 같이 다른 컨테이너를 지정함으로써 띄울 수 있는데, 이렇게 생성함으로써 생성된 도커 컨테이너는 지정한 도커 컨테이너와 네트워크를 공유하는 형태로 생성될 수 있게 된다  
```sh
$ docker run --net=container:{container_id} -d {image_name}
```
> 자세한 내용은 <https://bluese05.tistory.com/38>를 참조한다  

이렇게 생성된 컨테이너들을 각각 들어가서 ip 정보를 확인해보면, 모두 ip가 같음을 볼 수 있을 것이다  
즉 결론은, Pod 내의 도커 컨테이너들은 모두 위처럼 container 모드로 실행되었기 때문에 아래와 같은 특성이 만족되는 것이다  
- 외부에서 같은 IP로 접근할 수 있다(각 컨테이너들의 IP가 같음)
- 컨테이너간 localhost 로 통신할 수 있다
- 같은 port는 사용 불가능

쿠버네티스는 이러한 구조를 `pause` 라는 특별한 컨테이너를 통해 구현한다  
정확한 순서는 모르지만 pod 생성 시  
1. pause 컨테이너 생성
2. pause 컨테이너가 워커 컨테이너들을 container mode 로 생성

의 역할을 해주지 않을까 싶다  
이러한 역할을 하기 위해 pod 마다 하나씩 다 떠있는 것이고..  

## 노드가 다른 Pod 끼리의 통신
위의 모델의 경우 같은 `docker0` 브릿지 아래에서는 Pod 간의 통신이 전혀 문제될 것이 없지만,  
쿠버네티스 클러스터의 경우 보통 1개 이상의 노드를 관리하며, Pod 는 매번 다른 노드에 배포되는 특성이 있다  

중요한 점(문제되는 점)은 `docker0` 브릿지는 각 노드마다 생성되는데, 이 `docker0` 브릿지의 ip 대역대가 노드마다 겹칠 수 있다는 것이다  
`docker0` 아래의 컨테이너들은 전부 `docker0` 브릿지의 네트워크 대역을 따라가는데, 만약 `docker0` 브릿지의 네트워크 대역이 겹친다면 결국 Pod의 IP가 겹치는 문제가 발생한다  

![kubernetes-pod-network-multi-node1](https://joont92.github.io/temp/kubernetes-pod-network-multi-node1.png)  
> 출처 : <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>  

이런식으로 말이다  
이 상태에서는 src와 dest의 IP가 같기 때문에 Pod 간에 서로 통신할 수가 없다  
이런 형태는 일어날 확률이 매우 높은데, 이는 노드 안에서 생성되는 `doccker0` 브릿지의 경우 default 네트워크 대역이 정해져있기 때문이다  
그리고 만약 default 값을 사용하지 않는다고 해도, 다른 노드의 `docker0` 브릿지가 어떤 네트워크 대역을 사용하고 있는지 알지 못하기 때문에 문제가 해결되지는 않는다  

쿠버네티스는 이러한 문제점을 해결하기 위해 2가지 방법을 사용했다  
- `docker0` 브릿지들의 전체 네트워크 대역을 아우르는 주소 대역을 할당한다
- 들어오는 패킷들이 어떤 브릿지로 가야하는지에 대해 라우팅 테이블을 설정한다

이를 적용하면 아래와 같은 모습이 된다  

![kubernetes-pod-network-multi-node2](https://joont92.github.io/temp/kubernetes-pod-network-multi-node2.png)  
> 출처 : <https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727>  

(docker에서 사용하는 `docker0` 브릿지로는 이 문제를 해결할 수 없기 때문에 이를 커스텀한 `cbr` 브릿지를 사용했다)  

이제 Pod은 항상 라우팅 테이블을 거쳐 다른 Pod 이 존재하는 브릿지를 찾아갈 수 있기 때문에, 다른 노드에 있는 Pod 끼리도 원할하게 통신이 가능해진다!  

---

참고 : <https://coffeewhale.com/k8s/network/2019/04/19/k8s-network-01/>  
네트워크 브릿지, 라우팅에 대한 이해 필요