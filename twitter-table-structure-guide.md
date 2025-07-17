# Twitter Table Structure Guide

## 🐦 트위터 - Cassandra 테이블 설계 가이드

### 📋 개요

이 문서는 트위터와 같은 대규모 소셜 미디어 플랫폼의 데이터베이스 설계에 대한 종합 가이드입니다.&#x20;

* 왜 이렇게 설계해야 하는지
* 각 테이블의 역할은 무엇인지
* 어떻게 동작하는지

단계별로 이해할 수 있도록 작성되었습니다.



### 🤔 왜 Cassandra를 사용하는가?&#x20;

#### 트위터의 특징&#x20;

* **대량의 쓰기**: 초당 수십만 개의 트윗 생성
* **빠른 읽기**: 타임라인 조회 시 즉시 응답 필요
* **수평 확장**: 사용자 증가에 따른 서버 확장 필요
* **시간 기반**: 데이터 최신 트윗부터 시간순 정렬 중요



#### Cassandra의 장점

📊 관계형 DB (MySQL) vs Cassandra

관계형 DB:

* 복잡한 JOIN 연산 → 느린 읽기 성능
* 수직 확장 (서버 성능 향상) → 비용 ↗️
* ACID 보장 → 일관성은 좋지만 성능 ↓

Cassandra:

* 비정규화된 테이블 → 빠른 읽기 성능 ⚡
* 수평 확장 (서버 대수 증가) → 비용 효율적
* 최종 일관성 → 성능 우선, 일관성은 eventual

***

### 📊 4개 핵심 테이블 상세 설명

#### 1. 🏠 tweets 테이블 - 트윗의 원본 저장소

**역할**: 모든 트윗의 "진실의 원천(Source of Truth)"

```sql
-- Cassandra CQL
CREATE TABLE tweets (
    tweet_id UUID PRIMARY KEY,     -- 트윗 고유 ID
    user_id UUID,                  -- 작성자 ID  
    tweet_text TEXT,               -- 트윗 내용
    created_at TIMESTAMP,          -- 생성 시간
    modified_at TIMESTAMP          -- 수정 시간
);
```

**Java 엔티티**:

```java
@Table("tweets")
public class Tweet extends BaseEntity {
    @PrimaryKey
    private UUID tweetId;           // 파티션 키
    
    @Column("user_id")
    private UUID userId;
    
    @Column("tweet_text") 
    private String tweetText;
}
```

**데이터 예시**:

```
| tweet_id  | user_id | tweet_text      | created_at          |
|-----------|---------|-----------------|---------------------|
| tweet-123 | user-A  | 안녕하세요!      | 2025-01-15 14:30:00 |
| tweet-124 | user-B  | 점심 맛있다      | 2025-01-15 14:31:00 |
| tweet-125 | user-A  | 날씨 좋네요      | 2025-01-15 14:32:00 |
```

**🔍 주요 사용 패턴**:

* `POST /tweets`: 새 트윗 저장
* `GET /tweets/{tweetId}`: 특정 트윗 조회
* 다른 테이블에서 트윗 상세 정보 참조

**⚠️ 중요한 점**:

* 이 테이블은 `user_id`로 직접 조회하지 않습니다!
* "특정 사용자의 모든 트윗"은 다른 테이블에서 처리합니다.

***

#### 2. 👥 followers\_by\_user 테이블 - 팔로우 관계

**역할**: "특정 사용자를 팔로우하는 모든 사람들"을 빠르게 찾기

**복합 Primary Key 클래스**

```java
@PrimaryKeyClass
public class FollowersByUserKey {
    @PrimaryKeyColumn(name = "followed_user_id", type = PrimaryKeyType.PARTITIONED)
    private UUID followedUserId;    // 파티션 키: 팔로우 당하는 사람
    
    @PrimaryKeyColumn(name = "follower_id", type = PrimaryKeyType.CLUSTERED)  
    private UUID followerId;        // 클러스터링 키: 팔로우 하는 사람
}
```

**메인 엔티티**

```java
@Table("followers_by_user")
public class FollowersByUser extends BaseEntity {
    @PrimaryKey
    private FollowersByUserKey key;
    
    @Column("followed_at")
    private LocalDateTime followedAt;
}
```

**데이터 예시**:

```
🎯 user-A를 팔로우하는 사람들 (한 파티션에 저장)
| followed_user_id | follower_id | followed_at         |
|------------------|-------------|---------------------|
| user-A           | user-1      | 2025-01-10 09:00:00 |
| user-A           | user-2      | 2025-01-11 10:30:00 |
| user-A           | user-3      | 2025-01-12 15:20:00 |

🎯 user-B를 팔로우하는 사람들 (다른 파티션에 저장)  
| followed_user_id | follower_id | followed_at         |
|------------------|-------------|---------------------|
| user-B           | user-4      | 2025-01-13 11:15:00 |
| user-B           | user-5      | 2025-01-14 16:45:00 |
```

**🔍 핵심 쿼리**:

```java
// user-A가 트윗을 작성했을 때, 누구에게 알림을 보내야 할까?
List<FollowersByUser> followers = repository.findByKeyFollowedUserId("user-A");
// 결과: [user-1, user-2, user-3] - 한 번의 쿼리로 모든 팔로워 조회!
```

**💡 왜 이렇게 설계했나요?**

```
❌ 나쁜 설계 (관계형 DB 방식):
SELECT follower_id FROM follows WHERE followed_user_id = 'user-A'
→ 인덱스 스캔 + 여러 노드 접근 필요

✅ 좋은 설계 (Cassandra 방식):  
followed_user_id를 파티션 키로 사용
→ user-A의 모든 팔로워가 한 노드에 저장됨
→ 한 번의 네트워크 요청으로 모든 데이터 조회 가능!
```

***

#### 3. 📱 user\_timeline 테이블 - 개인 타임라인 (Fan-out on Write)

**역할**: 각 사용자의 개인 타임라인을 미리 생성하여 저장

**복합 Primary Key 클래스**

```java
@PrimaryKeyClass
public class UserTimelineKey {
    @PrimaryKeyColumn(name = "follower_id", type = PrimaryKeyType.PARTITIONED)
    private UUID followerId;        // 파티션 키: 타임라인 소유자
    
    @PrimaryKeyColumn(name = "created_at", type = PrimaryKeyType.CLUSTERED, ordering = Ordering.DESCENDING)
    private LocalDateTime createdAt; // 첫 번째 클러스터링 키: 시간 (내림차순)
    
    @PrimaryKeyColumn(name = "tweet_id", type = PrimaryKeyType.CLUSTERED)
    private UUID tweetId;           // 두 번째 클러스터링 키: 고유성 보장
}
```

**메인 엔티티**

```java
@Table("user_timeline") 
public class UserTimeline extends BaseEntity {
    @PrimaryKey
    private UserTimelineKey key;
    
    @Column("author_id")
    private UUID authorId;          // 트윗 작성자
    
    @Column("tweet_text")
    private String tweetText;       // 트윗 내용 (비정규화)
}
```

**🔄 Fan-out on Write 과정**:

**1단계**: user-A가 "안녕하세요!" 트윗 작성

```java
// tweets 테이블에 원본 저장
Tweet tweet = new Tweet("user-A", "안녕하세요!");
tweetRepository.save(tweet);
```

**2단계**: user-A의 팔로워들 조회

```java
// followers_by_user 테이블에서 팔로워 목록 조회
List<UUID> followers = followRepository.findFollowersByUserId("user-A");
// 결과: [user-1, user-2, user-3]
```

**3단계**: 각 팔로워의 타임라인에 트윗 추가

```java
// 각 팔로워의 user_timeline에 저장
for (UUID followerId : followers) {
    UserTimeline timeline = new UserTimeline(
        followerId,           // user-1, user-2, user-3
        tweet.getTweetId(),   // tweet-123
        "user-A",            // 작성자
        "안녕하세요!"         // 내용
    );
    timelineRepository.save(timeline);
}
```

**결과 데이터**:

```
🎯 user-1의 타임라인 (follower_id = user-1인 파티션)
| follower_id | created_at          | tweet_id  | author_id | tweet_text    |
|-------------|---------------------|-----------|-----------|---------------|
| user-1      | 2025-01-15 14:30:00 | tweet-123 | user-A    | 안녕하세요!    |
| user-1      | 2025-01-15 14:25:00 | tweet-122 | user-B    | 점심 맛있다    |
| user-1      | 2025-01-15 14:20:00 | tweet-121 | user-C    | 날씨 좋네요    |

🎯 user-2의 타임라인 (follower_id = user-2인 파티션)  
| follower_id | created_at          | tweet_id  | author_id | tweet_text    |
|-------------|---------------------|-----------|-----------|---------------|
| user-2      | 2025-01-15 14:30:00 | tweet-123 | user-A    | 안녕하세요!    |
| user-2      | 2025-01-15 14:28:00 | tweet-124 | user-D    | 커피 한 잔     |

🎯 user-3의 타임라인 (follower_id = user-3인 파티션)
| follower_id | created_at          | tweet_id  | author_id | tweet_text    |
|-------------|---------------------|-----------|-----------|---------------|  
| user-3      | 2025-01-15 14:30:00 | tweet-123 | user-A    | 안녕하세요!    |
| user-3      | 2025-01-15 14:26:00 | tweet-125 | user-E    | 운동 갔다 와야지 |
```

**🔍 타임라인 조회 (매우 빠름!)**:

```java
// GET /timeline - user-1의 타임라인 조회
List<UserTimeline> timeline = repository.findByKeyFollowerIdOrderByCreatedAtDesc("user-1", 20);
// 결과: user-1의 최신 트윗 20개를 한 번의 쿼리로 조회!
// 복잡한 JOIN이나 정렬 연산 없이 이미 정렬된 데이터를 바로 반환
```

**⚡ 성능 특징**:

* **쓰기**: O(팔로워 수) - 팔로워 수만큼 쓰기 발생
* **읽기**: O(1) + 조회할 트윗 수 - 매우 빠름!

***

#### 4. 🌟 celebrity\_tweets 테이블 - 인플루언서 트윗 (Fan-out on Read)

**역할**: 팔로워가 많은 인플루언서의 트윗을 별도 저장

**복합 Primary Key 클래스**

```java
@PrimaryKeyClass  
public class CelebrityTweetKey {
    @PrimaryKeyColumn(name = "author_id", type = PrimaryKeyType.PARTITIONED)
    private UUID authorId;          // 파티션 키: 인플루언서 ID
    
    @PrimaryKeyColumn(name = "created_at", type = PrimaryKeyType.CLUSTERED, ordering = Ordering.DESCENDING)
    private LocalDateTime createdAt; // 첫 번째 클러스터링 키: 시간 (내림차순)
    
    @PrimaryKeyColumn(name = "tweet_id", type = PrimaryKeyType.CLUSTERED)
    private UUID tweetId;           // 두 번째 클러스터링 키: 고유성 보장
}
```

**메인 엔티티**

```java
@Table("celebrity_tweets")
public class CelebrityTweet extends BaseEntity {
    @PrimaryKey
    private CelebrityTweetKey key;
    
    @Column("tweet_text")
    private String tweetText;       // 트윗 내용
}
```

**🤔 왜 별도 테이블이 필요한가요?**

**문제 상황**:

```
아이유(팔로워 1,000만 명)가 트윗을 작성한다면?

Fan-out on Write 방식으로 처리하면:
1. tweets 테이블에 원본 저장 (1번 쓰기)
2. 1,000만 명의 user_timeline에 모두 저장 (1,000만 번 쓰기) 💥

결과: 
- 서버 과부하 💻💥
- 비용 폭발 💸💸💸  
- 응답 시간 증가 ⏰
```

**해결책: Fan-out on Read**:

```
아이유가 트윗 작성:
1. tweets 테이블에 원본 저장 (1번 쓰기)
2. celebrity_tweets 테이블에 저장 (1번 쓰기)
3. 팔로워들의 user_timeline에는 저장하지 않음 ❌

사용자가 타임라인 조회 시:
1. user_timeline에서 일반 트윗들 조회
2. 팔로우하는 셀럽들의 celebrity_tweets에서 최신 트윗 조회  
3. 두 결과를 시간순으로 병합
```

**데이터 예시**:

```
🎯 아이유의 트윗들 (author_id = 아이유-id인 파티션)
| author_id | created_at          | tweet_id  | tweet_text         |
|-----------|---------------------|-----------|-------------------|
| 아이유-id  | 2025-01-15 14:30:00 | tweet-456 | 새 앨범 발매했어요! |
| 아이유-id  | 2025-01-15 12:15:00 | tweet-455 | 콘서트 재밌었어요   |
| 아이유-id  | 2025-01-15 10:20:00 | tweet-454 | 좋은 아침이에요     |

🎯 BTS의 트윗들 (author_id = BTS-id인 파티션)
| author_id | created_at          | tweet_id  | tweet_text       |
|-----------|---------------------|-----------|-----------------|
| BTS-id    | 2025-01-15 15:00:00 | tweet-460 | 새 뮤비 공개!     |
| BTS-id    | 2025-01-15 13:30:00 | tweet-459 | 팬 여러분 감사해요 |
```

***

### 🔄 하이브리드 전략: Fan-out on Write + Fan-out on Read

#### 전체 프로세스 시나리오

**시나리오 1: 일반 사용자(user-A)가 트윗 작성**

```java
// 1. 원본 저장
Tweet tweet = new Tweet("user-A", "오늘 날씨 좋네요");
tweetRepository.save(tweet);

// 2. 팔로워 조회 (소수)
List<UUID> followers = followRepository.findFollowersByUserId("user-A");
// 결과: [user-1, user-2, user-3] - 100명

// 3. Fan-out on Write
for (UUID followerId : followers) {
    UserTimeline timeline = new UserTimeline(followerId, tweet);
    timelineRepository.save(timeline);
}
// 100번의 쓰기 발생 (비용 적음)
```

**시나리오 2: 인플루언서(아이유)가 트윗 작성**

```java
// 1. 원본 저장
Tweet tweet = new Tweet("아이유-id", "새 앨범 발매했어요!");
tweetRepository.save(tweet);

// 2. 인플루언서 여부 확인
if (celebrityService.isCelebrity("아이유-id")) {
    // 3. celebrity_tweets에만 저장
    CelebrityTweet celebrityTweet = new CelebrityTweet("아이유-id", tweet);
    celebrityTweetRepository.save(celebrityTweet);
    // 1번의 쓰기만 발생 (비용 절약!)
}
```

**시나리오 3: 사용자(user-1)가 타임라인 조회**

```java
// 1. 일반 트윗들 조회 (user_timeline)
List<UserTimeline> regularTweets = timelineRepository
    .findByKeyFollowerIdOrderByCreatedAtDesc("user-1", 20);

// 2. 팔로우하는 셀럽들의 트윗 조회
List<UUID> followedCelebrities = followRepository.getFollowedCelebrities("user-1");
List<CelebrityTweet> celebrityTweets = new ArrayList<>();

for (UUID celebrityId : followedCelebrities) {
    List<CelebrityTweet> tweets = celebrityTweetRepository
        .findByKeyAuthorIdOrderByCreatedAtDesc(celebrityId, 10);
    celebrityTweets.addAll(tweets);
}

// 3. 시간순으로 병합
List<TweetResponse> finalTimeline = mergeAndSort(regularTweets, celebrityTweets);
```

#### 성능 비교

```
📊 일반 사용자 vs 인플루언서

일반 사용자 (팔로워 100명):
- 쓰기: 100번 (적음)
- 읽기: 1번 (매우 빠름)
- 저장공간: 100배 (적음)

인플루언서 (팔로워 1,000만 명):
- 쓰기: 1번 (매우 적음) ⭐
- 읽기: 2-3번 (빠름)  
- 저장공간: 1배 (매우 적음) ⭐

결론: 상황에 따라 최적의 전략 선택!
```

***

### 🔍 Cassandra 핵심 개념 이해

#### 파티션 키 vs 클러스터링 키

**파티션 키 (Partition Key)**:

```
역할: 데이터를 어느 노드에 저장할지 결정
예시: user_timeline 테이블의 follower_id

follower_id = "user-1" → Node A에 저장
follower_id = "user-2" → Node B에 저장  
follower_id = "user-3" → Node C에 저장

장점: 각 사용자의 타임라인이 한 노드에서 처리되어 빠름
```

**클러스터링 키 (Clustering Key)**:

```
역할: 같은 파티션 내에서 데이터 정렬 순서 결정
예시: user_timeline 테이블의 (created_at DESC, tweet_id)

user-1의 파티션 내부:
├── created_at: 2025-01-15 14:30:00, tweet_id: tweet-123
├── created_at: 2025-01-15 14:25:00, tweet_id: tweet-122  
└── created_at: 2025-01-15 14:20:00, tweet_id: tweet-121

장점: 이미 정렬되어 저장되므로 조회 시 빠름
```

#### 커서 기반 페이지네이션

**기존 OFFSET 방식의 문제점**:

```sql
-- 관계형 DB 방식
SELECT * FROM timeline ORDER BY created_at DESC LIMIT 20 OFFSET 1000;

문제점:
- OFFSET이 클수록 성능 저하 (1000개를 모두 스캔해야 함)
- 실시간으로 데이터가 추가되면 중복/누락 발생 가능
```

**Cassandra 커서 방식**:

```java
// 1페이지: 최신 20개 조회
List<UserTimeline> page1 = repository.findByKeyFollowerIdOrderByCreatedAtDesc("user-1", 20);
String cursor = generateCursor(page1.get(19)); // 마지막 아이템으로 커서 생성

// 2페이지: 커서 이후 20개 조회  
List<UserTimeline> page2 = repository.findByKeyFollowerIdAndCreatedAtBefore("user-1", cursor, 20);

장점:
- 항상 O(1) 성능 (OFFSET 크기와 무관)
- 실시간 데이터 추가와 상관없이 일관된 결과
```

***

### 🚀 구현 시 고려사항

#### 1. 인플루언서 판단 기준

```java
@Service
public class CelebrityService {
    
    private static final int CELEBRITY_FOLLOWER_THRESHOLD = 10_000;
    
    public boolean isCelebrity(UUID userId) {
        // 방법 1: 팔로워 수 기준
        long followerCount = followRepository.countFollowers(userId);
        if (followerCount > CELEBRITY_FOLLOWER_THRESHOLD) {
            return true;
        }
        
        // 방법 2: 수동 지정된 VIP 계정
        return vipAccountService.isVipAccount(userId);
    }
}
```

#### 2. 배치 처리 최적화

```java
// Fan-out 시 배치 삽입으로 성능 향상
@Async
public void fanOutTweetBatch(Tweet tweet, List<UUID> followers) {
    List<UserTimeline> timelineEntries = followers.stream()
        .map(followerId -> new UserTimeline(followerId, tweet))
        .collect(toList());
    
    // 배치로 한 번에 저장 (개별 저장보다 빠름)
    timelineRepository.saveAll(timelineEntries);
}
```

#### 3. 캐시 전략

```java
@Service  
public class TimelineService {
    
    @Cacheable(key = "#userId", value = "timeline")
    public TimelineResponse getTimeline(UUID userId, String cursor) {
        // 자주 조회되는 타임라인은 Redis에 캐시
        // TTL: 5분 (실시간성과 성능의 균형)
    }
}
```

#### 4. 모니터링 지표

```java
// 중요한 메트릭들
- Fan-out 시간: 트윗 작성 후 모든 팔로워 타임라인 업데이트 완료 시간
- 타임라인 조회 응답시간: 95 percentile < 100ms 목표
- 저장 공간 사용량: 특히 user_timeline 테이블 모니터링
- 셀럽 트윗 병합 성능: Fan-out on Read 시 병합 시간
```

***

### 📊 성능 예상치

#### 트래픽 규모별 성능

**소규모 서비스 (일일 활성 사용자 10만 명)**:

```
- 평균 팔로워 수: 100명
- 일일 트윗 수: 50만 개
- Fan-out 쓰기: 5,000만 회/일 (관리 가능)
- 타임라인 조회: 1,000만 회/일 (빠름)

추천: Fan-out on Write 위주 전략
```

**대규모 서비스 (일일 활성 사용자 1억 명)**:

```
- 평균 팔로워 수: 200명  
- 일일 트윗 수: 5억 개
- Fan-out 쓰기: 1,000억 회/일 (비용 부담)
- 인플루언서 비율: 1% (100만 명)

추천: 하이브리드 전략 (일반 사용자 Fan-out on Write + 인플루언서 Fan-out on Read)
```

#### 하드웨어 요구사항

**Cassandra 클러스터 구성 예시**:

```
소규모: 3노드 (각 16GB RAM, 1TB SSD)
중규모: 9노드 (각 32GB RAM, 2TB SSD)  
대규모: 27노드 (각 64GB RAM, 4TB SSD)

복제 팩터: 3 (고가용성 보장)
일관성 레벨: LOCAL_QUORUM (성능과 일관성 균형)
```

***

### 🔧 실제 구현 코드 예시

#### Repository 계층

```java
// UserTimelineRepository.java
public interface UserTimelineRepository extends CassandraRepository<UserTimeline, UserTimelineKey> {
    
    @Query("SELECT * FROM user_timeline WHERE follower_id = ?0 ORDER BY created_at DESC LIMIT ?1")
    List<UserTimeline> findLatestTimeline(UUID followerId, int limit);
    
    @Query("SELECT * FROM user_timeline WHERE follower_id = ?0 AND created_at < ?1 ORDER BY created_at DESC LIMIT ?2")  
    List<UserTimeline> findTimelineBeforeCursor(UUID followerId, LocalDateTime cursor, int limit);
}
```

#### Service 계층

```java
// TimelineService.java
@Service
@RequiredArgsConstructor
public class TimelineService {
    
    private final UserTimelineRepository userTimelineRepository;
    private final CelebrityTweetRepository celebrityTweetRepository;
    private final FollowRepository followRepository;
    
    public TimelineResponse getTimeline(UUID userId, String cursor, int size) {
        // 1. 일반 트윗 조회
        List<UserTimeline> regularTweets = getRegularTweets(userId, cursor, size);
        
        // 2. 셀럽 트윗 조회  
        List<CelebrityTweet> celebrityTweets = getCelebrityTweets(userId, cursor, size);
        
        // 3. 시간순 병합
        List<TweetResponse> mergedTweets = mergeByTimestamp(regularTweets, celebrityTweets);
        
        // 4. 응답 생성
        return TimelineResponse.builder()
            .tweets(mergedTweets.subList(0, Math.min(size, mergedTweets.size())))
            .nextCursor(generateNextCursor(mergedTweets))
            .hasMore(mergedTweets.size() > size)
            .build();
    }
    
    @Async
    public void fanOutTweet(Tweet tweet) {
        if (celebrityService.isCelebrity(tweet.getUserId())) {
            // Fan-out on Read: celebrity_tweets에만 저장
            saveToCelebrityTweets(tweet);
        } else {
            // Fan-out on Write: 모든 팔로워의 타임라인에 저장
            List<UUID> followers = followRepository.findFollowersByUserId(tweet.getUserId());
            fanOutToFollowers(followers, tweet);
        }
    }
}
```

***

### 💡 트러블슈팅 가이드

#### 자주 발생하는 문제들

**1. 타임라인 조회가 느려요**

```
원인: Celebrity 트윗 병합 시 너무 많은 인플루언서 조회
해결: 
- 팔로우하는 셀럽 수 제한 (최대 100명)
- 셀럽 트윗 캐시 적극 활용
- 비동기로 병합 후 WebSocket으로 푸시
```

**2. Fan-out 쓰기가 너무 느려요**

```
원인: 팔로워가 많은 사용자를 일반 사용자로 처리
해결:
- 인플루언서 판단 임계값 조정 (1만 명 → 5천 명)
- 배치 사이즈 최적화 (한 번에 1000명씩 처리)
- 비동기 처리 with 메시지 큐
```

**3. 저장 공간이 부족해요**

```
원인: user_timeline 테이블의 과도한 데이터 중복
해결:
- 오래된 타임라인 데이터 정리 (3개월 이상)
- 압축 정책 적용
- 더 많은 사용자를 celebrity로 분류
```

**4. 일관성 문제가 발생해요**

```
문제: 트윗 삭제했는데 타임라인에 아직 보임
해결:
- 트윗 삭제 시 관련된 모든 타임라인에서도 삭제
- 최종 일관성 특성상 약간의 지연은 정상
- 중요한 경우 강한 일관성 레벨 사용 (QUORUM)
```

***

### 🎯 다음 단계 확장 방안

#### 1. 멀티미디어 지원

```java
// 이미지/비디오가 포함된 트윗 처리
@Column("media_urls")
private List<String> mediaUrls;

@Column("media_type") 
private String mediaType; // IMAGE, VIDEO, GIF
```

#### 2. 실시간 알림

```java
// WebSocket + 메시지 브로커로 실시간 타임라인 업데이트
@EventListener
public void onTweetCreated(TweetCreatedEvent event) {
    // Fan-out과 동시에 실시간 알림 발송
    notificationService.notifyFollowers(event.getTweet());
}
```

#### 3. 추천 알고리즘

```java
// 머신러닝 기반 트윗 순서 조정
public List<TweetResponse> applyRecommendationAlgorithm(List<TweetResponse> tweets) {
    // 사용자 관심사, 상호작용 기록 등을 고려한 순서 조정
    return mlService.rankTweets(tweets, userPreferences);
}
```

#### 4. 지역별 분산

```java
// 지역별 데이터센터 분산으로 레이턴시 최적화
@Table("user_timeline_asia")
@Table("user_timeline_americas") 
@Table("user_timeline_europe")
```

***

### 📚 참고 자료

* [Apache Cassandra 공식 문서](https://cassandra.apache.org/doc/)
* [Spring Data Cassandra 레퍼런스](https://docs.spring.io/spring-data/cassandra/docs/current/reference/html/)

***

### 🎉 마무리

대규모 소셜 미디어 플랫폼의 데이터베이스 설계 원칙의 핵심은 **사용 패턴에 맞는 최적화**와 **트레이드오프의 균형.**

**기억해야 할 핵심 포인트**:

1. **읽기 최적화**: 자주 조회되는 데이터는 미리 계산하여 저장
2. **쓰기 분산**: 비용과 성능을 고려한 하이브리드 전략
3. **확장성**: 사용자 증가에 따른 유연한 아키텍처
4. **모니터링**: 지속적인 성능 측정과 최적화
