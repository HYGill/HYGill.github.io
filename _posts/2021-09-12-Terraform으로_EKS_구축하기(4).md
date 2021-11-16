# Terraform으로 EKS 구축하기(4)



1. EKS 소개
2. Kubernetes 란
3. EKS workshop을 통해 실습
4. Terraform 작성
5. Istio 설치
6. **Test Application 올린 후 Domain과 API 연결**

------

### Nginx 설치

- deployment.yaml

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 2
    selector:                   # deployment가 관리할 pod를 찾는 방법
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:                 # pod의 label
          app: nginx
      spec:
        containers:             # 컨테이너 설정
        - name: nginx
          image: nginx:latest
          ports:
          - containerPort: 80
  ```

  

- service.yaml

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
    labels:
      run: nginx
  spec:
    type: NodePort     # 서비스 타입
    ports:
    - port: 8080       # 서비스 포트
      targetPort: 80   # 타켓, 즉 pod의 포트
      protocol: TCP
      name: http
    selector:
      app: nginx
  
  ```

  

  이렇게 했을 시 pod가 service에 잘 연결된 것을 확인할 수 있었다.

  ```bash
  ubuntu@ip-10-0-27-0:~/nginx$ k get pod -o wide
  NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES
  nginx-deployment-585449566-6s92g   1/1     Running   0          139m   10.0.230.98   ip-10-0-226-225.ap-northeast-2.compute.internal   <none>           <none>
  nginx-deployment-585449566-8n9dm   1/1     Running   0          139m   10.0.249.95   ip-10-0-249-77.ap-northeast-2.compute.internal    <none>           <none>
  
  ubuntu@ip-10-0-27-0:~/nginx$  k describe service nginx-service
  Name:                     nginx-service
  Namespace:                default
  Labels:                   run=nginx
  Annotations:              <none>
  Selector:                 app=nginx
  Type:                     NodePort
  IP Families:              <none>
  IP:                       172.20.87.16
  IPs:                      172.20.87.16
  Port:                     http  8080/TCP
  TargetPort:               80/TCP
  NodePort:                 http  30041/TCP
  Endpoints:                10.0.230.98:80,10.0.249.95:80
  Session Affinity:         None
  External Traffic Policy:  Cluster
  Events:                   <none>
  
  ```

  

  그리고 pod들은 각각의 worker node에서 running 중임을 볼 수 있음!

  ```bash
  ubuntu@ip-10-0-27-0:~/nginx$ k get nodes
  NAME                                              STATUS   ROLES    AGE     VERSION
  ip-10-0-226-225.ap-northeast-2.compute.internal   Ready    <none>   3h33m   v1.20.11-eks-f17b81
  ip-10-0-249-77.ap-northeast-2.compute.internal    Ready    <none>   3h33m   v1.20.11-eks-f17b81
  
  ubuntu@ip-10-0-27-0:~/nginx$ k describe pod | grep Node
  Node:         ip-10-0-226-225.ap-northeast-2.compute.internal/10.0.226.225
  Node-Selectors:  <none>
  Node:         ip-10-0-249-77.ap-northeast-2.compute.internal/10.0.249.77
  Node-Selectors:  <none>
  ```



### Nginx 설치

nginx가 잘 띄워져있는지 확인해보자

- worker node용 instance에 연결되어있는 보안그룹

  - eks-cluster-sg-eks-cluster-~~ : cluster용 SG
  - eks-remoteAccess-~~ : Worker node용 SG

  

```bash
# eks-cluster-sg-eks-cluster SG에 접근 허용해주기
# nodeport로 연결된 port를 열어줘야함
ubuntu@ip-10-0-27-0:~$ curl http://{worker node INTERNAL-IP}:30041
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and ...
```



성공~

### LB 연결

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-subnets: {public-subnet-id}
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 8080
      targetPort: 80
      protocol: TCP
      name: http
```

nodePort service 삭제한 후 다시 저 yaml으로 apply하면 aws에 nlb가 생긴 후에 연결된다

![nlb성공](https://user-images.githubusercontent.com/47243329/141934007-1045a7c1-6aea-42c4-b237-900edb0e5e3b.PNG)

### Nginx Ingress Controller

: ingress-nginx는 [NGINX](https://www.nginx.org/) 를 역 프록시 및 로드 밸런서로 사용하는 Kubernetes용 Ingress 컨트롤러

service마다 loadbalancer가 붙는다면 많은 자원이 사용되므로 nginx-controller에서 규칙으로 lb를 컨트롤해보자!

```bash
# ingress-nginx 설치
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.4/deploy/static/provider/aws/deploy.yaml

# 설치 확인
ubuntu@ip-10-0-27-0:~$ k get svc -n ingress-nginx
NAME TYPE  CLUSTER-IP  EXTERNAL-IP  PORT(S)  AGE
ingress-nginx-controller  LoadBalancer   172.20.206.245   aa91e3602f5b4455eaed779f0ac70ec2-8177385d9f4c1cec.elb.ap-                           northeast-2.amazonaws.com   80:32317/TCP,443:31109/TCP   22s
ingress-nginx-controller-admission   ClusterIP      172.20.208.78    <none>               443/TCP                      23s

```

ingress-nginx를 설치하면 aws 유형 NLB가 설치된다.

```yaml
#deployement.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: test
  labels:
    app: nginx
spec:
  replicas: 2
  selector:                   
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:              
        app: nginx
    spec:
      containers:          
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```



```yaml
#service.yamlapiVersion: v1kind: Servicemetadata:  name: nginx-service  namespace: testspec:  selector:    app: nginx  ports:    - port: 8080 #  Cluster 내부에서 사용할 Service 객체의 포트      targetPort: 80 # Service객체로 전달된 요청을 Pod(deployment)로 전달할때 사용하는 포트      protocol: TCP   
```



```yaml
#ingress.yamlapiVersion: extensions/v1beta1kind: Ingressmetadata:  name: nginx-ingress  namespace: test  annotations:  	# 어떤 인그레스 컨트롤러를 사용하는지 표시    kubernetes.io/ingress.class: nginx    # 백엔드 서비스의 노출된 URL이 수신 규칙의 지정된 경로와 다를 시 서비스에서 예상하는 경로로 설정하기 위한 annotations    # nginx.ingress.kubernetes.io/rewrite-target: /spec:  rules:  - host: {hostDomain}    http:      paths:      - path: /        backend:          serviceName: nginx-service          servicePort: 8080
```



이렇게 yaml을 만들고 apply를 하면 {hostDomain}에서 화면을 확인 가능하다!



------

### SSH와 Port

- SSH(Secure Shell) : 원격지 호스트 컴퓨터에 접속하기 위해 사용되는 인터넷 프로토콜

- Port : 인터넷 프로토콜 스위트에서 **포트**(port)는 **운영 체제 통신의 종단점**이다. 이 용어는 하드웨어 장치에도 사용되지만, 소프트웨어에서는 네트워크 서비스나 특정 프로세스를 식별하는 논리 단위이다.

  

### Kubernetes

- 컨테이너화된 애플리케이션을 자동으로 배포, 스케일링 및 관리해주는 오픈소스 시스템
- 컨테이너들을 논리적인 단위로 그룹화
- 쿠버네티스는 분산 시스템을 탄력적으로 실행하기 위한 프레임 워크를 제공한다. 애플리케이션의 확장과 장애 조치를 처리하고, 배포 패턴 등을 제공
