# Terraform으로 EKS 구축하기(2)



1. EKS 소개
2. Kubernetes 란
3. EKS workshop을 통해 실습
4. **Terraform 작성**



### Terraform 작성

(전체 소스 링크는 하단에 첨부)

EKS를 구성할 때 필요한 것은 기본 VPC를 이루는 구성요소가 모두 필요하다.

또 Bastion방식으로 eks에 접속할 것이라면 거기에 EC2도 따로 띄워야함 

두가지 방식으로 모두 해봤는데 굳이 bastion으로 접속할 필요가 없어서 뺐다.



그리고 eks 모듈을 사용했음

[eks module]: https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest

처음 다운받아서 소스보고 복잡해서 속상했지만 examples폴더에 사용 예시가 정말 친절하게 나와있다. 그리고 나머지 필요한 옵션들은 이어져있는 모듈 계속 타고들어가면서 설정하면 될듯!



```
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  cluster_name = local.cluster_name
  vpc_id      = "${aws_vpc.eks_vpc.id}"
  subnets     = ["${aws_subnet.eks_private_subnet.id}", "${aws_subnet.eks_private_subnet2.id}"]
  cluster_version = "1.20"
  cluster_endpoint_public_access_cidrs = ["내 PC의 IP", "${aws_nat_gateway.eks_nat.public_ip}/32"]

  node_groups = {
      eks_nodes = {
          desired_capacity = 2
          max_capacity     = 5
          min_capacity     = 2
          key_name         = aws_key_pair.public_bastion.key_name
          instance_types = ["t3.small"]
      }
    }
  manage_aws_auth = false
}
```

노드 그룹이 자꾸 안 생성되어서 도대체 뭘까했는데 **cluster_endpoint_public_access_cidrs** 설정을 안해주었었다.
examples 폴더에서 basic 예시에는 없는데 까보니 설정할 수 있는 값이어서 넣어주었음.. 

Bastion없이 내 PC에서 직접 접근할 것이기 때문에 PC IP랑 nat gateway의 ip를 넣어주었다. 



생성 성공~

------

베스쳔 있는 eks: https://github.com/HYGill/sava/tree/main/terraform-bastion-eks

instance만들어서 cluster_endpoint_public_access_cidrs에 인스턴스 ip 넣어주면된다.

베스쳔 없는 eks: https://github.com/HYGill/sava/tree/main/terraform-eks
