---
description: 무상태 웹 계층
---

# Stateless Web tier

사용자 세션 데이터와 같은 상태 정보를 웹 계층에서 제거한다.

바람직한 전략은 상태 정보를 관계형 데이터베이스나 NoSQL 같은 지속성 저장소에 보관하는 것.



상태 정보를 보관하는 서버와 아닌 서버들 사이에서는 상태를 유지하여 요청들 사이에 공유되어야한다.

무상태 서버에는 이런 장치가 없다.

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

A 사용자가 웹서버1로 인증을 하게 됬을때, 상태가 공유되지 않은 상태에서 웹서버2로 요청을 보내게되면 인증은 실패한다. 이런 문제를 해결하기 위해 로드밸런스는 고정 세션이라는 기능을 제공하지만, 이 기능은 로드밸런서에 부담을 준다.



상태정보를 공유 저장소에 저장하게 되면, 사용자의 요청은 어떤 웹 서버로든 전달될 수 있다.

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>



"프로젝트에서 왜 Redis를 사용했나요?"

일반적으로 웹서버는 사용자와의 세션을 유지하게 됩니다. 그러나 이러한 경우 웹서버가 다수일 때 트정 사용자의 요청을 항상 동일한 웹서버로 보내야하는 문제가 발생합니다. 이를 해결하기 위해 로드밸런서 내에서 세션 고정 기능을 사용할 수 있지만, 이는 로드밸런서에 부담을 줍니다.

이러한 문제를 방지하고 웹서버 간 세션 정보의 공유를 가능하게 하기 위해, 사용자 인증 정보를 중앙 데이터베이스에 저장하는 방식을 고려했습니다. 하지만 전통적인 데이터베이스는 I/O 오버헤드가 존재하여 성능 저하를 일으킬 수 있고, 이를 해결 하기 위해 해시 테이블 기반의 Redis를 도입하여 성능을 최적화했습니다.

Redis는 메모리 기반의 키-값 저장소로서 디스크 기반의 데이터베이스보다 빠른 읽기 및 쓰기 속도를 제공합니다. 이는 사용자 인증과 같이 빈번한 데이터 접근이 필요한 경우에 유리합니다. 또한 Redis의 스케일 아웃 능력은 확장될때 유연하게 대응할 수 있습니다.

이러한 이유로 Redis는 웹 서버 간 사용자 세션 정보를 공유하는데 이상적인 선택이 되었습니다.