# Go 프레임워크

### 목차

1. [Gin](#Gin)
2. [Echo](#Echo)
3. [Beego](#Beego)
4. [Revel](#Revel)
5. [기타](#기타)



## Gin

Gin은 Go언어로 쓰여진 web framework이고, martini와 같은 httprouter를 사용하여 40배 이상의 performance를 나타냄



![gin](https://user-images.githubusercontent.com/47243329/90867359-39c2ec80-e3d0-11ea-933f-e09f34d7570b.PNG)



### 환경설정

```
1. go 언어 설치
2. $ go get -u github.com/gin-gonic/gin (Gin 설치)
3. import "github.com/gin-gonic/gin" 코드에 import해서 사용
```

[Gin 초기설정 참조링크](https://github.com/gin-gonic/gin#installation)



### 파일구조

: 프로젝트 생성할 시 자동으로 폴더 구축이 되지 않음

![gin파일구조](https://user-images.githubusercontent.com/47243329/90867874-ffa61a80-e3d0-11ea-9263-03a34af7bcda.PNG)

vender : go mod 사용

main.go : router 용도

controllers/userController.go : database 실행

models/db.go : database 셋팅

models/user.go : 사용자 model



### 코드

https://gitlab.com/HYGill/go_gin_crud



### 특징

1. 첫 글자가 소문자 인 경우 식별자는 비공개이며 선언 된 패키지 내에서만 액세스 가능(go언어 특징)
2. Go로 API 서버를 구축하고자 할 때 빠른 performace
3. 초기 설정이 간편



### 참고 자료

- [초기설정](https://github.com/gin-gonic/gin#installation)

- [패키지폴더구조](https://riptutorial.com/ko/go/example/29299/gin을-사용한-restfull-프로젝트-api)

- [에러 해결(소문자, 대문자)](https://www.sneppets.com/golang/cannot-refer-unexported-name-undefined-error-go/)







## Echo

Echo is a high performance, extensible, minimalist web framework for Go (Golang)

echo는 go(golang)의 웹 프레임워크



### 특징

- **Optimized Router** 경로의 우선 순위를 최적화로 지정하는 동적 메모리 할당이없는 최적화 된 HTTP 라우터

- **Scalable** 강력하고 확장 가능한 RESTful API를 빌드하고 그룹으로 쉽게 구성 가능

- **Automatic TLS** TLS 인증서를 자동으로 설치

- **HTTP/2** HTTP / 2 지원은 속도를 향상시키고, **Middleware** 많은 내장 미들웨어를 사용하거나 직접 정의. 또한 미들웨어는 루트, 그룹 또는 경로 수준에서 설정 가능

- **Data Binding** JSON, XML 또는 양식 데이터를 포함한 HTTP 요청 페이로드에 대한 데이터 바인딩

- **Data Rendering** JSON, XML, HTML, 파일, 첨부 파일, 인라인, 스트림 또는 Blob을 포함한 다양한 HTTP 응답을 보내는 API

- **Templates** 템플릿 엔진을 사용한 템플릿 렌더링

- **Extensible** 맞춤형 중앙 HTTP 오류 처리가 가능하고, 쉽게 확장 가능한 API



### 환경설정

```
$ go get -u github.com/labstack/echo/…
$ go run server.go
```

[echo 초기설정 참조링크](https://echo.labstack.com/guide/installation)



### 파일구조

![echo파일구조](https://user-images.githubusercontent.com/47243329/90868482-da65dc00-e3d1-11ea-91be-8c198313f470.PNG)

server.go : 서버 최초 시작

env: APP, DB의 정보를 관리

src/routers/Router.go : 라우팅 설정 & 컨트롤러 연결

src/controllers/UserController.go : 각 url과 method에 맞는 컨트롤러

src/configs/Database.go : mysql db와 연동 & db(CRUD)

src/models/UserModel.go : 구성할 table 형태



### 코드

https://gitlab.com/Uyoo/go_echo_crud



### 참고 링크

- [echo 공식문서](https://echo.labstack.com/guide/installation)

- [GORM 문서](https://gorm.io/ko_KR/docs/)







## Beego

Go 언어의 Application framework 중 하나. Beego 는 기본적으로는 ORM, Web Framework, Caching, Logging 등 Application 개발에 필요한 많은 기능을 제공해주는 프레임워크이고 “bee” 라는 명령행 도구를 제공해주는데 이 도구를 이용하여 Beego 에서 제공하는 다양한 기능을 사용 가능

- `generate` : scaffold 코드를 만들어 주는 도구
- `migrate` : database table의 변경 사항을 추적 실행해 주는 도구
- `run` : go application을 실행해주는 도구로 이 도구를 실행하면 코드 수정이 되면 자동으로 컴파일하고 재시작 없이 반영되도록 하는 도구



### 특징

- 기본 코드 제공 (Scaffold 기능)

- DB 스키마 변경 등을 쉽게 관리할 수 있는 기능 제공

  (migration 명령을 사용하여 실행, Rollback 이 가능)



### 환경 설정

1. beego 설치

   ```
   $ go get -u github.com/astaxie/beego
   ```

2. bee 명령어 설치

   ```
   $ go get -u github.com/beego/bee
   ```

3. 프로젝트 생성

   ```
   $ bee new [프로젝트 이름]
   ```



### 파일구조

![img](https://lh6.googleusercontent.com/Bgp7jxyPvBdr7vd7uwN-4yA3FvOpodWz7EggCM3Zr_znI_VzrgzkOae7WBujFnXoLbzA2f6WSf53R_0rJX0MxYHniKu4puzY7cClGKaGgXyxqfRwuNG_NsLKAh47cI_kDNkJyAaz)

main.go : 서버 시작 & DB 연결

conf/app.conf: APP, DB의 정보를 관리

routers/router.go : 라우팅 설정 & 컨트롤러 연결

controllers/UserController.go : 각 url과 method에 맞는 컨트롤러

models/UserModel.go : 구성할 table 형태 & db(CRUD)



### 코드

https://gitlab.com/Uyoo/go_beego_crud



### 참고

- [공식 문서](https://beego.me/quickstart)
- [사용후기](https://www.popit.kr/beego-go-application-framework-초간단-사용-소감/)







## Revel  ![img](https://lh3.googleusercontent.com/a5iW7goueB0Fz7dEuvhb5b7lqYFF-dM9l1mc-YipqTmw2uFe6yrs1HQ_CPbfkYd2lQSq4oxxGRGWbW49ONDnBRuDVCHSkUyNrgpBgtQxzx7eTesd5Hf_3cRmB9HfM4N9fcJ6jfrs)

Revel is a high-productivity, extremely flexible web framework for the Go language which provides routing, parameter parsing, validation, session/flash, templating, caching, job running, a testing framework, and even internationalization and lots more. Revel provides the flexibility of custom Server, Session and Template engines.



### 특징

- Full-fledged Framewor

- No Third-Party Plugins, Middleware, Configuration or Setup

- Huge & Active Community



### 환경 설정

- To get the Revel framework

  ```
  $ go get github.com/revel/revel
  ```

- Get and build the revel command line tool

  ```
  $ go get github.com/revel/cmd/revel
  ```

- Creating a new Revel application

  ```
  $ revel new -a myapp
  ```

- Run

  ```
  $ revel run -a myapp
  ```

  

### 파일구조

**![img](https://lh5.googleusercontent.com/6rsQeFkotm23C0uHaDxd6cldPdovmxCxa-8f2oYSYabnqQjodhC9ZOVjAPM-356vBpFMY83HGyB-vtaEU3KUY2JIL4LAjMetXKlx-BM6O9mgMc7ugmXK2d14HsLZHtJwMe4YiKAS)**

app : 애플리케이션(view, controller, db)

conf : 애플리케이션 설정 정보, 라우팅

public : 라우팅 없이 불러올 수 있는 퍼블릭 디렉토리

test : 기본 유닛테스트



### 코드

https://gitlab.com/chajk8111/golang-revel



### 참고

- 공식 문서 : https://revel.github.io/
- 환경설정 : https://www.hahwul.com/2019/11/21/go-revel-mvc/







## 기타

[Go Web Framework 선택하기](http://blog.naver.com/PostView.nhn?blogId=sory1008&logNo=221531591210&categoryNo=80&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView)

[웹 프레임워크 속도 측정(benchmark)](https://github.com/vishr/web-framework-benchmark)