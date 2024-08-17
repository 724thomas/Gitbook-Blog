# deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
apiVersion: apps/v1
  name: web
apiVersion: v1
spec:
  replicas: 3 #총 3개의 pod
  selector:
    matchLabels:
      app: web #web 이라는 pod
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web #service의 selector에 해당되는 이름
    spec:
      containers:
      - name: nginx-container #nginx 컨테이너
        image: nginx:1.14 #nginx 이미지
```

명령어:

*   kubectl apply -f deployment.yaml

    \-> deployment.apps/**web** created
*   kubectl get pods | grep -i **web**

    \->&#x20;

    <figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
*   kubectl get pods -o wide | grep -i **web**

    \->&#x20;

    <figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
* kubectl delete -f deployment.yaml\
  \-> deployment.apps "web" deleted
