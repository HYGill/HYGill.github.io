# Terraform으로 EKS 구축하기(3)



1. EKS 소개
2. Kubernetes 란
3. EKS workshop을 통해 실습
4. Terraform 작성
5. **Istio 설치**
6. Test Application 올린 후 Domain과 API 연결

------

### Istio 설치

- Istio 란 ?  서비스메시 툴

![istio구조](https://user-images.githubusercontent.com/47243329/141933867-ed8eac60-568e-40c6-b288-9ae1eef00893.PNG)

> 배포된 모든 애플리케이션과 함께 프록시 sidecar를 추가하여 Istio를 사용하면 애플리케이션 트래픽 관리, 강력한 보안 기능을 네트워크에 프로그래밍 가능함
>
> envoy를 사용하여 네트워크 흐름을 제어

- **서비스 메시** : 최신 애플리케이션은 일반적으로 분산된 마이크로서비스로 설계되며, 각 마이크로서비스는 개별 비즈니스 기능을 수행합니다. 이때 서비스 메시는 애플리케이션에 추가할 수 있는 전용 인프라 계층으로써 자체 코드에 추가하지 않고도 관찰, 트래픽 관리 및 보안과 같은 기능을 추가할 수 있다.
- **주요기능** 
  1. **교통관리** : 단일 클러스터 내 및 클러스터 간 트래픽 라우팅은 성능에 영향을 미치고 더 나은 배포 전략을 가능하게 합니다
  2. **관찰가능성** : Istio는 서비스 메시 내의 모든 통신에 대한 상세한 원격 측정을 생성
  3. **보안기능** : 마이크로서비스는 메시지 가로채기(man-in-the-middle) 공격에 대한 보호, 유연한 액세스 제어, 감사 도구 및 상호 TLS를 비롯한 특정 보안 요구 사항이 있습니다. Istio에는 운영자가 이러한 모든 문제를 해결할 수 있는 포괄적인 보안 솔루션이 포함되어 있습니다. 강력한 ID, 강력한 정책, 투명한 TLS 암호화, AAA(인증, 권한 부여 및 감사) 도구를 제공하여 서비스와 데이터를 보호합니다



### Istio 설치

- Istio 란 ?  서비스메시 툴

![istio구조](C:\Users\superuser\Downloads\sava\istio구조.PNG)

> 배포된 모든 애플리케이션과 함께 프록시 sidecar를 추가하여 Istio를 사용하면 애플리케이션 트래픽 관리, 강력한 보안 기능을 네트워크에 프로그래밍 가능함. Istio는 Pod에 envoy 를 sidecar 패턴으로 삽입하여, 트래픽을 컨트롤 하는 구조

>
> envoy를 사용하여 네트워크 흐름을 제어

- **서비스 메시** : 최신 애플리케이션은 일반적으로 분산된 마이크로서비스로 설계되며, 각 마이크로서비스는 개별 비즈니스 기능을 수행합니다. 이때 서비스 메시는 애플리케이션에 추가할 수 있는 전용 인프라 계층으로써 자체 코드에 추가하지 않고도 관찰, 트래픽 관리 및 보안과 같은 기능을 추가할 수 있다.
- **주요기능** 
  1. **교통관리** : 단일 클러스터 내 및 클러스터 간 트래픽 라우팅은 성능에 영향을 미치고 더 나은 배포 전략을 가능하게 합니다
  2. **관찰가능성** : Istio는 서비스 메시 내의 모든 통신에 대한 상세한 원격 측정을 생성
  3. **보안기능** : 마이크로서비스는 메시지 가로채기(man-in-the-middle) 공격에 대한 보호, 유연한 액세스 제어, 감사 도구 및 상호 TLS를 비롯한 특정 보안 요구 사항이 있습니다. Istio에는 운영자가 이러한 모든 문제를 해결할 수 있는 포괄적인 보안 솔루션이 포함되어 있습니다. 강력한 ID, 강력한 정책, 투명한 TLS 암호화, AAA(인증, 권한 부여 및 감사) 도구를 제공하여 서비스와 데이터를 보호합니다



- **istio 설치**

  ```bash
  #istioctl 설치
  % curl -sL https://istio.io/downloadIstioctl | sh -
  Downloading istioctl-1.11.4 from https://github.com/istio/istio/releases/download/1.11.4/istioctl-1.11.4-linux-amd64.tar.gz ...
  istioctl-1.11.4-linux-amd64.tar.gz download complete!
  
  Add the istioctl to your path with:
    export PATH=$PATH:$HOME/.istioctl/bin
  
  Begin the Istio pre-installation check by running:
           istioctl x precheck
  
  Need more information? Visit https://istio.io/docs/reference/commands/istioctl/
  
  # istioctl를 path에 추가
  % export PATH=$PATH:$HOME/.istioctl/bin
  
  #istio-opertor 설치
  % istioctl operator init --tag 1.7.5
  Installing operator controller in namespace: istio-operator using image: docker.io/istio/operator:1.7.5
  Operator controller will watch namespaces: istio-system
  ✔ Istio operator installed
  ✔ Installation complete
  
  % k get ns
  NAME              STATUS   AGE
  default           Active   39m
  istio-operator    Active   30s
  istio-system      Active   30s
  kube-node-lease   Active   39m
  kube-public       Active   39m
  kube-system       Active   39m
  
  % helm install install/kubernetes/helm/istio \
  --name istio \
  --namespace istio-system \
  --set tracing.enabled=true \
  --set global.mtls.enabled=true \
  --set grafana.enabled=true \
  --set kiali.enabled=true \
  --set servicegraph.enabled=true
  
  # 네임스페이스에서 자동 Istio 사이드카 삽입 활성화를 위해 namespace에 레이블을 지정
  # 배포를 수동으로 삽입하려면 istioctl kube-inject
  % kubectl label namespace istio istio-injection=enabled
  ```
  
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-istio
    namespace: istio
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
  
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-istio
    namespace: istio
    labels:
      app: nginx
      service: nginx-istio
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
  
  ```yaml
  # 들어온 트래픽을 서비스로 라우팅
  apiVersion: networking.istio.io/v1alpha3
  kind: VirtualService
  metadata:
    name: nginx-virtual
    namespace: istio
  spec:
    hosts:
    - "*"
    gateways:
    - nginx-istio-gateway
    http:
    - match:
      - uri:
          exact: /
      # URL에 따라서 어느 서비스로 라우팅할 지를 정한다
      route:
      - destination: # {domain:port}로 포워딩해서 서비스를 제공
          host: nginx-istio
          port:
            number: 8080
  ```
  
  ```yaml
  apiVersion: networking.istio.io/v1alpha3
  kind: Gateway
  metadata:
    name: nginx-istio-gateway
    namespace: istio
  spec:
    selector:
      istio: ingressgateway # use istio default controller
    servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
      - "*"
  ```
  
   -  Virtual Service 의 http/match/url
      - `exact: "value"` for exact string match
      - `prefix: "value"` for prefix-based match
      - `regex: "value"` for RE2 style regex-based match (https://github.com/google/re2/wiki/Syntax).
  
- 설치 시 nlb로 변경하는 방법

  ```yaml
  # override.yaml
  gateways:
    istio-ingressgateway:
      serviceAnnotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
  ```

  ```yaml
  $ helm template install/kubernetes/helm/istio --namespace istio -f override.yaml > $HOME/istio.yaml
  ```

  설치시 파일을 바꿔주는 것이 필요하다.

------

### Kubernetes 명령어 Alias 하기

- 자동 완성 스크립트를 `~/.bash_profile` 파일에서 소싱한다.

  ```bash
  echo 'source <(kubectl completion bash)' >>~/.bash_profile
  ```

- 자동 완성 스크립트를 `/usr/local/etc/bash_completion.d` 디렉터리에 추가한다.

  ```bash
  kubectl completion bash >/usr/local/etc/bash_completion.d/kubectl
  ```

- kubectl에 대한 앨리어스가 있는 경우, 해당 앨리어스로 작업하기 위해 셸 자동 완성을 확장할 수 있다.

  ```bash
  echo 'alias k=kubectl' >>~/.bash_profile
  echo 'complete -F __start_kubectl k' >>~/.bash_profile
  ```


