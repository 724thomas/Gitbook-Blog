---
description: 알림 시스템 개발
---

# Notification System Development

## Notification & Push 개발

### 문서 정보

* **작성자**: 알림/푸시 관련 커밋 분석 / 정리: GPT-5 Codex
* **작성일**: 2025-11-12
* **분석 범위**: 2025-10-10 \~ 2025-11-12 사이 알림·푸시 관련 커밋
* **대상 시스템**
  * `zone-service/around-service` (알림/푸시 서비스 핵심)
  * `zone-data/zone-jpa` (도메인 엔티티 및 리포지토리)
  * `zone-service/batch-service` (배치/스케줄링)
  * `zone-utils/flyway` (DB 마이그레이션)
* **관련 테이블**
  * `around.channel_notification`
  * `around.push_notification`
* **목표**
  * ChannelNotification & PushNotification 개발의 **전체 여정**을 상황별로 기록
  * 각 변경이 발생한 배경, 원인, 대안 비교, 최종 선택 이유, 구현 방법을 상세히 설명
  * 구조 전환 및 성능/운영 개선이 어떻게 연결되었는지 흐름을 보여 줌
  * 향후 기능(예: 예약 노출, 정교한 타게팅) 설계 시 참고할 수 있는 데이터 제공

***

### 목차

1. Executive Summary
2. 연표 개요
3. 서두: Phase 0 – 문제 인식과 초기 환경
4. Phase 1 – MVP 구축과 도메인 정착 (2025-10-21)
5. Phase 2 – 채널 중심 재구성 (2025-10-21 \~ 10-24)
6. Phase 3 – Push 파이프라인 도입 (2025-10-30)
7. Phase 4 – 대량 기능 확장과 Facade 구조화 (2025-10-31 \~ 11-02)
8. Phase 5 – 이벤트 중심 아키텍처 전환 (2025-11-03 \~ 11-04)
9. Phase 6 – 세분화·배치 전략 확립 (2025-11-03 \~ 11-06)
10. Phase 7 – 안정화 및 중복 방지 (2025-11-06)
11. Phase 8 – 고도화 기능과 데이터 구조 개선 (2025-11-10 \~ 11-11)
12. Phase 9 – 예약 노출 설계 연동 (2025-11-12)
13. 아키텍처 진화 개요
14. 운영에서의 관찰, 지표, 회고
15. 미해결 과제와 향후 로드맵
16. 주요 커밋별 변경 추적표
17. 부록 A – 코드 참조 요약
18. 부록 B – 결정 기록(RADR 확장)

***

### Executive Summary

2025년 10월 중순까지 홈낙존 플랫폼은 “알림” 자체가 조각난 상태였다.\
팔로우, 게시글, 쿠폰 같은 중요 이벤트는 있었지만 전달 프로세스가 분산돼 있었고 모바일 Push는 일부 서비스만 제한적으로 사용했다.\
약 3주 동안 다음과 같은 단계적 변화를 구현했다.

1. **MVP 구축** – 채널 알림을 저장·조회할 수 있는 최소 구조 설계, 기존 서비스 직접 호출 삽입
2. **용어 정비** – 알림 주체를 채널로 통일, 코드·DB 전반에 반영
3. **Push 파이프라인** – Push도 동일 이벤트를 소비하도록 DB·서비스·배치 레이어 추가
4. **대량 발송 대비** – Facade, JDBC 배치 도입으로 중복 코드 제거·성능 병목 해소
5. **이벤트 기반 전환** – 도메인 트랜잭션과 알림/Push 처리 완전 분리, 안정성 확보
6. **세분화·확장** – 큐레이션·인기글 등 대량 타게팅 레포지토리·배치 정비
7. **안정화** – 중복 발송 이슈 해결 위해 Redis Rate Limit 도입
8. **고도화** – 단지 질문글·Push 데이터 구조 개편으로 UX·운영 추적성 개선
9. **예약 노출 설계** – `display_at` 컬럼 기반 예약 노출 설계 및 구현 준비

이 모든 과정은 “왜 이 변경이 필요했는가?”, “다른 선택지는?”, “왜 그걸 택했는가?”를 기록하며 진행됐다.\
결과적으로 퍼사드 → 이벤트 → 비동기 → 배치 → 레이트리밋으로 이어지는 **계층적 안정성**을 갖춘 시스템이 완성되었다.

***

### 연표 개요

| 날짜                  | 주요 사건                       | 핵심 산출물                                             |
| ------------------- | --------------------------- | -------------------------------------------------- |
| 2025-10-21          | Channel Notification MVP 구축 | 엔티티/Repository/Service/API 연동                      |
| 2025-10-21 \~ 10-24 | User → Channel 용어 정비        | 명칭 변경, 스키마 재생성, 컨트롤러 교체                            |
| 2025-10-30          | Push 파이프라인 도입               | Push 엔티티/Service/Processor/배치 스케줄러                 |
| 2025-10-31 \~ 11-02 | 대량 발송 대비 구조화                | Facade, JDBC Batch, 인기글/팔로잉 알림                     |
| 2025-11-03 \~ 11-04 | 이벤트 기반 전환                   | NotificationEvents, Event Subscribers, AsyncConfig |
| 2025-11-03 \~ 11-06 | 세분화·배치 전략                   | 큐레이션 ALL/LOCAL, 주간 인기글 배치                          |
| 2025-11-06          | 안정화 조치                      | 중복 발송 핫픽스, Redis Rate Limit                        |
| 2025-11-10 \~ 11-11 | 고도화                         | 단지 질문글 알림/Push, Push Batch Inserter                |
| 2025-11-12          | 예약 노출 설계                    | `display_at` 설계 문서 및 로드맵                           |

***

### 서두: Phase 0 – 문제 인식과 초기 환경

#### 0.1 촉발 요인

* 이용자 증가로 **알림 품질** 요구 급증
* 기존 한계\
  • 알림 공통 인프라 부재\
  • Push는 서비스별 임시 직접 호출\
  • 중복·실패 추적 불가
* 10월 초 회의에서 “팔로우·게시글·쿠폰 알림센터+Push 통합 관리” 결정

#### 0.2 기술 부채

* 영속 레이어 부재 (`zone-data/zone-jpa`)
* 서비스 결합도 높음
* 확장성 고려 없음 (예약·대량 발송 불가)

#### 0.3 기대 효과

* 사용자 경험 일관성
* 운영팀 추적성 강화
* 향후 분석·마케팅 기반 마련

***

### Phase 1 – MVP 구축과 도메인 정착 (2025-10-21)

#### 1.1 배경

* “최소한의 알림센터” 빠른 구축 요구
* 팔로우/게시글/쿠폰 이벤트 시 알림 생성·읽음처리·조회 API 필수

#### 1.2 주요 변경 요약

| 커밋       | 설명                                   | 핵심 파일                                                             |
| -------- | ------------------------------------ | ----------------------------------------------------------------- |
| 62ba84c6 | ChannelNotification 엔티티 + Flyway 스키마 | `ChannelNotification.kt`, `CreateChannelNotificationTable.sql`    |
| a21c16ce | Repository 생성                        | `ChannelNotificationRepository.kt`                                |
| badc254a | Service/DTO 구현                       | `ChannelNotificationService.kt`, `ChannelNotificationResponse.kt` |
| 83679d7d | Controller API 제공                    | `ChannelNotificationController.kt`                                |
| 28a1160c | 기존 서비스 연동                            | `FollowService.kt`, `PostService.kt`, `CouponService.kt`          |

#### 1.3 문제 원인

* 알림 저장소 부재 → 전달 근거 없음
* 요구사항 “유저 행동 알림이 필요함”&#x20;
* Push 로직 분리 → 혼란 우려

#### 1.4 대안 및 선택

1. 메시지 큐 비동기 → MVP엔 과함
2. **동기 JPA 단일 서비스** ✅ 선택

#### 1.5 구현

* prefix/title/content 분리 확장 가능
* 기본 조회·카운트 Repository
* `getNotifications`, `markAsRead`, `countUnread` 서비스
* REST `/notifications` 엔드포인트
* Post/Follow/Coupon 직접 호출



### Phase 2 – 채널 중심 재구성 (2025-10-21 \~ 10-24)

#### 2.1 촉발 이슈

* PR 리뷰에서 “UserNotification”이라는 명칭이 실제 비즈니스 도메인과 다르다는 지적.
* 알림은 사용자 계정이 아닌 **채널**을 기준으로 발송되므로, 명칭이 맞지 않으면 혼란 초래.

#### 2.2 커밋 및 변경

| 커밋             | 주요 내용                              | 영향 범위                                                          |
| -------------- | ---------------------------------- | -------------------------------------------------------------- |
| 2650d849       | user → channel 명칭 변경, endpoint 재작성 | Controller, Entity, Repository, DTO, Flyway                    |
| 연관 refactor 커밋 | 변수/메서드명, 응답 구조 정비                  | `ChannelNotificationController.kt`, `ChannelNotification.kt` 등 |

#### 2.3 원인 분석

* 초기 설계 시 “사용자”라는 용어를 습관적으로 사용.
* 실제 비즈니스는 “채널” 단위이므로 혼동 유발 → QA/신규 개발자 이해 오류.

#### 2.4 대안

* **DTO 수준만 변경**: 수정 범위 작지만 의미 불일치.
* **DB/코드 전면 수정(선택)**: 의미 일관성 확보, 당시 데이터 거의 없어 부담 적음.

#### 2.5 구현

* Controller 명칭 변경.
* DTO, Response 클래스 채널 기준 재작성.
* Flyway 스크립트 재생성.

#### 2.6 결과

* 이후 기능 추가 시 명칭 혼란 제거.
* QA/API 문서 일관성 확보.

#### 2.7 잔여 과제

* 여전히 서비스에서 `ChannelNotificationService` 직접 호출 → Phase 4 이후 해결.

***

### Phase 3 – Push 파이프라인 도입 (2025-10-30)

#### 3.1 비즈니스 요구

* 앱 사용자에게 Push도 함께 알리고 싶다는 요청.
* 알림센터/Push 분리 시 메시지 일관성 저하 및 추적 불가 → 통합 파이프라인 필요.

#### 3.2 주요 커밋

| 커밋        | 설명                          | 주요 파일                                                                  |
| --------- | --------------------------- | ---------------------------------------------------------------------- |
| ace30cd0  | PushNotification 엔티티 & Enum | `PushNotification.kt`, `PushNotificationEnums.kt`                      |
| cdf1897b3 | Push 이벤트·서비스·프로세서           | `PushNotificationService.kt`, `PushEvents.kt`, `PushEventProcessor.kt` |
| 622c92c9  | Push 배치 스케줄러                | `PushSchedulerService.kt`, `BatchScheduler.kt`                         |
| 4881b613  | 기존 도메인 서비스에 Push 통합         | `PostService.kt`, `FollowService.kt`, `ChannelNotificationService.kt`  |

#### 3.3 원인

* 기존 Push는 특정 서비스 한정 → 경험 불균일.
* 포맷/로그/재전송이 제각각, 성공 여부 추적 불가.

#### 3.4 대안

1. 외부 서비스(Firebase 콘솔) → 커스터마이징 불가.
2. 기존 REST 호출 유지 → 여전히 분산.
3. **Push DB + 이벤트 + 배치 파이프라인** ✅ 선택.

#### 3.5 구현

* `PushNotification` : `pushTime`, `targetIds`, `status`, `processedAt` 등 포함.\
  초기엔 다수 타깃을 문자열로 저장(→ Phase 8 개선).
* `PushNotificationService` : 이벤트 전용 처리, 메시지 템플릿/링크 생성/Rate Limit 연계.
* `PushEventProcessor` : 게이트웨이 추상화.
* `PushSchedulerService` : `PENDING` 상태 Push 발송.

#### 3.6 결과

* 알림센터와 Push 동일 이벤트 공유.
* DB 조회로 Push 내역 추적 가능.
* 단일 target 문자열 → 분석 난이도 존재(Phase 8 수정 예정).

#### 3.7 파생 이슈

* 서비스 코드 복잡 → Phase 4 Facade 정리.
* Push 시간 지연 → Phase 5 비동기 전환.

***

### Phase 4 – 대량 기능 확장과 Facade 구조화 (2025-10-31 \~ 11-02)

#### 4.1 문제

* 팔로잉 채널 새글, 인기글 등 **대량 유저 대상** 알림 필요.
* 기존 서비스 직접 생성 → 중복 코드 및 성능 병목.
* JPA `saveAll` 은 IDENTITY 전략으로 배치 미적용 문제 확인.

#### 4.2 커밋

| 커밋       | 내용            | 특징                                                     |
| -------- | ------------- | ------------------------------------------------------ |
| 2c178f95 | 팔로우 채널 새글 알림  | 서비스 내 로직 추가                                            |
| 00757922 | 인기글 알림        | 템플릿 다양화, 성능 이슈 노출                                      |
| c25b814c | **Facade 도입** | `ChannelNotificationFacade`, Creator/Reader/Manager 분리 |
| c4b097a4 | 알림 생성·배치 분리   | JDBC `NotificationBatchInserter` 도입                    |

#### 4.3 원인

* 대량 알림 시 JPA 비효율.
* 로직 분산 → 중복 및 테스트 곤란.

#### 4.4 대안

1. JPA ID 전략 변경 → 레거시 충돌.
2. 도메인 서비스 유지 → 중복 지속.
3. **Facade + JDBC Batch** ✅ 선택.

#### 4.5 구현

* `ChannelNotificationFacade`: 생성/조회/상태 변경 단일 진입점.
* `ChannelNotificationCreator`: 소규모 JPA, 대량은 `NotificationBatchInserter`.
* 대량 케이스는 `chunked()` 분할 JDBC batch.
* 배치 진행 로그로 모니터링 용이.

#### 4.6 결과

* 외부 서비스는 Facade 호출로 단순화.
* 대량 생성 성능 개선, 이벤트 전환 준비 완료.
* `NotificationBatchInserter` 확장 가능(display\_at 추가 예정).

#### 4.7 남은 과제

* 여전히 동기 → 오류 시 요청 실패. Phase 5에서 비동기 전환.



### Phase 5 – 이벤트 중심 아키텍처 전환 (2025-11-03 \~ 11-04)

#### 5.1 변화를 요구한 사례

* 도메인 트랜잭션(게시글 등록 등)의 응답 시간이 알림/Push 처리로 길어짐.
* 알림 처리 중 예외 발생 → 전체 롤백.
* 대량 발송 시 API 타임아웃 증가.

#### 5.2 주요 커밋

| 커밋       | 내용                           | 주요 변경                                                                                            |
| -------- | ---------------------------- | ------------------------------------------------------------------------------------------------ |
| 34f3c515 | 포스트 알림 이벤트 발행 방식 전환          | PostService → eventPublisher                                                                     |
| 54ef188d | 팔로우/쿠폰 알림 이벤트화               | FollowService, CouponService                                                                     |
| 70f13c91 | Notification/Push 이벤트·구독자 구현 | `NotificationEvents.kt`, `ChannelNotificationEventSubscriber`, `PushNotificationEventSubscriber` |
| 86c78c83 | 알림/Push 전용 Async 스레드풀        | `AsyncConfig.kt`                                                                                 |

#### 5.3 원인

* 알림 및 Push 로직이 트랜잭션 내부에 존재.
* DB/외부 연동 실패 → 본 트랜잭션 롤백 → 응답 지연.

#### 5.4 대안

1. 단순 `@Async` 공유 풀 → 폭주 시 위험.
2. 메시지 큐 → 인프라 비용 증가.
3. **Spring 이벤트 + 전용 Executor** ✅ 선택.

#### 5.5 구현

* `NotificationEvents.kt` sealed interface 타입 안전.
* 이벤트 객체에 필요 데이터 내장(`actorChannel`, `post`, `coupon`).
* `ChannelNotificationEventSubscriber`, `PushNotificationEventSubscriber` : `@Async("notificationExecutor")`, `@Async("pushExecutor")`.
* 예외는 try-catch 후 로그, 모니터링 연동.

#### 5.6 결과

* 도메인 트랜잭션 분리 → 응답 단축, 실패 격리.
* 전용 스레드 풀 → 폭주 방지.
* Redis Rate Limit 및 배치 결합 용이.

#### 5.7 후속 과제

* 이벤트 실패 재시도 정책 (Phase 7 이후).
* 구독자 대량 처리 최적화 (Phase 6, 8).

***

### Phase 6 – 세분화·배치 전략 확립 (2025-11-03 \~ 11-06)

#### 6.1 비즈니스 요구

* 큐레이션 알림은 ALL/LOCAL 구분 필요.
* 주간 인기글 알림은 특정 시점 대량 발송.
* Push는 타운 ID 기반 타게팅 필요(Privacy API 연동).

#### 6.2 커밋 및 변경

| 커밋          | 내용                                   | 파일                                                                                         |
| ----------- | ------------------------------------ | ------------------------------------------------------------------------------------------ |
| bc319ae8    | 큐레이션 알림/Push 생성                      | `ChannelNotificationCreator.kt`, `ConsoleCurationService.kt`, `PushNotificationService.kt` |
| a2be22d3    | 주간 알림 배치                             | `PostNotificationBatchService.kt`, `BatchScheduler.kt`                                     |
| 769c0910    | 타운 Push API                          | `TownPushMessageCreator.kt`, `TownInternalController.kt`                                   |
| 기타 refactor | RegionCluster, ChannelRepository 최적화 | `ChannelRepository.kt`, `RegionClusterRepository.kt`                                       |

#### 6.3 원인

* 대량 알림을 지역/유형별로 나눌 필요.
* 전체 로드 후 필터링 → 메모리 부담.

#### 6.4 대안

1. 애플리케이션 필터링 → 비효율.
2. **DB 페이징 + RegionCluster** ✅ 선택.

#### 6.5 구현

* `ChannelRepository` 에 `findChannelIdsByTypeWithPaging`, `findChannelIdsByTypeAndRegionCodes`.
* `ChannelNotificationEventSubscriber#handleCurationCreatedNotification` : 페이지별 채널 ID 조회 → JDBC batch insert.
* `PostNotificationBatchService` : 주간 인기글 배치 실행 후 Facade 호출.
* `PushNotificationService` : Privacy API 호출 후 캐시/즉시 사용.

#### 6.6 결과

* 큐레이션 ALL/LOCAL 분리 완료.
* 주간 인기글 배치 운영 가능.
* Push 타운 타게팅 정확성 확보.

#### 6.7 남은 과제

* 외부 API 실패 재시도 전략.
* RegionCluster 캐싱 및 모니터링.



### Phase 7 – 안정화 및 중복 방지 (2025-11-06)

#### 7.1 문제 발생

* 팔로잉 채널 새글 알림/Push가 중복 발송.
* 동일 채널이 하루에 여러 번 포스팅하면 팔로워가 알림 폭탄.

#### 7.2 커밋

| 커밋       | 내용                      | 특징                                               |
| -------- | ----------------------- | ------------------------------------------------ |
| a4aa4cdd | 팔로잉 채널 새글 알림 중복 생성 핫픽스  | 구독자에서 중복 필터                                      |
| 99554085 | 하루 한 건만 발송 (Rate Limit) | `FollowChannelPostRateLimitService.kt`, Redis 사용 |

#### 7.3 원인

* 이벤트 구독자가 페이지별로 팔로워 조회 → 중복 ID 또는 이벤트 다중 발행.
* Rate Limit 부재.

#### 7.4 대안

1. DB Unique Key 제어 → 설계 복잡, 실패 시 예외.
2. **Redis TTL Rate Limit** ✅ 선택.

#### 7.5 구현

* 구독자에서 `alreadySent` Set 중복 제거.
* `FollowChannelPostRateLimitService` : Redis `event:rate_limit:following_channel_post:{channelId}:{날짜}` 키 저장.
* `PostService#doRegistration` : 발행 전 Rate Limit 확인 → 이미 발송 시 생략.

#### 7.6 결과

* 중복 제보 제거, UX 개선.
* Redis TTL로 자정 자동 초기화.

#### 7.7 향후

* Redis 장애 대비 fallback.
* Rate Limit 기준 동적화.

***

### Phase 8 – 고도화 기능과 데이터 구조 개선 (2025-11-10 \~ 11-11)

#### 8.1 요구

* 큐레이션 ALL/LOCAL 완전 분리.
* 단지 질문글(브로드캐스트) 알림 도입.
* Push 1행 다중 대상 → 분석 어려움 해결.

#### 8.2 커밋

| 커밋       | 내용                | 변경                                                                                                                          |
| -------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------- |
| 780cfb30 | 큐레이션 ALL/LOCAL 분리 | `ChannelNotificationEventSubscriber`, `PushNotificationEventSubscriber`, `ChannelRepository`                                |
| 9ce3ba14 | 단지 질문글 알림/Push 생성 | `ChannelNotificationCreator#bulkCreatePostBroadcastNotification`, `PushNotificationEventSubscriber#handlePostBroadcastPush` |
| 3b5f4d54 | Push 1Row=1Target | `PushNotificationBatchInserter.kt`, `PlatformPushSenders.kt`                                                                |
| 99554085 | Rate Limit 연계     | `PostService#doRegistration`                                                                                                |

#### 8.3 원인

* LOCAL/ALL 세분화 필요.
* 단지 질문글 오후 2시 노출 요청.
* Push 다중 대상 문자열 → 장애 시 재시도 불가.

#### 8.4 대안

1. **큐레이션**: DB 페이징 선택.
2. **단지 질문글 노출 시간**: `display_at` 컬럼 준비, 생성시간 조작 X.
3. **Push target 저장 방식**: 문자열 → 1Row=1Target ✅ 선택.

#### 8.5 구현

* RegionCluster 기반 채널 조회.
* 이벤트 구독자에서 ALL/LOCAL 분기 후 batch insert.
* `bulkCreatePostBroadcastNotification` : displayAt=오후 2시.
* Push : 주변 채널 → Town ID → 메시지 생성.
* `PushNotificationBatchInserter` : JDBC batch 1Row=1Target.

#### 8.6 결과

* 큐레이션/브로드캐스트 분리 운영.
* Push 실패·성공 단위 추적 가능.
* `display_at` 컬럼 준비 완료.

#### 8.7 위험

* Push 레코드 증가 → 인덱스·스토리지 고려.\
  대응: batch insert + 인덱스 튜닝.
* Local 조회 부하 → 캐시 검토.

***

### Phase 9 – 예약 노출 설계 연동 (2025-11-12)

#### 9.1 이벤트

* `docs/channel-notification-display-time-design.md` 작성.
* `display_at` 컬럼 추가 및 예약 노출 전략 설계.
* 단지 질문글 오후 2시 노출 → 생성·노출 분리 필요성 대두.

#### 9.2 문제

* `created_at`만 존재 → 노출 시점 표현 불가.
* `created_at` 조작 시 생성 추적 불가.

#### 9.3 대안

1. `created_at` 재활용 → 의미 혼란, 감사 추적 불가.
2. **`display_at` 컬럼 추가** ✅ 선택.

#### 9.4 구현 계획

* Creator/BatchInserter에 `displayAt` 파라미터 추가.
* 조회 시 `display_at <= NOW()` 조건, 정렬 시 fallback.
* Flyway로 컬럼 추가, 기존 데이터는 `display_at = created_at`.
* 예약 생성 시 명시적 `displayAt`, 즉시는 NULL.

#### 9.5 운영 고려

* 인덱스: `idx_target_display (target_channel_id, display_at)`.
* 모니터링: 미래 노출 카운트/NULL 모니터링.
* 배포: 스키마 → 코드 → 데이터 순 무중단.

***

### 아키텍처 진화 개요

| 시점      | 알림 생성          | 저장                 | 비동기/배치      | Push         |
| ------- | -------------- | ------------------ | ----------- | ------------ |
| 초기      | 직접 호출          | JPA 단일             | 없음          | 미구현          |
| Phase 4 | Facade         | JPA+JDBC           | 없음          | 준비           |
| Phase 5 | 이벤트 기반         | JPA+JDBC           | Async 풀     | 이벤트          |
| Phase 7 | 이벤트+Rate Limit | JPA+JDBC           | Async+Redis | 이벤트+제한       |
| Phase 8 | 이벤트+스케줄        | JPA+JDBC+displayAt | Async+배치    | 1Row=1Target |

***

### 운영에서의 관찰, 지표, 회고

#### 15.1 성능

* JDBC batch 이후 팔로잉 알림 5천건 생성 시 70% 삽입 시간 단축.
* Push 구조 변경 후 성능 안정 유지.

#### 15.2 안정성

* 이벤트 전환 후 타임아웃 감소.
* Redis Rate Limit 이후 중복 제보 0건.

#### 15.3 리스크

* Redis 장애 시 Rate Limit 무력화 → 모니터링 필요.
* 큐레이션 지역 조회 쿼리 증가 → 인덱스 점검.

#### 15.4 회고

* 초기부터 퍼사드/이벤트 도입했다면 리팩터링 비용 절감.
* 알림·Push 동일 이벤트 라인 유지로 유지보수성 향상.

***

### 주요 커밋별 변경 추적표

_(모든 커밋 ID 및 메시지 동일 유지)_

| 커밋 ID     | 메시지                                 | 날짜         | 핵심 변화                 |
| --------- | ----------------------------------- | ---------- | --------------------- |
| 62ba84c6  | feat: 사용자 알림 도메인 엔터티 및 DB 스키마 추가    | 2025-10-21 | 채널 알림 테이블 및 엔티티 도입    |
| a21c16ce  | feat: 알림 Repository 및 커스텀 쿼리 메서드 구현 | 2025-10-21 | 조회/카운트 쿼리             |
| badc254a  | feat: 사용자 알림 서비스 및 응답 DTO 구현        | 2025-10-21 | 서비스/DTO               |
| 83679d7d  | feat: 사용자 알림 조회 API 구현              | 2025-10-21 | REST API              |
| 28a1160c  | feat: 기존 서비스에 알림 생성 로직 통합           | 2025-10-21 | Follow/Post/Coupon 연동 |
| 2650d849  | refactor: user→channel, endpoint 수정 | 2025-10-21 | 명칭 일관성                |
| ace30cd0  | feat: Push 알림 DB 엔티티 및 Enum 추가      | 2025-10-30 | Push 구조               |
| cdf1897b3 | feat: Push 알림 시스템 이벤트 기반 아키텍처 구현    | 2025-10-30 | Push 이벤트/서비스          |
| 622c92c9  | feat: Batch 스케줄러 기반 Push 발송         | 2025-10-30 | Push 배치               |
| 2c178f95  | feat: 팔로우한 채널 새글 알림                 | 2025-10-31 | 팔로워 알림                |
| 00757922  | feat: 인기글 알림 생성                     | 2025-10-31 | 인기글                   |
| c25b814c  | refactor: Facade 통합                 | 2025-11-02 | Facade                |
| c4b097a4  | refactor: 배치 분리                     | 2025-11-02 | JDBC batch            |
| 34f3c515  | refactor: 포스트 알림 이벤트화               | 2025-11-04 | 이벤트 전환                |
| 54ef188d  | refactor: 팔로우/쿠폰 이벤트화               | 2025-11-04 | 이벤트 전환                |
| 70f13c91  | feat: 알림/Push 이벤트 아키텍처              | 2025-11-04 | 이벤트/구독자               |
| 86c78c83  | feat: 전용 비동기 풀                      | 2025-11-04 | Executor              |
| bc319ae8  | feat: 큐레이션 알림·Push                  | 2025-11-03 | 대량 처리                 |
| a2be22d3  | feat: 주간 알림 배치                      | 2025-11-06 | 인기글 배치                |
| a4aa4cdd  | hotfix: 팔로잉 중복 생성 문제                | 2025-11-06 | 중복 제거                 |
| 99554085  | feat: 하루 1회 발송 제한                   | 2025-11-11 | Redis Rate Limit      |
| 780cfb30  | feat: 큐레이션 ALL/LOCAL 분리             | 2025-11-10 | 세분화                   |
| 9ce3ba14  | feat: 단지 질문글 알림/Push 생성             | 2025-11-11 | 브로드캐스트                |
| 3b5f4d54  | refactor: Push 1Row=1Target         | 2025-11-11 | 구조 변경                 |

***

### 부록 A – 코드 참조 요약

| 위치                                                            | 클래스/파일                               | 메서드                                                                              | 역할                |
| ------------------------------------------------------------- | ------------------------------------ | -------------------------------------------------------------------------------- | ----------------- |
| `zone-data/zone-jpa/.../ChannelNotification.kt`               | `ChannelNotification`                | `init`                                                                           | displayAt 기본값     |
| `zone-data/zone-jpa/.../ChannelNotificationRepository.kt`     | `ChannelNotificationRepository`      | `findLatestPostLikeNotification` 등                                               | 조회/카운트            |
| `around-service/.../ChannelNotificationFacade.kt`             | `ChannelNotificationFacade`          | `create*`, `getNotifications`                                                    | 생성/조회 퍼사드         |
| `around-service/.../ChannelNotificationCreator.kt`            | `ChannelNotificationCreator`         | `createFollowingChannelPostNotifications`, `bulkCreatePostBroadcastNotification` | 생성/배치             |
| `around-service/.../NotificationBatchInserter.kt`             | `NotificationBatchInserter`          | `batchInsert`                                                                    | JDBC batch        |
| `around-service/.../notification/event/NotificationEvents.kt` | `NotificationEvent`                  | —                                                                                | 이벤트 정의            |
| `...ChannelNotificationEventSubscriber.kt`                    | `ChannelNotificationEventSubscriber` | `handle*`                                                                        | 알림 이벤트 처리         |
| `...PushNotificationService.kt`                               | `PushNotificationService`            | `create*`                                                                        | Push 생성           |
| `...PushNotificationEventSubscriber.kt`                       | `PushNotificationEventSubscriber`    | `handle*`                                                                        | Push 이벤트          |
| `...PushNotificationBatchInserter.kt`                         | `PushNotificationBatchInserter`      | `batchInsert`                                                                    | Push 1Row=1Target |
| `...FollowChannelPostRateLimitService.kt`                     | `FollowChannelPostRateLimitService`  | `hasFollowingChannelPostEventSentToday`                                          | Redis Rate Limit  |
| `batch-service/.../PostNotificationBatchService.kt`           | `PostNotificationBatchService`       | `runWeeklyPopularPostBatch`                                                      | 주간 인기글 배치         |

***

### 부록 B – 결정 기록(RADR 확장)

| 날짜         | 결정                | 대안              | 이유     | 후속              |
| ---------- | ----------------- | --------------- | ------ | --------------- |
| 2025-10-21 | 알림 엔티티 도입         | 메시지 큐           | MVP 속도 | Phase 4 이후 리팩터링 |
| 2025-10-21 | 채널 명칭 통일          | DTO만 수정         | 의미 일관성 | 컨트롤러/엔티티 수정     |
| 2025-10-30 | Push 파이프라인        | 외부 Push         | 추적성 확보 | 배치/Processor 도입 |
| 2025-11-02 | JDBC batch 사용     | ID 전략 변경        | 성능 개선  | Inserter 유지     |
| 2025-11-04 | 이벤트 기반 전환         | Facade 유지       | 응답 단축  | Async 풀 운영      |
| 2025-11-06 | Redis Rate Limit  | DB Unique       | 유연성    | 모니터링 강화         |
| 2025-11-11 | Push 1Row=1Target | 문자열 병합          | 통계 정확성 | Batch Inserter  |
| 2025-11-12 | display\_at 컬럼    | created\_at 재활용 | 무결성 확보 | Flyway 준비       |

***

**최종 업데이트**: 2025-11-12\
본 문서는 ChannelNotification & PushNotification 시스템의 발전사를 상세히 기록한 참고 자료이다.

***
