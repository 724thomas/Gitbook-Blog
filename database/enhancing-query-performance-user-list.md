---
description: 조회 쿼리 성능 향상 - 유저 목록
---

# Enhancing Query Performance - User list

## 1. 상황 설명:

<figure><img src="../.gitbook/assets/image (257).png" alt="" width="360"><figcaption></figcaption></figure>

기존 요구사항:

* 유저 조회 API를 통해 유저 목록을 가져옵니다.
* 삭제된 유저, 차단한 유저는 제외합니다.
* 팔로워 수를 기준으로 정렬하여, 20개씩 불러옵니다.
* 유저 검색시 `User`, `UserStats`, `BlockUser` 테이블을 조인하고, 팔로워에 따른 정렬을 하게 됩니다.
  * `User`: 유저 정보
  * `UserStats`: 유저 메타정보 (`User`와 1:1 관계)
  * `BlockUser`: 유저의 차단 유저 정보 (`User`와 1:N 관계)



문제점:

* 출시한지 1달채 안됬지만, 유저 수가 빠르게 증가하고 있었습니다(DAU: 800, peak RPS: 1-4, peek QPS: 1-3).&#x20;
* 인덱스를 걸지 않은 상태이며, 현재 매번 조인, 정렬함에 있어서 성능저하가 발생할 수 있다고 판단했습니다.

***

## 2. 아키텍처

<figure><img src="../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

***

## 3. 테스트 환경 세팅

실제로 얼만큼의 성능 저하가 발생할지 파악하기 위해 로컬 테스트 환경을 구축했습니다. 테이블을 생성하고, 10만개의 랜덤 데이터를 삽입하였습니다.

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

***

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
* **INNER JOIN:** User와 UserSocialStats는 항상 1:1 관계.
* **LEFT JOIN**: BlockUser와의 조인을 통해 특정 유저(`12345`)가 차단한 유저를 식별.
* **WHERE**: 삭제된 유저와 차단된 유저를 제외.
* **ORDER BY**: 팔로워 수(followerCount) 기준 내림차순으로 정렬.
* **LIMIT**: 상위 20개의 결과를 반환.

***

## 5. 현재 상태

EXPLAIN, EXPLAIN ANALZY 키워드를 통해, 쿼리가 어떻게 실행이 되고 있는지 확인해봤습니다.

아래는 EXPLAIN, EXPLAIN ANALYZE, 시각화한 흐름도입니다.

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure></div>

{% code fullWidth="true" %}
```
-> Limit: 20 row(s)  (actual time=596..596 rows=20 loops=1)
    -> Sort: uss.followerCount DESC, limit input to 20 row(s) per chunk  (actual time=596..596 rows=20 loops=1)
        -> Stream results  (cost=17181 rows=9857) (actual time=0.0347..573 rows=100000 loops=1)
            -> Filter: (bu.userId is null)  (cost=17181 rows=9857) (actual time=0.0325..518 rows=100000 loops=1)
                -> Nested loop antijoin  (cost=17181 rows=9857) (actual time=0.0322..509 rows=100000 loops=1)
                    -> Nested loop inner join  (cost=13732 rows=9857) (actual time=0.0248..183 rows=100000 loops=1)
                        -> Filter: (u.deleted = false)  (cost=10282 rows=9857) (actual time=0.0194..63.5 rows=100000 loops=1)
                            -> Table scan on u  (cost=10282 rows=98569) (actual time=0.0185..49 rows=100000 loops=1)
                        -> Single-row index lookup on uss using PRIMARY (userId=u.id)  (cost=0.25 rows=1) (actual time=979e-6..0.001 rows=1 loops=100000)
                    -> Filter: (bu.blocked = true)  (cost=0.25 rows=1) (actual time=0.00309..0.00309 rows=0 loops=100000)
                        -> Single-row index lookup on bu using PRIMARY (userId=12345, targetUserId=u.id)  (cost=0.25 rows=1) (actual time=0.00297..0.00297 rows=0 loops=100000)

```
{% endcode %}

```
흐름도
1. Limit: 20 rows
    └─ 2. Sort: uss.followerCount DESC
        └─ 3. Filter: bu.userId is null
            └─ 4. Nested loop anti join
                ├─ 5. Nested loop inner join
                │   ├─ 6. Filter: u.deleted = false
                │   │   └─ 7. Table scan on User (10만 건)
                │   └─ 8. Index lookup on UserSocialStats (userId)
                └─ 9. Index lookup on BlockUser (userId=12345, targetUserId=u.id)

```

* **실행 시간**: 총 596**ms**가 걸렸습니다.&#x20;
* 문제 되는 병목 구간은 아래와 같습니다:
  * **User 테이블의 전체 스캔 (7)**
  * **UserSocialStats 테이블의 조인 10만번 (8)**
  * **BlockUser 테이블의 조인 10만번 (9)**
  * **OrderBy 후 Limit으로 조인 결과를 생성한 후 LIMIT 적용**

***

## 6. 첫번째 해결 방안

### 6.1. 인덱스 추가

* 정렬 비용 감소
  * 정렬에 걸리는 비용을 줄이기 위해 followerCount DESC 인덱스를 추가.
  * CREATE INDEX idx\_followerCount\_desc ON `UserSocialStats` (followerCount DESC);
* 안티조인 최적화
  * BlockUser(userId, targetUserId, blocked)에 복합 인덱스를 추가.
  * CREATE INDEX idx\_block\_user ON `BlockUser` (userId, targetUserId, blocked);
* INNER JOIN 최적화
  * UserSocialStats(userId)에 인덱스 추가.
  * CREATE INDEX idx\_userId ON `UserSocialStats` (userId);
* 조건 필터 최적화
  * deleted 필드에 인덱스를 추가
  * CREATE INDEX idx\_deleted ON `User` (deleted);

<div data-full-width="true"><figure><img src="../.gitbook/assets/image (414).png" alt=""><figcaption></figcaption></figure></div>

{% code fullWidth="true" %}
```
-> Limit: 20 row(s)  (cost=37334 rows=10) (actual time=0.0558..0.325 rows=20 loops=1)
    -> Filter: (bu.userId is null)  (cost=37334 rows=10) (actual time=0.0551..0.322 rows=20 loops=1)
        -> Nested loop antijoin  (cost=37334 rows=10) (actual time=0.0544..0.317 rows=20 loops=1)
            -> Nested loop inner join  (cost=24890 rows=10) (actual time=0.0407..0.14 rows=20 loops=1)
                -> Covering index scan on uss using idx_followerCount_desc  (cost=0.031 rows=20) (actual time=0.0217..0.0288 rows=20 loops=1)
                -> Filter: (u.deleted = false)  (cost=0.25 rows=0.5) (actual time=0.00506..0.0052 rows=1 loops=20)
                    -> Single-row index lookup on u using PRIMARY (id=uss.userId)  (cost=0.25 rows=1) (actual time=0.00461..0.00466 rows=1 loops=20)
            -> Filter: (bu.blocked = true)  (cost=0.25 rows=1) (actual time=0.00849..0.00849 rows=0 loops=20)
                -> Single-row index lookup on bu using PRIMARY (userId=12345, targetUserId=uss.userId)  (cost=0.25 rows=1) (actual time=0.0083..0.0083 rows=0 loops=20)
```
{% endcode %}

* **실행 시간**: 총 0.325**ms**가 걸렸습니다.
* 개선된 내용은 아래와 같습니다.
  * **ORDER BY 비용 제거:`UserSocialStats` 테이블에서 정렬된 인덱스를 사용됨.**&#x20;
    * **정렬 비용없이 상위 20개를 바로 읽음으로, Using filesort가 제거됐습니다.**
  * **조인 범위 축소: LIMIT이 먼저 적용됨**
    * **정렬된 테이블에서 LIMIT 20을 가져온 뒤 조인을 하게 됩니다.**
  * **JOIN 순서 최적화: 테이블 접근 횟수를 줄임**
    * **`UserSocialStats` -> `User` -> `BlockUser` 순으로 시작되며, User와 BlockUser는 인덱스가 사용됬습니다.**



***

## 7. 두번째 해결 방안

### 7.1. Materialized View 활용

Materialized View는 쿼리 데이터를 물리적으로 저장하는 View의 종류입니다. (일반적인 view는 캡슐화와 모듈화의 장점을 가지지만 실행시 실시간으로 계산하게 됩니다.)

MySQL에서는 아쉽게도 Mview가 없기때문에, 역정규화된 것 같은 테이블을 추가로 따로 생성합니다.

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

Mview를 만든 후에 동일한 결과를 가져오는 쿼리를 실행해보았습니다.

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
-> Limit: 20 row(s)  (cost=24826 rows=20) (actual time=0.071..0.11 rows=20 loops=1)
    -> Filter: (bu.userId is null)  (cost=24826 rows=20) (actual time=0.0706..0.109 rows=20 loops=1)
        -> Nested loop antijoin  (cost=24826 rows=20) (actual time=0.0699..0.107 rows=20 loops=1)
            -> Index scan on mv using idx_followerCount_desc  (cost=0.0472 rows=20) (actual time=0.064..0.0681 rows=20 loops=1)
            -> Filter: ((bu.blocked = true) and (bu.targetUserId = mv.id))  (cost=0.25 rows=1) (actual time=0.00173..0.00173 rows=0 loops=20)
                -> Single-row index lookup on bu using PRIMARY (userId=12345, targetUserId=mv.id)  (cost=0.25 rows=1) (actual time=0.00162..0.00162 rows=0 loops=20)
```
{% endcode %}

* **실행 시간**: 총 **0.11ms**가 걸렸습니다.&#x20;
* 추가 개선된 내용은 아래와 같습니다.
  * **조인 제거**
    * **`User`와 `UserSocialStats` 테이블을 미리 병합함으로써, 조인 연산이 완전히 사라졌습니다.**
  * **ORDER BY 비용 제거**
    * **`UserStatsMV`에 followerCount DESC 인덱스를 추가함으로써, using filesort 또한 사용되지 않습니다.**
  * **테이블 접근 최소화**
    * `UserStatsMV`에 필요한 데이터가 포함되고 있어서, 단일 테이블 조회 + 1회 안티 조인만 수행됩니다.

다만, MV를 사용하면 속도가 획기적으로 빨라지지만, 단점들이 존재하여, 적당한 trade-off를 고려해야했습니다.

***

## 8. MV의 문제점

현재 생성한 MV 테이블은 읽기 성능은 빠를지언정, 쓰기 성능에서 문제가 발생할 수 있습니다. 하나의 테이블을 갱신하면, MV 테이블도 같이 갱신해합니다. 그리고 갱신하는 과정에서 실시간 일관성이 깨질 수 있습니다.

현 상황에서는 데이터가 실시간으로 반영되지 않아도 사용자 신뢰성에 크게 영향을 미치지 않는다고 판단했습니다. 데이터 정확성을 조금 포기하는 방향으로, 3시간마다 동기화를 시켰습니다.



### 8.1. 동기화의 방법

Mview 테이블을 동기화하는 방법은 크게 2가지가 있고, 장단점이 또렷합니다.

* **Update**
  * 변경된 필드만 갱신하기 때문에 I/O 최소화.
  * 다만, <mark style="color:red;">데이터가 누락될 가능성</mark> 존재.
* **Delete -> Insert**
  * 정합성을 보장
  * 다만, <mark style="color:red;">가용성이 떨어짐.</mark>



### 8.2. 초기동기화 이벤트와 문제점(Delete -> Insert)

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

Delete->Insert 방식은 간단하게 구현이 가능했습니다. 다만, 다시 삽입을 하는 과정에서 테이블이 비어 있는 상태가 발생할 수 있습니다.&#x20;

이를 해결하기 위해 실시간 데이터 가용성을 보장하도록 로직을 변경했습니다.



### 8.2. Staging 테이블 교체

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
3. RENAME을 통해 Staging 테이블을 교체합니다. 교체하는 과정은 내부적으로 원자적으로 처리됩니다.
4. 기존 테이블을 삭제합니다.



해당 방식의 고려할 점들은 크게 2가지입니다.

* 쓰기 작업이 상대적으로 많아짐
  * 매번 새로운 데이터를 써야하기 때문에 다른 상황에서는 문제가 될 수 있습니다.
  * 다만, 3시간마다 10만개의 데이터를 쓰는 비용이 크지 않다고 판단하였습니다.
* 추가적인 디스크 공간 필요
  * 새로운 테이블을 생성하기때문에 추가적인 공간이 필요합니다.
  * 데이터 크기를 봤을때 상대적으로 적은 공간이기 때문에 수용 가능하다고 판단하였습니다.



## 9. 결과

* 미리 조인된 MV 테이블을 통해 기존 조회 속도 향상.  (364ms -> 0.011ms)
* Staging table 교체 방법으로 3시간마다 MV 동기화. (실시간 데이터 가용성 보장)

