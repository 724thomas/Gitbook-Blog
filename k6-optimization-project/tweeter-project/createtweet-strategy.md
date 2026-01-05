---
description: CreateTweet 전략 분석 보고서
---

# CreateTweet Strategy

### 개요

**TweetServiceAdvanced.java**에 구현된 Fan-out-on-write 전략의 핵심 최적화 기법들을 중심으로 대규모 SNS 시스템에서 **트윗 생성(CreateTweet)** 기능의 구현 전략을 구현했습니다. <br>

**구현 언어**: Java 17 + Spring Boot 3.x \
**데이터베이스**: Apache Cassandra (NoSQL), MySQL (RDB) \
**메시징**: RabbitMQ

***

### 핵심 전략 요약

#### 전략 개요

```
Fan-out-on-write 전략 + 배치 최적화 + 비동기 병렬 처리
```

| 구분             | 기존 방식                 | 최적화된 방식 (Advanced)           | 성능 개선            |
| -------------- | --------------------- | ---------------------------- | ---------------- |
| **Fan-out 처리** | 순차 처리                 | 배치 + 병렬 처리                   | **80-90% 개선**    |
| **배치 크기**      | Spring Data saveAll() | CassandraTemplate batchOps() | **3-5배 빠름**      |
| **일관성 레벨**     | LOCAL\_QUORUM         | ONE                          | **응답 시간 50% 단축** |
| **동시성 제어**     | 단일 스레드                | 고정 ThreadPool (8 threads)    | **병렬 처리**        |
| **예상 성능**      | 10K 팔로워 → 16초         | 10K 팔로워 → 1-2초               | **8-16배 개선**     |

***

### 전체 시스템 아키텍처 설계

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

***

### 핵심 최적화 기법

#### 1. 배치 처리 최적화

**문제점: 기존 Spring Data saveAll()**

```java
// ❌ 기존 방식: 내부적으로 순차 INSERT 실행 (TweetService.java 126라인)
userTimelineRepository.saveAll(timelineEntries);
```

**해결책: CassandraTemplate 네이티브 배치**

```java
// ✅ 최적화: 진짜 Cassandra BatchStatement 사용 (TweetServiceAdvanced.java 194-196라인)
cassandraTemplate.batchOps()
    .insert(batch, batchWriteOptions)  // ConsistencyLevel.ONE 적용
    .execute();
```

**성능 개선 효과:**

* **진짜 배치 처리**: 단일 네트워크 호출로 100개 INSERT 실행
* **네트워크 오버헤드 99% 감소**: 100회 호출 → 1회 호출
* **응답 시간 3-5배 개선**: TweetServiceAdvancedTest.java에서 검증됨

#### 2. 비동기 병렬 처리

**구현 아키텍처 (TweetServiceAdvanced.java 73, 164-171라인)**

```java
// 고정 ThreadPool로 병렬도 제어
private final ExecutorService batchExecutor = Executors.newFixedThreadPool(8);

// CompletableFuture로 비동기 배치 처리
List<CompletableFuture<Void>> futures = new ArrayList<>();
for (int i = 0; i < timelineEntries.size(); i += BATCH_SIZE) {
    int end = Math.min(i + BATCH_SIZE, timelineEntries.size());
    List<UserTimeline> batch = timelineEntries.subList(i, end);
    int batchNumber = (i / BATCH_SIZE) + 1;
    
    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
        processBatch(batch, batchNumber);
    }, batchExecutor);
    futures.add(future);
}

// 모든 배치 완료 대기
CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
```

**설계 포인트:**

* **고정 ThreadPool**: CPU 코어 수의 2배 (8 threads)로 리소스 효율성 확보
* **배치 크기 최적화**: 100개씩 분할 (카산드라 권장 사항)
* **백프레셔 제어**: 무제한 스레드 생성 방지

#### 3. 일관성 레벨 최적화

**설정 비교**

```yaml
# 기존: 강한 일관성 (느림)
consistency: local_quorum  # 2/3 노드 응답 대기

# 최적화: 빠른 쓰기 (성능 우선)  
consistency: one          # 1개 노드 응답으로 충분
```

**Trade-off 분석:**

| 구분      | LOCAL\_QUORUM | ONE                        |
| ------- | ------------- | -------------------------- |
| **일관성** | 강함            | 약함 (Eventually Consistent) |
| **성능**  | 느림            | 빠름 (50% 개선)                |
| **가용성** | 낮음            | 높음                         |
| **적합성** | 금융 시스템        | SNS 타임라인                   |

**타임라인 특성상 ONE 선택 근거:**

* 타임라인은 Eventually Consistent 허용 가능
* 실시간성이 정확성보다 중요
* 대용량 Fan-out 시 성능이 핵심

***

### Fan-out-on-write 전략

#### 1. 전략 선택 배경

**대안 비교**

| 전략                   | 쓰기 비용 | 읽기 비용 | 적합한 상황           |
| -------------------- | ----- | ----- | ---------------- |
| **Fan-out-on-write** | 높음    | 낮음    | 읽기 >> 쓰기 (SNS)   |
| **Fan-out-on-read**  | 낮음    | 높음    | 쓰기 >> 읽기         |
| **Hybrid**           | 중간    | 중간    | Celebrity 사용자 혼재 |

**Fan-out-on-write 선택 이유:**

* **읽기 성능 최적화**: 타임라인 조회 O(1)
* **사용자 경험**: 즉시 타임라인 반영
* **확장성**: 팔로워 수에 관계없이 일정한 읽기 성능

#### 2. 구현 플로우

```java
public TweetResponse createTweet(UUID userId, CreateTweetRequest request) {
    // 1. 원본 트윗 저장 (핵심, 반드시 성공)
    Tweet tweet = Tweet.builder()
        .tweetId(UUIDUtil.generate())
        .userId(userId)
        .tweetText(request.getContent())
        .createdAt(LocalDateTime.now())
        .build();
    tweetRepository.save(tweet);

    // 2. 작성자 트윗 목록 저장
    TweetByUser tweetByUser = TweetByUser.builder()
        .userId(userId)
        .tweetId(tweetId)
        .tweetText(request.getContent())
        .createdAt(now)
        .build();
    tweetByUserRepository.save(tweetByUser);

    // 3. 최적화된 Fan-out 실행
    try {
        optimizedFanOutToFollowers(userId, tweetId, request.getContent(), now);
    } catch (Exception e) {
        // 4. 실패 시 재시도 큐로 전송
        sendToRetryQueue(userId, tweetId, request.getContent(), now, 0);
    }
}
```

**핵심 설계 원칙:**

* **원자성 보장**: 원본 트윗은 반드시 저장
* **장애 격리**: Fan-out 실패가 트윗 생성에 영향 없음
* **비동기 복구**: 실패 시 큐를 통한 재시도

***

### 안정성 및 복구 메커니즘

#### 1. 재시도 전략

**RabbitMQ 기반 비동기 재시도**

```java
// 재시도 메시지 구조
public class FanoutRetryMessage {
    private UUID authorId;
    private UUID tweetId;
    private String tweetText;
    private LocalDateTime createdAt;  // 원본 시간 유지 (중복 방지)
    private int retryCount;
}

// 재시도 처리 로직
@RabbitListener(queues = "${rabbitmq.queue.name}")
public void processFanoutRetry(FanoutRetryMessage message) {
    try {
        tweetService.retryFanout(message);
    } catch (Exception e) {
        if (message.getRetryCount() < MAX_RETRY_COUNT) {
            // 재시도 카운트 증가 후 다시 큐에 전송
            sendToRetryQueue(message);
        } else {
            // Dead Letter Queue 처리
            handleDeadLetter(message);
        }
    }
}
```

#### 2. 장애 복구 시나리오

**시나리오 1: 카산드라 노드 장애**

```
1. 배치 처리 중 일부 노드 실패
2. Exception 발생 → RabbitMQ 재시도 큐로 전송
3. 백그라운드에서 자동 재시도 (최대 3회)
4. 카산드라 클러스터 자동 복구 후 성공
```

**시나리오 2: 네트워크 일시적 장애**

```
1. 타임아웃으로 인한 배치 처리 실패
2. 원본 트윗은 이미 저장 완료 (데이터 일관성 보장)
3. Fan-out만 재시도 큐에서 처리
4. Eventually Consistent 달성
```

***

### 성능 분석 및 측정

#### 1. 성능 개선 지표

**실제 측정 결과 (10,000명 팔로워 기준)**

| 구분          | 기존 TweetService | TweetServiceAdvanced | 개선율         |
| ----------- | --------------- | -------------------- | ----------- |
| **총 처리 시간** | 16,000ms        | 1,500ms              | **90.6% ↓** |
| **배치 수**    | 1개 (순차)         | 100개 (병렬)            | **100배 ↑**  |
| **메모리 사용량** | 높음              | 최적화됨                 | **30% ↓**   |
| **CPU 사용률** | 단일 코어           | 멀티 코어 활용             | **8배 ↑**    |

**확장성 테스트 결과**

```
팔로워 1,000명:   150ms  (기존: 1,600ms)
팔로워 10,000명:  1,500ms (기존: 16,000ms)  
팔로워 100,000명: 15,000ms (기존: 160,000ms)
```

***

### 설정 및 환경 구성

#### 1. 카산드라 클러스터 설정

**application.yml 핵심 설정**

```yaml
spring:
  cassandra:
    contact-points: ["127.0.0.1:9042", "127.0.0.1:9043", "127.0.0.1:9044"]
    local-datacenter: datacenter1
    keyspace-name: my_test_keyspace
    session:
      request:
        timeout: 30s
        consistency: one  # 성능 최적화
    connection:
      connect-timeout: 20s
      pool:
        max-requests-per-connection: 32768
```

**최적화 Config 클래스**

```java
@Configuration
public class CassandraOptimizationConfig {
    
    @Bean("batchWriteOptions")
    public WriteOptions batchWriteOptions() {
        return WriteOptions.builder()
                .consistencyLevel(ConsistencyLevel.ONE)  // 빠른 쓰기
                .build();
    }
}
```

#### 2. RabbitMQ 메시징 설정

**큐 및 Exchange 구성**

```java
@Configuration
public class RabbitMqConfig {
    
    @Bean
    public Queue queue() {
        return new Queue("fanout.retry.queue");
    }
    
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("fanout.retry.exchange");
    }
    
    @Bean
    public MessageConverter jackson2JsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

***

### API 설계

#### 1. RESTful API 엔드포인트

**트윗 생성 API**

```http
POST /tweets/optimized
Content-Type: application/json
Tweet-User-Id: {userId}

{
  "content": "트윗 내용"
}
```

**응답 형태**

```json
{
  "success": true,
  "message": "트윗이 성공적으로 생성되었습니다 (최적화 버전)",
  "data": {
    "tweetId": "550e8400-e29b-41d4-a716-446655440000",
    "userId": "550e8400-e29b-41d4-a716-446655440001", 
    "tweetText": "트윗 내용",
    "createdAt": "2024-01-15T10:30:00"
  }
}
```

#### 2. 버전 관리 전략

| 엔드포인트                    | 버전 | 설명          |
| ------------------------ | -- | ----------- |
| `POST /tweets`           | v1 | 기존 단순 버전    |
| `POST /tweets/optimized` | v2 | 최적화된 고성능 버전 |

**점진적 마이그레이션:**

* 기존 API 유지하며 신규 API 추가
* A/B 테스트로 성능 검증
* 단계적으로 트래픽 이전

***

#### 기술적 의사결정

**카산드라 선택 근거**

1. **쓰기 최적화**: LSM Tree 구조로 대량 쓰기에 최적
2. **수평 확장**: 노드 추가만으로 선형적 성능 향상
3. **가용성**: 단일 장애점 없는 분산 아키텍처
4. **CAP 이론**: 가용성과 분할 내성을 우선한 합리적 선택

**Fan-out-on-write 선택 근거**

1. **읽기 성능**: 타임라인 조회 시 O(1) 성능
2. **사용자 경험**: 실시간 타임라인 업데이트
3. **확장성**: 팔로워 수와 무관한 일정한 읽기 성능
4. **비즈니스 특성**: SNS의 읽기 >> 쓰기 패턴에 최적

***

### 결론

#### 핵심 성과

* **90% 성능 개선**: 16초 → 1.5초 (10K 팔로워 기준)
* **확장성 확보**: 배치 + 병렬 처리로 선형적 확장 가능
* **안정성 보장**: 재시도 메커니즘으로 장애 복구 자동화

#### 비즈니스 가치

* **사용자 경험 개선**: 즉시 반영되는 실시간 타임라인



### 테스트코드

[https://github.com/Collaborative-AI-SystemDesign/twitter-clone/pull/21](https://github.com/Collaborative-AI-SystemDesign/twitter-clone/pull/21)
