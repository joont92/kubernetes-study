# batch job
- 작업을 완료한 후에 종료되는 배치 어플리케이션도 쿠버네티스를 이용해 실행할 수 있다
- job 이 실행중인 노드에 장애가 발생하면 job 은 다른 노드로 스케줄링 된다
- job 을 사용할 경우 pod 의 restartPolicy 속성을 명시적으로 OnFailure 나 Never 로 설정해줘야 한다
    - job 의 pod 는 무한정 실행하는 리소스가 아니기 때문이다
    - 기본값은 Always 다
- 2개 이상의 pod 를 생성해 병렬 또는 순차적으로 실행하도록 구성할 수 있다
    - completions 와 parallelism 속성을 이용해 이를 조정한다
- job 의 실행 시간도 제어할 수 있다
    - pod 이 특정 상태에 빠져서 완료할 수 없는 상태가 될 수도 있기 때문이다
    - activeDeadlineSeconds 속성을 이용한다
- 아래는 5개의 pod 를 2개씩 병렬로 실행하는 job 의 예시이다
    ```shell
    $ kubectl create -f batchjob-multi-completion.yaml
    job.batch/multi-completion-batch-job created

    $ kubectl get po
    NAME                               READY   STATUS    RESTARTS   AGE
    multi-completion-batch-job-fs57p   1/1     Running   0          13s
    multi-completion-batch-job-nqnjv   1/1     Running   0          13s

    ...

    $ kubectl get po
    NAME                               READY   STATUS       RESTARTS   AGE
    multi-completion-batch-job-fs57p   0/1     Completed    0          213s
    multi-completion-batch-job-nqnjv   0/1     Completed    0          213s
    multi-completion-batch-job-auv21   1/1     Running      0          13s
    multi-completion-batch-job-zasdw   1/1     Running      0          13s
    ```

---

# cron job
- 위의 batch job 은 생성 시 바로 실행하지만, 대부분의 batch job 은 특정 시간 또는 지정된 간격으로 반복 실행해야 한다
- 이를 위해 보통 리눅스의 crontab 을 이용하는데, kubernetes 에도 이와 비슷한 기능을 제공해주는 CronJob 이라는 리소스가 있다
- 리소스 종류를 CronJob 으로 주고, schedule 속성에 crontab 에서 사용하던 표현식을 작성해주면 된다
- cron job 중 예정된 시간을 엄격하게 지켜야하는 경우도 있는데, 이때는 startingDeadlineSeconds 속성을 사용하면 된다
    - 이는 cron job 시작 시간으로부터 지정한 시간을 초과했을 시 실패로 간주하는 옵션이다
- cron job 으로 실행하는 어플리케이션은 아래의 사항을 충족하도록 작성해주는 것이 좋다
    - 동시에 2개의 job 이 생성될 수 있으므로 멱등성을 가지도록 작업을 작성해야 한다
    - 특정시간에 job 이 누락될 수 있으므로, 다음번 job 이 이전 job 을 잘 실행했는지 검사해야 한다