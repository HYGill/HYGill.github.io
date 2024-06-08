# openLdap Helm 설치

[OpenLdap Chart](https://github.com/helm/charts/tree/master/stable/openldap) 

#### 수정 부분 

1. values.yaml

   ```yaml
   service:
     externalIPs: [172.0.0.1]
     type: NodePort
   
   env:
     LDAP_ORGANISATION: "kbfg"
     LDAP_DOMAIN: "com"
     LDAP_BACKEND: "hdb"
   
   adminPassword: admin
   ```

   

2. helm 설치

   ```shell
   helm install ldap -f values.yaml stable/openldap -n ldap
   ```



#### 문제사항 

1. **chown: changing ownership of '/~~': Operation not permitted 오류 발생**
   : no_root_squash - NFS는 일반적으로 루트 사용자를 nobody로 변경합니다. 이는 여러 다른 사용자가 NFS 공유에 액세스할 때 좋은 보안 조치

   pv가 만들어질때 nobody, nogroup으로 생성되어서 pod 내에서 권한 변경을 못하고 있었다. 그래서

   ```shell
   apt-get update
   
   apt-get install -y nfs-common nfs-kernel-server portmap
   
   chmod 777 /data   # NFS 루트 디렉토리로 사용될 디렉토리 권한부여
   
   접근가능한 아이피 설정
   vim /etc/exports
   
   /data ~.~.~.~/16(rw,sync,no_subtree_check)
   /데이터디렉토리경로 아이피/대역폭 (rw,sync,no_subtree_check)
   
   데몬 재시작
   /etc/init.d/nfs-kernel-server restart
   ```

   https://batt.tistory.com/126

   

   그러면 생성된 pv group과  user가 root인걸 확인할 수 있다!
