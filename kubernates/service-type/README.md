# Service type

* ClusterIP(default):
  * Pod 그룹(동일한 서비스를 지원하는 Pod 모음)의 단일 진입점(Virtual IP(ClusterIP): Loadbalancer) 생성
  * 외부에서는 ClusterIP를 통해 들어오지 않고, 각 pod의 ip를 통해서만 들어올 수 있다.
* NodePort:
  * Cluster IP가 생성된 후 모든 Worker Node에 외부에서 접속 가능 한 포트가 예약
  * Cluster IP가 생성될때 각 pod들의 해당 port가 열리게 된다. 외부에서 해당 pod의 ip와 port를 통해 들어가게 되면, iptables로 인해 loadbalancing이 되면서 네트워크 통신이 가능하게 된다. (포트포워딩)
*   LoadBalancer:

    * 클라우드 인프라스트럭처 (AWS, Azure, GCP)에 적용
    * LoadBalancer를 자동으로 프로 비전하는 기능 지원



그러면 외부에서는 어떤 pod의 ip로 접속해야하나?

\-> Type을 Loadbalancer로 하게 되면, aws/gcp 같은 곳에서 자동으로 loadbalancer가 만들어진다. (부하분산을 위해)

