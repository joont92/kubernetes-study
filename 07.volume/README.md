# volume
- pod 내 컨테이너들끼리 CPU, RAM, NIC 등은 공유하지만 디스크는 공유하지 못한다
    - 파일 시스템은 컨테이너 이미지에서 제공되기 때문이다
    - 이 같은 특징 때문에, 새로 시작한 컨테이너는 이전에 실행했던 컨테이너에 쓰여진 파일시스템의 어떤것도 볼 수 없다(같은 pod 내에서 실행되었더라도 마찬가지)
- 위와 같은 요구사항은 언제든 발생할 수 있는데, 쿠버네티스는 이를 volume 이라는 개념을 통해 해결한다

- 예를 들어 한 pod 에 아래와 같은 3개의 컨테이너가 들어있다고 가정해보자
    - /var/htdocs 에 있는 html 페이지를 서비스하고, /var/logs 에 로그를 쌓는 웹 서버 컨테이너
    - /var/html 에 주기적으로 html 파일을 생성해서 넣는 컨테이너
    - /var/logs 에 있는 로그를 처리하는 컨테이너
- 컨테이너 3개가 같이 동작해야하며, 같은 파일 시스템을 공유해야 한다
- ![volume](img/volume.jpg)
    - publicHtml, logVol 이라는 volume 을 2개 만들고, 각 컨테이너에서 원하는 파일시스템의 경로에 volume 을 마운트했다
- 보다시피 volume 은 2가지 특성이 있다
    - 쿠버네티스 리소스가 아니므로 자체적으로 생성, 삭제될 수 없다
    - volume 을 생성만 한다고 되는것이 아니라, 각 컨테이너에서 직접 마운트 해줘야 한다

- volume 에도 여러 종류의 타입들이 있다
    - 위에서 사용한 volume 의 타입은 `emptyDir` 이다

## volume 을 사용한 컨테이너 간 데이터 공유
### emptyDir
- [fortuneloop.sh](fortuneloop.sh), [Dockerfile](Dockerfile), [fortune-pod.yaml](fortune-pod.yaml) 참조
- volume 이 빈 디렉터리로 시작된다
- 어떤 파일이든 volume 에 쓸 수 있다
- volume 의 라이프사이클이 pod 에 묶여 있으므로, pod 가 삭제되면 volume 의 컨텐츠도 삭제된다
- pod 를 호스팅하는 워커 노드의 실제 디스크에 생성된다 
    - 노드 디스크의 유형에 따라 성능이 결정된다
    - **여러 pod 간 volume 의 데이터를 공유할 수 없다**

### gitRepo
- 이름 그대로 git repository 에서 특정 리비전을 체크아웃해서 데이터를 채운다
- emptyDir volume 을 생성한 뒤 데이터를 채우는 것이다(emptyDir 의 확장)
