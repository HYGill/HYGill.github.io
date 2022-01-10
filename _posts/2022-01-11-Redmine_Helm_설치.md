# redmine Helm 설치



[Redmine Chart]: https://github.com/bitnami/charts/blob/master/bitnami/redmine/



#### 수정 부분 

1) deployment.yaml

   ```yaml
   name: REDMINE_DATABASE_TYPE
   value: mysql
   ```

    : values.yaml에서 제공하는 database_type에 mysql이 없어서 직접 String으로 넣었다. 만일 postgresql이나 mariadb를 사용한다면 안고치고 values.yaml에서 REDMINE_DATABASE_VALUE만 바꾸면 된다!

   

2. value.yaml 

   ```yaml
   # externalDatabase 설정
   externalDatabase:
     host: "~~~~~"
     port: 3306
     user: redmine
     password: "~~~~~"
     database: redmine
     existingSecret: ""
   
   # ingress 연결
   ingress:
     enabled: true
     ...
     hostname: redmine.kronos.dmz1.dev.tz.cip.digitalkds.co.kr
     
   # pv 설정
   persistence:
     enabled: true
     storageClassName: "{storageName}"
     accessModes:
       - ReadWriteOnce
     size: 5Gi
   
   # redmine 설정
   redmineUsername: redmine
   redminePassword: "~~"
   redmineEmail: user@email.com
   redmineLanguage: en
   ```



#### 기타자료

1) [minikube 연결 확인](https://minikube.sigs.k8s.io/docs/handbook/host-access/) : minikube에서 local mysql에 연결되는지 확인할 때 쓰인 링크
2) [variables 확인](https://github.com/bitnami/bitnami-docker-redmine/#environment-variables) : redmine values의 값 확인

3. [ingress 설정](https://github.com/kubernetes/kubernetes/issues/90077) : k8s version에 따른 ingress version 확인

   > networking.k8s.io/v1beta1 == 1.14 to 1.18
   >
   > networking.k8s.io/v1 = 1.19+

