# Twitter - Cassandra, Fan out on Write

## 카산드라

### 개요

카산드라(Cassandra)는 오픈 소스 분산형 NoSQL 데이터베이스 관리 시스템입니다. 특히 단일 장애점 없이 고성능을 제공하면서 수많은 서버 간의 대용량 데이터를 관리하기 위해 설계



### 특징

* 분산 아키텍처 (Masterless): 중앙 집중형 마스터 노드가 없는 '마스터리스' 아키텍처로, 모든 노드가 동등하게 데이터를 저장하고 처리합니다. 이는 단일 장애점(Single Point of Failure)이 없어 시스템의 고가용성을 보장합니다.
* 수평 확장성 (Scalability): 데이터를 여러 서버에 분산하여 저장하고 처리함으로써 시스템의 확장성을 높일 수 있습니다. 노드를 추가하는 것만으로 선형적으로 처리량을 증가시킬 수 있습니다.
* 고가용성 및 내결함성 (High Availability & Fault Tolerance): 데이터가 여러 노드에 복제되므로, 특정 노드에 장애가 발생하더라도 서비스 중단 없이 데이터에 접근할 수 있습니다.
* 유연한 스키마 (Flexible Schema): 관계형 데이터베이스와 달리 스키마를 미리 엄격하게 정의하지 않아도 됩니다. 컬럼을 동적으로 추가하거나 제거하는 것이 용이하여 동적인 데이터 모델링에 유리합니다.
* 컬럼 기반 데이터 모델 (Wide Column): 데이터를 열(Column) 단위로 저장하며, 각 열은 서로 다른 데이터 타입을 가질 수 있습니다. 특히 대용량 데이터에 대한 읽기 성능이 뛰어나며, 시계열 데이터 처리에 효과적입니다.
* CQL (Cassandra Query Language): SQL과 유사한 문법을 가진 질의 언어인 CQL을 지원하여 사용자가 비교적 쉽게 데이터에 접근하고 조작할 수 있습니다.



### 분산 아키텍처(Masterless)

중앙 마스터 노드가 없습니다. Peer to Peer 구조를 가지고 있어서, 특정 노드에 장애가 발생하더라도 계속 동작할 수 있습니다. 하나의 클러스터는 여러개의 노드로 구성되어 있습니다. 이는 링 구조처럼 생겼습니다. 각의 파티션은 파티션 키로 인해 묶이는 그룹입니다.

* 클러스터 (Cluster) : 카산드라의 가장 큰 단위로, 전체 데이터베이스 시스템을 의미합니다. 이 클러스터는 여러 대의 노드로 구성됩니다.
* 노드 (Node) : 클러스터를 구성하는 개별 서버 인스턴스입니다. 각 노드는 데이터를 저장하고 처리하는 물리적인 역할을 합니다.
* 링 (Ring) 구조 : 클러스터의 모든 노드가 논리적으로 \*\*환형(Ring)\*\*으로 연결되어 있습니다. 이 링은 데이터가 저장될 수 있는 전체 공간을 나타내며, 각 노드는 링의 특정 부분(토큰 범위)을 책임지고 해당 범위의 데이터를 담당합니다.
* 파티션 (Partition) : 데이터의 \*\*파티션 키(Partition Key)\*\*에 의해 논리적으로 묶이는 데이터 그룹입니다. 동일한 파티션 키를 가진 모든 데이터는 같은 파티션에 속하며, 이 파티션은 링 위의 특정 토큰 범위에 매핑되어 해당 범위를 소유하는 노드에 저장됩니다.

즉, 하나의 클러스터 안에 여러 노드가 링 형태로 배치되어 있고, 각 노드는 링의 일부를 담당하며, 그 담당하는 부분에 해당하는 여러 파티션들의 데이터를 저장하는 구조입니다. 이때, 데이터의 고가용성을 위해 각 파티션의 데이터는 \*\*복제 요소(Replication Factor)\*\*에 따라 여러 노드에 복제되어 분산 저장됩니다.



### 장점

* 대규모 데이터 처리 능력: 수십 테라바이트에서 페타바이트 규모의 대용량 데이터를 효율적으로 처리할 수 있습니다.
* 뛰어난 확장성: 수평적 확장이 용이하여 클러스터의 중단 없이 규모를 확대하거나 축소할 수 있습니다.
* 높은 가용성: 여러 노드에 데이터가 복제되어 있어 노드 장애 시에도 서비스가 계속됩니다.
* 빠른 쓰기 및 읽기 성능: 대량의 데이터를 실시간으로 처리하고, 낮은 지연 시간으로 데이터를 읽고 쓸 수 있습니다.
* 유연한 데이터 모델링: 스키마리스(schema-less) 또는 유연한 스키마를 통해 빠르게 변화하는 비정형 데이터를 다루기에 적합합니다.

***



### 단점

* 높은 진입 장벽: 관계형 데이터베이스와는 다른 데이터 모델링 방식(쿼리 기반 모델링)과 분산 시스템에 대한 이해가 필요하여 학습 곡선이 높을 수 있습니다.
* 복잡한 조건 검색 제한: 주로 파티션 키를 통한 쿼리에 최적화되어 있어, 관계형 데이터베이스처럼 복잡한 조인이나 다중 조건 검색이 어렵습니다. (보조 인덱스를 사용할 수 있으나 전체 클러스터 스캔이 발생할 수 있어 대규모 환경에서는 지양됩니다.)
* 데이터 일관성 문제 (Eventual Consistency): 고가용성과 성능을 위해 최종 일관성(Eventual Consistency) 모델을 채택합니다. 이는 데이터가 모든 노드에 즉시 동기화되지 않을 수 있음을 의미합니다. 강력한 일관성이 필요한 트랜잭션에는 적합하지 않을 수 있습니다.
* 데이터 중복 가능성: 조인을 지원하지 않으므로, 애플리케이션의 쿼리 패턴에 맞춰 데이터를 여러 테이블에 중복 저장해야 할 수 있습니다. 이 경우 데이터 업데이트 시 여러 테이블을 수정해야 하는 번거로움이 있습니다.



### 데이터 모델

카산드라의 데이터 모델은 \*\*쿼리 중심적(Query-Driven)\*\*입니다. 관계형 데이터베이스의 ER(Entity-Relationship) 모델과는 달리, 애플리케이션에서 발생할 쿼리 패턴을 미리 분석하여 그에 최적화된 테이블 구조를 설계하는 것이 중요합니다.

기본적으로 카산드라는 키스페이스(Keyspace), 테이블(Table), 로우(Row), \*\*컬럼(Column)\*\*으로 구성됩니다. 특히 중요한 개념은 다음과 같습니다.

* 파티션 키 (Partition Key): 데이터를 클러스터의 어떤 노드에 저장할지 결정하는 데 사용됩니다. 파티션 키가 같으면 같은 노드에 저장될 가능성이 높습니다.
* 클러스터링 키 (Clustering Key): 파티션 키 내에서 데이터를 정렬하는 데 사용됩니다. 쿼리 시 데이터를 효율적으로 찾을 수 있게 해줍니다.

관계형 데이터베이스처럼 조인(Join)을 지원하지 않으므로, 필요한 모든 필드를 하나의 테이블에 모아서 저장하는 비정규화(Denormalization) 전략을 주로 사용합니다. 이는 데이터 중복을 감수하더라도 읽기 성능을 극대화하기 위함입니다.





### 사용 사례

* 대규모 웹 및 클라우드 애플리케이션: 수많은 사용자가 동시에 접근하고 대량의 데이터를 처리해야 하는 서비스 (예: Facebook, Netflix, Apple 등)
* 분산 로깅 및 이벤트 처리: 실시간으로 발생하는 대량의 로그 데이터, 센서 데이터, 이벤트 데이터를 수집 및 저장하는 시스템
* 시계열 데이터 관리: 시간 순서로 정렬된 데이터 (예: IoT 센서 데이터, 주가 정보, 사용자 활동 로그 등)를 효율적으로 저장하고 분석하는 경우
* 추천 시스템: 사용자 행동 데이터를 기반으로 실시간 추천을 제공하는 시스템
* 채팅 서비스: Discord와 같은 대규모 메시징 플랫폼에서 수조 개의 메시지를 저장하고 관리하는 데 사용됩니다.





## Fan-out on Write

Fan-out on Write은 소셜 미디어 피드나 메시징 앱처럼 많은 구독자에게 빠르게 정보를 전달해야 하는 시스템에서 사용되는 데이터 쓰기 전략입니다. 말 그대로 "쓰기 시점에 (데이터가) 퍼져나간다(fan-out)"는 의미입니다.



### Fan-out on Write의 원리

이 방식의 핵심은 데이터가 생성되는 시점(쓰기 시점)에 해당 데이터를 받아볼 모든 수신자의 저장 공간(예: 개인 피드 테이블)에 미리 복사하여 저장해 두는 것입니다.

예를 들어, 트위터(현 X)의 경우를 생각해봅시다.

1. 사용자 A가 트윗을 작성합니다. (쓰기 이벤트 발생)
2. 이 트윗이 사용자 A의 팔로워 목록(예: 사용자 B, C, D)에 있는 모든 사람들에게 전달되어야 합니다.
3. "Fan-out on Write" 방식에서는 사용자 A의 트윗이 쓰기 시점에 사용자 B, 사용자 C, 사용자 D 각각의 개인 타임라인(피드)을 위한 저장소에 복사되어 미리 저장됩니다.
4. 사용자 B, C, D가 자신의 타임라인을 볼 때(읽기 요청), 시스템은 단순히 해당 사용자의 미리 채워진 피드 저장소에서 데이터를 읽어와서 보여주기만 하면 됩니다. 별도의 복잡한 연산이나 조인이 필요 없습니다.



### Fan-out on Write의 장점

* 빠른 읽기 성능: 데이터를 읽는 시점에는 이미 모든 데이터가 개인 피드에 정리되어 있으므로, 단순한 읽기 작업만 수행하면 됩니다. 이는 사용자에게 매우 빠른 응답 시간을 제공하여 쾌적한 사용 경험을 보장합니다. 실시간 피드 서비스에 특히 중요합니다.
* 간단한 읽기 로직: 읽기 시점에 복잡한 조인이나 연산이 필요 없어 데이터베이스의 부하가 줄어들고, 애플리케이션 로직도 단순해집니다.
* 고가용성: 쓰기 시점에 데이터가 여러 곳에 복제되므로, 일부 데이터베이스 노드에 문제가 발생하더라도 다른 복제본을 통해 데이터를 제공할 수 있어 서비스 중단 위험이 낮습니다.



### Fan-out on Write의 단점 및 고려사항

* 높은 쓰기 비용: 데이터가 생성될 때마다 팔로워 수만큼 쓰기 작업이 발생하므로, 팔로워가 많은 사용자(예: 인플루언서)의 경우 엄청난 양의 쓰기 부하가 발생할 수 있습니다.
* 데이터 중복: 같은 데이터가 여러 사용자의 피드 저장소에 중복으로 저장됩니다. 이는 스토리지 비용 증가로 이어질 수 있습니다.
* 일관성 문제: 모든 팔로워의 피드에 동시에 데이터가 완벽하게 기록되지 않을 수 있습니다 (최종 일관성). 일부 팔로워는 다른 팔로워보다 몇 초 늦게 새 글을 보게 될 수도 있습니다. 소셜 미디어 피드에서는 허용 가능한 수준이지만, 금융 거래와 같은 강력한 일관성이 필요한 시스템에는 적합하지 않습니다.
* 삭제 및 수정의 복잡성: 특정 게시물을 삭제하거나 수정할 경우, 해당 게시물이 미리 복사된 모든 팔로워의 피드 저장소에서 찾아 삭제하거나 수정해야 하므로, 작업이 복잡해지고 추가적인 쓰기 부하가 발생할 수 있습니다.





## Twitter에서의 Cassandra & Fan-out on Write

Fan-out on Write은 게시물이 생성될 때 팔로워들의 피드에 미리 써두는 방식이므로, 단일 쓰기 작업이 다수의 쓰기 작업으로 "퍼져나가는" 특성을 가집니다. 카산드라는 이러한 요구사항에 최적화되어 있습니다.



Fan-out on Write 방식에서 카산드라가 유리한 이유는 주로 압도적인 쓰기 성능, 뛰어난 수평 확장성, 높은 가용성, 그리고 유연한 데이터 모델링 능력 때문입니다.

***

### 📈 Fan-out on Write과 카산드라의 적합성

Fan-out on Write은 게시물이 생성될 때 팔로워들의 피드에 미리 써두는 방식이므로, 단일 쓰기 작업이 다수의 쓰기 작업으로 "퍼져나가는" 특성을 가집니다. 카산드라는 이러한 요구사항에 최적화되어 있습니다.

#### 1. 압도적인 쓰기 성능 💪

Fan-out on Write은 쓰기 부하가 매우 높습니다. 예를 들어, 100만 명의 팔로워를 가진 사용자가 트윗 하나를 작성하면, 해당 트윗은 100만 개의 각 팔로워 타임라인에 기록되어야 합니다.

* 분산 쓰기 최적화: 카산드라는 분산 아키텍처를 통해 여러 노드에 데이터를 동시에 병렬로 쓸 수 있습니다. 이는 단일 노드의 쓰기 한계를 뛰어넘어 대규모 동시 쓰기 요청을 효율적으로 처리할 수 있게 합니다.
* LSM-Tree 기반 스토리지 엔진: 카산드라의 내부 스토리지 엔진은 LSM-Tree(Log-Structured Merge-Tree) 기반으로 작동합니다. 이는 쓰기 작업이 주로 메모리 내에서 이루어지고 디스크에는 순차적으로 기록되므로, 무작위 쓰기가 많은 관계형 데이터베이스에 비해 쓰기 성능이 월등히 빠릅니다.

***

#### 2. 뛰어난 수평 확장성 🚀

트위터와 같이 사용자 수와 데이터 양이 폭발적으로 증가하는 서비스는 유연한 확장성이 필수적입니다.

* 선형적 확장: 카산드라는 클러스터에 새로운 노드를 추가하는 것만으로 거의 선형적으로 쓰기 및 읽기 처리량을 늘릴 수 있습니다. 이는 Fan-out on Write 방식에서 발생하는 지속적인 쓰기 부하 증가에 효과적으로 대응할 수 있게 합니다.
* 단일 장애점 없음 (Masterless): 중앙 마스터 노드가 없기 때문에, 마스터 노드가 병목 현상을 일으키거나 장애가 발생하여 시스템 전체에 영향을 주는 일이 없습니다.

***

#### 3. 높은 가용성 및 내결함성 🛡️

서비스가 24시간 중단 없이 운영되어야 하는 대규모 시스템에서는 데이터베이스의 가용성이 매우 중요합니다.

* 데이터 복제: 카산드라는 데이터를 여러 노드에 복제(Replication Factor)하여 저장합니다. 이는 특정 노드에 장애가 발생하더라도 데이터에 계속 접근할 수 있도록 보장합니다. Fan-out on Write 시 다수의 쓰기 작업 중 일부 노드에 문제가 생겨도, 설정된 일관성 수준만 충족되면 쓰기 작업은 성공적으로 완료됩니다.

***

#### 4. 유연한 데이터 모델링 (비정규화 최적화) 🤸

Fan-out on Write 방식은 각 팔로워의 피드 테이블에 데이터를 중복 저장하는 비정규화(Denormalization) 전략을 사용합니다.

* NoSQL의 강점: 카산드라는 관계형 데이터베이스와 달리 복잡한 조인을 지원하지 않는 NoSQL 데이터베이스입니다. 따라서 애초에 Fan-out on Write처럼 비정규화된 데이터 모델에 최적화되어 있습니다. 각 팔로워의 타임라인을 별도의 파티션으로 관리하기에 매우 적합합니다.

***

#### 5. 조정 가능한 일관성 (Tunable Consistency) ⚖️

Fan-out on Write 방식은 읽기 시점의 일관성보다 쓰기 시점의 성능과 가용성을 더 중요하게 여깁니다.

* 유연한 일관성 모델: 카산드라는 `ONE`, `QUORUM`, `ALL` 등 사용자가 쿼리별로 일관성 수준을 조정할 수 있는 기능을 제공합니다. 트위터 피드처럼 약간의 지연이 허용되는 시나리오에서는 `ONE`이나 `QUORUM` 같은 느슨한 일관성 수준을 선택하여 최대의 쓰기 성능과 가용성을 확보할 수 있습니다.

이러한 카산드라의 특징들이 Fan-out on Write 아키텍처의 요구사항과 완벽하게 일치하므로, 트위터와 같은 대규모 소셜 미디어 서비스에서 매우 효과적으로 사용될 수 있었습니다.

***

### 📊 카산드라 테이블 예시

카산드라 클러스터에는 다음과 같이 두 개의 테이블이 있다고 가정합니다.

#### 1. `user_tweets` 테이블 (사용자별 모든 트윗 저장)

* 용도: 각 사용자가 작성한 모든 트윗의 원본 기록을 저장합니다.
* 스키마:
  * `user_id` (Text, 파티션 키): 트윗을 작성한 사용자의 ID
  * `tweet_id` (TimeUUID, 클러스터링 키): 각 트윗의 고유 ID (시간 순으로 정렬됨)
  * `tweet_text` (Text): 트윗 내용
  * `timestamp` (Timestamp): 트윗 작성 시간
  * `media_url` (Text, Nullable): 첨부 미디어 URL

| user\_id (PK) | tweet\_id (CK)     | tweet\_text | timestamp             |
| ------------- | ------------------ | ----------- | --------------------- |
| `user_A`      | `d9f...` (첫 번째 트윗) | "안녕하세요!"    | `2025-07-13 19:00:00` |
| `user_A`      | `e1a...` (세 번째 트윗) | "오늘 날씨 좋네요" | `2025-07-13 19:10:00` |
| `user_B`      | `dff...` (두 번째 트윗) | "점심 뭐 먹지?"  | `2025-07-13 19:05:00` |

#### 2. `user_timeline` 테이블 (팔로워별 개인 피드 저장)

* 용도: 각 팔로워의 홈 타임라인에 표시될 트윗들을 미리 저장합니다 (Fan-out on Write의 핵심).
* 스키마:
  * `follower_id` (Text, 파티션 키): 이 타임라인을 볼 사용자의 ID
  * `timestamp` (Timestamp, 클러스터링 키): 트윗 작성 시간 (최신 순 정렬)
  * `author_id` (Text): 트윗을 작성한 원본 사용자의 ID
  * `tweet_id` (TimeUUID): 원본 트윗의 고유 ID
  * `tweet_text` (Text): 트윗 내용
  * `media_url` (Text, Nullable): 첨부 미디어 URL

| follower\_id (PK) | timestamp (CK)        | author\_id | tweet\_id                  | tweet\_text |
| ----------------- | --------------------- | ---------- | -------------------------- | ----------- |
| `user_B`          | `2025-07-13 19:10:00` | `user_A`   | `e1a...` (User A의 세 번째 트윗) | "오늘 날씨 좋네요" |
| `user_B`          | `2025-07-13 19:00:00` | `user_A`   | `d9f...` (User A의 첫 번째 트윗) | "안녕하세요!"    |
| `user_C`          | `2025-07-13 19:10:00` | `user_A`   | `e1a...` (User A의 세 번째 트윗) | "오늘 날씨 좋네요" |
| `user_C`          | `2025-07-13 19:05:00` | `user_B`   | `dff...` (User B의 두 번째 트윗) | "점심 뭐 먹지?"  |
| `user_C`          | `2025-07-13 19:00:00` | `user_A`   | `d9f...` (User A의 첫 번째 트윗) | "안녕하세요!"    |



### 트윗이 게시될 때의 순차적 흐름 (Fan-out on Write)

이제 `user_A`가 새로운 트윗 (`tweet_4`: "저녁 먹으러 가자!")을 작성하고, `user_A`를 `user_B`와 `user_C`가 팔로우하고 있다고 가정해봅시다.

* 사용자 A가 트윗을 작성합니다.
  * 클라이언트가 카산드라에 쓰기 요청을 보냅니다.
* `user_tweets` 테이블에 트윗 원본 저장 (1번 쓰기)
  * `user_tweets` 테이블에 `user_A`의 새 트윗이 추가됩니다.
* 팔로워 목록 조회
  * 백엔드 서비스는 `user_A`를 팔로우하는 모든 사용자의 ID를 조회합니다. 이 정보는 별도의 `followers_by_user` 또는 `user_followers` 같은 테이블에 저장되어 있을 수 있습니다 (예시에서는 `user_B`, `user_C`라고 가정).
* `user_timeline` 테이블에 트윗 Fan-out (팔로워 수만큼 쓰기)
  * 조회된 팔로워(`user_B`, `user_C`) 각각의 `user_timeline` 파티션에 `user_A`의 새 트윗이 복사되어 저장됩니다.
  * `user_B`의 타임라인에 쓰기,`user_C`의 타임라인에 쓰기

| follower\_id (PK) | timestamp (CK)        | author\_id | tweet\_id         | tweet\_text  |
| ----------------- | --------------------- | ---------- | ----------------- | ------------ |
| `user_B`          | `2025-07-13 19:15:00` | `user_A`   | `f0c...` (새로운 트윗) | "저녁 먹으러 가자!" |
| `user_B`          | `2025-07-13 19:10:00` | `user_A`   | `e1a...`          | "오늘 날씨 좋네요"  |
| `user_B`          | `2025-07-13 19:00:00` | `user_A`   | `d9f...`          | "안녕하세요!"     |
| `user_C`          | `2025-07-13 19:15:00` | `user_A`   | `f0c...` (새로운 트윗) | "저녁 먹으러 가자!" |
| `user_C`          | `2025-07-13 19:05:00` | `user_B`   | `dff...`          | "점심 뭐 먹지?"   |
| `user_C`          | `2025-07-13 19:00:00` | `user_A`   | `d9f...`          | "안녕하세요!"     |



## 읽기 성능 최적화

캐싱은 `user_timeline` 테이블에서 특정 `follower_id`에 해당하는 파티션의 일부 또는 전체를 메모리에 올려두는 개념입니다.

1. 캐시 저장 단위:
   * 캐시는 보통 `user_timeline:{follower_id}`와 같은 키를 사용하며, 값으로는 해당 팔로워의 \*\*최신 트윗 목록(예: 최신 100개, 200개 등)\*\*을 저장합니다. 이 목록은 `timestamp`를 기준으로 정렬되어 있습니다.
   * 예시: `user_B`의 타임라인 캐시에는 `2025-07-13 19:15:00`부터 `2025-07-13 18:00:00`까지의 트윗 100개가 저장되어 있을 수 있습니다.
2. 초기 타임라인 로드 (새로고침):
   * 사용자 B가 타임라인을 새로고침하면, 애플리케이션은 `user_timeline:user_B` 캐시 키로 데이터를 조회합니다.
   * 캐시 히트: 캐시에 최신 트윗 목록이 있다면, 그중 최신 20개를 사용자에게 보여줍니다.
   * 캐시 미스: 캐시에 데이터가 없거나, 캐시가 무효화되었다면, 카산드라에서 최신 20개(또는 더 많은 개수)의 트윗을 가져와 캐시에 저장하고 사용자에게 보여줍니다.



프론트엔드에서 `cursor` (주로 `timestamp`)를 매개변수로 보내는 것은 "이 `timestamp` 이전의 트윗을 보여줘"라는 의미입니다.

* 커서 기반 쿼리 예시:
  * 사용자 B가 스크롤을 내려서 현재 보이는 트윗 중 가장 오래된 트윗의 `timestamp`가 `2025-07-13 18:30:00`이라고 가정합니다.
  * 프론트엔드는 다음 20개의 트윗을 가져오기 위해 `timestamp < '2025-07-13 18:30:00'` 조건을 포함하여 요청을 보냅니다.
* 캐시 히트/미스 판단:
  * 애플리케이션 서버는 이 요청을 받으면, `user_timeline:user_B` 캐시를 확인합니다.
  * 캐시 히트: 만약 캐시된 100개 또는 200개의 트윗 목록 안에 `2025-07-13 18:30:00` 이전의 데이터가 충분히 포함되어 있다면, 캐시에서 해당 범위의 20개 트윗을 찾아 사용자에게 반환합니다. 이 경우에도 카산드라에 접근할 필요가 없습니다.
  * 캐시 미스: 캐시된 범위(`2025-07-13 19:15:00` \~ `2025-07-13 18:00:00`)를 벗어나는 더 오래된 데이터(`2025-07-13 17:50:00`부터의 트윗)를 요청하는 경우 캐시 미스가 발생합니다. 이때는 카산드라로 직접 쿼리를 보내 데이터를 가져옵니다.

***

### 🔄 캐시 무효화와 갱신

* 새로운 트윗 발생 시:
  * `user_A`가 새 트윗을 작성하고, 이 트윗이 `user_B`의 `user_timeline` 파티션에 Fan-out 되어 저장됩니다.
  * 이때, `user_timeline:user_B` 캐시는 \*\*무효화(invalidate)\*\*됩니다.
  * `user_B`가 다음에 타임라인을 새로고침하거나 스크롤하여 데이터를 요청하면, 캐시 미스가 발생하고 카산드라에서 새로운 트윗을 포함한 최신 데이터 범위를 가져와 캐시를 갱신합니다.

따라서 캐시는 특정 시점의 고정된 데이터가 아니라, \*\*사용자가 가장 자주 접근하는 최신 데이터의 '범위'\*\*를 저장하여 읽기 성능을 최적화하고, 커서 방식은 이 캐시된 범위 내에서 효율적으로 데이터를 탐색하거나, 캐시 범위를 벗어나는 경우 데이터베이스로 직접 접근하는 방식으로 동작합니다.
