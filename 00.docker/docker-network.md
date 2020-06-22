<https://bluese05.tistory.com/15>  
docker0 이라는 브릿지 생성  
컨테이너 하나 생성할 때 마다 veth interface 1쌍 생성  
(veth interface 는 단순히 응답을 연결된 다른곳으로 보내주는 용도)  
veth interface 한쪽은 호스트 os 와 동일한 namespace 에 두고, 한쪽은 컨테이너가 사용하는 namespace 에 두면서 이름을 eth0 으로 설정함  
즉 컨테이너에서 ifconfig 로 볼 수 있는 eth0 은 사실 host(node)의 veth interface 임(namespace 로 보이지않게 해놓은것일 뿐)  
그리고 이 veth interface 는 docker0 이라는 브릿지에 연결됨  
(네트워크 브릿지 모드 공부 필요)  
그리고 이 브릿지에 해당하는 ip 대역대로 veth interface 를 생성함  

docker-compose 로 도커 컨테이너를 생성할 경우 docker0 과 같은 브릿지가 하나 더 연결됨  

<https://bluese05.tistory.com/38>  
container 모드를 사용하면 아예 네트워크를 공유한다(ip, mac 이 동일함)  
이건 어떤 기술을 써서 하는걸까?