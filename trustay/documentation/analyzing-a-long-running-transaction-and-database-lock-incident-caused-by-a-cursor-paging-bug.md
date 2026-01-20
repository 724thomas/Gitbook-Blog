---
description: 커서 페이징 오류로 인한 장시간 트랜잭션과 DB Lock 장애 분석
---

# Analyzing a Long-Running Transaction and Database Lock Incident Caused by a Cursor Paging Bug

(사내 보안 규정에 따라 실제 서비스·테이블·코드 식별자는 모두 일반화되어 있습니다)



**단 하나의 커서 업데이트 실수**가 어떻게

* 장시간 트랜잭션 유지
* 테이블 락 확산
* 커넥션 풀 고갈
* 서비스 장애

로 이어졌는지를, **실제 장애 분석 흐름 그대로** 정리하였습니다.



## TL;DR

* **문제**\
  커서 기반 페이징 로직의 커서 값 불일치로 루프가 비정상적으로 길어졌고,\
  해당 로직이 트랜잭션 내부 + 동기 이벤트 리스너에서 실행되며 DB 락이 장시간 유지됨
* **결과**\
  여러 테이블 락 → 커넥션 풀 고갈 → 요청 타임아웃 발생
* **해결**\
  문제 트랜잭션 식별 후 강제 종료(KILL),\
  커서 기준 오류 수정 및 이벤트/트랜잭션 구조 개선



## 배경

게시글(Post) 등록시 다음 작업이 함께 수행되고 있었습니다.

* 게시글 저장
* 미디어 저장
* 팔로워 대상 알림 생성

이 중 **팔로워 목록을 커서 기반 페이징으로 순회하는 로직**이\
게시글 등록 트랜잭션 내부에서 동기 이벤트 리스너를 통해 실행되고 있었습니다.

즉,

하나의 Post 등록 트랜잭션이 팔로워 전체 순회 + 알림 생성이 끝날 때까지 종료되지 않는 구조



## 문제 발생 징후

### 1. 애플리케이션 로그에서 발견된 신호

운영 중 아래와 같은 로그가 다수 발생하기 시작했습니다.

```sql
Connection is not available, request timed out after 3000ms
HikariPool-1 - Connection is not available, request timed out after 3000ms
```

이는 **DB 커넥션 풀 고갈**의 전형적인 증상입니다.



## 장애 분석 과정

### 2. 실행 중인 DB 작업 확인

먼저 MySQL에서 현재 실행 중인 스레드를 확인했습니다.

```sql
SHOW FULL PROCESSLIST;
```

확인 포인트는 다음과 같았습니다.

* **Time**: 수십 분 이상 살아있는 커넥션
* **State**: `Sleep` 또는 `Query`
* **Host**: 동일한 애플리케이션 노드

이미 이 단계에서 **정상적이지 않은 장시간 트랜잭션**이 의심되었습니다.

### 3. Lock 대기 상태 확인

락 대기 여부를 확인하기 위해 다음 쿼리를 실행했습니다.

```sql
SELECT * FROM performance_schema.data_lock_waits;
```

결과:

* 여러 트랜잭션이 락 대기 상태
* 특정 트랜잭션이 다수의 락을 점유 중

즉, **하나의 트랜잭션이 다른 모든 트랜잭션을 막고 있는 상황**이었습니다.

### 4. 장시간 트랜잭션 식별

다음으로 InnoDB 트랜잭션 정보를 확인했습니다.

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

문제가 된 트랜잭션의 특징:

* `trx_seconds`: 수 시간 이상
* `trx_rows_modified > 0`
* `trx_tables_locked >= 2`
* 상태는 `Sleep`

즉, **이미 데이터를 수정했고, 여러 테이블에 락을 잡은 채, 트랜잭션이 종료되지 않고 대기 중**인 상태였습니다.

### 5. 실행 SQL 역추적의 어려움

해당 스레드의 실행 SQL을 확인하려 했으나,

```sql
SELECT
  th.PROCESSLIST_ID,
  es.SQL_TEXT
FROM performance_schema.events_statements_history_long es
JOIN performance_schema.threads th
ON th.THREAD_ID = es.THREAD_ID
WHERE th.PROCESSLIST_ID = <thread_id>;
```

트랜잭션이 이미 `Sleep` 상태로 전환된 뒤라\
실행 SQL이 남아있지 않은 상황이었습니다.

그래서, **애플리케이션 로그를 통해 역추적**할 수밖에 없었습니다.



## 응급 조치: 트랜잭션 강제 종료

우선 시스템 정상화를 위해 문제 스레드를 종료했습니다.

```sql
KILL <thread_id>;
```

#### KILL 시 데이터는 어떻게 되는가?

* COMMIT 되지 않은 변경 사항 → **모두 자동 롤백**
* 이미 COMMIT 된 데이터 → 영향 없음
* InnoDB 트랜잭션 원자성 보장

다시말해, **post 등록 자체가 통째로 취소되었고, 데이터 정합성은 유지**되었습니다.

락이 해제되자 대기 중이던 다음 트랜잭션이 즉시 실행되었고,\
이때는 실행 중인 SQL이 PROCESSLIST에 남아 있었습니다.



## 근본 원인 분석

#### 문제의 커서 페이징 로직

```kotlin
// Repository
findAllFollowersIdWithCursor(
  channelId,
  offSetId,
  size
): List<Follow> {
  where follow.id > offsetId
  order by follow.id asc
}

// Service
offsetId = idsCursor.last().follower.id
```

#### 문제 핵심

* **Repository**
  * `follow.id` 기준으로 정렬 + 페이징
* **Service**
  * 마지막 커서로 `follower.id`를 사용

**조회 기준과 커서 업데이트 기준이 서로 달랐음**

이로 인해:

* 동일한 follow 레코드가 반복 조회
* 커서가 정상적으로 전진하지 않음
* while 루프가 비정상적으로 오래 실행



## 왜 장애로 이어졌는가

### 1. 트랜잭션 내부에서 실행

해당 로직은:

* Post 등록 트랜잭션 내부
* DB Insert/Update 이후
* COMMIT 전 상태

에서 실행되고 있었습니다.

### 2. 동기 이벤트 리스너 사용

* `@EventListener` (동기)
* 이벤트 처리 중에도 트랜잭션 유지
* 내부에서 `Thread.sleep()` 포함

결과적으로, **DB 락을 잡은 상태로 애플리케이션 스레드가 장시간 대기**하게 되었습니다.

### 3. 락 -> 커넥션 풀 고갈 -> 장애

* post, image, notification, xxx 테이블 락 유지
* 다른 트랜잭션이 락 대기 상태로 전환
* 커넥션 반환 불가
* HikariPool 고갈
* 서비스 요청 타임아웃 발생



## 개선 방향 및 재발 방지

### 1. 트랜잭션 경계 축소

* 긴 로직을 하나의 트랜잭션에 포함하지 않고, 트랜잭션을 작게 쪼개기

### 2. 이벤트 처리 구조 개선

* **TransactionalEventListener(phase = AFTER\_COMMIT):** 트랜잭션 종료 후 이벤트 처리
* **@Async + REQUIRES\_NEW**: 독립 트랜잭션에서 비동기 처리



## 체크리스트

장애를 겪고난뒤, 체크리스트를 만들었습니다.

#### 1. DB 관점

```sql
SELECT
  trx_id,
  trx_started,
  TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS trx_seconds,
  trx_rows_modified,
  trx_tables_locked
FROM information_schema.innodb_trx;
```

* 오래 살아있는 트랜잭션
* rows\_modified > 0
* tables\_locked >= 1\
  → **즉시 위험 신호**

```sql
SELECT * FROM performance_schema.data_lock_waits;
```

* 결과가 존재하면 이미 장애 진행 중

***

#### 2. 애플리케이션 관점

* `Connection is not available`
* `HikariPool - Timeout`
* 요청 처리 시간 급증

***
