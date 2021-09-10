# Terraform으로 EKS 구축하기(1)



1. **EKS 소개**
2. **Kubernetes 란**
3. **EKS workshop을 통해 실습**
4. Terraform 작성



### EKS 소개

Amazon Elastic Kubernetes Service(Amazon EKS)는 AWS 클라우드 또는 온프레미스에서 Kubernetes 애플리케이션을 시작, 실행 및 조정할 수 있는 유연성을 제공하는 서비스

![eks통신아키텍쳐](https://user-images.githubusercontent.com/47243329/128975349-c11390ba-05bc-464a-961d-36a300976990.PNG)

- EKS 클러스터가 준비되면, API 엔드포인트 공급

  

### Kubernetes 란

![kubernetes 구조](https://user-images.githubusercontent.com/47243329/128975450-f0f43b90-3f92-4fe1-98d9-b3eb8efe2941.PNG)

 : 쿠버네티스 클러스터를 구성하는 머신들은 **노드(node)** 라고 불립니다. 쿠버네티스 클러스터 내 노드들은 물리적이거나 가상 머신일 수 있습니다.

- **노드의 종류**

  : 콘트롤 플레인(Control Plane)을 형성하며, 클러스터의 “두뇌”로 역할하는 마스터 노드(Master-node).

  - 1개 이상의 API 서버: kubectl 의 REST 진입점
  - etcd: 분산 키/벨류 저장소
  - 콘트롤 매니저: 항상 현재 상태와 원하는 상태 평가
  - 스케줄러: 작업 노드에 파드를 스케줄링

  : 데이터 플레인(Data Plane)을 형성하며, 파드(pod)들을 통해 실제 컨테이너 이미지들을 작동시키는 워커 노드(Worker-node).

  - 작업 노드로 구성

  - kubelet: API 서버와 노드 간에 연결 역할

  - kube-proxy: IP 변환과 라우팅 관리. 노드의 네트워크 규칙을 유지 관리하고 이 네트워크 규칙이 내부 네트워크 세션이나 클러스터 바깥에서 파드로 네트워크 통신을 할 수 있도록 해준다.

    

- **Kubernetes 용어**

  [파드(Pod)](https://kubernetes.io/docs/concepts/workloads/pods/pod/) : 하나 이상의 컨테이너를 둘러싼 가장 작은 래퍼(wrapper) 단위.

  [데몬셋(DaemonSet)](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) : 워커 노드에 파드의 단일 인스턴스를 실행합니다.

  [디플로이먼트(Deployment)](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) : 어플리케이션 버전의 롤아웃(또는 롤백) 방법에 대한 세부내용.

  [레플리카 셋(ReplicaSet)](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) : 계속 동작할 파드의 개수를 정의합니다.

  [잡(Job)](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) : 파드가 제대로 완성(completion)되어 동작할 수 있도록 합니다.

  [서비스(Service)](https://kubernetes.io/docs/concepts/services-networking/service/) : 고정 IP 주소를 파드의 논리 그룹과 매핑합니다.

  [레이블(Label)](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) : 연결(association)과 필터링에 사용되는 키/밸류(Key/Value) 쌍.



### EKS workshop을 통해 실습

: 마이크로서비스 예제 배포

![label, nodeSelector](https://user-images.githubusercontent.com/47243329/128975391-3d0e783b-a48a-42b2-93fa-cf3730fa29ab.PNG)

구조는 두개의 Backend(crystal, nodeJs)와 하나의 Frontend(Ruby)로 구성된다

[nodejs deployment]

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-nodejs
  labels:
    app: ecsdemo-nodejs
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-nodejs
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-nodejs
    spec:
      containers:
      - image: brentley/ecsdemo-nodejs:latest
        imagePullPolicy: Always
        name: ecsdemo-nodejs
        ports:
        - containerPort: 3000
          protocol: TCP
```

[crystal deployment]

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ecsdemo-nodejs
  labels:
    app: ecsdemo-nodejs
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ecsdemo-crystal
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: ecsdemo-crystal
    spec:
      containers:
      - image: brentley/ecsdemo-crystal:latest
        imagePullPolicy: Always
        name: ecsdemo-crystal
        ports:
        - containerPort: 3000
          protocol: TCP
```

[frontend Service]

```
apiVersion: v1
kind: Service
metadata:
	name: ecsdemo-frontend
spec:
	selector:
		app: ecsdemo-nodejs
	type: LoadBalancer
	ports:
	 - protocol: TCP
	   port: 80
	   targetPort: 3000
```
=> CLB로 로드밸런서 생성

[backend Service]

```
apiVersion: v1
kind: Service
metadata:
	name: ecsdemo-nodejs
spec:
	selector:
		app: ecsdemo-nodejs
	ports:
	 - protocol: TCP
	   port: 80
	   targetPort: 3000
```
```
apiVersion: v1
kind: Service
metadata:
	name: ecsdemo-crystal
spec:
	selector:
		app: ecsdemo-crystal
	ports:
	 - protocol: TCP
	   port: 80
	   targetPort: 3000
```

- service 종류를 선택하지 않으면 default는 clusterIP(클러스터 내부 IP를 서비스로 노출. 클러스터 내부에서만 서비스에 접근 가능)
- 이전에 LB를 생성하지않은 AWS 계정인 경우 AWSServiceRoleForElasticLoadBalancing을 추가한다

