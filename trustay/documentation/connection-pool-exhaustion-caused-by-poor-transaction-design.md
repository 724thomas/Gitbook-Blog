---
description: 트랜잭션 설계 문제로 발생한 커넥션 풀 고갈 이슈와 해결 과정 정리
---

# Connection Pool Exhaustion Caused by Poor Transaction Design

## 상황

앱에서 게시글 등록시 요청이 실패하는 상황.

***

## 원인

* 팔로워 목록을 커서 방식으로 조회하는 로직에서 마지막 id 업데이트가 잘못되어 **루프가 비정상적으로 오래 실행되는 상황이 발생. 이 로직이 트랜잭션 내부에서 실행되면서 문제가 증폭됨.**
* 해당 로직은 **Post 등록 트랜잭션 내부에서 동기 이벤트 리스너를 통해 실행되고 있었고**, 이벤트 처리 과정에서 DB 수정 작업과 `Thread.sleep` 이 반복되면서 **트랜잭션이 장시간 종료되지 않은 채 유지되었다.**
* 이로 인해 post, postMedia, channelNotification, pushNotification 테이블에 대한 락이 장시간 유지되었고, 락 대기 상태에 빠진 트랜잭션들이 DB 커넥션을 반환하지 못하면서 커넥션 풀 고갈로 이어졌다.

***

## 해결 과정

### 1. 쓰레드 풀 고갈이라는 메시지를 **애플리케이션 로그, 모니터링(Datadog)** 에서 확인

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* "Connection is not available, request timed out after 3000ms."
* HikariPool-1 - Connection is not available, request timed out after 3000ms.

***

### 2. DB에서 실행 중인 작업들을 **`SHOW FULL PROCESSLIST`** 명령어를 통해 확인

```sql
SHOW FULL PROCESSLIST;
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

* User: 어떤 계정이
* Host: 어떤 host(IP)에서
* States: 어떤 상태(Sleep / Query)로
* Time: 얼마나 오래 살아있는지를 1차로 파악

***

### 3. 락 대기 중인 작업들을 **`performance_schema.data_lock_waits`** 명령어를 통해 확인

```sql
SELECT * FROM performance_schema.data_lock_waits;
```

<div data-full-width="false"><figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure></div>

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

* 현재 **락 때문에 대기 중인 트랜잭션이 존재하는지**
* 있다면 **누가 기다리고 있고, 누가 막고 있는지** 확인

***

### 4. 실행 중인 작업들 중, 종료되지 않고 오랫동안 살아있는 쓰레드를

**`information_schema.innodb_trx` + `PROCESSLIST` 조합**을 통해 확인

```sql
SELECT
    trx_id,
    trx_started,
    TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS trx_seconds,
    trx_mysql_thread_id,
    trx_rows_modified,
    trx_tables_locked
FROM information_schema.innodb_trx;
```

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure></div>

* trx\_seconds: 수 시간 이상 살아있는 트랜잭션
* `trx_rows_modified: Sleep` 상태인데 `rows_modified > 0`
* trx\_tables\_locked: 여러 테이블에 락을 잡고 있는 트랜잭션

이 단계에서 **문제의 쓰레드(thread id)** 를 특정

***

### 5. 해당 쓰레드의 실행 SQL을

**`performance_schema.events_statements_history_long`** 를 통해 확인 시도. 설정값에 따라 너무 오래되면 제공하지 않음.

```sql
SELECT
    th.PROCESSLIST_ID,
    es.SQL_TEXT,
    es.EVENT_ID
FROM performance_schema.events_statements_history_long es
JOIN performance_schema.threads th
  ON th.THREAD_ID = es.THREAD_ID
WHERE th.PROCESSLIST_ID = <문제_thread_id>
ORDER BY es.EVENT_ID DESC
LIMIT 20;
```

* 트랜잭션이 `Sleep` 상태라면 SQL이 남아있지 않을 수도 있음
* 이 경우 **실행 SQL은 애플리케이션 로그를 통해 역추적**

어떤 SQL이 문제를 발생시켰는지 트랜잭션이 sleep 상태라서 SQL이 남아있지 않아 **역추적이 어려웠던 상황**이었고, **우선적으로 해당 프로세스를 KILL 명령어를 통해 종료하여 락을 해제하였습니다.**

```sql
KILL <thread_id>;
```

락이 해제되자, 대기 중이던 다음 트랜잭션이 즉시 실행되었는데, 해당 트랜잭션 역시 동일한 구조의 문제를 가지고 있었고, 이번에는 실행 중인 SQL이 PROCESSLIST 상에 남아 있어 **SQL 상태를 역추적할 수 있었습니다.**

이를 통해 문제의 원인이 post를 작성하는 메서드 내부에서 발생한 트랜잭션 설계 문제임을 확인할 수 있었습니다.\
동일하게 해당 트랜잭션 역시 KILL 하여 종료하였습니다.

#### KILL시 트랜잭션을 강제로 종료하는데, 데이터는 어떻게 되는가?

**커밋되지 않은 변경 사항은 모두 자동으로 롤백.**

* KILL로 종료된 트랜잭션은
  * 아직 COMMIT 되지 않은 변경 사항만 롤백.
  * 이미 커밋된 데이터에는 영향 없음
* InnoDB는 트랜잭션 단위의 원자성을 보장하기에 중간 상태의 데이터가 남는 일은 없음.

그러므로, post 등록 상황 자체가 취소되며 모두 안전하게 롤백되었고, 락만 해제된 상태로 DB는 정상화되었습니다.

***

### 6. 문제 발생한 Cursor방식의 페이징 처리.

```kotlin
    // Repository
    override fun findAllFollowersIdWithCursor(
        channelId: Long,
        lastId: Long?,
        size: Long,
    ): List<Follow> {
        val whereCondition = BooleanBuilder()
            .and(follow.following.id.eq(channelId))
        if (lastId != null) {
            whereCondition.and(follow.id.gt(lastId))
        }

        return queryFactory.select(follow)
            .from(follow)
            .where(whereCondition)
            .orderBy(follow.id.asc())
            .limit(size)
            .fetch()
    }
    
    // Service
    ...
    var lastId: Long? = null

        while (true) {
            try {
                val followingChannelIdsCursor = followRepository.findAllFollowersIdWithCursor(
                    channelId = event.actorChannel.id,
                    lastId = lastChannelId,
                    size = BATCH_SIZE,
                )

                if (followingChannelIdsCursor.isEmpty()) {
                    break
                }

                lastId = followingChannelIdsCursor.last().follower.id
                // 문제되는 부분!!!!!
                ....
            }
        }
    ...
    }    
```

Repository에서는 follow 엔티티의 id를 기준으로 오름차순 정렬 및 페이징을 수행하고 있었으나,\
Service 레이어에서는 마지막 커서 값으로 follow.id가 아닌 follower.id를 사용하고 있었다.

이로 인해 다음 페이지 조회 시 이미 조회한 follow **레코드가 반복 조회되거나, 동일 범위를 반복 조회하는 문제**가 발생하였고, 결과적으로 루프가 비정상적으로 오래 실행되는 상황이 만들어졌다.



### 7. 추후 고려해야할 것

1. 긴 로직을 하나의 트랜잭션으로 감싸고 있음
   1. 트랜잭션을 작게 쪼갠다
2. @Transactional + 동기 @EventListener
   1. **@TransactionalEventListener(phase = AFTER\_COMMIT):** 이벤트는 AFTER\_COMMIT에서 처리
   2. **@Async + Transactional(propagation = REQUIRES\_NEW):** 비동기 이벤트 + 독립 트랜잭션

## 체크리스트

### 1. 열린 트랜잭션 확인 (위험 신호)

```sql
SELECT
  trx_id,
  trx_started,
  TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS trx_seconds,
  trx_mysql_thread_id,
  trx_rows_modified,
  trx_tables_locked
FROM information_schema.innodb_trx
ORDER BY trx_started;
```

* trx\_seconds: 오래 실행됨
* trx\_rows\_modiefied>0: Update/Insert 쿼리
* trx\_tables\_locked >=1: 락으로 점유하고 있는 테이블들



```sql
SELECT * FROM performance_schema.data_lock_waits;
```

* 결과가 나오면 이미 장애 발생



### 2. 애플리케이션 관점에서의 로그 확인

* "Connection is not available, request timed out"
* "HikariPool - Timeout after xxx ms
* 요청 처리 시간이 비정상적으로 길어짐



## 배운점

이번 장애를 통해&#x20;

* 트랜잭션의 크기와 수명이 시스템 안정성에 직접적인 영향을 준다는 것을 체감하였고,
* DB 락 문제의 대부분은 트랜잭션 설계 문제에서 시작될 수 있다는 것을 알 수 있었고,
* 응급조치를 어떻게 해야하는지 큰 그림이 그려졌습니다.
