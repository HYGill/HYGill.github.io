# Introduction to Infrastructure as Code with Terraform

------

#### Terraform은 Hashicorp에서 오픈소스로 개발중인 인프라스트럭처 관리 도구입니다. 

서비스 실행에 필요한 환경을 구축하는 도구라는 점에서 프로비저닝 도구로 분류됩니다. 테라폼은 코드로서의 인프라스트럭처 Infrstructure as Code를 지향하고 있는 도구로서, GUI나 웹 콘솔을 사용해 서비스 실행에 필요한 리소스를 관리하는 대신 필요한 리소스들을 선언적인 코드로 작성해 관리할 수 있도록 해줍니다.



#### Introduction to Infrastructure as Code with Terraform

- Terraform은 멀티 클라우드 플랫폼을 관리할 수 있다.

- 사람이 읽을 수 있는 언어는  infrastructure code를 빨리 작성할 수 있게 도와준다.

- Terraform의 상태를 사용하면 배포 전반에 걸쳐 리소스 변경을 추적 할 수 있습니다.

- 구성을 버전 제어에 적용하여 인프라에서 안전하게 협업 할 수 있습니다.

  

#### Standardize your deployment workflow

- 공급자는 컴퓨팅 인스턴스 또는 사설 네트워크와 같은 인프라의 개별 단위를 리소스로 정의

- 다양한 공급자의 리소스를 모듈이라는 재사용 가능한 Terraform 구성으로 구성하고 일관된 언어와 워크플로로 관리

- Terraform의 구성 언어는 선언적이기 때문에 원하는 최종 상태를 설명

  

  To deploy infrastructure with Terraform:

  - **Scope** - Identify the infrastructure for your project.
  - **Author** - Write the configuration for your infrastructure.
  - **Initialize** - Install the plugins Terraform needs to manage the infrastructure.
  - **Plan** - Preview the changes Terraform will make to match your configuration.
  - **Apply** - Make the planned changes.



#### Track your infrastructure

상태 파일에서 실제 인프라를 추적



#### Collaborate

애플리케이션 코드 에서처럼 버전 제어를 통해 인프라 변경을 관리
