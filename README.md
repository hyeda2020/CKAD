# CKAD
리포지토리 설명 : CKAD 자격증 공부 내용 정리  
참고 :  
https://kubernetes.io/ko/  
https://www.udemy.com/course/certified-kubernetes-application-developer/  

# 1. Core Concepts
![image](https://github.com/user-attachments/assets/8788bc3b-d7ec-42bc-b03f-6c8cf2dcabc6)
- Node : 클러스터 내 가상의 머신으로, 각 노드는 컨트롤 플레인에 의해 관리되고 kublet, kube-proxy와 같은 파드를 실행하는 데 필요한 서비스를 포함함.  
  - Control Plane Component : 클러스터에 관한 전반적인 결정(스케줄링 등)을 수행하고 클러스터 이벤트를 감지 및 반응  
    - kube-apiserver : 쿠버네티스 클러스터의 프론트 엔드로, CLI 인터페이스 제공함과 동시에 클러스터 내부와 상호작용 함.  
    - etcd : 클러스터에 대한 모든 데이터를 key-value 쌍 형태로 저장하는 고가용성 저장소.  
    - kube-scheduler : 새로 생성된 파드를 어느 노드에 배치하여 실행할지 결정하는 역할 수행.  
    - Controller : 노드, 파드가 다운되는 경우를 인지하고 대응하는 역할 수행.  
  - Node Component : 동작 중인 파드를 유지시키고 쿠버네티스 런타임 환경을 제공  
    - kubelet : 클러스터 각 노드에서 실행되는 에이전트로, 파드에서 컨테이너가 제대로 실행되는지 확인하고, 정상적으로 동작되도록 하는 역할 수행.  
    - kube-proxy : 클러스터 각 노드에서 실행되는 네트워크 프록시로, 내부 네트워크 세션이나 클러스터 바깥에서 파드로 네트워크 통신을 할 수 있도록 해줌.  
    - container runtime : 컨테이너를 실행하는데 필요한 기본 소프트웨어(ex. Docker)  
- Pod : 쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위로, 하나 이상의 컨테이너로 구성됨.  
  파드 내 컨테이너 그룹은 항상 함께 배치되고, 함께 스케줄되며, 네트워크와 스토리지를 서로 공유함.  

  사용량 증가로 인해 앱에 부하가 발생한다면 해당 노드에 새로운 파드를 추가로 생성하여 부하를 분산시키는 방식으로 앱의 규모를 늘림.  
  이후 계속된 사용량 증가로 인해 노드 내의 앱 규모가 확장되어 리소스 자원(용량, CPU 등)이 부족해지면 새로운 노드에 파드를 추가로 배포.  
  - Pod 생성 방법 예시(kubectl 명령어)  
  `kubectl run <pod-name> --image=<image-name>`


※ yml in Kubernetes : .yml 파일은 쿠버네티스 내에서 Pod, ReplicaSet, Deployment, Service 등의 개체 생성을 위한 일종의 '명세'로서 기능.  
```
# pod-definition.yml
apiVersion: v1  # Pod, Service의 경우는 v1, ReplicaSet, Deployment는 app/v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:  # 리스트 형식
  - name: nginx-container
    image: nginx
```

  
- ReplicaSet : 파드에 이슈가 발생되어 앱 전체가 다운되는 경우 등을 방지하기 위해 클러스터 내 특정 파드 인스턴스들이 지정된 개수 만큼 실행되는 것을 보장.  
```
# replica-definition.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: myapp
spec:
  # 케이스에 따라 레플리카 개수를 수정.
  replicas: 3
  selector:  # 인스턴스 개수를 보장할 파드를 레이블 기준으로 선택
    matchLabels: 
      tier: frontend
  template:  # 이하 Pod에 대한 정보 명시
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
```

  
- Deployment : Pod와 ReplicaSet에 대한 선언적 업데이트를 제공함으로써 이를 기반으로 쿠버네티스에서 롤링 업데이트/롤백를 안정적으로 운영 가능.
  
![image](https://github.com/user-attachments/assets/cb399c6a-72a3-4a68-b6e3-e49b9906a1bd)
```
# deploy-definition.yml
apiVersion: apps/v1
kind: Deployment  # 사실상 이 부분이 ReplicaSet 와의 차이점
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx
```

- Namespace : 단일 클러스터 내에서 리소스 그룹을 서로 나누고 분리시키는 것으로, 클러스터 자원을 여러 사용자 사이에서 나누는 방법.  
  이때, 리소스의 이름은 네임스페이스 내에서 유일해야 하며, 전체 네임스페이스를 통틀어서 유일할 필요는 없음.  
  이러한 리소스 범위 지정은 네임스페이스 기반 오브젝트(Deployment, Service 등)에만 적용되며,  
  Node, Persistent Volume 등의 클러스터 단위 오브젝트에는 적용 불가.  
  (처음엔 `default` 네임스페이스가 기본적으로 세팅됨.)

  - 현재 요청에 네임스페이스 설정 방법 예시  
    ```
    kubectl run nginx --image=nginx --namespace=<namespace-name>
    kubectl get pods --namespace=<namespace-name>
    ```
  - 네임스페이스 컨텍스트 지정 방법 예시
    ```
    kubectl config set-context --current --namespace=<namespace-name>
    ```
# 2. Configuration
- ConfigMap : 키-값 쌍으로 데이터를 저장하는데 사용하는 API 오브젝트로, 파드는 볼륨에서 환경 변수, 커맨드-라인 인수 또는 구성 파일로 컨피그맵을 사용할 수 있음.
  - ConfigMap 생성 예시(kubectl 명령어)
  ```
  kubectl create configmap <config-name> --from-literal=<key>=<value> # key-value 값 직접 지정하여 생성  
  kubectl create configmap <config-name> --from-file=<path-to-file>   # 파일을 직접 지정하여 생성  
  kubectl create configmap <config-name> --from-file=<directory>/     # 디렉토리를 직접 지정하여 생성(디렉토리 내 파일은 key, 파일 내용은 value가 됨)  
  ```
    
  - yml 파일을 활용한 ConfigMap 생성 및 적용 예시
  ```
  # config-map.yml 파일 정의
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    player_initial_lives: "3"
    ui_properties_file_name: "user-interface.properties"
  ```  
  ```  
  # pod-definition.yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
    labels:
      app: myapp
  spec:
    containers
    - name: nginx-container
      image: nginx
  envFrom:
    - configMapRef: # ConfigMap 적용 
      name: app-config
  ```

- Secret : 컨테이너에서 사용하는 password, auth token, ssh key와 같은 중요 민감 정보들을 저장하고 base64로 인코딩해서 관리.
  민감하지 않은 일반 설정 데이터는 ConfigMap을 사용하고, 민감한 정보는 Scret 사용.  
  - Secret 생성 예시(kubectl 명령어)  
  ```
  kubectl create secret generic <secret-name> --from-literal=<key>=<value>
  kubectl create secret generic <secret-name> --from-file=<path-to-file>
  ```
  - yml 파일을 활용한 Secret 생성 및 적용 예시
  ```
  # secret.yml
  apiVersion: v1
  kind: Secret
  metadata:
    name: mysecret
  data:  # 이 값은 Secret 생성 후 base64로 인코딩되어 관리됨
    USER_NAME: root
    PASSWORD: pswrd
  ```  
  ```
  # pod-definition.yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
    labels:
      app: myapp
  spec:
    containers:
    - name: nginx-container
      image: nginx
  envFrom:
    - secretRef: # Secret 적용
      name: mysecret
  ```  
- Service Account : 파드의 일부 컨테이너에서 실행되는 애플리케이션 프로세스(젠킨스, 프로메테우스 등)이 클러스터의 API 서버에 인증하기 위해 사용.  
  - Service Account 생성 예시  
   `kubectl create serviceaccount <sa-name>`  
  쿠버네티스는 파드에 Service Account를 명시하지 않을 경우 기본적으로 `default` Service Account와 그 토큰이 볼륨으로 마운트 됨.  
  쿠버네티스 v1.32을 포함한 최신 버전에서는  API 자격 증명들은 TokenRequest API를 사용하여 직접 얻거나 Projected Volume을 사용하여 파드에 마운트할 수 있음.  
  (단, 이 방법으로 취득한 토큰은 시간 제한이 있으며, 마운트 되었던 파드가 삭제되는 경우 자동으로 만료됨.)   
  만약 만료되지 않는 토큰이 필요한 경우, 다음과 같이 해당 서비스어카운트를 참조하는 어노테이션을 갖는  
 `kubernetes.io/service-account-token` 타입의 시크릿을 생성.  
  ```
  apiVersion: v1
  kind: Secret
  type: kubernetes.io/service-account-token
  metadata:
    name: mysecretname
    annotations:
      kubernetes.io/service-account.name: myserviceaccount # 서비스 계정과 연결
  ```  
- Resource Requirements : 파드가 사용하는 노드의 리소스 양을 제약 및 관리.
  - Pod별 리소스 관리  
    ```
    # pod-definition.yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
      labels:
        app: myapp
    spec:
      containers:  # 리스트 형식
      - name: nginx-container
        image: nginx
        resources:
          requests:
            memory: "1Gi"
            cpu: 1
          limits:
            memory: "2Gi"
            cpu: 1
    ```  
  - `Resource Quota`를 사용한 네임스페이스 단위 리소스 관리  
    ```
    # resource-quota.yml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: my-resource-quota
    spec:
      hard:
        requests.cpu: 1
        requests.memory: 1Gi
        limits.cpu: 2
        limits.memory: 2Gi
        requests.nvidia.com/gpu: 4
    ```

- Taints & Tolerations : 워커 노드에 Taint가 설정된 경우, 동일 값의 Toleration이 있는 파드만 해당 워커 노드에 배치 가능.  
  Toleration이 있는 파드는 동일한 Taint가 있는 노드를 포함하여 모든 노드에 배치 가능.  
  가령, `kubectl create deployment testdep --image=nginx --replicas=5` 명령어를 통해 파드를 생성한 경우,  
  오직 워커노드에만 배치되고 절대 마스터 노드에는 배치되지 않는데,  
  그 이유는 마스터 노드에는 `node-role.kubernetes.io/master:NoSchedule` 라는 Taint가 설정되어 있기 때문.  
  - Taints 생성 예시  
    `kubectl taint nodes node1 key1=value1:NoSchedule`  
  - Taints 제거 예시  
    `kubectl taint nodes node1 key1=value1:NoSchedule-`
  - Tolerations 적용 예시
    ```
    # pod-definition.yml
    apiVersion: v1 
    kind: Pod
    metadata:
      name: my-pod
      labels:
        app: myapp
    spec:
      containers: 
      - name: nginx-container
        image: nginx
      tolerations: # tolerations 적용
      - key: "key1"
        operator: "Equal" # Taint가 '일치'하는 노드에 배치 가능
        value: "value1"
        effect: "NoSchedule"
    ```

  - effect 종류
  1) NoSchedule : toleration이 맞지 않으면 배치되지 않음.  
  2) PreferNoSchedule : toleration이 맞지 않으면 배치되지 않으나, 클러스터 리소스가 부족하면 배치됨.  
  3) NoExecute : toleration이 맞지 않으면 동작중인 파드를 종료시키고 해당 Taint 노드에 스케줄시키지 않음.  

- Node Selector & Node Affinity  
  - Node Selector : 파드가 특정 노드에만 할당되도록 설정  
  `kubectl label nodes <node-name> color=blue # 노드의 레이블 생성`  
    ```
    # pod-definition.yml
    apiVersion: v1 
    kind: Pod
    metadata:
      name: my-pod
      labels:
        app: myapp
    spec:
      containers: 
      - name: nginx-container
        image: nginx
      nodeSelector:
        color: blue # 노드의 레이블을 통해 선택
    ```
  => 단, 이러한 Node Selector는 오로지 명시한 레이블이 있는 노드만 선택이 가능하다는 제약이 있으며,  
     특정 파드가 여러 노드들 중 어느 하나라도 배치될 수 있게끔 하거나,  
     혹은 특정 노드를 제외하고 다른 노드에는 모두 배치 가능하게끔 하는 등 복잡한 노드 배치 조건으로써는 부족함.  
    
  - Node Affinity : `nodeSelector`처럼 노드의 레이블을 기반으로 파드가 스케줄링될 수 있는 노드를 제한할 수 있지만,  
    `nodeSelector`보다 다양한 제약 조건 사용 가능.  
    ```
    # pod-definition.yml
    apiVersion: v1 
    kind: Pod
    metadata:
      name: my-pod
      labels:
        app: myapp
    spec:
      containers: 
      - name: nginx-container
        image: nginx
      affinity:  
        nodeAffinity:  # Node Affinity 적용
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In  # NotIn, Exists 조건도 있음.
                values: # blue 또는 red 중에서는 어느 것에 배치되어도 상관 없음
                - blue
                - red
    ```
    - Node Affinity 종류  
    1) `requiredDuringSchedulingIgnoredDuringExecution` : 조건이 만족되지 않으면 스케줄러는 파드를 아예 배치하지 않음.  
    2) `preferredDuringSchedulingIgnoredDuringExecution` : 스케줄러가 최대한 조건에 맞는 노드를 찾되,  
       해당되는 노드가 없으면 파드를 아무 노드에 배치해도 됨.  
    => 단, 두 종류 모두 파드가 이미 실행중인 경우엔 Node Affinity가 중간에 바뀌어도 변화가 적용되지 않음.(`IgnoredDuringExecution`)  

# 3. Multi-Container Pod  
보통 하나의 파드엔 하나의 컨테이너가 포함되는 것이 일반적이지만, 가령 웹서비스와 로깅 서비스처럼 여러 컨테이너들이  
서로 긴밀하게 상호작용하는 경우처럼 여러 개의 컨테이너가 하나의 파드에 함께 포함되어 서로 자원을 공유하고 협력하는 구조.  
  
- 특징
   - 네트워크 공유 : 파드 내 컨테이너는 같은 IP/Port 주소 공유. 따라서, 각 컨테이너는 `localhost`를 통해 서로 통신 가능.  
   - 스토리지 공유 : 파드 내의 컨테이너는 쿠버네티스 볼륨을 사용하여 동일한 디스크 공간을 공유 가능.  
   - 같은 수명주기 공유
  
- yml을 활용한 생성
  ```
  # pod-definition.yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
      labels:
        app: myapp
    spec:
      containers: # 멀티 컨테이너
      - name: nginx-container
        image: nginx
  
      - name: log-agent
        image: log-agent
  ```
  
- 패턴
  - Sidecar 패턴 : 하나의 주요 애플리케이션이 실행되고, 그 애플리케이션을 보조하는 컨테이너가 함께 실행될 때 사용.   
  - Adaptor 패턴 : 애플리케이션이 직접 네트워크 통신을 처리하지 않고, 별도의 프록시 컨테이너를 통해 트래픽을 처리하는 경우에 사용.  
  - Ambassador 패턴 : 메인 컨테이너의 데이터를 변환하거나 처리하는 중간 단계 컨테이너가 필요한 경우에 사용.  

# 4. Observability  
- Probe : kubelet에 의해 주기적으로 수행되는 컨테이너 상태를 체크하기 위한 진단.  
  - Readiness Probe : 해당 컨테이너가 트래픽을 수용할 수 있는 상태인지 진단.  
    가령, 특정 애플리케이션의 경우 Pod가 생성되어 Running 상태일지라도, 정상적으로 앱 서비스가 띄워져서 트래픽을 수용할 수 있기까지  
    몇 분 이상 걸릴 수도 있으며 이런 경우에 서비스에서 해당 Pod로 트래픽을 보내도 바로 응답을 받지 못할 수 있음.  
    따라서, Readiness Probe에 응답이 없을 경우 앱 구동 순간에 트래픽이 흐르지 않게 해당 파드를 아예 서비스에서 제외시킴.  
  - Liveness Probe : 해당 컨테이너를 재기동 해야하는 상태인지 주기적으로 진단. 만약 컨테이너가 응답이 없다면 해당 컨테이너를 재기동.  

  - Probe 방식  
    1) HTTP Test : Http Get을 통해 컨테이너 상태 진단.  
    2) TCP Test : 지정된 포트에 TCP 연결을 시도하여 상태 진단.  
    3) Exec Command : 쉘 명령을 수행하여 그 결과에 따라 컨테이너 상태 진단.  

  - Probe 옵션  
    1) initialDelaySeconds : 최초 Probe 수행까지의 지연 시간  
    2) periodSeconds : Probe를 수행하는 시간 간격  
    3) timeoutSeconds : 진단 결과를 대기하기까지의 시간  
    4) successThreshold : 몇 번의 성공 결과를 수신받아야 정상 상태로 판단할 것인지 (default : 1번)  
    5) failureThreshold : 몇 번의 실패 결과를 수신받아야 실패 상태로 판단할 것인지 (default : 3번)  
  
  ```
  # pod-definition.yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx-container
        image: nginx
      ports:
      - containerPort: 8080
      livenessProbe:
        httpGet: # Http Test(Get)
          path: /api/healthy
          port: 8080
        initialDelaySeconds: 5 # 최초 Probe는 컨테이너 start 후 5초 뒤에 수행
        periodSeconds: 10      # 10초 간격으로 Probe 수행
        failureThreshold: 3    # 3번 실패 응답읇 받으면 실패 상태로 간주
  ```  

- Container Logging : Pod 내의 특정 컨테이너 로그 확인 방법  
`kubectl logs -f <pod-name> <container-name>`
  
# 5. Pod Design  
- Rolling Updates & Rollback : 파드 인스턴스를 점진적으로 새로운/이전 것으로 업데이트하여  
  Deployment 업데이트가 서비스 중단 없이 이루어질 수 있도록 함.  

  - Rolling Update 방법 1.(yml 파일 수정 후 적용)  
    ```
    # deploy-definition.yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx-container
            image: nginx:1.7.0  # -> nginx:1.7.1 으로 이미지 버전 변경
    ```
    `kubectl apply -f deploy-definition.yml # deploy-definition.yml 내용 수정 후 적용`  

  - Rolling Update 방법 2.(set image 명령어 사용)  
  `kubectl set image deployment/nginx-deployment nginx-container=nginx:1.7.1`

  - Rollback 방법  
  `kubectl rullout undo deployment/nginx-deployment`  
    
  - Rolling Update 상태 조회 방법  
  `kubectl rollout status deployment/nginx-deployment # 현재 상태 확인`  
  `kubectl rollout history deployment/nginx-deployment # 과거 이력 확인`  

- Deployment Strategy : Rolling Update는 구버전과 새버전이 공존하는 시간이 발생한다는 단점이 존재하며,  
  이러한 단점을 극복하기 위한 두 가지 전략 존재.  
  - Blue Green 배포 : 기존 버전(Blue)과 새 버전(Green)이 병렬로 실행된 다음 새 버전 테스트 후  
    이슈가 없으면 모든 트래픽을 새 버전으로 이동하는 방식의 배포 전략.  
    만약 다음과 같은 blue/green deployment와 service가 생성되어 있을 경우,  
    ```
    # myapp-blue.yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-blue
      labels:
        app: myapp
        type: front-end
    spec:
      template:
        metadata:
          name: myapp-pod
          labels:
            version: v1
        spec:
          containers:
          - name: app-container
            image: myapp-image:1.0
      replicas: 5
      selector:
        matchLabels:
          version: v1
    ```
    ```
    # myapp-green.yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-green
      labels:
        app: myapp
        type: front-end
    spec:
      template:
        metadata:
          name: myapp-pod
          labels:
            version: v2
        spec:
          containers:
          - name: app-container
            image: myapp-image:2.0
      replicas: 5
      selector:
        matchLabels:
          version: v2
    ```
    ```
    # serveice-definition.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      selector:
        version: v1  # v2 로 바꿔주면 blue -> green으로 트래픽 전환   
    ```
    service에서 seletor를 통해 version 레이블을 v1에서 v2으로 바꿔주면 my-service는 새로 배포된 Pod에만 트래픽을 보내게 됨.  
    게다가 만약 새로 배포된 버전에 문제가 발생한다면, 다시 my-service의 label을 v1로 돌려줌으로써 쉽게 롤백 가능.  
    
  - Canary 배포 : 기존 버전과 새로운 버전을 병렬로 실행하되, 새로운 버전에는 일부의 트래픽만 전달하여  
    초기 테스트로 일부 사용자에게 점진적으로 롤아웃하고, 새로운 버전에 이슈가 없으면 모든 트래픽을 새 버전으로 전달하는 전략.
    만약 다음과 같은 old/new deployment와 service가 생성되어 있을 경우,  
    ```
    # myapp-old.yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-old
      labels:
        app: myapp
        type: front-end
    spec:
      template:
        metadata:
          name: myapp-pod
          labels:
            version: v1
            app: front-end
        spec:
          containers:
          - name: app-container
            image: myapp-image:1.0
      replicas: 5
      selector:
        matchLabels:
          version: v1
    ```
    ```
    # myapp-new.yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-new
      labels:
        app: myapp
        type: front-end
    spec:
      template:
        metadata:
          name: myapp-pod
          labels:
            version: v2
            app: front-end
        spec:
          containers:
          - name: app-container
            image: myapp-image:2.0
      replicas: 1  # 파드 인스턴스를 하나만 지정하여 새로운 버전은 아주 일부의 트래픽만 수용하도록 설정
      selector:
        matchLabels:
          version: v2
    ```
    ```
    # serveice-definition.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      selector:
        app: front-end # 두 Deployment 모두에 트래픽 전달
    ```
    my-service에서 seletor를 통해 app 레이블이 front-end인 기존/신규 버전 모두에 트래픽을 전달하되,  
    신규 버전의 경우 replicas 사이즈를 1로 지정함으로써 아주 일부의 트래픽만 신규 버전으로 전달되게끔 설정.  
    이후, 테스트를 통해 새로운 버전에 이슈가 없을 경우 모든 트래픽을 신규 버전으로 전달되게끔 replicas 수정.  

- Job & CronJobs  
  - Job : 쿠버네티스에서는 일반적으로 파드 내 컨테이너가 종료되면 항상 재기동하여 Running 상태를 유지하는데,  
    Batch 작업이나 리포팅같은 경우엔 해당 작업이 완료되면 종료된 상태로 있어야 하는 경우가 많음.  
    이런 경우에 대해 쿠버네티스에서 제공하는 Job은 작업이 완료되는 실행이 중지되는 일회성 작업.
    ```
    # job-definition.yml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: math-add-job
    spec:
      completions: 3  # 해당 Job을 순차적으로 3번 실행
      parallelism: 2  # 해당 Job을 동시에 최대 2개 실행
      # 즉, 위 조건상으로는 job1, job2는 동시 실행, 그 후 두 잡이 완료되면 job3 실행
      template:
        spec:
          containers:
          - name: math-add
            image: perl:5.34.0
            command: ['expr',  '3', '+', '2']
          restartPolicy: Never  # 파드가 정상 종료되면 절대 재기동하지 않음.(필수 지정)
      backoffLimit: 4  # 중간에 실패할 경우 최대 몇 번 재기동 시도할 것인지.
    ```
      
  - CronJob : Job에 크론 스케줄 기능 추가.  
    ```
    # cron-job-definition.yml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: daily-add-job
    spec:
      schedule: "0 9 * * *"
      jobTemplate:  # Job의 spec 내용을 명시
        spec:
          completions: 3 
          parallelism: 2  
          template:
            spec:
              containers:
              - name: math-add
                image: perl:5.34.0
                command: ['expr',  '3', '+', '2']
              restartPolicy: Never  
          backoffLimit: 4 
    ```

# 6. Services & Networking
- Service : 특정 파드 그룹을 묶어서 연결하는 단일진입점을 구성하고(ClusterIP), 이를 외부로 노출시켜(NodePort) 로드밸런싱하는 기능 제공.  
  - ClusterIP : 특정 파드 그룹을 묶어서 연결하는 단일진입점을 구성하며, 클러스터 내부에서만 접근 가능한 IP. (서비스의 default 타입)
    - ClusterIP 타입 서비스 생성 예시
    ![clusterip](https://github.com/user-attachments/assets/8c8551b3-9237-460b-bff2-48abc0ff7fcb)
    ```
    # clusterip-service-def.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: clusterip-service
    spec:
      type: ClusterIP
    ports:
    - targetPort: 80
      port: 80
      selector:
        app: myapp
        type: back-end
    ```  
  - NodePort : 파드 그룹의 포트를 노드 포트와 매핑함으로써 애플리케이션을 클러스터 외부로 노출시킴.  
    ![nodeport](https://github.com/user-attachments/assets/bed2e050-f4ff-4285-9b5e-c42e32beed24)
    - NodePort 타입 서비스 생성 예시
    ```
    # nodeport-service-def.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nodeport-service
    spec:
      type: NodePort
      clusterIP: 10.100.100.200 # ClusterIP를 Wrapping하여 노드포드 생성 
      selector:
        app: myapp
        type: front-end
      ports:
      - port: 80  # 기본적으로 그리고 편의상 `targetPort`는 `port` 필드와 동일한 값으로 설정된다.
        targetPort: 80
        nodePort: 30008 # 기본값: 30000-32767
    ```  
- Ingress : 클러스터 외부의 트래픽을 클러스터 내부의 서비스로 라우팅하는 방법을 제공하는 리소스로,  
  Ingress를 사용하면 도메인 이름 기반의 HTTP 및 HTTPS 라우팅을 설정하여, 외부에서 들어오는 요청을 URL 경로에 따라 특정 서비스로 유도할 수 있음.  
    
  ※ Ingress의 기능  
    1) Service에 외부 URL을 제공  
    2) 트래픽을 로드밸런싱  
    3) SSL인증서 처리  
    4) 도메인 기반 가상 호스팅 제공
    
  ![Ingress](https://github.com/user-attachments/assets/96d19f83-7965-464e-b04b-32f905582648)  
  - `Ingress Controller` : 클러스터에서 `Ingress`가 작동하려면 반드시 하나 이상의 `Ingress Controller`가 실행되고 있어야 함.  
    `Ingress Controller`는 일종의 `Ingress`의 구현체로, `Ingress`가 작동하는 방식은 어느 `Ingress Controller`를 사용하는지에 따라 다름.  
    `Ingress Controller` 솔루션에는 NginX, traefik, Istio 등이 있으며, `Ingress`를 구성하려면 반드시 사전에 `Ingress Controller` 솔루션 이미지를 Deployment로 배포해야 함.  
    ```  
    # Nginx 컨트롤러 설치  
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.0/deploy/static/provider/baremetal/deploy.yaml
    ```
    
  - Ingress Resource : 트래픽을 어떤 방식으로 URL 경로에 따라 라우팅 할 것인지 규칙(Rule)을 명시.  
    ```
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: ingress-login-order
    spec:
      paths:
      - path: /login
        pathType: Prefix
        backend:
          serviceName: login-service
          servicePort: 80
      - path: /order
        pathType: Prefix
        backend:
          serviceName: order-service
          servicePort: 80
    ```  
- Network Policy : 파드가 네트워크상의 다양한 엔티티(파드, 네임스페이스, Ip블록)와 통신할 수 있도록 허용하는 방법 지정.  
  - 파드가 통신할 수 있는 엔티티는 아래와 같이 3개의 조합으로 식별.  
    1) 허용되는 다른 파드(Selector를 통해 지정)  
    2) 허용되는 네임스페이스(Selector를 통해 지정)  
    3) IP 블록(CIDR 범위)  

  ※ 파드에 대한 네트워크 허용 방법의 두 가지 종류  
  - Ingress : 해당 파드로 진입하는 트래픽(in)  
  - Egress : 해당 파드에서 다른 곳으로 나가는 트래픽(out)
  
  ```
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: network-policy
  spec:
    podSelector:
      matchLabels:
        role: db
    policyTypes:
      - Ingress
      - Egress
    ingress:  # 아래 대상들에 대해서만 DB에 대한 Ingress 트래픽 허용
      - from:    # ipBlock과 (namespacaceSelector & podSelector)는 OR 조건
          - ipBlock:            
              cidr: 172.17.0.0/16  
          - namespaceSelector:  # namespacaceSelector 와 podSelector는 AND 조건(같은 - 하위에 있으면 AND 조건)
              matchLabels:
                project: myproject
            podSelector:
              matchLabels:
                role: frontend
        ports:
          - protocol: TCP
            port: 6379
    egress:   # 아래 대상들에 대해서만 DB로부터의 Egress 트래픽 허용
      - to:
          - ipBlock:
              cidr: 10.0.0.0/24
        ports:
          - protocol: TCP
            port: 5978
  ```
  
