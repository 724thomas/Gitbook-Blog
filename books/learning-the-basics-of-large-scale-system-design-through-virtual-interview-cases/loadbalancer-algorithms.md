---
description: 대규모 시스템 설계 관점에서의 로드밸런스 알고리즘
---

# Loadbalancer Algorithms

로드 밸런싱 알고리즘들은 혼합하거나 수정해서 사용하기도 한다. 시스템 특성과 요구사항에 맞게 선택하고 조정하는게 중요하다.

* 라운드 로빈
  * 순차적으로 각 서버에 균등하게 분배한다.
  * 첫 번째 요청은 첫 번째 서버에, 두번째 요청은 두번째 서버에 보내지는 식이다.
  * 구현이 간단하고, 모든 서버에 균등하게 요청을 분배한다.
  * 서버의 성능이나 현재 부하를 고려하지 않는다. 과부하 상태일때도 같은 수의 요청을 받는다.
* 가중 라운드 로빈
  * 라운드 로빈과 유사하지만, 각 서버에 특정 가중치를 부여한다. 더 높은 가중치를 가진 서버가 더 많은 요청을 받는다.
  * 서버의 성능이나 용량에 따라 요청을 조정할 수 있다.
  * 올바른 가중치 설정이 필요하며, 서버의 상태나 용량에 따라 가중치를 주기적으로 조정해야 한다.
* 최소 연결
  * 현재 활성 연결이 가장 적은 서버에 새로운 요청을 보낸다.
  * 서버마다 처리 능력이 다르거나, 세션이 길게 유지되는 경우 적합하다.
  * 실시간 트래픽 분포에 반응하여 서버에 균등한 부하를 할당한다.
  * 연결 수만 고려하므로, 서버의 실제 부하나 처리 시간을 고려하지 않는다.
* 가중 최소 연결
  * 최소 연결 알고리즘과 유사하지만, 서버의 성능이나 용량에 따라 가중치를 부여한다.
  * 서버의 성능과 현재 부하를 더 정확히 반영할 수 있다.
  * 가중치 설정과 조정이 필요하며, 서버 상태의 변화에 따라 지속적인 관리가 요구된다.
* IP 해시(IP Hash)
  * 클라이언트의 IP 주소를 해싱하여 결정된 서버에 요청을 보낸다.
  * 클라이언트가 항상 동일한 서버로 연결되도록 한다.
  * 세션 지속성을 보장하며, 사용자별 정보가 필요한 경우 유용하다.
  * 트래픽 분포가 불균등할 수 있으며, 특정 서버에 과부하가 발생할 가능성이 있다.
* 최소 응답 시간(Least Response Time)
  * 응답 시간이 가장 짧은 서버에 요청을 보낸다. 이는 서버의 현재 부하와 성능을 실시간으로 고려한다.
  * 빠른 응답을 제공할 수 있는 서버를 선택함으로써 사용자의 경험을 향상
  * 실시간 모니터링과 높은 계산 복잡도가 필요하다.
* URL 해시(URL Hash)
  * 요청 URL을 기반으로 해시 값을 계산하여 결정된 서버에 요청을 보낸다.
  * 특정 URL을 항상 같은 서버가 처리하도록 한다.
  * 캐시 효율성이 높아지며, 특정 종류의 요청을 특화된 서버가 처리할 수 있다.
  * 특정 서버에 부하가 집중될 수 있으며, 모든 URL에 균등하게 트래픽을 분배하지 못할 수도 있다.
