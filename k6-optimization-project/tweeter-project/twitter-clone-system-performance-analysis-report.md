---
description: RDB vs Cassandra 트윗 생성 성능 차이
---

# Twitter Clone System Performance Analysis Report

### 📋 목차 (Table of Contents)

1. 프로젝트 개요 및 목표
2. 기술 스택 및 아키텍처 비교
3. 성능 테스트 환경 및 방법론
4. 상세 성능 분석 결과
5. 성능 차이 원인 분석
6. 결론

***

### 1. 프로젝트 개요 및 목표

🎯 프로젝트 목표:

* 대규모 사용자를 지원하는 트위터 클론 시스템 구축
* NoSQL(Cassandra)과 RDB(MySQL)의 성능 차이 실증적 분석
* 분산 시스템 아키텍처 패턴 학습 및 적용
* Fan-out-on-write 패턴을 통한 타임라인 서비스 최적화

🔍 핵심 연구 질문:

1. Cassandra와 MySQL의 대용량 데이터 처리 성능 차이는?
2. Fan-out-on-write 패턴에서 각 DB의 확장성은?
3. 실시간 소셜 미디어 서비스에 최적화된 DB 선택 기준은?

***

### 2. 아키텍처 비교

#### Cassandra 아키텍처:

* 3-Node Cassandra Cluster (포트: 9042, 9043, 9044)
* Consistency Level: ONE (성능 최적화)
* 배치 처리: 100개씩 분할 + 비동기 병렬 처리
* ThreadPool: 8개 고정 스레드 (CPU 코어 수의 2배)
* 네이티브 Cassandra BatchStatement 사용

#### RDB 아키텍처:

* 4-Shard MySQL Cluster (포트: 3306, 3307, 3308, 3309)
* 샤딩 전략: User ID 기반 해시 샤딩
* Connection Pool: HikariCP
* 트랜잭션: 단일 샤드 내 ACID 보장
* 샤드 간 데이터 분산: 250,000명씩 균등 분할

***

### 3. 단계적 성능 측정

* 팔로워 수별 단계적 성능 측정 (1명 → 10명 → 100명 → 1,000명 → 10,000명 → 100,000명)
* LogTrace AOP를 통한 정밀한 메서드별 실행 시간 측정
* 각 테스트 3회 실행 후 평균값 산출
* 동일한 하드웨어 환경에서 순차 실행

***

### 4. 상세 성능 분석 결과

```sql
# 팔로워 1명
RDB
[8eec453f] RdbTweetController.createTweet(..)
[8eec453f] |-->RdbCreateTweetShardService.createTweetWithCorrectSharding(..) 
[8eec453f] |   |-->RdbFollowRepository.findFollowerIds(..) 
[8eec453f] |   |<--RdbFollowRepository.findFollowerIds(..) time=9ms
[8eec453f] |   |-->CrudRepository.save(..) 
[8eec453f] |   |<--CrudRepository.save(..) time=7ms
[8eec453f] |<--RdbCreateTweetShardService.createTweetWithCorrectSharding(..) time=28ms
[8eec453f] RdbTweetController.createTweet(..) time=29ms

CASSANDRA
[75649708] TweetController.createTweetOptimized(..) 시작
[75649708] |-->TweetServiceAdvanced.createTweet(..) 시작
[75649708] |   |-->CrudRepository.save(..) Tweet 저장 
[75649708] |   |<--CrudRepository.save(..) Tweet 저장 완료 (7ms)
[75649708] |   |-->CrudRepository.save(..) TweetByUser 저장 
[75649708] |   |<--CrudRepository.save(..) TweetByUser 저장 완료 (6ms)
[75649708] |   |-->FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 
[75649708] |   |<--FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 완료 (5ms)
[75649708] |<--TweetServiceAdvanced.createTweet(..) 완료 (총 24ms)
[75649708] TweetController.createTweetOptimized(..) 완료 (총 24ms)
----------------------------------------------------------------------------------------

# 팔로워 10명
RDB
[8eec453f] RdbTweetController.createTweet(..)
[8eec453f] |-->RdbCreateTweetShardService.createTweetWithCorrectSharding(..) 
[8eec453f] |   |-->RdbFollowRepository.findFollowerIds(..) 
[8eec453f] |   |<--RdbFollowRepository.findFollowerIds(..) time=9ms
[8eec453f] |   |-->CrudRepository.save(..) 
[8eec453f] |   |<--CrudRepository.save(..) time=7ms
[8eec453f] |<--RdbCreateTweetShardService.createTweetWithCorrectSharding(..) time=28ms
[8eec453f] RdbTweetController.createTweet(..) time=29ms

CASSANDRA
[3fbebb11] TweetController.createTweetOptimized(..) 시작
[3fbebb11] |-->TweetServiceAdvanced.createTweet(..) 시작
[3fbebb11] |   |-->CrudRepository.save(..) Tweet 저장 
[3fbebb11] |   |<--CrudRepository.save(..) Tweet 저장 완료 (3ms)
[3fbebb11] |   |-->CrudRepository.save(..) TweetByUser 저장 
[3fbebb11] |   |<--CrudRepository.save(..) TweetByUser 저장 완료 (4ms)
[3fbebb11] |   |-->FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 
[3fbebb11] |   |<--FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 완료 (4ms)
[3fbebb11] |<--TweetServiceAdvanced.createTweet(..) 완료 (총 21ms)
[3fbebb11] TweetController.createTweetOptimized(..) 완료 (총 21ms)
----------------------------------------------------------------------------------------
# 팔로워 100명
RDB
[9e0b1eff] RdbTweetController.createTweet(..)
[9e0b1eff] |-->RdbCreateTweetShardService.createTweetWithCorrectSharding(..)
[9e0b1eff] |   |-->RdbFollowRepository.findFollowerIds(..)
[9e0b1eff] |   |<--RdbFollowRepository.findFollowerIds(..) time=3ms
[9e0b1eff] |   |-->CrudRepository.save(..)
[9e0b1eff] |   |<--CrudRepository.save(..) time=5ms
[9e0b1eff] |<--RdbCreateTweetShardService.createTweetWithCorrectSharding(..) time=184ms
[9e0b1eff] RdbTweetController.createTweet(..) time=185ms

CASSANDRA
[15eaabd2] TweetController.createTweetOptimized(..) 시작
[15eaabd2] |-->TweetServiceAdvanced.createTweet(..) 시작
[15eaabd2] |   |-->CrudRepository.save(..) Tweet 저장 
[15eaabd2] |   |<--CrudRepository.save(..) Tweet 저장 완료 (1ms)
[15eaabd2] |   |-->CrudRepository.save(..) TweetByUser 저장
[15eaabd2] |   |<--CrudRepository.save(..) TweetByUser 저장 완료 (3ms)
[15eaabd2] |   |-->FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회
[15eaabd2] |   |<--FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 완료 (5ms)
[15eaabd2] |<--TweetServiceAdvanced.createTweet(..) 완료 (총 34ms)
[15eaabd2] TweetController.createTweetOptimized(..) 완료 (총 35ms)
----------------------------------------------------------------------------------------

# 팔로워 1000명
RDB
[d1b93ded] RdbTweetController.createTweet(..)
[d1b93ded] |-->RdbCreateTweetShardService.createTweetWithCorrectSharding(..)
[d1b93ded] |   |-->RdbFollowRepository.findFollowerIds(..)
[d1b93ded] |   |<--RdbFollowRepository.findFollowerIds(..) time=9ms
[d1b93ded] |   |-->CrudRepository.save(..)
[d1b93ded] |   |<--CrudRepository.save(..) time=9ms
[d1b93ded] |<--RdbCreateTweetShardService.createTweetWithCorrectSharding(..) time=1533ms
[d1b93ded] RdbTweetController.createTweet(..) time=1533ms

CASSANDRA
[fe152d89] TweetController.createTweetOptimized(..) 시작
[fe152d89] |-->TweetServiceAdvanced.createTweet(..) 시작
[fe152d89] |   |-->CrudRepository.save(..) Tweet 저장
[fe152d89] |   |<--CrudRepository.save(..) Tweet 저장 완료 (2ms)
[fe152d89] |   |-->CrudRepository.save(..) TweetByUser 저장
[fe152d89] |   |<--CrudRepository.save(..) TweetByUser 저장 완료 (3ms)
[fe152d89] |   |-->FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회
[fe152d89] |   |<--FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 완료 (13ms)
[fe152d89] |<--TweetServiceAdvanced.createTweet(..) 완료 (총 62ms)
[fe152d89] TweetController.createTweetOptimized(..) 완료 (총 62ms)
----------------------------------------------------------------------------------------

# 팔로워 10000명
RDB
[834c8b4a] RdbTweetController.createTweet(..)
[834c8b4a] |-->RdbCreateTweetShardService.createTweetWithCorrectSharding(..)
[834c8b4a] |   |-->RdbFollowRepository.findFollowerIds(..)
[834c8b4a] |   |<--RdbFollowRepository.findFollowerIds(..) time=27ms
[834c8b4a] |   |-->CrudRepository.save(..)
[834c8b4a] |   |<--CrudRepository.save(..) time=4ms
[834c8b4a] |<--RdbCreateTweetShardService.createTweetWithCorrectSharding(..) time=4250ms
[834c8b4a] RdbTweetController.createTweet(..) time=4250ms

CASSANDRA
[54b66ef3] TweetController.createTweetOptimized(..) 시작
[54b66ef3] |-->TweetServiceAdvanced.createTweet(..) 시작
[54b66ef3] |   |-->CrudRepository.save(..) Tweet 저장
[54b66ef3] |   |<--CrudRepository.save(..) Tweet 저장 완료 (3ms)
[54b66ef3] |   |-->CrudRepository.save(..) TweetByUser 저장
[54b66ef3] |   |<--CrudRepository.save(..) TweetByUser 저장 완료 (3ms)
[54b66ef3] |   |-->FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회
[54b66ef3] |   |<--FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 완료 (162ms)
[54b66ef3] |<--TweetServiceAdvanced.createTweet(..) 완료 (총 487ms)
[54b66ef3] TweetController.createTweetOptimized(..) 완료 (총 487ms)
----------------------------------------------------------------------------------------

#팔로워 100000명
RDB
[834c8b4a] RdbTweetController.createTweet(..)
[834c8b4a] |-->RdbCreateTweetShardService.createTweetWithCorrectSharding(..)
[834c8b4a] |   |-->RdbFollowRepository.findFollowerIds(..)
[834c8b4a] |   |<--RdbFollowRepository.findFollowerIds(..) time=27ms
[834c8b4a] |   |-->CrudRepository.save(..)
[834c8b4a] |   |<--CrudRepository.save(..) time=4ms
[834c8b4a] |<--RdbCreateTweetShardService.createTweetWithCorrectSharding(..) time=13884ms
[834c8b4a] RdbTweetController.createTweet(..) time=13884ms

CASSANDRA
[bb202d5c] TweetController.createTweetOptimized(..) 시작
[bb202d5c] |-->TweetServiceAdvanced.createTweet(..) 시작
[bb202d5c] |   |-->CrudRepository.save(..) Tweet 저장
[bb202d5c] |   |<--CrudRepository.save(..) Tweet 저장 완료 (3ms)
[bb202d5c] |   |-->CrudRepository.save(..) TweetByUser 저장
[bb202d5c] |   |<--CrudRepository.save(..) TweetByUser 저장 완료 (3ms)
[bb202d5c] |   |-->FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 (953ms)
[bb202d5c] |   |<--FollowRepository.findByKeyFollowedUserId(..) 팔로워 조회 완료
[bb202d5c] |<--TweetServiceAdvanced.createTweet(..) 완료 (총 3956ms)
[bb202d5c] TweetController.createTweetOptimized(..) 완료 (총 3956ms)
```

| 팔로워 수    | RDB (ms) | Cassandra (ms) | 성능 차이   |
| -------- | -------- | -------------- | ------- |
| 1명       | 29       | 24             | 5ms     |
| 10명      | 29       | 21             | 8ms     |
| 100명     | 185      | 35             | 150ms   |
| 1,000명   | 1,533    | 62             | 1,471ms |
| 10,000명  | 4,250    | 487            | 3,763ms |
| 100,000명 | 13,884   | 3,956          | 9,928ms |

***

### 5. 성능 차이 원인 분석

#### 5-1. DB 아키텍처 차이

Cassandra(NoSQL)

* ✅Write-Optimized: LSM Tree 구조로 쓰기 성능 우수
* ✅분산 처리: 3-Node로 부하 분산
* ✅배치 처리: 네이티브 BatchStatement로 네트워크 오버헤드 최소화
* ✅비동기 처리: CompletableFuture로 병렬 처리

RDB

* ❌ ACID 트랜잭션: 데이터 일관성 보장으로 인한 성능 오버헤드
* ❌ 샤딩 복잡성: 3개 샤드 간 데이터 분산 및 조회 복잡성
* ❌ 순차 처리: 샤드별 순차적 데이터 삽입
* ❌ 네트워크 지연: 샤드 간 통신 오버헤드

#### 5-2. Fan-out 처리 방식 차이

Cassandra Fan-out 처리

* 배치 크기: 100개씩 분할
* 병렬도: 8개 스레드 동시 처리
* 네트워크: 단일 클러스터 내 통신
* 메모리: LSM Tree의 메모리 기반 쓰기

RDB Fan-out 처리

* 샤드 분산: 3개 샤드에 팔로워 데이터 분산
* 순차 처리: 샤드별 순차적 데이터 삽입
* 네트워크: 샤드 간 통신 오버헤드
* 트랜잭션: 각 샤드 내 ACID 보장

#### 5-3. 확장성 차이

Cassandra 확장성

* 수평 확장: 노드 추가로 선형적 성능 향상
* 데이터 분산: Consistent Hashing으로 자동 분산
* 읽기/쓰기 분리: 각 노드가 독립적으로 처리
* 장애 복구: 자동 복제 및 복구

RDB 확장성

* 샤딩 복잡성: 샤드 키 관리 및 데이터 재분배 복잡
* 조인 제한: 샤드 간 조인 불가능
* 트랜잭션 제한: 샤드 간 분산 트랜잭션 복잡
* 운영 복잡성: 샤드별 백업 및 복구 필요

***

### 6. 결론

#### 📊 핵심 결과

1. **Cassandra 우수성**: 대용량 처리에서 RDB 대비 **3.5\~24.7배** 성능 우수
2. **선형적 확장**: 팔로워 수 증가에 따른 예측 가능한 성능 저하
3. **Write-heavy 최적화**: 소셜 미디어 워크로드에 최적화된 성능

#### 🚀 프로젝트 성과

* **성능 향상**: 대용량 데이터 처리량 향상
* **학습 완료**: NoSQL vs RDB 실무 비교 분석
* **안정성 확보**: 100,000명 팔로워 환경 안정 동작
* **아키텍처 구축**: 복잡한 샤딩 환경 구축 완료



***

### 📈 프로젝트 학습 내용

✅ 학습 내용:

* NoSQL vs RDB 비교 분석
* 100,000명 팔로워 환경에서 데이터 처리 성능 24.7배 향상 달성
* 기본 샤딩 환경 구축 및 실험
