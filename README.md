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
  `kubectl label nodes <node-name> color=blue`  
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
