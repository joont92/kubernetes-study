spring config 는 어떻게 사용되는가?  
pod spec 에서 적용한 enviroment 는 컨테이너 내의 enviroment 를 덮어씀  
일반적으로 어플리케이션들은 어플리케이션 프로퍼티보다 시스템 프로퍼티를 더 우선순위 높게 설정할것이다?  
설정을 볼륨에 마운트하더라도 어플리케이션에서 파일에 있는 config 를 주기적으로 읽어오도록 설정하거나 reload 하도록 operation 을 날려줘야 한다  

컨피그맵이나 시크릿맵은 일반적으로 사용하는 방법은 똑같다  
시크릿은 describe 하면 base64 인코딩되어서 출력됨  
컨피그맵이랑 시크릿으로 2개를 분리해놓으면 RBAC 으로 권한 설정할 수 있음(configmap은 all, secret 은 인가된 사용자)
시크릿은 1MB 제한인가? 있다  

사내에서는 아래와 같이 사용 config map 을 선언한다  
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: region-env
  namespace: {{ .Values.namespace | default "vroong" | quote }}
data:
  {{- range $key, $val := .Values.envvars }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
```
> envvars 는 manifest 에 있는 파일이다  

그리고 deployment 에서 아래와 같이 사용  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: region
  namespace: {{ .Values.namespace | default "vroong" | quote }}
spec:
  replicas: 2
  selector:
    matchLabels:
      app: region
      version: "1.0"
  template:
    metadata:
      labels:
        app: region
        version: "1.0"
        filebeat-collect: "true"
        log-format: json
      annotations:
        iam.amazonaws.com/role: {{ .Values.iamRole | quote }}
    spec:
      containers:
      - name: region-app
        image: 200327251464.dkr.ecr.ap-northeast-2.amazonaws.com/vroong/region
        envFrom:
        - configMapRef:
            name: region-env
```