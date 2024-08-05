# services.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  clusterIP: 10.16.15.100 #단일 진입점을 설정. 안하게 되면 랜덤하게 생성된다.
  selector:
    app: web #web이라는 이름으로
  ports:
  - protocol: TCP #TCP 기반의
    port: 80 #10.16.15.100:80으로 접속하게 되면,
    targetPort: 80 #web이라는 pod들 중 하나의 80포트로 연결
```

명령어:

*   kubectl apply -f services.yaml

    \-> service/**web-svc** created
*   kubectl get svc

    \->

    <figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

web-svc의 10.16.15.100이 단일 진입점이 된다.

* kubectl delete -f services.yaml\
  \-> services "web-svc" deleted
