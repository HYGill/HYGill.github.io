# MatterMost Helm chart 

- 개요 : 메타모스트 Helm Chart와 Jenkins Job을 작성하여 사용자가 웹에서 필요한 정보를 입력하면 자동 설치가 되는 부분을 진행하게 되었다.



[메타모스트 helm chart ]: https://github.com/helm/charts/tree/master/stable/mattermost-team-edition

우선 mattermost 기본 helm chart를 다운받는다. 기본 제공하는대로 사용한다면 Readme에 있는 install 처럼 진행하면 된다.



프로젝트를 위해 수정해야했던 부분은 

1. application name, url과 같은 부분을 사용자가 적은 정보에 맞게 커스터마이징 해야함
2. mattermost는 웹소켓을 이용하여 통신하기 때문에 (메세지 기능이 있어서 일까) ingress proxy 부분 수정
3. mattermost에서 기본 제공하는 mysql을 사용하지 않고 프로젝트 내에 쓰고있는 db를 사용하기 때문에 external DB를 활성화 시키기



이런 부분을 수정하기 위해서는 helm chart의 수정도 중요하지만 helm을 실행시키기 위한 job을 어떻게 구성하는지 생각하는것도 오래걸렸다. 자세한 소스코드는 첨부하지 못하지만 겪었던 문제 사항을 공유하기 위해 블로그를 쓴다~ 다들 나처럼 삽질하지 말길~



1. `{.values. }` 라고 적힌 부분은 value.yaml에 적혀있는 부분이다. 

   ``` 
   helm install [] [] --set name=[]
   ```

      이처럼 install 할 때 set명령어를 통해 원하는 대로 주입이 가능하다. 그렇기 때문에 1번 조건을 해결하기 위해 Jenkins job에 변수로 처리하여 사용자마다 다른 namespce, app name, url을 주도록 하였다.

2. Ingress annotation중에는 nginx.conf에 직접 써주는 것과 전역 선언되는 것 두가지 타입이 있다.

   진짜 이것땜에 삼일 삽질했다. 다들 공식문서를 제발 꼭 보자

   ```shell
   nginx.org/server-snippets: |
         location ~ /api/v[0-9]+/(users/)?websocket$ {
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
           proxy_pass {{ .Values.proxyPass }};
   }
   ```

   

   [nginx 관련 helm 문법]: https://docs.nginx.com/nginx-ingress-controller/configuration/ingress-resources/advanced-configuration-with-annotations/

   여기에 보면 server-snippets와 location-snippets가 써져있는데 둘다 써본 결과 server는 그냥 쌩으로 저렇게 넣어준다. location은 이미 있는 곳에 추가해주는 것 같다. 나는 프록시 pass를 그냥 추가해줘야하는 것이라서 server를 써서 저렇게 그대로 넣었다. 

   저렇게 넣어야했던 이유는

   **websocket upgrade가 되지 않는 오류를 만났기 때문**

   [Proxy pass 추가 내용]: https://docs.mattermost.com/install/config-proxy-nginx.html

   메타모스트가 친절하게 내용을 알려줬는데 이거 넣는 방법을 몰라서 계속..계속..삽질했다. 이거 제대로 넣으니까 바로뜨더라~ 눈물이 좀 나네

   

3. 이것도 맨위에 readme에 있는 것 보면 나와있는데 externalDB 활성화 시키는 법이 나와있다. value에서 enable시키고 정보를 넣어주면 끝. DB정보를 넣어줘야하는데 mysql은 string으로 쭈욱 넣어주는 방식이기때문에 소스코드에서 가공한 후에 그냥 string으로 쭉 넘겼다. 





------



진짜 지금 돌아보면 너무 허둥지둥해서 삽질을 많이 했다는 생각만 든다. 

문제점을 파악하고 원리를 알고 했으면 야근은 안해도 됐을텐데

그래도 완성한거에 의의를 두고 이제 다시 해야하는 쥬피터 노트북은 삽질하지 말고 해야지..





