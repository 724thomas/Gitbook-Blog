# my-sql restarts

설명:

1. db-mysql 이름으로 mysql을 helm install을 통해 설치했다.
2. 초기 설정이 잘못되서 helm uninstall을 하게 되었고, 다시 db-mysql 이름으로 설치를 했다.
3. 설치를 하고 난 후에  db-mysql 내에서 find: '/docker-entrypoint-startdb.d/': no such file or directory 라는 `CrashLoopBackOff`문제가 생겼고, 계속 restart가 되고 있었다.



원인:

* uninstall 과정에서, 자동으로 pvc가 삭제되지 않는점이 문제였다.

해결:

1. 기존 db-mysql을 삭제
2. kubectl get pvc를 통해 pvc정보를 가져온다
3. kubectl delete pvc \[PVC\_NAME]를 통해 옛날 pvc를 삭제
4. 다시 helm install

