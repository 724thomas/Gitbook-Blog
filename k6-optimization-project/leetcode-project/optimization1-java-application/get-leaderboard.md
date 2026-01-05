---
description: 리더보드 조회 API 최적화
---

# GET Leaderboard

### 📙 리더보드 조회 API

**`GET /contests/{contest_id}/leaderboard`**

* ✅ 1. **DB 인덱스 최적화:** contestId, score 컬럼 인덱스 추가

#### 가설:

* 현재 leaderboard 조회 시 `contest_id`를 조건으로 주고 `score DESC`로 정렬하는 쿼리의 실행 성능이 낮은 것으로 확인되었습니다.
* 기존에는 `contest_id`에 대한 FK 인덱스만 존재하여 데이터 필터링 후 정렬이 발생합니다.
* MySQL의 경우, 정렬(`ORDER BY`)을 효율적으로 처리하기 위해서는 **정렬 컬럼이 포함된 인덱스**가 필요합니다.
* 따라서 `contest_id`와 `score DESC`를 포함하는 **복합 인덱스**를 생성하여 쿼리가 인덱스만으로 정렬까지 처리하도록 최적화하면 성능이 크게 향상될 것으로 예상됩니다.
* 복합 인덱스 추가로 leaderboard 조회 성능이 크게 개선될 것으로 기대됩니다.

<details>

<summary>MySQL 인덱스 동작 원리</summary>

* 단일 컬럼 인덱스(`contest_id`)만으로는 정렬 조건(`score DESC`)이 인덱스에 포함되지 않아 filesort 단계가 발생.
* `contest_id`와 `score DESC`를 함께 포함한 복합 인덱스를 사용하면 정렬 작업을 인덱스 레벨에서 해결하여 정렬 비용을 줄이고 성능을 향상시킬 수 있습니다.
* 일반적으로 MySQL의 인덱스 정렬 처리 비용은 \*\*O(log n)\*\*로, 인덱스를 사용한 정렬이 디스크 정렬보다 훨씬 빠릅니다.

</details>



#### 실험/과정:

**현재:**

SQL Plan 확인

* FK(`contest_id`) 인덱스를 사용하여 데이터를 필터링한 뒤 정렬 처리를 하고 있습니다.

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>SQL plan</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>EXPLAIN ANALYZE 결과</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Application 단 API 성능</p></figcaption></figure>

* DB 조회 시간: 약 **100ms**
* 총 서비스 호출 시간: **286ms (DB 조회, 네트워크 통신, 애플리케이션 처리시간을 모두 포함)**



**개선 후:**

<figure><img src="../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption><p>SQL Plan</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption><p>EXPLAIN ANALYZE 결과</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption><p>Application 단 API 성능</p></figcaption></figure>

* Plan 상에서는 FK 인덱스와 유사해 보이나 실제 실행 시 정렬 처리가 인덱스를 통해 최적화됨.
  * FK 인덱스만으로도 Plan은 비슷하게 출력되지만, **복합 인덱스**로 인해 `ORDER BY` 최적화가 이루어져 정렬 비용이 크게 줄어듦.
* DB 조회 시간: 약 **5.45ms**
* 총 서비스 호출 시간: **31ms**

#### 평가:

* 인덱스 추가 전이랑 추가 후 성능 비교 평가
  * 서비스 호출 시간: 인덱스 적용 전 **286ms** → 인덱스 추가 후 **31ms**로 응답 속도 대폭 개선
  * DB 조회 시간: 인덱스 적용 전 **100ms** → 인덱스 추가 후 **5.45ms**로 대폭 단축
  * 복합 인덱스 설계로 인해 `ORDER BY` 최적화 효과 발생 → 불필요한 정렬 연산 감소

***

* ✅ 2. **Redis Sorted Set 적용:** 상위 50명만 유지하는 구조를 사용하여, 빠른 조회 및 갱신이 가능하도록 할 수 있습니다.

#### 가설:

* 리더보드를 Redis ZSet에 유지하면, 반복적인 상위 50명 조회 요청에 대해 빠른 응답을 제공할 수 있을 것 같습니다.
* Redis의 Sorted Set은 Skip List 기반 자료구조로 `O(log n)`의 삽입/삭제 성능을 가지며, `reverseRangeWithScores(0, 49)`를 통해 효율적으로 상위 랭커를 반환할 수 있습니다.
* 상위 50명의 리스트만 Redis에 유지하고, `addScore()` 시점마다 51위 이하를 제거하는 방식으로 메모리 사용량을 제어할 수 있습니다.

<details>

<summary>Skip List</summary>

기본적으로 여러 개의 연결 리스트(linked list) 층이 있으며, **중간 노드들을 가리키는 추가 포인터**들을 포함해 효율적인 순회와 탐색이 가능합니다. 스킵 리스트는 기능적으로 **균형 이진 탐색 트리(Balanced BST)** 와 유사하지만, **연결 리스트와 난수(randomization)** 를 사용해 구현된다는 점이 다릅니다. 탐색, 삽입, 삭제의 평균 시간 복잡도는 **O(log n)** 입니다. 스킵 리스트는 전체 원소 수를 M이라 할 때, **상위 N개 요소를 반환하는 연산은 O(N × log M)** 의 시간 복잡도로 처리할 수 있기 때문에 적합합니다.

</details>

#### 실험/과정:

**현재:**

```json
[66eba4b1] LeaderBoardController.getLeaderBoardFromRdb(..)
[66eba4b1] |-->LeaderBoardService.getLeaderBoardInfo(..)
[66eba4b1] |   |-->LeaderBoardRepository.getTop50LeaderBoardByContestId(..)
[66eba4b1] |   |<--LeaderBoardRepository.getTop50LeaderBoardByContestId(..) time=62ms
[66eba4b1] |<--LeaderBoardService.getLeaderBoardInfo(..) time=64ms
[66eba4b1] LeaderBoardController.getLeaderBoardFromRdb(..) time=71ms

[6ea56b6c] LeaderBoardController.getLeaderBoardFromRdb(..)
[6ea56b6c] |-->LeaderBoardService.getLeaderBoardInfo(..)
[6ea56b6c] |   |-->LeaderBoardRepository.getTop50LeaderBoardByContestId(..)
[6ea56b6c] |   |<--LeaderBoardRepository.getTop50LeaderBoardByContestId(..) time=24ms
[6ea56b6c] |<--LeaderBoardService.getLeaderBoardInfo(..) time=24ms
[6ea56b6c] LeaderBoardController.getLeaderBoardFromRdb(..) time=29ms
```

`요청1`은 71ms, `요청2`는 29ms가 걸렸습니다.

두번째 요청은 빠르게 처리 되고 있는데, 애플리케이션에서 캐시 관련 로직은 존재하지 않지만, DB 자체의 쿼리 캐시 효과가 있습니다.

* `요청1`: 디스크 → DB 읽기 → 응답
* `요청2`: InnoDB buffer Pool → 메모리 히트 → 응답

**개선된 후:**

```json
[da6bae17] |-->LeaderBoardService.getLeaderBoardWithCache(..) args=[1]
[da6bae17] |   |-->LeaderBoardRedisService.getTopN(..) 
[da6bae17] |   |<--LeaderBoardRedisService.getTopN(..) time=26ms
[da6bae17] |   |-->LeaderBoardRepository.getTop50LeaderBoardByContestId(..) args=[1]
[da6bae17] |   |<--LeaderBoardRepository.getTop50LeaderBoardByContestId(..) args=[1] time=61ms
[da6bae17] |   |-->LeaderBoardRedisService.addScore(..) 
[da6bae17] |   |<--LeaderBoardRedisService.addScore(..) time=2ms
[da6bae17] |   |-->LeaderBoardRedisService.addScore(..) 
[da6bae17] |   |<--LeaderBoardRedisService.addScore(..) time=0ms
...
[da6bae17] |   |<--LeaderBoardRedisService.addScore(..) time=0ms
[da6bae17] |<--LeaderBoardService.getLeaderBoardWithCache(..) time=95ms
[da6bae17] LeaderBoardController.getLeaderBoardWithCache(..) time=102ms
[f6134134] LeaderBoardController.getLeaderBoardWithCache(..)
[f6134134] |-->LeaderBoardService.getLeaderBoardWithCache(..)
[f6134134] |   |-->LeaderBoardRedisService.getTopN(..)
[f6134134] |   |<--LeaderBoardRedisService.getTopN(..) time=2ms
[f6134134] |<--LeaderBoardService.getLeaderBoardWithCache(..) time=3ms
[f6134134] LeaderBoardController.getLeaderBoardWithCache(..) time=17ms

```

Redis의 ZSet를 적용했을때 결과입니다.

요청1은 102ms, 요청2는 17ms가 걸렸습니다.

* `요청1`: 애플리케이션 → 레디스 (캐시 미스) → 디스크 → DB 읽기 → 레디스 저장 → 응답
* `요청2`: 애플리케이션 → 레디스 (캐시 히트) → 응답

#### 평가:

* **기존 RDB 단독 조회 방식**
  * 첫 번째 요청은 InnoDB 버퍼 풀에 적재되지 않은 상태이므로 디스크 I/O가 발생 → **71ms 소요**
  * 두 번째 요청부터는 DB 메모리 캐시(InnoDB Buffer Pool)에 의해 **29ms**로 응답 속도 개선
  * 그러나 여전히 요청마다 **DB I/O 부하가 발생**하므로, 트래픽 증가 시 병목 가능성 존재
* **Redis Sorted Set 캐시 적용 방식**
  * 첫 요청(캐시 미스)은 Redis + DB를 모두 거쳐 **102ms**로 약간 지연됨
  * 이후 요청(캐시 히트)은 Redis에서 즉시 응답 → **17ms**로 대폭 단축
  * Redis는 ZSet + Skip List 기반 구조로 `reverseRangeWithScores`를 통해 **상위 N개 데이터를 O(N) 수준으로 매우 빠르게 조회** 가능
  * **ZSet 유지 전략**도 적절히 설계

#### 결론:

* 시스템의 초기 단계에서는 데이터베이스에 인덱스를 추가하여 검색 성능을 최적화합니다.
* 인덱스를 통해 일정 수준까지는 데이터베이스 부하를 효과적으로 줄일 수 있겠지만, 사용자 수가 증가하면 인덱스만으로는 부족할 수 있습니다. 이에 따라 Redis를 활용해 데이터를 캐싱하고, 자주 조회되는 요청을 빠르게 처리하도록 시스템을 개선했습니다.
* 이러한 단계적 접근 방식은 초기 단계에서 불필요한 오버엔지니어링을 피하면서도, 사용량 증가에 따라 적시에 캐시 도입을 통해 성능을 보장할 수 있는 구조를 마련할 수 있었습니다.
