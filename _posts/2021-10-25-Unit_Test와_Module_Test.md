# Unit Test와 Module Test

찾아보면 Unit Test와 Module Test를 하나로 말해서 두개의 차이점을 잘 몰랐었는데 설명을 들어서 정리~
Unit Test 경험밖에 없어서 나중에 API Test도 꼭 해보고싶다


1. #### Unit Test

   메서드 Level의 Test. 다른 dependency를 모두 없앤 후 순수 로직만 체크

   - 사용방법 
     - Spring일 경우, Junit dependency 추가하여 test폴더에 작성
     - Express일 경우, 별도의 Test 설치 후 사용(ex. Jest)



2. #### Module Test

   dependency를 살려서 실제 API 테스트

   - 사용방법

     - Newman, Postman, Jenkins로 API 테스트 자동화

       [참고자료1](https://co-de.tistory.com/19)
       
       [참고자료2](https://medium.com/dtevangelist/devops-jenkins-postman-newman%EC%9C%BC%EB%A1%9C-api-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%9E%90%EB%8F%99%ED%99%94%ED%95%98%EA%B8%B0-f08a155a949c)

       -> Postman에서 API 작성 후 export -> 개발 소스에 json 파일 업로드 -> newman 설치 후 pipeline에서 newman run (--reporters : 결과를 리턴하는 방식 지정) -> 결과값 확인
       
    - 실제 DB값이 변하기 때문에 DB의 Git인 flyway를 사용하여 테스트 후 초기화 시켜준다

       

   

   
