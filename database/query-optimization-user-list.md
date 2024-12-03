---
description: SQL 쿼리 최적화 - 유저 목록
---

# Query optimization - User list

## 1. 상황 설명:

<figure><img src="../.gitbook/assets/image (257).png" alt="" width="360"><figcaption></figcaption></figure>

* 유저 검색시 User, UserStats, BlockUser 테이블을 조인하고, 팔로워에 따른 정렬을 하게 됩니다.
* 매번 조인을 하고, 정렬함에 있어서 문제가 발생할 수 있다고 판단했습니다.



## 2. 아키텍처

<figure><img src="../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

* User와 Stats: 1대1 관계, User와 Block: 1대 N관계입니다.
* API 호출시, 유저ID를 받습니다. 유저 ID는 차단한 유저들을 제외시키기 위함입니다.
* 차단되지 않고 삭제되지 않은 유저들의 목록중 20개를 가져오게 됩니다.



## 3. 테스트 환경 세팅

<details>

<summary> 테이블 생성 코드 (User, UserSocialStats, BlockUser)</summary>

```sql
CREATE TABLE User (
    id BIGINT PRIMARY KEY,
    fcmToken VARCHAR(2000),
    serviceName VARCHAR(255) UNIQUE,
    thumbnailUrl VARCHAR(255),
    backgroundImageUrl VARCHAR(255),
    name VARCHAR(255),
    birthDate DATE,
    gender VARCHAR(255),
    nickname VARCHAR(255) NOT NULL,
    introduce LONGTEXT,
    linkUrl VARCHAR(255),
    followViewable BOOLEAN,
    deleted BOOLEAN NOT NULL
);

CREATE TABLE UserSocialStats (
    userId BIGINT PRIMARY KEY,
    viewCount BIGINT DEFAULT 0,
    episodeTotalLikeCount BIGINT DEFAULT 0,
    episodeTotalHateCount BIGINT DEFAULT 0,
    shareCount BIGINT DEFAULT 0,
    episodeCount BIGINT DEFAULT 0,
    followerCount BIGINT DEFAULT 0,
    followingCount BIGINT DEFAULT 0,
    FOREIGN KEY (userId) REFERENCES User(id) ON DELETE CASCADE
);

CREATE TABLE BlockUser (
    userId BIGINT NOT NULL,
    targetUserId BIGINT NOT NULL,
    blocked BOOLEAN NOT NULL,
    blockedDt DATETIME NOT NULL,
    ignored BOOLEAN NOT NULL,
    ignoredDt DATETIME NOT NULL,
    PRIMARY KEY (userId, targetUserId),
    FOREIGN KEY (userId) REFERENCES User(id) ON DELETE CASCADE,
    FOREIGN KEY (targetUserId) REFERENCES User(id) ON DELETE CASCADE
);

```

</details>

<details>

<summary>10만개의 더미데이터 삽입</summary>

{% code fullWidth="true" %}
```sql
-- 재귀 호출 최대 깊이 설정
SET SESSION cte_max_recursion_depth = 1000000;

-- User 테이블에 더미 데이터 삽입
INSERT INTO User (id, nickname, fcmToken, serviceName, thumbnailUrl, backgroundImageUrl, name, birthDate, gender, introduce, linkUrl, followViewable, deleted)
WITH RECURSIVE cte (n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 100000 -- 생성하고 싶은 더미 데이터의 개수
)
SELECT 
    n AS id,                                       -- 고유 ID
    CONCAT('nickname_', LPAD(n, 7, '0')) AS nickname, -- 'nickname_0000001' 형식
    CONCAT('fcm_token_', LPAD(n, 7, '0')) AS fcmToken, -- 'fcm_token_0000001' 형식
    CONCAT('service_user_', LPAD(n, 7, '0')) AS serviceName, -- 'service_user_0000001' 형식
    CONCAT('thumbnail_', LPAD(n, 7, '0'), '.png') AS thumbnailUrl,
    CONCAT('background_', LPAD(n, 7, '0'), '.png') AS backgroundImageUrl,
    CONCAT('User Name ', n) AS name,
    DATE_ADD('2000-01-01', INTERVAL n MOD 365 DAY) AS birthDate,
    CASE WHEN n % 2 = 0 THEN 'MALE' ELSE 'FEMALE' END AS gender,
    CONCAT('This is user number ', n) AS introduce,
    CONCAT('http://example.com/user', LPAD(n, 7, '0')) AS linkUrl,
    TRUE AS followViewable,
    FALSE AS deleted
FROM cte;

-- UserSocialStats 테이블에 더미 데이터 삽입
INSERT INTO UserSocialStats (userId, viewCount, episodeTotalLikeCount, episodeTotalHateCount, shareCount, episodeCount, followerCount, followingCount)
WITH RECURSIVE cte (n) AS (
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 100000 -- 생성하고 싶은 더미 데이터의 개수
)
SELECT 
    n AS userId,                           -- User 테이블의 ID와 매핑
    FLOOR(RAND() * 1000) AS viewCount,     -- 랜덤 조회수 (0~999)
    FLOOR(RAND() * 500) AS episodeTotalLikeCount, -- 랜덤 좋아요 수 (0~499)
    FLOOR(RAND() * 100) AS episodeTotalHateCount, -- 랜덤 싫어요 수 (0~99)
    FLOOR(RAND() * 300) AS shareCount,     -- 랜덤 공유 수 (0~299)
    FLOOR(RAND() * 50) AS episodeCount,    -- 랜덤 에피소드 수 (0~49)
    FLOOR(RAND() * 500) AS followerCount,  -- 랜덤 팔로워 수 (0~499)
    FLOOR(RAND() * 300) AS followingCount  -- 랜덤 팔로잉 수 (0~299)
FROM cte;

```
{% endcode %}

</details>



## 4. 실행 계획 분석을 위한 SQL 쿼리

{% code fullWidth="true" %}
```sql
SELECT 
    u.id AS id,
    u.nickname AS nickname,
    u.serviceName AS serviceName,
    u.thumbnailUrl AS thumbnailUrl,
    u.backgroundImageUrl AS backgroundImageUrl,
    uss.followerCount AS followerCount
FROM 
    User u
INNER JOIN 
    UserSocialStats uss ON uss.userId = u.id
LEFT JOIN 
    BlockUser bu ON bu.userId = 12345
                 AND bu.targetUserId = u.id 
                 AND bu.blocked = TRUE
WHERE 
    u.deleted = FALSE
    AND (bu.userId IS NULL)
ORDER BY uss.followerCount DESC
LIMIT 20 OFFSET 0;

```
{% endcode %}

* **SELECT**: User와 UserSocialStats 테이블에서 유저 정보(닉네임, 서비스 이름, 팔로워 수 등)를 가져옴.
* **FROM 및 JOIN**:
  * User와 UserSocialStats를 조인하여 유저 정보와 통계를 결합.
  * BlockUser와의 조인을 통해 특정 유저(`12345`)가 차단한 유저를 식별.
* **WHERE**: 삭제된 유저와 차단된 유저를 제외.
* **ORDER BY**: 팔로워 수(followerCount) 기준 내림차순으로 정렬.
* **LIMIT**: 상위 20개의 결과를 반환.



## 5. 현재 상태

### 5.1. EXPLAIN

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure></div>

* User 테이블의 전체 스캔이 발생하고 있습니다.
* 조인 키를 통해 UserSocialStats, BlockUser 테이블에서는 각 조인마다 1개의 Row가 읽히고 있습니다.

### 5.2. EXPLAIN ANALYZE

{% code fullWidth="true" %}
```
-> Limit: 20 row(s)  (actual time=364..364 rows=20 loops=1)
    -> Sort: uss.followerCount DESC, limit input to 20 row(s) per chunk  (actual time=364..364 rows=20 loops=1)
        -> Stream results  (cost=17000 rows=10000) (actual time=0.0708..343 rows=100000 loops=1)
            -> Filter: (bu.userId is null)  (cost=17000 rows=10000) (actual time=0.0679..290 rows=100000 loops=1)
                -> Nested loop antijoin  (cost=17000 rows=10000) (actual time=0.0675..281 rows=100000 loops=1)
                    -> Nested loop inner join  (cost=13500 rows=10000) (actual time=0.0639..163 rows=100000 loops=1)
                        -> Filter: (u.deleted = false)  (cost=10000 rows=10000) (actual time=0.0193..60.1 rows=100000 loops=1)
                            -> Table scan on u  (cost=10000 rows=99999) (actual time=0.0183..46.2 rows=100000 loops=1)
                        -> Single-row index lookup on uss using PRIMARY (userId=u.id)  (cost=0.25 rows=1) (actual time=831e-6..856e-6 rows=1 loops=100000)
                    -> Filter: (bu.blocked = true)  (cost=0.25 rows=1) (actual time=0.00101..0.00101 rows=0 loops=100000)
                        -> Single-row index lookup on bu using PRIMARY (userId=12345, targetUserId=u.id)  (cost=0.25 rows=1) (actual time=894e-6..894e-6 rows=0 loops=100000)

```
{% endcode %}

* **실행 시간**: 총 **364ms**가 걸렸습니다.&#x20;
* 문제 되는 병목 구간은 아래와 같습니다:
  * **User 테이블의 전체 스캔 (8쨰줄)**
  * **followerCount DESC 정렬 (2째줄)**
  * **BlockUser 필터링 (4쨰줄)**
  * **InnerJoin (5번째, 5번째)**



## 6. 첫번째 해결 방안

### 6.1. 인덱스 추가

* 정렬에 걸리는 비용을 줄이기 위해 followerCount DESC 인덱스를 추가.
  * CREATE INDEX idx\_followerCount\_desc ON UserSocialStats (followerCount DESC);
* BlockUser(userId, targetUserId, blocked)에 복합 인덱스를 추가.
  * CREATE INDEX idx\_block\_user ON BlockUser (userId, targetUserId, blocked);
* UserSocialStats(userId)에 인덱스 추가.
  * CREATE INDEX idx\_userId ON UserSocialStats (userId);
* deleted 필드에 인덱스를 추가
  * CREATE INDEX idx\_deleted ON User (deleted);

{% code fullWidth="true" %}
```
-> Limit: 20 row(s)  (actual time=425..425 rows=20 loops=1)
    -> Sort: uss.followerCount DESC, limit input to 20 row(s) per chunk  (actual time=425..425 rows=20 loops=1)
        -> Stream results  (cost=44522 rows=49284) (actual time=0.0444..404 rows=100000 loops=1)
            -> Filter: (bu.userId is null)  (cost=44522 rows=49284) (actual time=0.0416..354 rows=100000 loops=1)
                -> Nested loop antijoin  (cost=44522 rows=49284) (actual time=0.041..346 rows=100000 loops=1)
                    -> Nested loop inner join  (cost=27272 rows=49284) (actual time=0.0373..228 rows=100000 loops=1)
                        -> Index lookup on u using idx_deleted (deleted=false)  (cost=10023 rows=49284) (actual time=0.0316..124 rows=100000 loops=1)
                        -> Single-row index lookup on uss using PRIMARY (userId=u.id)  (cost=0.25 rows=1) (actual time=846e-6..871e-6 rows=1 loops=100000)
                    -> Filter: (bu.blocked = true)  (cost=0.25 rows=1) (actual time=0.00101..0.00101 rows=0 loops=100000)
                        -> Single-row index lookup on bu using PRIMARY (userId=12345, targetUserId=u.id)  (cost=0.25 rows=1) (actual time=895e-6..895e-6 rows=0 loops=100000)

```
{% endcode %}

**실행 시간**: 총 425**ms**가 걸렸습니다.  더 오래걸렸습니다.



### 6.2. 원인 분석

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure></div>

위 실행 계획 이미지를 보면 MySQL 옵티마이저가 인덱스를 추가했음에도 불구하고 효율적인 실행 계획을 선택하지 못하고 있습니다.

User 테이블: 실행 계획의 추가 정보에 using temporary; using filesort 를 통해 임시 테이블 작업을 하고 있습니다. 인덱스를 제대로 활용하지 않고있습니다.

UserSocialStarts 테이블:    ORDER BY에서 인덱스를 활용하지 않고 있습니다.

BlockUser 테이블: idx\_block\_user를 사용하고 있지만 반복 작업이 많습니다.



## 7. 두번째 해결 방안

### 7.1. Materialized View 활용

Materialized View는 쿼리 데이터를 물리적으로 저장하는 View의 종류입니다. (일반적인 view는 캡슐화와 모듈화의 장점을 가지지만 실행시 실시간으로 계산하게 됩니다.)

MySQL에서는 아쉽게도 MV가 없기때문에, 마치 역 정규화된 것 같은 테이블을 하나 생성합니다.

{% code fullWidth="false" %}
```sql
CREATE TABLE UserStatsMV (
    id INT PRIMARY KEY,                
    nickname VARCHAR(255),             
    serviceName VARCHAR(255),          
    thumbnailUrl VARCHAR(255),         
    backgroundImageUrl VARCHAR(255),   
    followerCount INT,                 
    viewCount INT,                     
    episodeTotalLikeCount INT,         
    episodeTotalHateCount INT,         
    shareCount INT,                    
    episodeCount INT                   
);

INSERT INTO UserStatsMV (id, nickname, serviceName, thumbnailUrl, backgroundImageUrl, followerCount, viewCount, episodeTotalLikeCount, episodeTotalHateCount, shareCount, episodeCount)
SELECT 
    u.id,
    u.nickname,
    u.serviceName,
    u.thumbnailUrl,
    u.backgroundImageUrl,
    uss.followerCount,
    uss.viewCount,
    uss.episodeTotalLikeCount,
    uss.episodeTotalHateCount,
    uss.shareCount,
    uss.episodeCount
FROM 
    User u
INNER JOIN 
    UserSocialStats uss ON u.id = uss.userId;
WHERE u.deleted=FALSE;
```
{% endcode %}

MV를 만든 후에 동일한 결과를 가져오는 쿼리를 실행해보겠습니다

{% code fullWidth="false" %}
```sql
EXPLAIN ANALYZE SELECT 
    mv.id AS id,
    mv.nickname AS nickname,
    mv.serviceName AS serviceName,
    mv.thumbnailUrl AS thumbnailUrl,
    mv.backgroundImageUrl AS backgroundImageUrl,
    mv.followerCount AS followerCount
FROM 
    UserStatsMV mv
LEFT JOIN 
    BlockUser bu ON bu.userId = 12345
                 AND bu.targetUserId = mv.id
                 AND bu.blocked = TRUE
WHERE 
    bu.userId IS NULL -- 차단되지 않은 사용자만 조회
ORDER BY 
    mv.followerCount DESC
LIMIT 20 OFFSET 0;
```
{% endcode %}

{% code fullWidth="true" %}
```
-> Limit: 20 row(s)  (cost=44915 rows=20) (actual time=96.5..96.6 rows=20 loops=1)
    -> Filter: (bu.userId is null)  (cost=44915 rows=99294) (actual time=96.5..96.6 rows=20 loops=1)
        -> Nested loop antijoin  (cost=44915 rows=99294) (actual time=96.5..96.6 rows=20 loops=1)
            -> Sort: mv.followerCount DESC  (cost=10162 rows=99294) (actual time=96.5..96.5 rows=20 loops=1)
                -> Table scan on mv  (cost=10162 rows=99294) (actual time=0.0237..36.8 rows=100000 loops=1)
            -> Filter: ((bu.blocked = true) and (bu.targetUserId = mv.id))  (cost=0.25 rows=1) (actual time=0.00203..0.00203 rows=0 loops=20)
                -> Single-row index lookup on bu using PRIMARY (userId=12345, targetUserId=mv.id)  (cost=0.25 rows=1) (actual time=0.0019..0.0019 rows=0 loops=20)
```
{% endcode %}

* **실행 시간**: 총 **96ms**가 걸렸습니다.&#x20;
* 문제 되는 병목 구간은 아래와 같습니다:
  * **UserStatsMV 테이블의 전체 스캔 (5쨰줄)**
  * **followerCount DESC 정렬 (4째줄)**

### 7.2. 추가 최적화

정렬에 걸리는 비용을 줄이기 위해 followerCount DESC 인덱스를 추가.

* CREATE INDEX idx\_followerCount\_desc ON UserStatsMV (followerCount DESC);

{% code fullWidth="true" %}
```
-> Limit: 20 row(s)  (cost=24826 rows=20) (actual time=0.071..0.11 rows=20 loops=1)
    -> Filter: (bu.userId is null)  (cost=24826 rows=20) (actual time=0.0706..0.109 rows=20 loops=1)
        -> Nested loop antijoin  (cost=24826 rows=20) (actual time=0.0699..0.107 rows=20 loops=1)
            -> Index scan on mv using idx_followerCount_desc  (cost=0.0472 rows=20) (actual time=0.064..0.0681 rows=20 loops=1)
            -> Filter: ((bu.blocked = true) and (bu.targetUserId = mv.id))  (cost=0.25 rows=1) (actual time=0.00173..0.00173 rows=0 loops=20)
                -> Single-row index lookup on bu using PRIMARY (userId=12345, targetUserId=mv.id)  (cost=0.25 rows=1) (actual time=0.00162..0.00162 rows=0 loops=20)

```
{% endcode %}

* 실행시간이 0.071ms로 줄었습니다. 기존 364ms에 비교했을때 99%의 시간을 단축할 수 있었습니다.



## 8. MV의 문제점

현재 생성한 MV 테이블은 읽기 성능은 빠를지언정, 쓰기 성능에서 문제가 발생할 수 있습니다. 하나의 테이블을 갱신하면, MV 테이블도 같이 갱신해야하기 떄문입니다.

현 상황에서 데이터가 실시간으로 변경되지 않아도 사용자 신뢰성에 크게 영향을 미치지 않는다고 판단했습니다. 데이터 정확성을 조금 포기하는 방향으로, 3시간마다 동기화를 시켰습니다.



### 8.1. 초기동기화 이벤트와 문제점

```sql
CREATE EVENT sync_user_stats_mv
ON SCHEDULE EVERY 1 HOUR
STARTS CURRENT_TIMESTAMP
ON COMPLETION PRESERVE
ENABLE
COMMENT 'Sync UserStatsMV table with User and UserSocialStats data every hour'
DO
BEGIN
    TRUNCATE TABLE UserStatsMV;

    INSERT INTO UserStatsMV (id, nickname, serviceName, thumbnailUrl, backgroundImageUrl, followerCount, viewCount, episodeTotalLikeCount, episodeTotalHateCount, shareCount, episodeCount)
    SELECT 
        u.id,
        u.nickname,
        u.serviceName,
        u.thumbnailUrl,
        u.backgroundImageUrl,
        uss.followerCount,
        uss.viewCount,
        uss.episodeTotalLikeCount,
        uss.episodeTotalHateCount,
        uss.shareCount,
        uss.episodeCount
    FROM User u
    INNER JOIN UserSocialStats uss ON u.id = uss.userId
    WHERE u.deleted = FALSE;
END;
```

위 이벤트는 기존 MV테이블의 데이터만 삭제한 후에, 다시 삽입하게 됩니다. 다만, 다시 삽입을 하는 과정에서 테이블이 비어 있는 상태가 발생할 수 있습니다. 이를 해결하기 위해 실시간 데이터 가용성을 보장하도록 로직을 변경했습니다.



### 8.2. 실시간 데이터 가용성 보장

```sql
SET GLOBAL event_scheduler = ON;

CREATE EVENT sync_user_stats_mv
ON SCHEDULE EVERY 3 HOUR
STARTS CURRENT_TIMESTAMP
ON COMPLETION PRESERVE
ENABLE
COMMENT 'Create temp MV, switch with current MV, remove orignal MV'
DO
BEGIN
    -- 1. 임시 테이블 생성
    CREATE TABLE IF NOT EXISTS UserStatsMV_temp LIKE UserStatsMV;

    -- 2. 임시 테이블 데이터 초기화
    TRUNCATE TABLE UserStatsMV_temp;

    -- 3. 임시 테이블에 최신 데이터 삽입
    INSERT INTO UserStatsMV_temp (id, nickname, serviceName, thumbnailUrl, backgroundImageUrl, followerCount, viewCount, episodeTotalLikeCount, episodeTotalHateCount, shareCount, episodeCount)
    SELECT 
        u.id,
        u.nickname,
        u.serviceName,
        u.thumbnailUrl,
        u.backgroundImageUrl,
        uss.followerCount,
        uss.viewCount,
        uss.episodeTotalLikeCount,
        uss.episodeTotalHateCount,
        uss.shareCount,
        uss.episodeCount
    FROM User u
    INNER JOIN UserSocialStats uss ON u.id = uss.userId
    WHERE u.deleted = FALSE;

    -- 4. 기존 테이블과 임시 테이블 교체
    RENAME TABLE UserStatsMV TO UserStatsMV_old, UserStatsMV_temp TO UserStatsMV;

    -- 5. 이전 테이블 삭제
    DROP TABLE UserStatsMV_old;
END;
```

프로세스:

1. 임시 테이블을 생성하게 됩니다. (UserStatsMV\_temp)
2. 임시 데이블에 데이터를 삽입합니다.
3. RENAME을 통해 기존 MV와 새로운 MV를 교체합니다. ( UserStatsMV <--> UserStatsMV\_temp)
4. 기존 테이블을 삭제합니다.



## 9. 결과

* 미리 조인된 MV 테이블을 통해 기존 조회 속도 향상.  (364ms -> 0.071ms)
* MV 동기화를 위해 3시간 단위로 실행되는 트리거 생성. (실시간 데이터 가용성 보장)
* 다만, 데이터 가용성을 보장하는 과정에서 추가적인 디스크 공간 필요.

