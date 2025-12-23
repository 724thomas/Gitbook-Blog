---
description: 전체 알림 생성시 CPU 부하 해결 과정
---

# CPU Load Analysis and Optimization for Bulk Notification Creation

콘솔에서 **전체 알림 생성 → 푸시 생성 → 발송**이 겹치는 구간에서 **DB CPU가 100%까지 치솟는 문제**가 발생했다.\
해당 DB는 다른 서비스와도 **공유 인스턴스**이기 때문에, 단일 작업의 부하가 전체 시스템에 영향을 주는 상황이었습니다.

<figure><img src="../../.gitbook/assets/image (458).png" alt=""><figcaption></figcaption></figure>

다른 서비스에서도 DB를 공유하고 있기때문에, 문제가 커지게됩니다.

문제는 크게 2가지였는데,

* 위치 기반 쿼리 효율성
* 알림 생성/소비 과정에서의 효율성

우선적으로 급히 사용해야하는 알림 생성/소비 과정에서의 효율성을 해결하려고 했습니다.



## Notification - read에 문제가 없는지 확인한다.



### &#x20;문제 발생 - writer 인스턴스

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



### 가설

문제의 원인은 channelRepository.findChannelIdsByTypeWithPaging에 있다고 판단.

A. 비효율적인 Count쿼리: 모든 데이터를 다 읽고 카운트를 실행하는 문제: fetch().size 방식

B. Count 쿼리를 매 페이징 사이즈만큼 조회때마다 집계: 실제로는 1번만 필요

C. Offset방식: 매번 0번째 데이터부터 조회

### 과정

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



## &#x20;Notification - write에 문제가 없는지 확인합니다.

<figure><img src="../../.gitbook/assets/Pasted Graphic 5.png" alt=""><figcaption></figcaption></figure>

* 200건 단위 INSERT
* 5개의 인덱스 동시 업데이트
* CPU 사용량 **약 5\~10% 증가**

200건씩의 INSERT가 발생할때마다 5번개의 인덱스가 업데이트되지만 대량 INSERT에서는 평소보다 5\~10% CPU 부하가 생기고 있고, 크게 문제가 되진 않는다고 판단됩니다.

<br>

## &#x20;Push - read에 문제가 없는지 확인합니다.

<figure><img src="../../.gitbook/assets/Pasted Graphic 6.png" alt=""><figcaption></figcaption></figure>

채널 조회 부분만 실행했을때 Notification - read와 동일한 문제로 CPU가 높아집니다. Notification - read와 동일한 문제

### A. Cursor 방식으로 처리.

<figure><img src="../../.gitbook/assets/Pasted Graphic 7.png" alt=""><figcaption></figcaption></figure>

Cursor방식 적용 후 CPU 부하가 낮아졌습니다.

### B. 나머지 조회(알림 허용, fcm 토큰 조회, 채널Id로 타운Id 조회)

<figure><img src="../../.gitbook/assets/Pasted Graphic 8.png" alt=""><figcaption></figcaption></figure>

CPU 13%로 push-read 조회는 크게 문제가 없습니다.

<br>

## Push - write에 문제가 없는지 확인합니다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* 처리 시간은 길지만
* CPU 부하는 크지 않음
* 대량 발송 시에도 **10\~15% 수준**



## Push 발송 - read에 문제가 없는지 확인한다.

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
```

푸시 대기 테이블에서는 대부분 PENDING 상태입니다. 반면, PROCESSING상태는 중간에 처리되다가 서버가 죽는 경우입니다.

현재 인덱스는 push\_notification\_status, push\_time만 설정되어있고, 쿼리 분석결과 type=range로 인덱스를 잘 활용하고 있습니다.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Push\_Notification에서 10만건의 발송 대기가 쌓여있고, 즉시 발송 알림들을 생성하여 발송했을때 CPU의 큰 변화가 없습니다.



## Push 발송 - write에 문제가 없는지 확인합니다.

Push를 발송할때, CHUNK\_SIZE만큼 조회를 한뒤, 개별적으로 PROCESSING으로 상태값을 바꾸고, 발송이 완료되면 테이블에서 삭제하는 로직으로 동작합니다. 10만개의 발송 대기 푸시를 즉시 발송하도록 모두 update를 해봤습니다.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

약 10\~15% CPU를 점유합니다.



BULK 처리를 하게될시 허용범위가 어느정도인지 감이 안오는 상태. 정보가 없고 GPT는 허용범위라고 하지만,  다른 팀은 어떤지 물어봐야할것 같습니다.





## 남은 문제:

*   처리량을 낮췄고 서비스를 상시 배포하고 있기때문에, 콘솔 알림 생성 스레드의 작업이 끝나기전에 재배포 되는 경우.

    * 기존: event의 상태를 PROCESSING으로 바꾸고, 처리. PROCESSING이 있으면 Skip. 작업이 완료되면 PROCESSED로 변경 (5초에 100개 처리)
    * 문제: 스레드가 죽으면 평생 작업이 시작/재시작 하지 않습니다.
    * 해결방법1: 현재까지 완료한 작업의 위치를 청크마다 기록 + 시간 업데이트. 스케줄러는 업데이트 시간이 너무 오래지났으면 다시 재시작. 기록한 마지막 위치부터 시작
      * 문제: 알림 생성과 푸시 생성은 순차적으로 처리되고 있어서, 오프셋을 기록하더라도 어떤게 처리된건지 알 수 없음
      * 해결방법: 표시를 해두지 않고, 아예 처음부터 작업이 되게한다. = 중복 생성 허용.


* 푸시 알림 구분이 없어서, 전체 대상 푸시가 쌓여있으면, 먼저 만들어진 전체 대상 푸시가 끝날때까지 일상 푸시는 발송이 안되는 경우
  * 기존: 발송 시간이 가장 오래된 푸시부터 처리
  * 문제: Head of line blocking
  * 해결방법1: 하나의 스케줄러를 두개로 나눠서 천천히 발송돼도 괜찮은 전체 푸시용, 즉시 발송되어야하는 개별 푸시용으로 분리한다.
  * 해결방법2: 우선순위큐와 같이 동작하게 두는 방법. 처리량이 낮으면 starvation
  * 해결방법3: 번갈아가면서 처리. 전체1 - 즉시3 - 전체1 - 즉시3 등..

