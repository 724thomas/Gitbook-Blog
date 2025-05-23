# Data Structure

**링크드 리스트와 배열의 차이를 설명하고, 각각의 장단점을 비교하세요.**



링크드리스트: 노드가 포인터를 통해 연결된 구조

* 동적 크기 조정 가능.삽입, 삭제 O(1)
* 임의 접근 불가

배열: 메모리 공간에 연속적으로 저장된 구조

* 인덱스를 통한 빠른 접근. O(1)
* 삽입 삭제가 비효율적



1. 링크드 리스트에서 중간 노드를 삭제하는 시간 복잡도는 무엇인가요?

삽입 삭제 자체는 O(1)이지만, 중간 노드를 순차적으로 탐색하는 경우에는 O(n)



2. 배열 기반의 데이터 구조에서 메모리 재할당이 발생하는 이유는 무엇인가요?

배열은 연속된 메모리를 요구하기 때문에 크기를 초과하면 새로운 더 큰 메모리 공간을 할당하고 데이터를 복사.



3. 링크드 리스트와 배열을 활용한 캐시(Cache) 설계 방법은 무엇인가요?

* 링크드리스트: Least Recently Used 캐시 사용. 가장 오래된 데이터를 관리
* 배열: CPU 캐시처럼 고정 크기와 빠른 접근을 위한 인덱싱



4. 링크드 리스트에서 순환(Cycle)을 탐지하는 알고리즘은 무엇인가요?

플로이드의 순환 탐지 알고리즘. slow는 1단계씩, fast는 2단계씩 움직이면서, 두 포인터가 만나면 순환



5. 배열과 링크드 리스트를 혼합한 데이터 구조의 설계 사례를 설명하세요.

링크드리스트+해시맵. 인덱스 조회를 해시맵을 통해서 O(1)로 접근.



**트리와 그래프의 차이를 설명하고, 각각의 활용 사례를 제시하세요.**

* 트리는 방향성이 있고, 사이클이 없습니다. 파일시스템 / 조직도에서 활용
* 그래프는 방향성 및 사이클 여부가 상관없습니다. 네트워크 라우팅 / 소셜 네트워크에서 사용



1. 이진 탐색 트리(Binary Search Tree)에서 균형을 유지하는 알고리즘은 무엇인가요?

**LL 회전 (왼쪽-왼쪽 불균형)**: 오른쪽으로 단순 회전.

**RR 회전 (오른쪽-오른쪽 불균형)**: 왼쪽으로 단순 회전.

**LR 회전 (왼쪽-오른쪽 불균형)**: 왼쪽 서브트리를 왼쪽으로 회전 후, 오른쪽으로 회전.

**RL 회전 (오른쪽-왼쪽 불균형)**: 오른쪽 서브트리를 오른쪽으로 회전 후, 왼쪽으로 회전.

* AVL트리
  * 리프 노드들의 깊이가 1이상 차이가 나지 않아야하고, 만약 차이가 나는 경우, 회전을 사용해 트리의 균형을 맞춥니다.
* 레드-블랙 트리
  * 루트노드와 리프노드는 Black이고, 나머지 노드는 RED 또는 Black.
  * RED 노드의 자식 노드는 항상 Black. 모든 노드에서 리프 노드까지의 Black 노드 개수는 같아야합니다.
  * 트리의 높이가 군형 잡히지 않아도 항상 O(logN)을 보장. AVL보다 삽입/삭제 효율이 높지만 검색 시간이 느릴 수 있음.



2. 그래프에서 최단 경로를 찾는 알고리즘 중 다익스트라(Dijkstra)와 벨만-포드(Bellman-Ford)의 차이는 무엇인가요?

* 다익스트라:&#x20;
  * O(V+ElogV)
  * 음수 가중치를 허용하지 않습니다
  * 가중치가 양수인 그래프에서만 사용
  * 탐욕적 접근법으로, 현재까지의 최단 경로를 확정하며 진행.
* 벨만-포드
  * O(VE)
  * 모든 간선을 V-1번 반복 탐색하여 최단 경로를 구합니다.
  * 음수 사이클이 존재하는지도 탐지 가능.
  * 동적 프로그래밍 접근법. 모든 간선을 반복적으로 탐색하여 거리 값을 갱신.



3. 트리 구조에서 순회(Traversal) 방법 중 In-order와 Pre-order의 차이를 설명하세요.

* Pre-order: 좌-루트-우
* In-order: 루트-좌-우
* Post-order: 좌-우-루트



4. 그래프의 인접 리스트와 인접 행렬의 차이점은 무엇인가요?

* 인접 리스트
  * 정점 수는 많고 간선 수가 적은 희소 그래프(Sparse Graph)
  * 메모리 효율적. (해시맵)
  * 특정 간선을 조회시 O(V)
* 인접 행렬
  * 정점과 간선 수가 모두 많은 밀집 그래프(Dense Graph)
  * 메모리  비효율적. 2차원 배열.



5. 트리와 그래프를 활용한 네트워크 설계 사례를 설명하세요.

* 트리
  * 분산 네트워크에서 계층적 구조 설계
  * 사이클이 없고, 부모-자식 관계로 정의
  * DNS 시스템  회사 조직망
* 그래프
  * 노드와 간선으로 구성
  * 사이클 및 양방향 데이터 흐름 허용
  * 인터넷 라우팅 프로토콜 / 최적의 길 찾기
