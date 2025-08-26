---
description: 일관성 해싱
---

# Consistent Hashing

## 🎯 일관성 해싱 (보강 버전 Part 1)

### 1. 왜 해싱이 필요할까?

* 서버가 여러 대 → 데이터를 나눠 담아야 함
* 단순 방식: `key % 서버개수` (예: 손님 번호 % 마트 개수)
* 문제: 서버 하나 추가/삭제 시 **전체 데이터 재분배 발생**
* 실제로 수백만 건 데이터가 전부 흔들리면 → 서비스 멈춤/대규모 지연 위험 🚨
* 운영 환경에서는 **거의 불가능한 방식**

***

### 2. 해시 링(Hash Ring) 비유

* 원형 시계판에 서버 = 마트, 데이터 = 손님을 배치
* 손님은 **자기보다 시계방향으로 가장 가까운 마트**로 이동
* 해시 함수: 데이터 → 숫자 → 링 위 위치 변환
* 장점: 구조가 단순하면서도 **유연하고 안정적**

***

### 3. 노드 추가/삭제 시 변화

* 새 서버 등장 → **해당 구간 손님만** 새 서버로 이동
* 서버 하나 제거 → 그 서버 손님만 옆 서버로 이동
* 전체 재분배 ❌, 일부 이동 ⭕
* 결과: 확장성·장애 대응력이 뛰어남
* 운영 관점: 서버 증설 시에도 서비스 중단이 거의 없음

***

### 4. 가상 노드(Virtual Node)

* 현실: 서버 성능·용량은 제각각
* 단순 배치 시 특정 서버에 데이터 몰려 **Hotspot(핫스팟)** 발생
* 해결책: 하나의 물리 서버를 여러 가상 노드로 쪼개서 링에 흩뿌림
* 마치 대형 마트를 작은 체인점 여러 개로 나눠 동네 곳곳에 배치하는 느낌
* 효과: 데이터 분산 균형 ↑, 병목 ↓

***

### 5. 실제 서비스 활용

* **Redis Cluster**: 키 → CRC16 해시 → 16384 슬롯 → 서버 매핑
* **Cassandra**: 파티션 키 → 토큰 링 → 노드 분산
* **Memcached**: 서버 증설/삭제에도 데이터 최소 이동으로 안정 운영
* 공통점: “일관성 해싱 원리”를 각 서비스에 맞게 변형
* 결론: 단순한 이론이 아니라 **실무 핵심 기술**

***

### 6. 개발자가 얻는 이점

* 서버 확장(스케일 아웃)이 두렵지 않음
* 장애 발생 시 전체 재분배가 아닌 **일부만 이동**
* 데이터가 고르게 퍼져 특정 서버 병목 방지
* 운영 비용 절감 + 서비스 안정성 확보
* 실무에서 “캐시, 세션, DB 파티셔닝” 문제를 풀어주는 열쇠

***

### 7. 단점도 있다

* 완벽히 균등 분산 ❌ → 여전히 Hotspot 가능
* 해시 함수 선택에 따라 성능 차이 발생 (예: SHA1 vs MurmurHash)
* 가상 노드 설계/운영이 추가 관리 포인트
* 모니터링 없으면 **특정 노드 과부하**를 놓치기 쉬움
* 즉, “도입만 하면 끝”은 아님 → 운영 지식 필요

***

### 8. 대안 알고리즘도 있다

* **Rendezvous Hashing(HRW)**: 키마다 모든 서버와 점수 계산 → 최고 점수 서버 선택, 구현 단순
* **Weighted Hashing**: 서버 성능에 맞춰 가중치 적용 가능
* **Jump Consistent Hash**: 최근 등장, 계산 빠르고 균형 분배 우수
* 상황별로 장단점 존재 → 서비스 특성에 맞는 선택 필요

***

### 9. 그림으로 정리하기

* **해시 링 기본 구조**: 서버와 데이터가 원형에 배치
* **노드 추가/삭제 전후**: 일부만 이동하는 모습 강조
* **가상 노드**: 큰 서버가 여러 점으로 흩어져 있는 그림
* **Hotspot 예시**: 특정 구간에 데이터가 몰려 과부하 발생
* 시각 자료가 있으면 초보자도 직관적으로 이해 가능

***

### 10. 마무리: 이걸 알면 뭐가 좋을까?

* 일관성 해싱은 “확장성과 유연성”을 제공하는 핵심 아이디어
* 단순 수학 문제가 아니라 **대규모 서비스 운영 필수 지식**
* 캐시, 세션 저장소, DB 파티셔닝, 로드밸런서 설계 등 광범위하게 활용됨
* 실무 메시지: **규모가 커지는 순간 반드시 마주치는 문제** → 알아두면 필살기

***

### 11. Redis Cluster의 슬롯 개념

* Redis는 데이터를 **16,384개의 슬롯(slot)** 으로 쪼갬
* 키 → `CRC16 해시` → `mod 16384` → 슬롯 번호 결정
* 각 슬롯은 특정 노드가 맡아서 저장/관리
* 노드가 추가/삭제돼도 → 슬롯만 재배치하면 됨
* 👉 “큰 창고를 작은 칸으로 나눠두면, 옮길 때 일부 칸만 움직이면 되는 구조”

***

### 12. 슬롯과 일관성 해싱 비교

* **Consistent Hashing**: 원형 링 위에서 노드 배치 → 유연하지만 구현 복잡
* **Redis Cluster**: 미리 정해진 슬롯 16,384개 → 단순, 예측 가능
* 장점: 리밸런싱 범위가 명확, 관리 쉬움
* 단점: 슬롯 수는 고정 → 동적 확장 불가
* 👉 “원형 링(자유로운 지도)” vs “칸 나눠진 룰렛판(정해진 틀)”

***

### 13. 노드 추가/삭제 시 슬롯 재분배

* 새 노드 추가 → 기존 노드가 가진 **일부 슬롯만 양도**
* 노드 삭제 → 그 노드 슬롯을 다른 노드가 인수
* 전체 데이터 이동 ❌, 슬롯 단위 일부 이동 ⭕
* 👉 “마트 점포가 늘어나면 주변 물건 몇 박스만 새 점포로 옮기는 느낌”

***

### 14. 마스터-슬레이브 구조와 고가용성

* 각 슬롯 = 기본적으로 **마스터 노드** 담당
* 마스터마다 **슬레이브 노드**(복제본) 붙음
* 마스터 장애 → 슬레이브 자동 승격
* 데이터 손실은 최소화, 서비스는 계속 진행
* 👉 “마트 본점(마스터) + 지점(슬레이브). 본점이 닫아도 지점이 바로 가게 열어줌”

***

### 15. 데이터 조회 시 해시 태그(Hash Tag)

* 키 안에 `{}`를 쓰면 그 안 문자열만 해시 계산
* 예: `{user:1000}:name`, `{user:1000}:tweets` → 같은 슬롯으로 묶임
* 덕분에 멀티키 연산/트랜잭션 가능
* 👉 “같은 박스에 담아놔야 같이 꺼낼 수 있음”

***

### 16. 클라이언트-서버 간 키 라우팅

* 클라이언트가 직접 키 → 슬롯 → 노드 계산
* 클러스터 구성이 바뀌면 서버가 `MOVED`, `ASK` 신호를 보내 새 노드 알려줌
* 클라이언트는 다시 요청을 재전송 → 최신 구조 따라감
* 👉 “택배 기사님이 이사 간 집 주소 알려주는 안내판”

***

### 17. 실제 장애 상황 시 동작

* 노드 다운 → 해당 슬롯의 슬레이브가 자동 승격
* 잠깐의 지연은 있을 수 있지만, 데이터 유실 최소화
* 심각한 장애 → 관리자가 `redis-cli cluster`로 수동 조정
* 👉 “자동 대체 시스템이 있지만, 최악의 경우 매니저가 직접 개입”

***

### 18. Redis Cluster의 장단점

**장점**

* 구조 단순, 관리 예측 가능
* 빠른 확장성과 높은 가용성
* 일관성 해싱의 장점을 실제로 구현

**단점**

* 슬롯 개수 고정(16384) → 동적 확장 ❌
* 멀티키 연산은 같은 슬롯에서만 가능
* 특정 슬롯에 트래픽 몰리면 **핫스팟 문제** 발생
* 👉 “깔끔한 룰렛판이지만, 룰렛판 크기를 바꾸긴 힘듦”

***

### 19. Redis Cluster vs Consistent Hashing 총정리

* **공통점**: 노드 변경 시 전체가 아닌 일부만 이동
* **차이점**
  * Consistent Hashing → 원형 링, 유연함
  * Redis Cluster → 고정 슬롯, 단순/관리 쉬움
* Redis가 슬롯 방식을 택한 이유: 개발자·운영자 입장에서 실용적
* 👉 “철학은 같지만, 구현은 현실적으로 간소화”

***

### 20. 실무 적용 인사이트

* **Redis Cluster**는 캐시/세션 저장소로 많이 사용
* Hash Tag 설계를 잘못하면 → 멀티키 연산 불가 → 큰 문제 발생
* 장애 대응, 슬롯 재분배 전략을 사전에 연습해 두는 게 중요
* 결론: Redis Cluster는 단순 DB가 아니라 **확장성과 안정성을 갖춘 분산 설계 도구**
* 👉 “그냥 창고가 아니라, 확장 계획까지 세워둔 스마트 창고”

***

## Redis Cluster: docker-compose로 5노드 시작 → 노드 추가/삭제 튜토리얼

### 0) 구성 개요

* **노드 수**: 5 (마스터 3 + 슬레이브 2)로 시작
* **포트**: 7001\~7005 (각 컨테이너 내부 6379에 매핑)
* **클러스터 생성**: `redis-cli --cluster create ... --cluster-replicas 1`
* **확장(추가)**: 새 컨테이너 띄운 뒤 `add-node` → (원하면) 리밸런싱(reshard)
* **축소(삭제)**: 대상 노드의 슬롯을 다른 노드로 옮긴 뒤 `del-node`

***

### 1) docker-compose.yml (5노드 시작)

> Bitnami 이미지로 간단하게. 컨테이너 **호스트네임**을 클러스터에 알리도록 설정해 재분배/리다이렉트가 잘 동작하게 함.

```yaml
version: "3.9"
services:
  redis-node-1:
    image: bitnami/redis-cluster:7.2
    container_name: redis-node-1
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - REDIS_PASSWORD=
      - REDIS_NODES=redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5
      - REDIS_CLUSTER_ANNOUNCE_HOSTNAME=redis-node-1
    ports: ["7001:6379"]
    networks: [redis-net]

  redis-node-2:
    image: bitnami/redis-cluster:7.2
    container_name: redis-node-2
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - REDIS_PASSWORD=
      - REDIS_NODES=redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5
      - REDIS_CLUSTER_ANNOUNCE_HOSTNAME=redis-node-2
    ports: ["7002:6379"]
    networks: [redis-net]

  redis-node-3:
    image: bitnami/redis-cluster:7.2
    container_name: redis-node-3
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - REDIS_PASSWORD=
      - REDIS_NODES=redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5
      - REDIS_CLUSTER_ANNOUNCE_HOSTNAME=redis-node-3
    ports: ["7003:6379"]
    networks: [redis-net]

  redis-node-4:
    image: bitnami/redis-cluster:7.2
    container_name: redis-node-4
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - REDIS_PASSWORD=
      - REDIS_NODES=redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5
      - REDIS_CLUSTER_ANNOUNCE_HOSTNAME=redis-node-4
    ports: ["7004:6379"]
    networks: [redis-net]

  redis-node-5:
    image: bitnami/redis-cluster:7.2
    container_name: redis-node-5
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - REDIS_PASSWORD=
      - REDIS_NODES=redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5
      - REDIS_CLUSTER_ANNOUNCE_HOSTNAME=redis-node-5
    ports: ["7005:6379"]
    networks: [redis-net]

networks:
  redis-net:
    driver: bridge
```

> **참고**
>
> * 비밀번호가 필요하면 `REDIS_PASSWORD=yourpass`를 모든 노드에 동일하게 지정하고, 아래 `redis-cli` 호출 시 `-a yourpass` 추가.
> * 로컬 포트를 700x로 나눈 건 보기 편하라고 한 것. 내부 포트는 항상 6379다.

***

### 2) 컨테이너 기동 & 클러스터 생성

```bash
# 컨테이너 기동
docker compose up -d

# 클러스터 생성(마스터/슬레이브 자동 배치, replicas=1)
docker exec -it redis-node-1 redis-cli --cluster create \
  redis-node-1:6379 redis-node-2:6379 redis-node-3:6379 \
  redis-node-4:6379 redis-node-5:6379 \
  --cluster-replicas 1
# 비밀번호 쓰는 경우: redis-cli -a yourpass --cluster create ...
```

* 위 명령이 끝나면 **마스터 3개 + 슬레이브 2개**로 자동 구성된다.
* 상태 확인:

```bash
docker exec -it redis-node-1 redis-cli cluster nodes
docker exec -it redis-node-1 redis-cli cluster info
```

***

### 3) (선택) 슬롯 분포/건강 체크

```bash
docker exec -it redis-node-1 redis-cli cluster slots
docker exec -it redis-node-1 redis-cli ping
```

* `cluster slots`로 **슬롯 범위(0\~16383)가 어떤 마스터에 할당**됐는지 확인.
* 마스터 각각에 5\~6천 슬롯 정도가 분산돼 있으면 정상.

***

### 4) **노드 추가**(확장): 5 → 6 노드

#### 4-1) compose에 새 서비스 추가

```yaml
  redis-node-6:
    image: bitnami/redis-cluster:7.2
    container_name: redis-node-6
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - REDIS_PASSWORD=
      - REDIS_NODES=redis-node-1 redis-node-2 redis-node-3 redis-node-4 redis-node-5 redis-node-6
      - REDIS_CLUSTER_ANNOUNCE_HOSTNAME=redis-node-6
    ports: ["7006:6379"]
    networks: [redis-net]
```

```bash
docker compose up -d redis-node-6
```

#### 4-2) 클러스터에 노드 합류(add-node)

```bash
# 새 노드를 '마스터'로 추가
docker exec -it redis-node-1 redis-cli --cluster add-node \
  redis-node-6:6379 redis-node-1:6379
# (비밀번호 있으면 -a yourpass)
```

* 이 시점엔 **슬롯을 아직 받지 않은 빈 마스터** 상태.

#### 4-3) 리밸런싱(슬롯 재분배, reshard)

```bash
# 자동 리밸런싱(모든 마스터에서 공평하게 슬롯 가져오기)
docker exec -it redis-node-1 redis-cli --cluster rebalance \
  --cluster-use-empty-masters redis-node-1:6379
# 또는 특정 양(respect weight) 조절 옵션 가능
```

* 결과: 기존 마스터들 일부 슬롯이 **redis-node-6**으로 이동.

> **팁**
>
> * “새 노드를 슬레이브(복제본)로만 추가”하고 싶다면:
>   1. `add-node`로 일단 추가
>   2. `redis-cli --cluster replicate <new-node-id> <master-node-id>` 실행

***

### 5) **노드 삭제**(축소): 6 → 5 노드

> 핵심: **해당 노드의 슬롯을 먼저 다른 마스터로 모두 옮긴 후** `del-node`.

#### 5-1) 슬롯 비우기(reshard로 슬롯 이동)

```bash
# 예: redis-node-6의 슬롯을 다른 마스터로 이동 (자동 분산)
docker exec -it redis-node-1 redis-cli --cluster rebalance \
  --cluster-weight redis-node-6:0 redis-node-1:1 redis-node-2:1 redis-node-3:1 \
  redis-node-1:6379
# 또는 명시적으로 '이 노드의 모든 슬롯을 옮겨라' 형태:
# redis-cli --cluster reshard <some-master:6379>
#   -> amount: 0 (all)
#   -> target node: (옮길 대상 마스터의 node-id)
#   -> source nodes: (삭제할 노드의 node-id)
```

#### 5-2) del-node (클러스터에서 제거)

```bash
# 삭제할 노드의 node-id 확인
docker exec -it redis-node-1 redis-cli cluster nodes | grep redis-node-6

# 슬롯이 0개인지 꼭 확인한 뒤(중요) del-node 수행
docker exec -it redis-node-1 redis-cli --cluster del-node \
  redis-node-1:6379 <NODE_ID_OF_REDIS_NODE_6>
```

#### 5-3) 컨테이너 정리

```bash
docker stop redis-node-6 && docker rm redis-node-6
# compose 파일에서 redis-node-6 섹션 삭제 or 주석 처리
```

> **주의**
>
> * 삭제하려는 노드가 **마스터**라면 반드시 **슬롯 0개** 상태에서 del-node.
> * 삭제하려는 노드가 **슬레이브**라면, 먼저 `cluster nodes`에서 **master/slave 관계**를 확인하고, 필요 시 다른 슬레이브로 재배치.

***

### 6) 장애/Failover 빠른 연습

```bash
# 마스터 하나 강제 종료(테스트)
docker stop redis-node-2

# 잠시 후 자동 failover 되는지 확인
docker exec -it redis-node-1 redis-cli cluster nodes | grep failover
docker exec -it redis-node-1 redis-cli cluster info
```

* 슬레이브가 같은 슬롯을 이어받으면 정상.
* 복구 후:

```bash
docker start redis-node-2
# 필요시 재복제/역할 조정: redis-cli --cluster replicate <slave-id> <master-id>
```

***

### 7) Spring 연결 포인트(요약)

> 이미 스프링 설정은 진행 중이니, 클러스터 호스트는 **서비스명:6379**로 쓰면 깔끔.

```yaml
spring:
  redis:
    cluster:
      nodes:
        - redis-node-1:6379
        - redis-node-2:6379
        - redis-node-3:6379
        - redis-node-4:6379
        - redis-node-5:6379
    timeout: 3000
    lettuce:
      cluster:
        refresh:
          adaptive: true
          period: 15s
```

* **추가/삭제 후**에도 Lettuce가 `MOVED/ASK`를 따라가며 토폴로지를 갱신.
* 해시 태그(`{user:42}`)로 **멀티키 연산**을 같은 슬롯에 묶는 패턴은 그대로 유지.

***

### 8) 운영 팁 (요약 체크리스트)

* **추가**: 컨테이너 띄우기 → `add-node` → (필요 시) `rebalance`
* **삭제**: `rebalance/reshard`로 슬롯 0 만들기 → `del-node` → 컨테이너 제거
* **모니터링**: `cluster info`, `cluster nodes`, Prometheus + Grafana
* **보안**: `REDIS_PASSWORD` 통일 + TLS(필요 시), ACL
* **백업**: RDB/AOF, 스냅샷 경로 볼륨 마운트 권장



***

## Spring 애플리케이션에서 Consistent Hashing/Redis Cluster 쓰는 법

### 1) 키 네이밍 & Hash Tag 유틸 (슬롯 고정)

* 멀티키/트랜잭션을 쓸 가능성이 있는 도메인은 **반드시 Hash Tag**로 묶자: `{user:42}:profile`

```java
public final class RedisKeys {
    private RedisKeys() {}
    public static String userTag(long userId) { return "{user:" + userId + "}"; }
    public static String userProfile(long userId) { return userTag(userId) + ":profile"; }
    public static String userTimeline(long userId) { return userTag(userId) + ":timeline"; }
    public static String rateLimiter(long userId) { return userTag(userId) + ":rl"; }
}
```

***

### 2) RedisTemplate 기반 Repository (Value/Hash)

* 직렬화는 `StringRedisSerializer` + `GenericJackson2JsonRedisSerializer` 추천(이미 설정했다고 가정).

```java
@Service
@RequiredArgsConstructor
public class UserProfileRepository {
    private final RedisTemplate<String, Object> redis;

    public void saveProfile(long userId, UserProfile profile, Duration ttl) {
        String key = RedisKeys.userProfile(userId);
        redis.opsForValue().set(key, profile, ttl);
    }

    @SuppressWarnings("unchecked")
    public Optional<UserProfile> findProfile(long userId) {
        Object v = redis.opsForValue().get(RedisKeys.userProfile(userId));
        return Optional.ofNullable((UserProfile) v);
    }

    public void saveProfileFields(long userId, Map<String, String> fields, Duration ttl) {
        String key = RedisKeys.userProfile(userId);
        redis.opsForHash().putAll(key, fields);
        redis.expire(key, ttl);
    }

    public Map<Object, Object> getProfileFields(long userId) {
        return redis.opsForHash().entries(RedisKeys.userProfile(userId));
    }
}
```

***

### 3) 멀티키 조회(MGET) & 파이프라인 (같은 슬롯에 묶어야 함)

* 같은 `{user:42}` 태그로 묶이면 클러스터에서도 안전하게 배치/파이프라인 가능.

```java
@Service
@RequiredArgsConstructor
public class TimelineService {
    private final RedisTemplate<String, Object> redis;

    public List<Object> mgetTimelineBatch(long userId, List<Long> itemIds) {
        List<String> keys = itemIds.stream()
                .map(id -> RedisKeys.userTimeline(userId) + ":" + id) // 같은 슬롯
                .toList();
        return redis.opsForValue().multiGet(keys);
    }

    public List<Object> pipelineGet(List<String> keys) {
        return redis.executePipelined((RedisCallback<Object>) conn -> {
            keys.forEach(k -> conn.stringCommands().get(k.getBytes()));
            return null;
        });
    }
}
```

***

### 4) Spring Cache (+ Hash Tag 포함 KeyGenerator)

* 캐시 키에도 **태그를 포함**시켜 같은 슬롯 유지.

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory cf) {
        RedisCacheConfiguration base = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .disableCachingNullValues();
        return RedisCacheManager.builder(cf)
                .cacheDefaults(base)
                .withCacheConfiguration("userCache",
                        base.entryTtl(Duration.ofMinutes(5)))
                .build();
    }

    @Bean("tagKeyGen")
    public KeyGenerator tagKeyGenerator() {
        return (target, method, params) -> {
            // 예: 첫 파라미터가 userId라고 가정
            long userId = (long) params[0];
            return RedisKeys.userTag(userId) + ":" + method.getName() + ":" + Arrays.hashCode(params);
        };
    }
}

@Service
@RequiredArgsConstructor
public class UserService {
    private final UserProfileRepository repo;

    @Cacheable(cacheNames = "userCache", keyGenerator = "tagKeyGen")
    public UserProfile getUserProfile(long userId) {
        return repo.findProfile(userId)
                .orElseGet(() -> loadFromDbAndCache(userId));
    }

    private UserProfile loadFromDbAndCache(long userId) {
        // ... DB 조회
        UserProfile profile = new UserProfile(/*...*/);
        repo.saveProfile(userId, profile, Duration.ofMinutes(5));
        return profile;
    }
}
```

***

### 5) 레이트리미터(Token Bucket) – Lua (원자적)

* **클러스터에서 Lua/EVAL은 같은 슬롯 키만** 안전. 키를 태그로 묶자.

```java
@Configuration
public class LuaConfig {
    @Bean
    public DefaultRedisScript<Long> tokenBucketScript() {
        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setResultType(Long.class);
        script.setScriptText(
            // KEYS[1] = bucket key, ARGV: capacity, refillTokens, refillIntervalMillis, nowMillis, tokensNeeded
            "local key=KEYS[1]\n" +
            "local capacity=tonumber(ARGV[1])\n" +
            "local refill=tonumber(ARGV[2])\n" +
            "local interval=tonumber(ARGV[3])\n" +
            "local now=tonumber(ARGV[4])\n" +
            "local need=tonumber(ARGV[5])\n" +
            "local data=redis.call('HMGET',key,'tokens','ts')\n" +
            "local tokens=tonumber(data[1]) or capacity\n" +
            "local ts=tonumber(data[2]) or now\n" +
            "local elapsed=math.max(0, now - ts)\n" +
            "local add=math.floor(elapsed/interval)*refill\n" +
            "tokens=math.min(capacity, tokens+add)\n" +
            "if tokens>=need then tokens=tokens-need; ts=now; " +
            "  redis.call('HMSET',key,'tokens',tokens,'ts',ts); return 1 " +
            "else redis.call('HMSET',key,'tokens',tokens,'ts',ts); return 0 end"
        );
        return script;
    }
}

@Service
@RequiredArgsConstructor
public class RateLimiterService {
    private final RedisTemplate<String, Object> redis;
    private final DefaultRedisScript<Long> tokenBucketScript;

    public boolean tryConsume(long userId, int tokens) {
        String key = RedisKeys.rateLimiter(userId);
        long capacity = 100, refill = 100, intervalMs = 1000, now = System.currentTimeMillis();
        Long ok = redis.execute(tokenBucketScript, List.of(key),
                capacity, refill, intervalMs, now, tokens);
        return ok != null && ok == 1L;
    }
}
```

***

### 6) 분산락(선택: Redisson) – 임계 구역 보호

* Redisson이면 코드가 간단해짐. (Lettuce로 SET NX 구현도 가능하지만 Redisson이 안전장치 풍부)

```java
@Service
@RequiredArgsConstructor
public class InventoryService {
    private final RedissonClient redisson;

    public void purchase(long userId, long productId) {
        String lockName = RedisKeys.userTag(userId) + ":lock:purchase";
        RLock lock = redisson.getLock(lockName);
        boolean acquired = false;
        try {
            acquired = lock.tryLock(2, 5, TimeUnit.SECONDS); // wait 2s, lease 5s
            if (!acquired) throw new IllegalStateException("busy");
            // 임계 구역: 재고 차감, 주문 생성
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException(e);
        } finally {
            if (acquired) lock.unlock();
        }
    }
}
```

***

### 7) 세션 공유 (Spring Session)

* Sticky Session 없이도 세션 공유. (Hash Tag는 필요 없음)

```yaml
spring:
  session:
    store-type: redis
```

```java
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
public class SessionConfig {}
```

***

### 8) 멀티키 트랜잭션(동일 슬롯 전제) & TxCallback

* `SessionCallback`으로 MULTI/EXEC. 키들은 **같은 태그**로!

```java
@Service
@RequiredArgsConstructor
public class BasketService {
    private final RedisTemplate<String, Object> redis;

    public void addItems(long userId, List<String> items) {
        String base = RedisKeys.userTag(userId) + ":basket";
        redis.execute(new SessionCallback<Void>() {
            @Override public Void execute(RedisOperations ops) {
                ops.multi();
                for (String it : items) {
                    ops.opsForList().rightPush(base, it); // 같은 슬롯
                }
                ops.exec();
                return null;
            }
        });
    }
}
```

***

### 9) 장애/리밸런싱 중 리다이렉션 & 재시도

* Lettuce는 `MOVED/ASK` 자동 처리 + 토폴로지 리프레시.
* 네트워크/타임아웃 재시도는 **Spring Retry**로 보완.

```java
@EnableRetry
@Service
@RequiredArgsConstructor
public class ResilientReadService {
    private final RedisTemplate<String, Object> redis;

    @Retryable(maxAttempts = 3, backoff = @Backoff(delay = 100, multiplier = 2))
    public Object get(String key) {
        return redis.opsForValue().get(key);
    }
}
```

***

### 10) 관측성: 캐시 미스/Latency 메트릭

* Micrometer로 간단히 지표 남기기.

```java
@Component
@RequiredArgsConstructor
public class CacheMetrics {
    private final Counter cacheMiss;

    public CacheMetrics(MeterRegistry reg) {
        this.cacheMiss = Counter.builder("cache_miss_total")
                .tag("cache", "userCache").register(reg);
    }

    public void onMiss() { cacheMiss.increment(); }
}
```

***

### 11) 통합 테스트 (Testcontainers로 클러스터 연결 확인)

* 단일 RedisContainer로도 연결 검증은 가능. (로컬에 이미 클러스터가 있다면 생략)

```java
@SpringBootTest
@Testcontainers
class RedisIT {
    @Container
    static RedisContainer REDIS = new RedisContainer("redis:7.2-alpine")
            .withExposedPorts(6379);

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry r) {
        String host = REDIS.getHost();
        Integer port = REDIS.getFirstMappedPort();
        r.add("spring.redis.cluster.nodes", () -> host + ":" + port); // 데모용(실클러스터는 여러 노드)
    }

    @Autowired RedisTemplate<String, Object> redis;

    @Test
    void ping() {
        redis.opsForValue().set("k","v");
        assertEquals("v", redis.opsForValue().get("k"));
    }
}
```

***

### 12) 성능 팁 & 체크리스트 (짧게)

* **키 설계**: 반드시 Hash Tag로 멀티키/Tx 가능한 그룹 정의
* **직렬화 비용**: 큰 객체라면 JSON 대신 바이너리(FST/Smile) 고려
* **TTL 전략**: 핫키는 짧게, 백오프 재시도 시 TTL 재설정 주의
* **파이프라인**: 읽기/쓰기 벌크 처리 시 `executePipelined` 적극 활용
* **핫스팟**: 특정 태그에 과도한 트래픽이면 태그 스키마(샤딩 태그 추가)로 분산

***

#### 끝으로

* **핵심 1**: 키를 태그로 “묶는 설계”가 클러스터에서 모든 걸 결정한다.
* **핵심 2**: Lua/멀티키/트랜잭션/락은 **같은 슬롯**이 전제.
* **핵심 3**: 장애/리밸런싱은 클라이언트가 따라가므로, **타임아웃/재시도/모니터링**만 잘 세팅하면 실무 내구성 충분.

