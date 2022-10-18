# ElasticSearch 맛보기

좋은 기회로 ElasticSearch 세미나에 가게되었다! 

근데 아무것도 아는 것이 없어서 가기 전에 기본적인 것은 알아놔야 한다는 생각에 정리해보기

------



- #### Elastic Search 란?

  Elasticsearch는 시간이 갈수록 증가하는 문제를 처리하는 분산형 RESTful 검색 및 분석 엔진

  

- #### 특징

  1. Apache Lucene을 기반으로 구축

  2. JSON 문서로 데이터를 저장

  3. 클러스터 상태 확인, 인덱스에 대한 CRUD(생성, 읽기, 업데이트 삭제) 및 검색 작업 실행, 필터링 및 집계 같은 고급 검색 작업 수행을 위한 종합적이며 강력한 [REST API 세트](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html) 사용

  4. 역 인덱스라고 하는 데이터 구조를 사용

     -> 문서에 나타나는 모든 고유한 단어의 목록을 만들고, 각 단어가 발생하는 모든 문서를 식별

  5. 저장된 문서는 샤드라고 하는 여러 다른 컨테이너에 걸쳐 분산되며, 이 샤드는 복제되어 하드웨어 장애 시에 중복되는 데이터 사본을 제공

  6. Kibana에서 사용자는 데이터를 강력하게 시각화하고, 대시보드를 공유

  7. Logstash는 데이터를 집계하고 처리하여 Elasticsearch로 전송하는 데 사용. 서버 사이드 오픈 소스 데이터 처리 파이프라인

- #### 용어 정리

  1. Cluster : Elasticsearch에서 가장 큰 시스템 단위이고, 최소 하나 이상의 노드로 이루어진 노드들의 집합

     서로 다른 클러스터는 데이터의 접근, 교환을 할 수 없는 독립적인 시스템으로 유지

  2. Node : Elasticsearch를 구성하는 하나의 단위 프로세스

     <노드의 종류>

     1. **[Master-eligible node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#master-node)**

        클러스터를 제어하는 마스터노드

     2. **[Data node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#data-node)**

        데이터 노드는 데이터를 보유하고 CRUD, 검색 및 집계와 같은 데이터 관련 작업을 수행

     3. **[Ingest node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#node-ingest-node)**

        수집 노드는 인덱싱 전에 문서를 변환하고 강화하기 위해 수집 파이프라인 을 문서에 적용. master나 data role이 없는 노드를 선정하는 것이 좋다

     4. **[Remote-eligible node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#remote-node)**

        원격 클라이언트로 작동할 수 있는 역할 이 있는 노드

     5. **[Machine learning node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#ml-node)**

        기계 학습 기능을 사용하려면 클러스터에 기계 학습 노드가 하나 이상 있어야함

     6. **[Transform node](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#transform-node)**

        데이터변환 노드

  3. Shard(샤드) : Elasticsearch에서 스케일 아웃을 위해 index를 여러 shard로 쪼갠 것

  4. Replica : 노드를 손실했을 경우 데이터의 신뢰성을 위해 샤드들을 복제. 서로 다른 노드에 있는 것을 권장

     -> primary Shard(데이터를 저장할 때 나눠진 파티션)와 replica Shard(primary shard의 복제본)는 같은 노드에 존재할 수 없음

     ![img](https://t1.daumcdn.net/cfile/tistory/99AB08425C9F17D928)

[출처](https://github.com/exo-archives/exo-es-search)

- #### Docker로 ElasticSearch 띄워보기



```shell
-- ElasticSearch 이미지 다운
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.4.3

-- 네트워크 생성(kibana를 이 네트워크에 설치할)
docker network create elastic

-- 이미지 실행
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.4.3

-- 이미지 실행 시 출력되는 로그를 꼭 꼭 다른 파일에 복사해놓자. kibana 실행시 토큰 필요
-> Elasticsearch security features have been automatically configured!
-> Authentication is enabled and cluster connections are encrypted.

->  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  ~~~

-- 컨테이너에서 로컬로 인증서 복사
docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt .

-- curl을 통해 인증서가 유효한지 확인. 가능하면 이미지 실행시 출력된 패스워드 입력
curl --cacert http_ca.crt -u elastic https://localhost:9200


```

```shell
-- elasticSearch 실행 확인

example@host ~ % docker ps
CONTAINER ID   IMAGE                                                 COMMAND                  CREATED         STATUS         PORTS                                            NAMES
1cb5460a7ba5   docker.elastic.co/elasticsearch/elasticsearch:8.4.3   "/bin/tini -- /usr/l…"   6 minutes ago   Up 6 minutes   0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   es01
example@host ~ % curl -XGet http://localhost:9200/_cluster/health?pretty=true

-- elasticSearch 설정 확인

example@host ~ % docker exec -i -t es01 cat /usr/share/elasticsearch/config/elasticsearch.yml
cluster.name: "docker-cluster"
network.host: 0.0.0.0
```

```shell
docker pull docker.elastic.co/kibana/kibana:8.4.3
docker run --name kibana --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.4.3
```



키바나 run 후 출력되는 url에 복사한 kibana token을 넣은 후 

elaticSearch 실행 시 출력되었던 로그 중 user와 password를 입력하면 된다!

Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):

![스크린샷 2022-10-18 오후 8.39.05](https://user-images.githubusercontent.com/47243329/196449646-cec9ed15-8e00-4dde-b9b5-5603fcf47aa3.png)



그러면 Elastic Cloud에서 보았던 화면이 그대로 출력된다! 뿌듯



데이터 하나만 POST, GET 실습만 진행해보기로 결정!

```shell
example@host ~ % curl -X POST "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "firstname": "Jennifer",
  "lastname": "Walters"
}
'

curl: (52) Empty reply from server

```

..? 이것만하고 집에가려고 했는데 집에 가고싶다

docker log를 확인해보니

```shell
"message":"received plaintext http traffic on an https channel, closing connection Netty4HttpChannel{localAddress=/~~, remoteAddress=/~~}"
```

이러한 오류가 확인된다. https가 안된다고 하는 것 같아서 elasticSearch의 설정파일(elasticsearch.yaml)을 찾아보니

```yaml
# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
# Create a new cluster with the current node only
# Additional nodes can still join the cluster later
cluster.initial_master_nodes: ["1cb5460a7ba5"]
```

이렇게 확인된다. 컨테이너 안에 들어가서 수정하려고했으나 vi가 안깔려있는 container였다 .. docker를 올릴때 설정하는 것이 좋겠다고 생각이 들었다. 찾아보니 공식 페이지에 docker-compose가 있었다. 한번에 할수있는 파일이 있었잖아 ?

근데 여러개의 노드를 만드는것같아서 우선 ssl을 false로 바꾸고 인증서 부분을 삭제한 후 es01만 사용할수있게 수정하고 실행시켰다.

```yaml
-- docker-compose.yaml

version: "2.2"

services:
  setup:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.4.3
    user: "0"
    command: >
      bash -c '
        if [ x${ELASTIC_PASSWORD} == x ]; then
          echo "Set the ELASTIC_PASSWORD environment variable in the .env file";
          exit 1;
        elif [ x${KIBANA_PASSWORD} == x ]; then
          echo "Set the KIBANA_PASSWORD environment variable in the .env file";
          exit 1;
        fi;
      '

  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.4.3
    volumes:
      - esdata01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    environment:
      - node.name=es01
      - cluster.name=${CLUSTER_NAME}
      - cluster.initial_master_nodes=es01
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1

  kibana:
    image: docker.elastic.co/kibana/kibana:8.4.3
    volumes:
      - kibanadata:/usr/share/kibana/data
    ports:
      - 5601:5601
    environment:
      - SERVERNAME=kibana
      - ELASTICSEARCH_HOSTS=http://es01:9200
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
    mem_limit: ${MEM_LIMIT}

volumes:
  certs:
    driver: local
  esdata01:
    driver: local
  kibanadata:
    driver: local

```

```shell
-- 실행
example@host elasticSearch % docker-compose up -d

-- 확인
example@host elasticSearch % docker ps
CONTAINER ID   IMAGE                                                 COMMAND                  CREATED              STATUS              PORTS                              NAMES
2e60d7be03c5   docker.elastic.co/kibana/kibana:8.4.3                 "/bin/tini -- /usr/l…"   About a minute ago   Up About a minute   0.0.0.0:5601->5601/tcp             elasticsearch_kibana_1
2d6c6ef27579   docker.elastic.co/elasticsearch/elasticsearch:8.4.3   "/bin/tini -- /usr/l…"   About a minute ago   Up About a minute   0.0.0.0:9200->9200/tcp, 9300/tcp   elasticsearch_es01_1
```

잘 실행된것을 확인하였으니 다시 실습을 해보았다!

```shell
example@host elasticSearch % curl -X POST "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "firstname": "Jennifer",
  "lastname": "Walters"
}
'
{
  "_index" : "customer",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

example@host elasticSearch % curl -X GET "localhost:9200/customer/_doc/1?pretty"

{
  "_index" : "customer",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "firstname" : "Jennifer",
    "lastname" : "Walters"
  }
}
```

등록과 조회를 해봤으니 키바나에서 확인을 해보면! 



<img width="1412" alt="스크린샷 2022-10-18 오후 10 48 21" src="https://user-images.githubusercontent.com/47243329/196450179-1225fa5b-33a7-4613-b5ba-19394095bb33.png">

<img width="1434" alt="스크린샷 2022-10-18 오후 10 48 33" src="https://user-images.githubusercontent.com/47243329/196450099-aabe4a3d-ab94-446d-9a51-6f972dafdc7b.png">


이렇게 데이터를 확인할 수 있다! 휴 뿌듯하고 신기하다 ... 

강의를 듣기에는 너무 겉핥기만 해본 느낌이지만 ,, 열심히 이해해봐야지!

다음 블로그에서 그 내용은 정리해야겠다.
