---
description: 알림센터 알림 노출 시간 디자인
---

# channel notification display time design

**📘 채널 알림 예약 노출 기능 설계 문서**

***

#### 문서 정보

* **작성일**: 2025-11-12
* **대상 모듈**: zone-service/around-service, zone-data/zone-jpa
* **관련 테이블**: `around.channel_notification`
* **주요 변경**: 알림 예약 노출 기능 추가 (`display_at` 컬럼)

***

#### 목차

1. 배경 및 문제 정의
2. 요구사항 분석
3. 해결 방안 검토
4. 최종 설계 결정
5. 구현 상세
6. 마이그레이션 전략
7. Init 블록 설계 결정
8. 주의사항 및 제약사항

***

### 1. 배경 및 문제 정의

#### 1.1 비즈니스 요구사항

채널 알림 시스템에 새로운 요구사항이 추가되었습니다:

* **현재**: 알림 생성 즉시 사용자에게 노출
* **요구사항**: 특정 시점(예: 오후 2시)에 노출되도록 예약 기능 필요

#### 1.2 구체적 사례

단지 질문글 알림(`HOUSE_QUESTION`)의 경우, 알림이 생성되는 시점은 오전이지만 사용자에게는 오후 2시에 노출되어야 합니다.\
이는 사용자 경험 최적화와 알림 집중도를 높이기 위한 전략입니다.

#### 1.3 현재 시스템 한계

현재 `channel_notification` 테이블 구조에서는 알림의 노출 시점을 제어할 수 있는 별도 필드가 존재하지 않습니다.\
`created_at` 필드만으로는 생성 시간과 노출 시간을 구분할 수 없습니다.

***

### 2. 요구사항 분석

#### 2.1 기능 요구사항

1. 알림을 미래 시점에 노출할 수 있어야 함
2. 알림 조회 시 현재 시간 이전의 알림만 반환
3. 즉시 노출 알림과 예약 알림을 모두 지원
4. 기존 알림 시스템과의 하위 호환성 유지

#### 2.2 비기능 요구사항

1. 기존 운영 중인 시스템의 무중단 배포
2. 성능 저하 최소화
3. 인덱스 활용 최적화
4. 데이터 일관성 보장

#### 2.3 제약사항

1. 알림은 높은 Insert 빈도를 가짐 (인덱스 최소화 필요)
2. Batch Insert 사용 중
3. 기존 데이터의 마이그레이션 필요

***

### 3. 해결 방안 검토

#### 3.1 방안 A: `created_at` 컬럼 활용

* 테이블 구조 변경 불필요하지만 의미적 혼란, 디버깅 불편
* 생성/노출 시간 구분 불가
* **결론**: 비추천

#### 3.2 방안 B: `display_at` 컬럼 추가 (채택)

* 생성 시간(`created_at`)과 노출 시간(`display_at`)을 명확히 구분
* 감사 추적 및 하위 호환성 유지
* **결론**: 채택

***

### 4. 최종 설계 결정

#### 선택된 방안

✅ **방안 B: `display_at` 컬럼 추가**

#### 이유

1. 데이터 무결성 확보
2. 감사 및 추적 가능성 향상
3. 장기적 유지보수성 확보
4. 비용 대비 명확성의 가치가 큼

#### 테이블 스키마

```sql
ALTER TABLE `around`.`channel_notification`
ADD COLUMN `display_at` DATETIME(6) NULL
COMMENT '노출 시간 (NULL인 경우 created_at 사용)';
```

#### 인덱스

```sql
ALTER TABLE `around`.`channel_notification`
ADD INDEX `idx_target_display` (`target_channel_id`, `display_at`);
```

***

### 5. 구현 상세

#### 5.1 영향받는 컴포넌트

* **엔티티**: `ChannelNotification.kt` → `display_at` 필드 추가
* **리포지토리**: 조회 쿼리에 `display_at <= NOW()` 조건 추가
* **서비스**: 즉시 노출은 생략 시 init 자동, 예약은 명시적 전달
* **Batch Insert**: SQL 및 DTO에 `display_at` 필드 추가

#### 5.2 조회 로직

```kotlin
.where(
    channelNotification.displayAt.isNull
        .or(channelNotification.displayAt.loe(LocalDateTime.now()))
)
.orderBy(
    channelNotification.displayAt.coalesce(channelNotification.createdAt).desc()
)
```

***

### 6. 마이그레이션 전략

#### 3단계 무중단 배포

1. **Phase 1**: 컬럼 추가 및 기존 데이터 업데이트 (`display_at = created_at`)
2. **Phase 2**: 코드 수정 후 배포 (NULL 안전 처리 포함)
3. **Phase 3**: NULL 제거 후 `NOT NULL` 제약 추가

#### Flyway 스크립트 예시

```sql
ALTER TABLE `around`.`channel_notification`
ADD COLUMN `display_at` DATETIME(6) NULL;
UPDATE `around`.`channel_notification` SET display_at = created_at WHERE display_at IS NULL;
ALTER TABLE `around`.`channel_notification`
ADD INDEX `idx_target_display` (`target_channel_id`, `display_at`);
```

***

### 7. Init 블록 설계 결정

#### 채택안: Init 블록 유지

```kotlin
@Column(name = "display_at", nullable = true)
@Comment("노출 시간 (NULL인 경우 created_at 사용)")
var displayAt: LocalDateTime? = displayAt
    protected set

init {
    if (this.displayAt == null) this.displayAt = LocalDateTime.now()
}
```

**이유**

* Fail-safe 보장
* 코드 수정 최소화
* 예약 알림은 명시, 즉시 알림은 생략 가능
* 성능 영향 무시 가능 수준

***

### 8. 주의사항 및 제약사항

#### 예약 알림 생성 시

✅

```kotlin
val displayAt = NotificationTimeCalculator.calculateAfternoonTwoOClockTime(now)
ChannelNotification(displayAt = displayAt)
```

❌

```kotlin
ChannelNotification(displayAt = entity.createdAt)
```

#### Batch Insert 시

* `displayAt` 반드시 명시 필요 (init 미적용)

#### 인덱스 모니터링

```sql
SHOW INDEX FROM channel_notification;
SELECT * FROM sys.schema_unused_indexes;
```

***

### 9. 결론

1. `display_at` 컬럼 추가로 생성/노출 시간 명확히 분리
2. Init 블록 유지로 안정성과 생산성 확보
3. 무중단 3단계 마이그레이션으로 운영 리스크 최소화

***

### 부록

* **엔티티**: `zone-data/zone-jpa/.../ChannelNotification.kt`
* **리포지토리**: `.../ChannelNotificationRepository.kt`
* **서비스**: `.../ChannelNotificationCreator.kt`, `.../ChannelNotificationBatchInserter.kt`
* **유틸리티**: `NotificationTimeCalculator.kt`
* **마이그레이션**: `V{TIMESTAMP}__AddDisplayAtToChannelNotification.sql`
