# Pod network

Pod Network = CNI (Container Network interface plugin) 에서 관리하는 포드 간 통신에 사용되는 클러스터 전체 네트워크

컨테이너와 컨테이너 간의 네트워크를 지원해준다.

원래는같은 host내에 있는 컨테이너들끼리는 통신이 가능하지만,

다른 host의 컨테이너와 통신은 지원을 안해준다.

각각의 host에서 라우팅 테이블을 가지고 있는 gateway를 통해, 네트워크 통신이 가능하다.
