# IAM

: AWS 리소스에 대한 액세스를 안전하게 제어할 수 있는 웹 서비스. IAM을 사용하여 리소스를 사용하도록 인증(로그인) 및 권한 부여(권한 있음)된 대상을 제어하는 서비스

![img](https://user-images.githubusercontent.com/47243329/128975212-9959c905-8c61-42eb-b893-3bd3113e99ce.PNG)

- **RBAC (Role-Based Access Control)**: Group에다가 Role을 바로 주는 대신에 권한의 논리적 집합으로서 Role을 만들고 Role을 Group이나 User에 연결합니다. 즉 Group은 UserGroup으로 주로 사용하고, Role은 권한의 Group으로 사용하는 것입니다. (간접 계층 추가)

- **principal** : 리소스 기반 정책에서, 보안 주체를 지정합니다. 리소스에 대한 액세스가 허용되거나 거부되는 보안 주체를 지정. 

  - IAM 자격 증명 기반 정책에서는 `Principal` 요소를 사용할 수 없습니다.

- **assume role** : role을 만들어 사용자(서비스)에게 권한을 위임하는 것

- **role** : 사용자가 어떠한 작업을 할때 권한을 획득한다는 의미를 지님. IAM 역할을 만들 때 누가 이 역할을 맡을 수 있는지 담당자(Principal)를 지정해야 하는 신뢰 정책(Trust Policy)이 추가되어야 해당하는 IAM 사용자나 AWS 리소스가 역할을 얻을 수 있다.

- **Policy** : 기존 RBAC 구조를 그대로 유지하면서, 권한을 부여하는 형식으로서 Policy라는 개념이 추가

  ex) 탄약관리 역할은 7내무반 그룹에 부여하되, 그것이 사용자 계급이 간부인지, 병사인지에 따라서 적용이 달리 된다면



- **루트 계정 밑에 사용자 추가하기**

  : IAM Group 생성 -> User 추가 -> Policy 연결 

  콘솔 접근 가능

- **자격 증명 획득**

  : IAM Role 생성 -> Policy 연결 -> 보안 주체, 사용자나 리소스 연결 -> cli나 콘솔이 아닌 곳에서 사용





## 과제

1) A의 aws space에 B가 instance 설치

2) Assume 관련 조건 3개 조사 후 이식

3) 설치한 instance에 jenkins, gitlab 설치

=> 모두 Terraform으로 진행



4) 알아볼 것 : SSM, EC2 크레딧 사양 옵션, flow log, EC2 Instance Connect, 보안그룹 outbound  규칙, SSM, yum, t2(인스턴스 종류)



### 과제 진행 순서

1) A(liivAgile)의 aws space에 B(developer)가 instance 설치

- A Role 생성 : EC2 접근, 생성, VPC, Subnet, RouteTable, InternetGateway, SecurityGroup 생성 권한부여

- B User, Role 생성 : AssumeRole 권한 부여

- B가 자신의 aws config에서 만든 user로 접속

- assume-role 명령어 실행 후 받은 정보로 접속

  ```
  aws sts assume-role --role-arn arn:aws:iam::352682994136:role/developer-role --role-session-name session-test
  ```

- Terraform apply 후 리소스 확인

  

2) 설치한 instance에 jenkins, gitlab 설치

- jenkins, gitlab 설치 script 작성

- connection 정보 입력

  ```
  connection {
      host        = "${aws_instance.public_instance.public_ip}"
      user        = "ec2-user"
      type        = "ssh"
      private_key = "${file("./ssh/###.pem")}"
      timeout     = "2m"
    }
  ```

  

- provisioner 에 작성한 script 넣기

  ```
  provisioner "remote-exec" {
    inline = [
    ...
    ]
   }
  ```

- Terraform apply 후 설치 확인



### 겪었던 문제상황

- 1번 진행 중 만난 오류

  : A의 role에 B를 등록한 상태에서 aws assume-role이 진행되지 않음

  ```
  An error occurred (AccessDenied) when calling the AssumeRole operation: Roles may not be assumed by root accounts.
  ```

  해결 : 

  > Roles can be used by the following:
  >
  > - An IAM user in the same AWS account as the role
  > - An IAM user in a different AWS account than the role
  > - A web service offered by AWS such as Amazon Elastic Compute Cloud (Amazon EC2)
  > - An external user authenticated by an external identity provider (IdP) service that is compatible with SAML 2.0 or OpenID Connect, or a custom-built identity broker.




- principal에 직접 user의 ID를 넣었는데 IAM user로 사용한다고 하여서 B 계정에서 User 생성 후 assume 권한 부여

  

- 에러메세지가 변형되어 나와서 이렇게 진행하여 결과값 받음

  => sts 권한을 iam에 줘야한다

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": "sts:DecodeAuthorizationMessage",
              "Resource": "*"
          }
      ]
  }
  ```

  ```
  $ aws sts decode-authorization-message --encoded-message encoded-message
  ```

  

- local에서 terraform이 실행중이어서 lock이 걸림
  => cpu 프로세스에서 종료
  https://stackoverflow.com/questions/55212864/error-ebusy-resource-busy-or-locked-rmdir

  

- Route Table에 local만 있으면 외부통신 안되니까 꼭 확인하자..아님 죽음뿐..

  

- provisioner "remote-exec" 실행이 안됨

  => connection 정보를 미리 입력해주어야함. 그래야 ssh에 접근해서 설치하지

  ```
  connection {
      host        = "${aws_instance.public_instance.public_ip}"
      user        = "ec2-user"
      type        = "ssh"
      private_key = "${file("./ssh/liiv_agile.pem")}"
      timeout     = "2m"
  }
  ```

  

- GitLab 설치 시 마지막에 오류가 남

  > Error executing action `run` on resource 'bash[migrate gitlab-rails database]'

  [참고페이지]: https://stackoverflow.com/questions/46907157/cannot-install-gitlab-using-omnibus-error-executing-action-run-on-resource-b

  메모리 부족이었을 때 날 수 있다는 정보를 얻고 t2.micro였던 것을 t2.medium으로 변경하니 해결..진짜 공짜좋아하다가 벌받기

  

- EC2는 처음 만들었을 때 아무런 권한이 없기 때문에 IAM을 연결시켜줘야한다

