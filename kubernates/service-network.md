# Service network

Service discovery를 위해 kube-proxy가 관리하는 Cluster-wide 범위의 Virtual IP



Service는 k8s의 api중 하나이다.

쿠버네티스의 node들간의 loadbalancing을 만들어주는 역할을 한다.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

하나의 IP = Virtual IP, k8s = Cluster IP

10.96.100.100 으로 요청을 보내면, loadbalance를 통해, 10.244.1.1, 10.244.2.2, 10.244.2.1 중 부하분산시켜서, 요청을 보내게 된다.\
\
**msa 환경에서 container끼리 통신을 그러면 어떻게 하냐?**\
main이라는 컨테이너가 있다고 했을때, service에 의해 만들어진 Virtual IP를 통해 요청을 보내게 되고, loadbalacer에 의해 부하분산 요청을 보내게 된다.  (loadbalancer는 각 node에 존재하게 된다.)

이러한 loadbalancer를 만드는건, iptalbes rules에 의해 만들어진다. (방화벽 기능 뿐만 아니라, loadbalance도 해준다)



정리해서,&#x20;

1. "쿠버야 web 파드들을 하나의 IP(Virtual IP / Cluster IP)로 묶어서 관리해줘! 요청을 Service에 보낸다.
2. etcd에 endpoint(app: web)들이있는 Cluster IP(Virtual IP)가 만들어진다.
3. Cluster IP가 만들어지게 되면, Service에게 알려주고, Service는 각 노드에 있는 Kubelet에게 보내게 된다.
4. "Kubelet아, IPtables rules을 이용해서 Loadbalancing 되는 Rule을 만들어줘" 라고 보내게 되는데, kubelet은 IPtables Rules을 만들 수 있는 기능이 없다. (방화벽 룰은 root 권한을 가져야 설정이 가능한데, kubelet은 이런 권한이 없다.)
5. 하지만, kube-proxy 라는 애가 root 권한을 가지고 있어서, kubelet은 kube-proxy에게 요청을 보내게 된다.
6. kube-proxy가 리눅스 커널에, iptables rules을 만들어달라고 요청할 수 있다.
7. iptables rule이 만들어지고, 이 rule은 loadbalancing을 지원해준다.





