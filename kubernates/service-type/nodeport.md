# NodePort

모든 노드를 대상으로 외부 접속 가능한 포트를 예약

Default NodePort 범위 : 30000\~327677

ClusterIP를 생성 후 NodePort를 예약

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service #노드 포트를 생성
spec:
  type: NodePort #노드 포트를 열어주겠다
  clusterIP: 10.100.100.200 #클러스터 IP 새ㅐ애성
  selector:
    app: webui
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30200 #생략 가능. 생략하게 되면 30000~32767중 랜덤
```



위 yaml 파일로 인해 clusterIP가 먼저생성되고(현재는 지정),  kube-proxy에 의해 nodePort 30200번으로접속했을때 loadbalancing이되는  iptables rules가 생성된다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

그 후에, 10.0.2.11:30200 또는 10.0.2.12:30200로 접속하게 되면, node1, 2내 iptables rules에 의해 loadbalancing을 통해 app:web에 접속이 가능하다. (각 node의 kube-proxy가 listening을 하게 된다)
