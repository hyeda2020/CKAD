# CKAD
리포지토리 설명 : CKAD 자격증 공부 내용 정리  
참고 :  
https://kubernetes.io/ko/  
https://www.udemy.com/course/certified-kubernetes-application-developer/  

# Core Concepts
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
  containers  # 리스트 형식
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
