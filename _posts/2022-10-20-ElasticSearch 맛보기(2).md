# ElasticSearch 맛보기(2)

세미나에서 들은 내용을 정리해보겠다!

------

- **Elasticsearch** : 분산 시스템

  - 쉬운 스케일 아웃
  - 고 가용성 보장
  - Apache Lucene(풀텍스트 검색 엔진을 만들 수 있는 자바 라이브러리) 사용
  - Rest API를 지원하며 RESTFul 한 속성을 가지고 있음
  - Json 도큐먼트 기반으로 동작. 쿼리도 json 형식!
  - 풀 텍스트 검색이 가능

- **Elasticsearch 클러스터링 과정**

  - 데이터를 샤드(Shard) 단위로 분리해서 저장

    -> 샤드 : 루씬 검색 쓰레드, 노드 : Elasticsearch 실행 프로세스

![클러스터 구조](https://user-images.githubusercontent.com/47243329/198865403-7bb59be9-cac8-4f5b-bdea-437ee878c7b1.jpg)



- **클러스터(Cluster)**

  - Elasticsearch 시스템의 가장 큰 단위
  - 하나의 클러스터는 다수의 노드로 구성
  - 하나의 클러스터를 다수의 서버로 바인딩해서 운영, 또는 역으로 하나의 서버에서 다수의 클러스터로 운용 가능

- **노드(Node)**

  - 엘라스틱서치를 구성하는 하나의 단위 프로세스
  - 다수의 샤드로 구성
  - 같은 클러스터명을 가진 노드들은 자동으로 바인딩

- **클러스터 구성**

  - 하나의 물리 장비에서 하나 이상 노드 실행 가능
  - 디폴트로 2개의 포트를 통해 클라이언트(http:9200), 노드(tcp:9300) 통신
  - 하나의 물리 장비에서 다수의 클러스터도 실행 가능
  - Discovery : 같은 클러스터의 다른 노드를 찾아 바인딩하는 과정

- **데이터 구조**

  - RDBMS에서는 데이터를 테이블 형태로 저장

    -> 인덱스를 특정 열을 기준으로 만들기 때문에 내용을 검색하기 위해서는 Like 검색을 통해 한 행씩 찾아야함

  - 검색엔진에서는 텍스트를 분석하여 검색어 사전을 만들어 놓아서 인덱스로 검색 가능

    -> 텍스트 분석(Analyzer)을 통해 Term을 분석

- **텍스트 분석 과정(검색어도 동일하게 분석)**

  - Tokenized된 Term들을 가공하는 Token Filtering 수행
    1. Lowercase Token Filter로 대소문자 변환
    2. 대소문자가 변환되어 같아진 텀들을 합쳐줌
    3. 검색어로서의 가치가 없는 단어들을 제거(a, an, but ..) 
       Stop Token Filter가 사용
    4. s, ~ing 등을 제거하는 형태소 분석 과정을 거침
       Snowball Token Filter를 사용
    5. 동의어 처리
       Synonym Token Filter를 이용해 동의어 사전을 정의

- **매핑 및 필드**

  - 각 필드별로 데이터를 저장하는 스키마 명세이며 인덱스/타입 별로 구분

  - 자동 생성된 매핑의 텍스트 필드는 기본적으로 standard analyzer가 적용되며 keyword 타입의 멀티 필드 생성

    ```json
    ...
    "pages" : {
    		"type" : "long" 
    },
    "publish_date" : { 
    		"type" : "date"
    },
    "author" : {
    		"type" : "text", "fields" : {
    		"keyword" : {
    				"type" : "keyword", "ignore_above" : 256
    		} ... 
    ```

- **Elasticsearch 집계 Aggregation**

  **종류**

  **Bucket** : 주어진 조건으로 분류된 버킷을 만듦. 도큐먼트 집단 단위인 버킷을 생성

  -> 또 다른 Bucket 또는 Metric의 Sub-aggregation을 포함

  ```json
  // 1) keyword 필드의 문자열 별로 bucket을 나누어 집계하도록 설정
  {
    "aggs" : {
      "authors" : {
        "terms" : {
          "field" : "author.keyword"
        }
      }
    }
  }
  // 1)의 결과
  "aggreagtions" : {
  	"authors" : {
  		"doc_count_error_upper_bound" : 0,
  		"sum_other_doc_count" : 0,
  		"buckets" : [
  			{
  				"key" : "William shakespeare",
  				"doc_count" : 2
  			},
  			..
  		]
  	}
  }
  
  // 2) 숫자 필드 값으로 범위를 지정하고 해당하는 버킷을 만드는 집합
  {
    "aggs" : {
      "page_range" : {
        "range" : {
          "field" : "pages",
          "ranges" : [
            {"from" : 0, "to" : 50},
            ...
          ]
        }
      }
    }
  }
  // 2)의 결과
  "aggregations" : {
    "page_range" : {
      "buckets" : [
        {
          "key" : "0.0-50.0",
          "from" : 0.0,
          "to" : 50.0,
          "doc_count" : 0
        },
  			...
      ]
    }
  }
  ```

  **Metric** : 도큐먼트 집단에 대한 하나 또는 그 이상의 계산된 수치를 포함

  ```json
  // 특정 쿼리문에 대한 avg와 sum 결과값 포함
  {
  	"aggs" : {
      "page_avg" : {
        "avg" : {
          "field" : "pages"
        }
      },
      "page_sum" : {
        "sum" : {
          "field" : "pages"
        }
      }
    }
  }
  // 결과
  {
    "aggregations" : {
      "page_sum" : {
        "value" : 376.0
      },
      "page_avg" : {
        "value" : 125.333333333
      }
    }
  }
  ```

  -> 두가지를 콤보로 사용할 수 있으나 3 Level까지 depth가 가게되면 시스템 뻗음

  **Pipeline** : 다른 집계 또는 Metric 결과에 대한 새로운 계산을 실행

  

- **도큐먼트 색인 및 검색**

  - 입력받은 도큐먼트들은 해당 인덱스의 샤드들에 Round Robin 방식으로 분배되어 저장됨

  - 도큐먼트 불러오기(retrieve) : 도큐먼트 ID를 입력하여 검색과정 없이 실행. inverted index를 사용하지 않고 실행

  - 과정

    1. 처음 쿼리 수행 명령을 받은 노드(코디네이트 노드)는 모든 샤드에게 쿼리를 전달

    2. 각 샤드들은 요청된 크기만큼의 검색 결과 큐를 코디네이트 노드로 리턴. 리턴된 결과는 루씬 doc id와 랭킹 점수만 포함

       -> 요청 size가 10이었으면 각 10size만큼 리턴

    3. 노드는 리턴된 결과들의 랭킹 점수를 기반으로 정렬

       -> 이에 각 샤드들이 리턴한 10개 중에 유효한 값들만 의미있게 된다.

    4. 유효한 샤드들에게 최종 결과들을 다시 요청

       -> 이 전에는 id와 랭킹 점수만 리턴받았으니 전체 문서 요청

    5. 클라이언트에게 전체 문서 내용 정보 리턴



------

#### 강의 들으면서 정리없이 필기했던 부분

- Integrations Server : datadog과 같은 것을 연결하기 위하여 클러스터 생성 시 설정하여야함

- 스케일 아웃이 잘된다

- 아파치 루씬 : 텍스트 서칭을 할 수 있게하는 library. 

- 샤드 : 루씬 검색 쓰레드. 인덱싱이 샤드에 분배되어 저장된다.

- 노드 : ElasticSearch 실행 프로세스. 같은 클러스터명을 가진 노드들은 자동으로 바인딩. 다수의 샤드로 구성

- ElasticSearch 클러스터링 과정 : 노드를 여러개 실행시키면 같은 클러스터로 묶임

- 클러스터 구성 : 디폴트로 2개의 포트를 통해 클라이언트(9200) 노드(9300) 통신 
  내부통신이 9300

- security 설정이 default가 true이게 바뀌었다. 이전에 실습할때 그래서 xpack 보안 설정 바꿔야했던듯

- discovery.seed_hosts 설정에서 실행이고 클러스터 명이 같은 노드가 없다면 새로운 클러스터로 시작 아니라면 확인된 노드의 클러스터에 합류

- 역인덱싱을 위해 텍스트 분석(Analyzer)을 저장 전에 수행한다.

- Keyword Type(대소문자까지 정확하게 일치)은 집계를 위해 사용. 

  

윌리엄 셰익스피어가 집계를 위해 keyword 검색도 되어야하지만 검색에서 윌리엄만 검색했을때 결과가 나와야한다면 text와 keyword 타입을 모두 지정할 수있다.

- Histogram aggregation : interval 설정도 가능. 막대 차트같은 것에서 필요

- 검색 과정 

  받은 쿼리를 해당하는 모든 샤드에 전달 -> 각각의 샤드는 샤드별로 쿼리를 병렬적으로 수행. 해당하는 정보를 10개씩 id와 얼마나 쿼리에 적합한지 _score 점수로 코디네이트 노드로 전송 -> 코디네이트 노드는 상위 10개를 남기고 나머지는 버림 -> 코디네이트 노드는 유효한 검색 결과를 가진 샤드에 다시 검색 요청

- 검색 랭킹 알고리즘 

  Term Frequency : 찾는 검색어의 갯수 (정확도 높음)

​		Inverse Document Frequency : 흔한 단어는 점수가 낮음

- 세그먼트 병합

  : 도큐먼트 단위로 삭제하는 것은 작업이 크기때문에 삭제를 해도 index를 사용하라

  삭제하지 않고 오래된 인덱스에 병합
