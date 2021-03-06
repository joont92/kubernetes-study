# 쿠버네티스 등장 배경
- 최근 몇년간 애플리케이션의 개발과 배포 방식이 바뀌었다
    - 거대한 모놀리스를 만들기보다는 작은 마이크로서비스들을 만드는 방식으로 변화하고 있다
- 모놀리스는 아래와 같은 특징을 가지고 있다
    - 구축하기가 쉽다
    - 애플리케이션의 한 부분을 변경하더라도 전체를 재배포해야 한다
    - 시간이 지남에 따라 구성요소간 경계가 불분명해지고, 상호 의존성이 커진다
    - 시간이 지남에 따라 유지관리하기가 어려워지고, 확장도 어려워진다
- 마이크로서비스는 아래와 같은 특징을 가지고 있다
    - 마이크로서비스는 독립적으로 배포되고 실행될 수 있다 
    - 마이크로서비스들끼리는 잘 정의된 인터페이스를 통해 통신한다
        - HTTP 같은 동기 프로토콜 또는 AMQP 같은 비동기 프로토콜을 사용해 통신한다
    - 마이크로서비스는 전체를 확장할 필요없이 리소스가 필요한 서비스만 별도로 확장할 수 있다
    - 여러 시스템들이 하나의 시스템으로 작동하도록 배포하고 구성하기가 어렵다
    - 분산 시스템이다 보니 에러를 추적하거나 디버깅하는 것이 힘들다
    
- 마이크로서비스가 아니더라도 시스템에 배포하는 애플리케이션 구성 요소의 수가 많아지면 이를 관리하기가 어려워지게 되고, **이를 잘 처리해주지 않는 무언가**가 있지 않는 이상 체계를 계속 유지하기 힘들다
    - 이를 매우 잘 처리해주는 무언가 중에 가장 유명한것이 쿠버네티스라고 보면 된다
    - 쿠버네티스의 모든 오브젝트들은 결국 필요에 의해 만들어진 것이다

# 쿠버네티스 소개
- 쿠버네티스는 구글에서 개발자와 시스템 관리자가 수천개의 애플리케이션과 서비스를 관리하는데 도움을 준 Borg(Omega)를 기반으로 만들어졌다
- 쿠버네티스는 마스터 노드와 워커 노드로 구성된다
    - 마스터 노드는 쿠버네티스 시스템을 제어하고 관리하는 `쿠버네티스 컨트롤 플레인`을 실행한다
    - 워커 노드에는 실제 애플리케이션들이 컨테이너 형태로 배포된다
- 쿠버네티스를 사용하면 아래와 같은 이점들을 누릴 수 있다
    - 사용자가 제공한 디스크립션과 현재 애플리케이션들의 상태가 맞는지 지속적으로 확인하고, 이를 계속 유지하려고 시도한다(쿠버네티스의 메인 컨셉)
    - 어플리케이션을 간단하게 수평 확장/축소 할 수 있다
    - 어플리케이션들은 어떤 노드(하드웨어)든 관계없이 가장 효율적인 위치에 배포되므로 하드웨어 활용도를 높일 수 있다
    - 수평확장된 어플리케이션들이 서로 다른 노드들에 퍼져있어도 외부에서는 이를 신경쓸 필요없이 계속 동일하게 접근할 수 있다
    - 어플리케이션들이 배포된 노드에서 장애 발생 시 자동으로 다른 노드로 어플리케이션들을 스케줄링한다
    - 컨테이너 기술을 기반으로 하였으므로 컨테이너 기술의 장점을 모두 누릴 수 있다(운영과 개발 환경간의 동일함)

- 스케일아웃이 항상 가능한것도 아님(RDB)
- 여러 서비스가 하나의 시스템처럼 동작하지 않는다면 딱히 마이크로서비스의 장점이 없다
- 모놀리스를 사용할 경우 모놀리스 내에서 분리된 서비스들끼리 다른 라이브러리 버전을 사용할 수 없다
- 도커 이미지 레이어?
- 특정 하드웨어 기반으로 작성된 어플리케이션은 도커라도 구동되지 않을 수 있음(IOT)
- eks 를 사용하면 k8s 서비스에 elb 를 쉽게 붙일 수 있다
    - route53도 등록해서 외부로 url 을 제공해줄 수 있다
- 워커노드 내 컨테이너 런타임 : 컨테이너(도커)를 실행함
- 워커노드내 pod 를 띄울 공간이 부족할 경우 새로운 노드를 띄움
    - 이 때 기존의 노드에 있던 pod 들도 어느정도 이사시킴(분산을 위하여)