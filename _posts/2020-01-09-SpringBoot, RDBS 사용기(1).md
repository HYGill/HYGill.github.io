# Serverless프로젝트 수정사항(SpringBoot, RDBS 사용기)

**프로젝트 개요**

Communication App을 만드는 프로젝트에서 서버를 구현했었다.. 
처음 프로젝트 개요는 서버리스로 구현하는 것이어서 lambda와 dynamodb를 사용하였는데 그 것을 보완하는 과정을 포스팅하려고 한다.
이 프로젝트는 차장님과 페어코딩으로 진행되어서 제대로 서버의 구조를 배울 수 있는 좋은 기회가 될 것 같다!!

## 현재 프로젝트의 문제점
* DynamoDB를 사용하기로 결정되어 진행해보니 Dynamo는 sorting기능을 구현할 때 복잡성을 피할 수 없게 되어 있는 것을 알았다.
기존 프로젝트는 DynamoDB 에서 별도의 index 테이블을 생성하여 정렬하고 기존 content를 Mapping하는 방법을 택하였는데, likeHash별로 sorting 한 후 그 sorting 된 결과를 원래 content 와 mapping 하는 과정에서 시간이 많이 소요되는 것으로 판단되었다.

* DynamoDB에서 sorting을 사용하기 위해서는 **primary key와 sorting key**의 사용이 불가피한데 다른 기능을 구현할 때 sorting key를 지정한 Table에 접근하려면 저 pk sk 두개의 key가 필요하다. 그래서 sorting key를 따로 저장하는 Table을 구성하여서 key값을 꺼내 온 다음 접근을 해야한다. 
이 과정이 필요하다고 생각했기 때문에 이 Table이 필요한(content Table이니 거의 모든) 기능을 구현할 때 **복잡성이 추가**된다.

> 결론 : DynamoDB를 사용하지 않는 것이 좋겠다.


## 프로젝트의 재구성
* 새로운 Database 선택 과정
처음에 선택해야 할 것은 NoSQL형태 그대로 할 것이냐 RDBS로 바꿀 것 이냐 였다. 후보군은
	1. RDBS: 제일 빨리 만들 수 있으나 속도 문제 미지수  
	2. DocumentDB: 속도 매우 빠르고 기존과 다르게 sorting,max,min 등 함수를 제공한다. 
그러나 Aggeration 이라는 Function이 DB프로시져 같은 느낌이라 데이터 분석 용도로 짜는거 아니면 잘 안한다고 한다. 

	이러한 특징을 가지고 있는 두가지에서 고민을 하였고, 결과로 목표 성능이 천만개인 메가통 서버에서 RDBS 속도는 별로 문제가 되지 않을 것이라고 판단하였다. 또한 DocumentDB를 하게되면 러닝커브가 너무 많아 기한 안에 끝내지 못할 것을 고려해서 RDBS로 교체하기로 했다. 
	
---
*   DB의 종류를 바꾸면서 바뀌는 점
	
	**1. 람다함수를 사용하지 못하게 되었다.**
	
	NoSQL은 쿼리를 쏘고 Response를 응답받는 stateless로 사용되는데 RDBS는 소켓통신을 해서 미리 커넥션을 할 후 쿼리를 보내고 응답받는 형태이다. 만일 람다를 계속 사용한다면 람다는 사용할때만 켜져있는 프로세스라서 소켓 커넥션이 유지될 수 없다. 그렇다면 실행될때마다 커넥션을 새로 생성하고 삭제해야하는데 이러한 과정을 RDBS가 버티지 못할 것이라고 판단했기 때문이다. 
		
	이것을 배울 때 한가지 새로운 점을 알게 되었는데, 이전에 소켓통신 실습을 했을 때는 당근 가상의 선에 연결되는 것이라고 생각했는데 사실 네트워크로 통신을 할 때마다 .sock파일이 생성되고 주고받는 패킷이 여기에 write되어 새로운 파일이 생성되어 계속 write하면 cpu자원을 많이 소모하게 된다는 점이다. 
		
	이렇기 때문에 RDBS는 람다를 사용할 수 없다는 판단하에 그냥 SpringBoot로 개발하기로 하였다.
	
	**2. 랭킹순 sorting을 실시간 반영이아닌 배치형태로 진행하기로 하였다.** 
	

# 마무리
설계와 앞으로 나아갈 점에 대해서 배웠으니 정말 스프링부트를 사용하여 서버를 개발하는 걸 다음에 배울 예정이다
그 과정을 앞으로 적어나가려고 한다.
