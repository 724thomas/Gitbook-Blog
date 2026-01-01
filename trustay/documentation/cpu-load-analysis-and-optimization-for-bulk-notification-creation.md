---
description: 대량 작업시 CPU 부하 해결 과정
---

# CPU Load Analysis and Optimization for Bulk Notification Creation

## 1. 콘솔 알림, 푸시 발송

콘솔에서 **전체 알림 생성 → 푸시 생성 → 발송**이 겹치는 구간에서 **DB CPU가 100%까지 치솟는 문제**가 발생했다.\
해당 DB는 다른 서비스와도 **공유 인스턴스**이기 때문에, 단일 작업의 부하가 전체 시스템에 영향을 주는 상황이었습니다.

<figure><img src="../../.gitbook/assets/image (458).png" alt=""><figcaption></figcaption></figure>

다른 서비스에서도 DB를 공유하고 있기때문에, 문제가 커지게됩니다.

문제는 크게 2가지였는데,

* 위치 기반 쿼리 효율성
* 알림 생성/소비 과정에서의 효율성

우선적으로 급히 사용해야하는 알림 생성/소비 과정에서의 효율성을 해결하려고 했습니다.



### Notification - read에 문제가 없는지 확인한다.

#### 문제 발생 - writer 인스턴스

* 특정 시점부터 **CPU가 급격히 상승**
* 알림 생성 작업과 정확히 시간대가 겹침

<figure><img src="../../.gitbook/assets/Pasted Graphic 1 (1).png" alt=""><figcaption></figcaption></figure>

<pre class="language-kotlin"><code class="lang-kotlin"><strong>A. QueryDSL
</strong><strong>val whereCondition =
</strong>    BooleanBuilder()
        .and(channel.type.eq(channelType))
        .and(channel.dataStatus.eq(DataStatus.USED))

val totalCount = // TotalCount에서 모든 데이터를 읽고 사이즈를 읽는다.
    queryFactory
        .select(channel.id)
        .from(channel)
        .where(whereCondition)
        .fetch()
        .size

val result =
    queryFactory
        .select(channel.id)
        .from(channel)
        .where(whereCondition)
        .offset(pageable.offset)
        .limit(pageable.pageSize.toLong())
        .fetch()


return PageImpl(
    result,
    pageable,
    totalCount.toLong(),
).toResultPage()


B. Service
for (page in 0 until defaultPage.totalPageSize) {
            try {
                val pageable = PageRequest.of(page, BATCH_SIZE)
                val pagedChannelIds =
                    channelRepository.findChannelIdsByTypeWithPaging(
                        channelType = ChannelType.USER,
                        pageable = pageable,
                    )
//                channelNotificationFacade.bulkCreateConsoleNotification(
//                    targetChannelIds = pagedChannelIds.items,
//                    event = event,
//                )

                totalProcessed += pagedChannelIds.items.size
            } catch (e: Exception) {
                log.error(e) { "콘솔 알림 페이지 ${page + 1} 생성 실패 - title: ${event.title}" }
            }
        }
        log.info { "콘솔 알림 생성 완료 - title: ${event.title}, 총: ${totalProcessed}개" }


</code></pre>



#### 가설

문제의 원인은 channelRepository.findChannelIdsByTypeWithPaging에 있다고 판단.

A. 비효율적인 Count쿼리: 모든 데이터를 다 읽고 카운트를 실행하는 문제: fetch().size 방식

B. Count 쿼리를 매 페이징 사이즈만큼 조회때마다 집계: 실제로는 1번만 필요

C. Offset방식: 매번 0번째 데이터부터 조회

#### 과정

#### A. Count쿼리 변경

```kotlin
val totalCount =
    queryFactory
        .select(channel.id.count())
        .from(channel)
        .where(whereCondition)
        .fetchOne() ?: 0L

```

<figure><img src="../../.gitbook/assets/Pasted Graphic 2.png" alt=""><figcaption></figcaption></figure>

동일한 결과

* CPU 패턴 변화 없음
* Count 쿼리는 핵심 원인이 아님



#### B. Count쿼리와 결과 쿼리 분리

```kotlin
override fun countChannelIdsByType(channelType: ChannelType): Long {
    return queryFactory
        .select(channel.id.count())
        .from(channel)
        .where(
            channel.type.eq(channelType)
                .and(channel.dataStatus.eq(DataStatus.USED)),
        ).fetchOne() ?: 0L
}


override fun findChannelIdsByTypeWithPaging(
    channelType: ChannelType,
    pageable: Pageable,
): ResultPage<Long> {
    val whereCondition =
        BooleanBuilder()
            .and(channel.type.eq(channelType))
            .and(channel.dataStatus.eq(DataStatus.USED))


    val result =
        queryFactory
            .select(channel.id)
            .from(channel)
            .where(whereCondition)
            .offset(pageable.offset)
            .limit(pageable.pageSize.toLong())
            .fetch()


    return PageImpl(
        result,
        pageable,
        0L,
    ).toResultPage()
}
```

<figure><img src="../../.gitbook/assets/Pasted Graphic 3.png" alt=""><figcaption></figcaption></figure>

동일한 결과

* 여전히 동일한 CPU 부하
* Offset Paging 자체가 문제일까 의심<br>

#### C. 오프셋 페이징 처리를 커서 방식으로 변경

```kotlin
var lastChannelId: Long? = null


while (true) {
    val channelIdsCursor = channelRepository.findChannelIdsByTypeWithCursor(
        channelType = ChannelType.USER,
        lastChannelId = lastChannelId,
        size = BATCH_SIZE,
    )
    if (channelIdsCursor.isEmpty()) {
        break
    }


    lastChannelId = channelIdsCursor.last()
}


override fun findChannelIdsByTypeWithCursor(
    channelType: ChannelType,
    lastChannelId: Long?,
    size: Int,
): List<Long> {
    val condition = BooleanBuilder()
        .and(channel.type.eq(channelType))
        .and(channel.dataStatus.eq(DataStatus.USED))


    if (lastChannelId != null) {
        condition.and(channel.id.gt(lastChannelId))
    }


    return queryFactory
        .select(channel.id)
        .from(channel)
        .where(condition)
        .orderBy(channel.id.asc())
        .limit(size.toLong())
        .fetch()
}

```

<figure><img src="../../.gitbook/assets/Pasted Graphic 4.png" alt=""><figcaption></figcaption></figure>

확실히 부하가 줄어든걸 볼 수 있습니다.

DB CPU 부하가 **눈에 띄게 감소: 조회 병목의 핵심 원인은 Offset Paging**



### Notification - write에 문제가 없는지 확인합니다.

<figure><img src="../../.gitbook/assets/Pasted Graphic 5.png" alt=""><figcaption></figcaption></figure>

* 200건 단위 INSERT
* 5개의 인덱스 동시 업데이트
* CPU 사용량 **약 5\~10% 증가**

200건씩의 INSERT가 발생할때마다 5번개의 인덱스가 업데이트되지만 대량 INSERT에서는 평소보다 5\~10% CPU 부하가 생기고 있고, 크게 문제가 되진 않는다고 판단됩니다.

<br>

### Push - read에 문제가 없는지 확인합니다.

<figure><img src="../../.gitbook/assets/Pasted Graphic 6.png" alt=""><figcaption></figcaption></figure>

채널 조회 부분만 실행했을때 Notification - read와 동일한 문제로 CPU가 높아집니다. Notification - read와 동일한 문제

#### A. Cursor 방식으로 처리.

<figure><img src="../../.gitbook/assets/Pasted Graphic 7.png" alt=""><figcaption></figcaption></figure>

Cursor방식 적용 후 CPU 부하가 낮아졌습니다.

#### B. 나머지 푸시를 생성하는 과정에서의 조회

```kotlin
userNotificationSettingRepository.findByUserIdsAndType(userIds, type) // 알림 설정 조회
- type: Range
- extra: using Index
channelLinkRepository.findChannelIdsByUserIds // userId -> channelId 변환
- type: Range
- extra: using Index
channelDeviceTokenRepository.findAllByChannelIds // fcm토큰 조회
- type: Range
- extra: using Index
```

<figure><img src="../../.gitbook/assets/Pasted Graphic 8.png" alt=""><figcaption></figcaption></figure>

CPU 13%로 push-read 조회는 크게 문제가 없습니다.



### Push - write에 문제가 없는지 확인합니다.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

* 처리 시간은 길지만
* CPU 부하는 크지 않음
* 대량 발송 시에도 **10\~15% 수준**



### Push 발송 - read에 문제가 없는지 확인한다.

처리되지 않은 알림 또는 처리중인 알림을 조회하는 코드는 아래와 같습니다.

```kotlin
return queryFactory
            .selectFrom(pushNotification)
            .where(
                pushNotification.pushNotificationStatus.eq(PushNotificationStatus.PENDING)
                    .and(pushNotification.pushTime.loe(now))
                    .or(
                        pushNotification.pushNotificationStatus.eq(PushNotificationStatus.PROCESSING)
                            .and(pushNotification.processedAt.loe(now)),
                    ),
            )
            .orderBy(pushNotification.pushTime.asc())
            .limit(limit.toLong())
            .fetch()
            
- type: Range
- extra: Using Index


templateRepository.getSystemNotificationByContentType
내부적으로 2개의 쿼리가 실행됩니다. 갯수가 많이 않아(100개 이하) 문제가 되지 않습니다.
```

푸시 대기 테이블에서는 대부분 PENDING 상태입니다. 반면, PROCESSING상태의 알림 조회는 중간에 처리되다가 서버가 죽는 경우 남아있는 알림입니다.&#x20;

현재 인덱스는 push\_notification\_status, push\_time만 설정되어있고, 쿼리 분석결과 type=range로 인덱스를 잘 활용하고 있습니다.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Push\_Notification에서 10만건의 발송 대기가 쌓여있고, 즉시 발송 알림들을 생성하여 발송했을때 CPU의 큰 변화가 없습니다.



### Push 발송 - write에 문제가 없는지 확인합니다.

스케줄러가 돌면서 처리안된 200개를 조회하고, 상태값을 변경합니다.



Push를 발송할때, CHUNK\_SIZE만큼 조회를 한뒤, 순차 개별적으로 PROCESSING으로 상태값을 바꾸고, 발송이 완료되면 테이블에서 삭제하는 로직으로 동작합니다. 10만개의 발송 대기 푸시를 즉시 발송하도록 모두 조회, 업데이트를 해봤습니다.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

약 10\~15% CPU를 점유합니다.



### 최종 기능 단위 연결 테스트

이번에는 푸시 발송하기전, 활동탭 알림과 푸시 알림 생성 CPU값입니다.

<figure><img src="../../.gitbook/assets/image (459).png" alt=""><figcaption></figcaption></figure>

처음 10\~15%를 점유하고, 점차 내려갑니다. CPU가 처음에 튀는 이유는 **InnoDB Buffer Pool Cold Start**때문에 잠시 부하가 걸립니다. ([https://yesjuhee.tistory.com/31](https://yesjuhee.tistory.com/31))

BULK Insert처리를 하게될시 허용범위가 어느정도인지 감이 안오는 상태인데, 단순히 생각하기로는 최근 기준으로 CPU 최대치에 10\~15%를 비교하면 될것이라 생각을 했습니다.

#### Master(Writer) 인스턴스 최대치 - 대략 25%

<figure><img src="../../.gitbook/assets/image (460).png" alt=""><figcaption></figcaption></figure>

#### Slave(Reader) 인스턴스 최대치 - 대략 40%

<figure><img src="../../.gitbook/assets/image (461).png" alt=""><figcaption></figcaption></figure>



### 남은 문제:

*   처리량을 낮췄고 서비스를 상시 배포하고 있기때문에, 콘솔 알림 생성 스레드의 작업이 끝나기전에 재배포 되는 경우.

    * 기존: event의 상태를 PROCESSING으로 바꾸고, 처리. PROCESSING이 있으면 Skip. 작업이 완료되면 PROCESSED로 변경 (5초에 100개 처리)
    * 문제:&#x20;
      * 스레드가 죽으면 평생 작업이 시작/재시작 하지 않습니다.&#x20;
      * 마지막 위치를 기록하기가 어렵습니다. 알림센터 -> 푸시 순차적으로 기능이 실행되기때문
    * 해결방법1. 중복 생성:&#x20;
      * PROCESSING이 2시간 이상이면, 시간 update 후 재처리. 재처리로 인해 중복으로 생성
    * 해결방법2. 누락:&#x20;
      * PROCESSING이 2시간 이상이면, PROCESSED로 변경 후, PENDING 작업 처리


* 푸시 알림 구분이 없어서, 전체 대상 푸시가 쌓여있으면, 먼저 만들어진 전체 대상 푸시가 끝날때까지 일상 푸시는 발송이 안되는 경우
  * 기존: 발송 시간이 가장 오래된 푸시부터 처리
  * 문제: Head of line blocking
  * 해결방법1: 하나의 스케줄러를 두개로 나눠서 천천히 발송돼도 괜찮은 전체 푸시용, 즉시 발송되어야하는 개별 푸시용으로 분리한다.
  * 해결방법2: 우선순위큐와 같이 동작하게 두는 방법. 처리량이 낮으면 starvation
  * 해결방법3: 번갈아가면서 처리. 전체1 - 즉시3 - 전체1 - 즉시3 등..



## 2. x일 미접속자 인기글 알림

비효율적인 쿼리를 개선

<pre class="language-kotlin"><code class="lang-kotlin">// 개선 전aasdf
<strong>
</strong>override fun findAllChannelIdsWithCursor(
        beforeDateTime: LocalDateTime?,
        lastChannelId: Long?,
        size: Long,
    ): List&#x3C;Long> {
        val havingCondition = if (beforeDateTime != null) {
            channelDeviceToken.updatedAt.max().lt(beforeDateTime)
        } else {
            null
        }

        val result = queryFactory
            .select(channelDeviceToken.channelId)
            .from(channelDeviceToken)
            .groupBy(channelDeviceToken.channelId)
            .having(havingCondition)
            .where(
                if (lastChannelId != null) {
                    channelDeviceToken.channelId.gt(lastChannelId)
                } else {
                    null
                },
            )
            .orderBy(channelDeviceToken.channelId.asc())
            .limit(size)
            .fetch()

        return result
    }
    
// 개선 후
    override fun findAllChannelIdsWithCursor(
        beforeDateTime: LocalDateTime?,
        lastChannelId: Long?,
        size: Long,
    ): List&#x3C;Long> {
        val havingCondition = if (beforeDateTime != null) {
            channelDeviceToken.updatedAt.max().lt(beforeDateTime)
        } else {
            null
        }

        val subQuery = JPAExpressions
            .select(channelDeviceToken.channelId)
            .from(channelDeviceToken)
            .where(channelDeviceToken.dataStatus.eq(DataStatus.USED))
            .groupBy(channelDeviceToken.channelId)
            .having(havingCondition)

        val whereCondition = BooleanBuilder()
            .and(channel.dataStatus.eq(DataStatus.USED))
            .and(channel.type.`in`(ChannelType.USER, ChannelType.KID_USER))

        if (lastChannelId != null) {
            whereCondition.and(channel.id.gt(lastChannelId))
        }

        return queryFactory
            .select(channel.id)
            .from(channel)
            .where(
                channel.id.`in`(subQuery)
                    .and(whereCondition),
            )
            .orderBy(channel.id.asc())
            .limit(size)
            .fetch()
    }
</code></pre>

```
    
type: Range
extra: Using where; Using index	
-> Limit: 500 row(s)  (cost=133 rows=365) (actual time=0.0346..0.501 rows=358 loops=1)
    -> Filter: (max(cdt.updated_at) < <cache>((now() - interval 3 day)))  (cost=133 rows=365) (actual time=0.0337..0.476 rows=358 loops=1)
        -> Group aggregate: max(cdt.updated_at)  (cost=133 rows=365) (actual time=0.0263..0.411 rows=403 loops=1)
            -> Filter: (cdt.channel_id > 501)  (cost=89 rows=442) (actual time=0.0178..0.287 rows=442 loops=1)
                -> Covering index range scan on cdt using idx_channel_id_updated_at over (501 < channel_id)  (cost=89 rows=442) (actual time=0.0165..0.243 rows=442 loops=1)
	

type1: range, type2: index
extra1: Using where, extra2: Using where	
-> Limit: 500 row(s)  (cost=25137 rows=500) (actual time=1.8..502 rows=346 loops=1)
    -> Filter: ((c.data_status = 'USED') and (c.`type` in ('USER','KID_USER')) and (c.id > 501) and <in_optimizer>(c.id,c.id in (select #2)))  (cost=25137 rows=2504) (actual time=1.8..502 rows=346 loops=1)
        -> Index range scan on c using PRIMARY over (501 < id)  (cost=25137 rows=125206) (actual time=0.0213..173 rows=256414 loops=1)
        -> Select #2 (subquery in condition; run only once)
            -> Filter: ((c.id = `<materialized_subquery>`.channel_id))  (cost=58.9..58.9 rows=1) (actual time=693e-6..693e-6 rows=0.00136 loops=253582)
                -> Limit: 1 row(s)  (cost=58.8..58.8 rows=1) (actual time=556e-6..556e-6 rows=0.00136 loops=253582)
                    -> Index lookup on <materialized_subquery> using <auto_distinct_key> (channel_id=c.id)  (actual time=416e-6..416e-6 rows=0.00136 loops=253582)
                        -> Materialize with deduplication  (cost=58.8..58.8 rows=46.7) (actual time=1.51..1.51 rows=365 loops=1)
                            -> Filter: (max(cdt.updated_at) < <cache>((now() - interval 3 day)))  (cost=54.1 rows=46.7) (actual time=0.11..1.39 rows=365 loops=1)
                                -> Group aggregate: max(cdt.updated_at)  (cost=54.1 rows=46.7) (actual time=0.104..1.33 rows=417 loops=1)
                                    -> Filter: (cdt.data_status = 'USED')  (cost=49.5 rows=46.7) (actual time=0.0967..1.18 rows=467 loops=1)
                                        -> Index scan on cdt using idx_channel_id_device_id  (cost=49.5 rows=467) (actual time=0.0958..1.12 rows=467 loops=1)

개선을 하려고했지만, 결과를 봤을때 Driving table이 잘못 잡혀서 문제가 생김.
결론: 현재 잘 동작한다.
```

```kotlin
// 개선 전
override fun findAllInactiveUserIdsWithCursor(
        beforeDateTime: LocalDateTime,
        lastUserId: Long?,
        size: Long,
    ): List<Long> {
        val whereCondition = BooleanBuilder()
            .and(user.lastLoginAt.before(beforeDateTime))
            .and(user.status.eq(UserStatus.ACTIVATION))
            .and(user.dataStatus.eq(DataStatus.USED))
            .and(userSocialAccount.role.eq(UserRole.USER))

        if (lastUserId != null) {
            whereCondition.and(user.id.lt(lastUserId))
        }

        val result = queryFactory.select(user.id)
            .from(user)
            .join(userSocialAccount)
            .on(userSocialAccount.user.id.eq(user.id))
            .where(whereCondition)
            .orderBy(user.id.desc())
            .limit(size)
            .fetch()

        return result
    }

// 개선 후
    override fun findAllInactiveUserIdsWithCursor(
        beforeDateTime: LocalDateTime,
        lastUserId: Long?,
        size: Long,
    ): List<Long> {

        val whereCondition = BooleanBuilder()
            .and(user.lastLoginAt.lt(beforeDateTime))
            .and(user.status.eq(UserStatus.ACTIVATION))
            .and(user.dataStatus.eq(DataStatus.USED))

        if (lastUserId != null) {
            whereCondition.and(user.id.gt(lastUserId))
        }

        val isUserExists = JPAExpressions
            .selectOne()
            .from(userSocialAccount)
            .where(
                userSocialAccount.user.id.eq(user.id)
                    .and(userSocialAccount.role.eq(UserRole.USER)),
            )
            .exists()

        val inactiveUsers = queryFactory
            .select(user.id)
            .from(user)
            .where(whereCondition.and(isUserExists))
            .orderBy(user.id.asc())
            .limit(size)
            .fetch()

        return inactiveUsers
```

```
type1: range, type2: ref
extra1: Using where; Backward index scan, extra2: Using where
-> Limit: 500 row(s)  (cost=101 rows=0.167) (actual time=0.103..3.49 rows=278 loops=1)
    -> Nested loop inner join  (cost=101 rows=0.167) (actual time=0.102..3.47 rows=278 loops=1)
        -> Filter: ((u.last_login_at < <cache>((now() - interval 3 day))) and (u.`status` = 'ACTIVATION') and (u.data_status = 'USED') and (u.id < 501))  (cost=100 rows=1.67) (actual time=0.0796..0.696 rows=419 loops=1)
            -> Index range scan on u using PRIMARY over (id < 501) (reverse)  (cost=100 rows=500) (actual time=0.0714..0.522 rows=500 loops=1)
        -> Filter: (usa.`role` = 'USER')  (cost=0.475 rows=0.1) (actual time=0.0059..0.00638 rows=0.663 loops=419)
            -> Index lookup on usa using user_id (user_id=u.id)  (cost=0.475 rows=1) (actual time=0.00543..0.00608 rows=1.06 loops=419)
	

type1: range, type2: ref
extra1: Using where, extra2: Using where; FirstMatch(u)	
-> Limit: 500 row(s)  (cost=32017 rows=52.9) (actual time=0.0544..3.7 rows=500 loops=1)
    -> Nested loop semijoin  (cost=32017 rows=52.9) (actual time=0.0533..3.65 rows=500 loops=1)
        -> Filter: ((u.last_login_at < <cache>((now() - interval 3 day))) and (u.`status` = 'ACTIVATION') and (u.data_status = 'USED') and (u.id > 501))  (cost=31716 rows=529) (actual time=0.0345..0.769 rows=508 loops=1)
            -> Index range scan on u using PRIMARY over (501 < id)  (cost=31716 rows=158603) (actual time=0.0206..0.575 rows=570 loops=1)
        -> Filter: (usa.`role` = 'USER')  (cost=0.0469 rows=0.1) (actual time=0.00551..0.00551 rows=0.984 loops=508)
            -> Index lookup on usa using user_id (user_id=u.id)  (cost=0.0469 rows=1) (actual time=0.00527..0.00528 rows=1 loops=508)
	
개선 전과 비교해서, JOIN을 사용했을때 N건이면 N번 비교, EXIST는 1건 발견 즉시 종료
```

## 3. 단지 질문글 알림센터, 푸시 성능 개선
