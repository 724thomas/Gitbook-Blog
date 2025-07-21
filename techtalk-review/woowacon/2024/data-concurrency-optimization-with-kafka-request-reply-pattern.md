---
description: 'Kafka 기반 대규모 데이터 동시성 최적화: Request-Reply 패턴 활용 사례'
---

# Data Concurrency Optimization with Kafka – Request-Reply Pattern

## 🛠 Kafka와 Request-Reply로 푼 대규모 동시성 이슈 해결기

배달의민족은 수십만 개의 가게 주문 처리 과정에서 발생하는 **대규모 동시성 문제**를 Kafka와 Request-Reply 패턴을 이용해 해결했습니다. 이 글은 해당 장애 분석, 구조 설계, 코드 구현까지의 전 과정을 개발자 친화적으로 설명합니다.

***

### 🚨 장애 상황: 수십만 건의 실시간 데이터 변경

배민의 시스템은 주문량이 폭증하거나 기상 악화 등 **트리거**가 발생하면, 특정 가게의 **배달 반경을 실시간으로 축소**시켜 유입되는 주문을 줄입니다.

하지만 이때 수십만 개 가게에 대해 동시에 반경 변경이 발생하고, 동일한 칼럼(데이터 열)을 수정하게 되면 다음과 같은 문제가 발생합니다:

* 💥 **Deadlock (교착 상태)**: 서로 락을 잡은 상태에서 아무도 작업을 끝내지 못하는 상태
* ⏳ **Lock Timeout**: 락이 걸린 상태에서 시간이 지나도 작업이 안 되면 에러 발생
* ⚠️ **Race Condition (경쟁 조건)**: 여러 쓰레드가 동시에 접근해 순서가 꼬이며 잘못된 결과가 나오는 문제

***

### 🔄 순서가 중요한 이유

예를 들어,

1. 라이더 부족 트리거 발생 → 반경 3km로 축소
2. 시스템 장애 트리거 발생 → 반경 1km로 더 축소

이렇게 순서가 지켜져야 하는데, 1번만 적용되고 2번이 무시되면 시스템이 **장애 상태임에도 주문을 계속 받게 되는 오류**가 발생합니다.

💡 _순서 보장이란?_

여러 요청이 있을 때 “처리 순서”가 예상대로 유지되는 것을 의미합니다. 장애 시에는 더 강력한 제한이 나중에 적용되어야 합니다.

***

### 🔐 분산 락? 해결되지 않았다

초기에는 `Pessimistic Lock`(비관적 락)이나 `Redis 분산 락`을 사용하려 했습니다.

💡 _락(Lock)이란?_

여러 프로세스가 동시에 데이터를 수정하지 못하게 막는 장치입니다.

하지만 다음 문제들이 있었습니다:

* 락 경합 → 성능 저하
* 락을 걸어도 정확한 순서 보장이 안 됨
* **수평 확장 불가** (서버를 여러 대 늘려도 처리량이 증가하지 않음)
* 응답 결과를 받을 수 없음

***

### 📦 Kafka의 특성 활용

Kafka는 동일한 **Key**를 사용하면 **같은 Partition**으로 메시지를 보냅니다. 이 Partition 내부에서는 순서가 보장됩니다.

💡 _Kafka란?_

분산 메시지 큐 시스템으로, 데이터를 보내는 쪽(Producer)과 받는 쪽(Consumer)을 분리시켜 대규모 트래픽을 처리할 수 있습니다.

💡 _Partition이란?_

Kafka에서 토픽을 나누는 단위입니다. 동일한 Key가 항상 같은 Partition으로 가면 순서가 유지됩니다.

***

### 📬 Request-Reply 패턴 도입

Kafka는 기본적으로 **단방향**이기 때문에 처리 결과를 받을 수 없습니다. 이를 해결하기 위해 `Request-Reply` 패턴을 도입합니다.

💡 _Request-Reply란?_

요청자가 메시지를 보내고, 응답자는 처리 후 응답 메시지를 돌려주는 구조입니다. 요청자 → 응답자 → 다시 요청자.

Spring Kafka에서는 이를 위해 **`ReplyingKafkaTemplate`** 클래스를 제공합니다.

[![Image](https://private-user-images.githubusercontent.com/113500771/468525509-d52f9efd-7984-450c-8370-eedb716cca17.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTMwODc3NjcsIm5iZiI6MTc1MzA4NzQ2NywicGF0aCI6Ii8xMTM1MDA3NzEvNDY4NTI1NTA5LWQ1MmY5ZWZkLTc5ODQtNDUwYy04MzcwLWVlZGI3MTZjY2ExNy5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcyMVQwODQ0MjdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT05NWY5ZTk5M2E2ZTRkZjJjNzA0NTg0OWJjYTA5NzA4OWViZDJkZTY1ODU3OTMzNWQyMzk5YzY5NTI2ZDc1ZTA2JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.CAPLC1Bntbx79ReBMNy-fNokXvl8eyEnrTeLiT370-0)](https://private-user-images.githubusercontent.com/113500771/468525509-d52f9efd-7984-450c-8370-eedb716cca17.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTMwODc3NjcsIm5iZiI6MTc1MzA4NzQ2NywicGF0aCI6Ii8xMTM1MDA3NzEvNDY4NTI1NTA5LWQ1MmY5ZWZkLTc5ODQtNDUwYy04MzcwLWVlZGI3MTZjY2ExNy5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcyMVQwODQ0MjdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT05NWY5ZTk5M2E2ZTRkZjJjNzA0NTg0OWJjYTA5NzA4OWViZDJkZTY1ODU3OTMzNWQyMzk5YzY5NTI2ZDc1ZTA2JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.CAPLC1Bntbx79ReBMNy-fNokXvl8eyEnrTeLiT370-0)

[![Image](https://private-user-images.githubusercontent.com/113500771/468525427-896f344f-7f91-4489-90e1-60ab406ac1bb.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTMwODc3NjcsIm5iZiI6MTc1MzA4NzQ2NywicGF0aCI6Ii8xMTM1MDA3NzEvNDY4NTI1NDI3LTg5NmYzNDRmLTdmOTEtNDQ4OS05MGUxLTYwYWI0MDZhYzFiYi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcyMVQwODQ0MjdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT02N2I3ZDEyMWY4YzlkZjkwMTBiZTAzN2FhN2M4M2FkOTIwNjkxZTNhNDViNjBiMjkxOTEwYTliNTdiZGJmZTYzJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.FaWju5kA60OWrcl9kKZgVCv1zqu0VnsOUqc0orDxm5Q)](https://private-user-images.githubusercontent.com/113500771/468525427-896f344f-7f91-4489-90e1-60ab406ac1bb.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTMwODc3NjcsIm5iZiI6MTc1MzA4NzQ2NywicGF0aCI6Ii8xMTM1MDA3NzEvNDY4NTI1NDI3LTg5NmYzNDRmLTdmOTEtNDQ4OS05MGUxLTYwYWI0MDZhYzFiYi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcyMVQwODQ0MjdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT02N2I3ZDEyMWY4YzlkZjkwMTBiZTAzN2FhN2M4M2FkOTIwNjkxZTNhNDViNjBiMjkxOTEwYTliNTdiZGJmZTYzJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.FaWju5kA60OWrcl9kKZgVCv1zqu0VnsOUqc0orDxm5Q)

#### ReplyingKafkaTemplate(Requester) 요소

* KafkaProducer
  * 메시지 `전송`
* ReplyContainer
  * 전송한 메시지에 대한 `응답을 수신`

#### Kafka Broker(Request Channel / Reply Channel) 요소

* Request Topic
* Reply Topic

#### Consumer Group(Replier) 요소

* Consumer

| 항목          | KafkaTemplate         | ReplyingKafkaTemplate                               |
| ----------- | --------------------- | --------------------------------------------------- |
| ✅ 주요 목적     | 단방향 메시지 전송            | 요청-응답 구조 구현                                         |
| 🔄 메시지 흐름   | Producer → Topic      | Producer → Topic → Consumer → 응답 Topic → Producer   |
| 📥 응답 수신 기능 | 없음                    | 있음 (CompletableFuture로 응답 대기)                       |
| 🧠 내부 구성    | KafkaProducer 만 포함    | KafkaProducer + ReplyListenerContainer 포함           |
| 🧩 사용 예     | 이벤트 발행, 로그 전송, 알림 등   | 주문 처리 결과 요청, 동기적 응답이 필요한 경우                         |
| 🧵 비동기 처리   | 가능 (ListenableFuture) | 가능 (RequestReplyFuture, 내부적으로 CompletableFuture 사용) |
| ⚙️ 설정 복잡도   | 낮음 (간단히 사용 가능)        | 높음 (응답 토픽, 컨테이너 설정 필요)                              |

### 🧠 Correlation ID로 요청-응답 매핑

Kafka는 비동기 처리이기 때문에, 어떤 응답이 어떤 요청에 대한 것인지 구분이 필요합니다.

이를 위해 `correlationId`라는 고유 식별자(요청의 Key)를 사용합니다.

```
// 요청 시
correlationId = UUID.randomUUID();
futureMap.put(correlationId, new CompletableFuture<>());
```

응답 수신 시:

```
// 응답에 포함된 correlationId로 future를 찾아 응답 완료 처리
CompletableFuture<?> future = futureMap.remove(correlationId);
future.complete(response);
```

💡 _CompletableFuture란?_

비동기 결과를 기다리는 객체입니다. 응답이 오면 future.complete()으로 값을 채워넣습니다.

***

### 🧪 단일 토픽 운영 전략

#### RequestDispatcher 클래스 작성

**Request-Topic과 Reply-Topic을 도메인 기준으로 단일 토픽**으로 운영했습니다.

```
@Component
public class RequestDispatcher<V extends Request<?>, R extends Reply<?>> {
private final ReplyingKafkaTemplate&lt;String, V, R&gt; replyingKafkaTemplate;private final String requestTopic;public RequestDispatcher(ReplyingKafkaTemplate&lt;String, V, R&gt; replyingKafkaTemplate, String requestTopic) {    this.requestTopic = requestTopic;    this.replyingKafkaTemplate = replyingKafkaTemplate;}public CompletableFuture&lt;Void&gt; send(V request) {    return replyingKafkaTemplate.send(requestTopic, request.getKey(), request) //ListenableFuture&lt;SendResult&lt;...&gt;&gt;            .completable() // CompletableFuture&lt;SendResult&lt;...&gt;&gt;            .thenRun(() -&gt; {});}
    
  
}
```

장점:

* 설정 관리 간단
* 공통 코드를 도출해 Requester 재사용 가능
* 기능 증설/확장 용이

단점:

* 컨슈머가 필요 없는 메시지도 구독하게 되므로 필터링 필요

💡 _토픽(Topic)이란?_

Kafka에서 메시지를 분류하는 논리적 단위입니다.

***

### 🧵 Reply 메시지 처리 전략: sharedReplyTopic

모든 인스턴스가 reply-topic을 구독하면 **자신과 관련 없는 응답도 수신**하게 됩니다.

[![Image](https://private-user-images.githubusercontent.com/113500771/468525259-4e3fe04f-8467-405f-9f7b-570ca9d9dfd1.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTMwODc3NjcsIm5iZiI6MTc1MzA4NzQ2NywicGF0aCI6Ii8xMTM1MDA3NzEvNDY4NTI1MjU5LTRlM2ZlMDRmLTg0NjctNDA1Zi05ZjdiLTU3MGNhOWQ5ZGZkMS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcyMVQwODQ0MjdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0zYzEzMzk3OWM5ZTVlYTg2MWIwMzNkOThkMGEyMGViZDhiZmUzODA0ZWRjZGQ2NDljYmM2NDZiMWU5MmI2NjdkJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.YqbMSmsdmewwxwxUAyPiH4tYTdIbQzMfkwYbZApj2jQ)](https://private-user-images.githubusercontent.com/113500771/468525259-4e3fe04f-8467-405f-9f7b-570ca9d9dfd1.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTMwODc3NjcsIm5iZiI6MTc1MzA4NzQ2NywicGF0aCI6Ii8xMTM1MDA3NzEvNDY4NTI1MjU5LTRlM2ZlMDRmLTg0NjctNDA1Zi05ZjdiLTU3MGNhOWQ5ZGZkMS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcyMVQwODQ0MjdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0zYzEzMzk3OWM5ZTVlYTg2MWIwMzNkOThkMGEyMGViZDhiZmUzODA0ZWRjZGQ2NDljYmM2NDZiMWU5MmI2NjdkJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.YqbMSmsdmewwxwxUAyPiH4tYTdIbQzMfkwYbZApj2jQ)

해결책:

* 각 인스턴스는 **고유한 Consumer Group ID**를 갖도록 설정
  * 응답에 대한 요청이 없을때 처리하지 않고 넘어가는 이점
* `sharedReplyTopic = true` 설정 시
  * 요청이 누락된 응답에 대해 `DEBUG` 로그만 남기고 무시
* Consumer Group ID 는 UUID를 활용
  * 중복을 최대한 제거할 수 있었음

💡 _Consumer Group ID란?_

Kafka에서 메시지를 처리하는 컨슈머 그룹을 식별하는 ID입니다. 각 그룹은 메시지를 중복 없이 처리합니다.

***

### 응답 메시지 발행 정리

### Spring Kafka의 `@SendTo` 활용

* 요청 메시지를 수신한 `카프카 리스너에 @SendTo 추가`하기
* `카프카 헤더`에 직접 Reply Topic 지정 / [@sendto](https://github.com/sendto) value 지정
* ListenerContainer 구성시 `응답 메시지를 돌려줄 ReplyTemplate` 정의

#### 위 구조 : 아래의 파란색 눈금에 관련된 응답 메시지 발행

[![Image](https://private-user-images.githubusercontent.com/113500771/468525158-41748021-d74a-4f88-9a64-d89b1ba919b0.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTMwODc3NjcsIm5iZiI6MTc1MzA4NzQ2NywicGF0aCI6Ii8xMTM1MDA3NzEvNDY4NTI1MTU4LTQxNzQ4MDIxLWQ3NGEtNGY4OC05YTY0LWQ4OWIxYmE5MTliMC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcyMVQwODQ0MjdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1mMjkxMTQyZTdjYmQxNTE1NDE2N2UxNTZlYTMwYTBiNTk3OGQ3NjVhNzAzZDFlMWFlZWQ1ZTQyM2QzNDE2NjM4JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.SV1nxjT0-XKWbPn47nWxy-99RMpd35cf6ASUsv0hEFc)](https://private-user-images.githubusercontent.com/113500771/468525158-41748021-d74a-4f88-9a64-d89b1ba919b0.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTMwODc3NjcsIm5iZiI6MTc1MzA4NzQ2NywicGF0aCI6Ii8xMTM1MDA3NzEvNDY4NTI1MTU4LTQxNzQ4MDIxLWQ3NGEtNGY4OC05YTY0LWQ4OWIxYmE5MTliMC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwNzIxJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDcyMVQwODQ0MjdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1mMjkxMTQyZTdjYmQxNTE1NDE2N2UxNTZlYTMwYTBiNTk3OGQ3NjVhNzAzZDFlMWFlZWQ1ZTQyM2QzNDE2NjM4JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.SV1nxjT0-XKWbPn47nWxy-99RMpd35cf6ASUsv0hEFc)

***

### ⚖️ 분산 락 vs Kafka Request-Reply 비교

| 항목    | 분산 락           | Kafka + Request-Reply    |
| ----- | -------------- | ------------------------ |
| 처리 방식 | 동기적 (blocking) | 비동기 (non-blocking)       |
| 순서 보장 | 어렵고 직접 구현 필요   | Kafka가 자동 보장             |
| 확장성   | 락 경합 ↑, 성능 ↓   | 파티션/브로커 수평 확장 가능         |
| 결과 응답 | 별도 로직 필요       | CompletableFuture로 간편 수신 |

***

### 🏁 기존 구조 vs 개선 구조

***

### 🏁 기존 구조 vs 개선 구조

#### 🧱 기존 구조

* 각 요청이 서버에서 DB를 직접 수정
* 락 충돌/데이터 꼬임 빈번
* 응답 결과 없음

#### 🚀 개선 구조 (Kafka 기반)

* 모든 요청을 Kafka에 전송
* 순서 보장 + 병렬 처리 가능
* 결과는 Reply-Topic을 통해 응답
* 응답 수신 → 클라이언트에 정확히 전달

***

### ✨ 실무 인사이트 요약

* Kafka의 **Key-Partition 구조**는 동시성 문제와 순서 보장에 매우 유리
* `ReplyingKafkaTemplate`을 이용한 Request-Reply 패턴은 **비동기 결과 확인**에 최적
* 단일 토픽 운영 전략 + Consumer Group 분리 + sharedReplyTopic 설정은 **확장성과 안정성**을 동시에 확보하는 키

***

### ✅ 마무리하며

이번 사례는 단순 코드 개선이 아니라, 시스템 아키텍처를 **이벤트 기반 메시지 처리 구조로 전환**한 대표적인 예입니다.

> “장애는 코드가 아니라 아키텍처로 막는다.”

Kafka + Request-Reply 패턴은 다른 실시간 시스템에서도 충분히 적용할 수 있습니다. 유사한 장애 상황이 있다면 꼭 이 구조를 고려해보시길 추천드립니다.







### 느낀점: 확장성과 안정성은 얻었지만, 과연 최선이었을까?

이번 구조는 **Kafka의 파티셔닝과 Request-Reply 패턴을 조합하여 동시성과 순서를 해결했다는 점에서는 매우 인상적**이었습니다. 특히 Kafka의 특성을 잘 활용해 분산락이나 DB 중심의 동기 처리보다 훨씬 확장 가능한 시스템을 만들었다는 점은 분명히 배울 점이었습니다.

하지만 개인적으로는 이 구조가 **약간의 오버엔지니어링**이라고 느껴졌습니다. 그 이유는 다음과 같습니다:

***

#### 모든 컨슈머 그룹이 리플라이 토픽을 계속 **pull**한다는 점

현재 구조에서는 `sharedReplyTopic = true` 설정을 통해, \*\*모든 컨슈머 그룹이 같은 reply topic을 구독(poll)\*\*하고,\
자신의 요청이 아닌 응답이면 **그냥 무시**하는 방식으로 동작합니다.

이 말은 곧, **N개의 Consumer Group이 모두 같은 메시지를 읽고, 그 중 하나만 처리하고, 나머지는 "내 것 아님"이라며 버린다는 뜻**입니다.

→ **비효율적입니다.**

***

#### 더 나은 구조는 없을까?

Kafka의 구조상 파티션 기반 순서를 보장하기 위해 단일 리플라이 토픽을 유지하는 전략은 이해됩니다.\
하지만, 응답 메시지를 처리하는 측에서 **라우팅이 가능하거나**,\
Kafka 헤더에 들어 있는 `correlationId`나 `replyTo` 필드를 이용해 **명확한 응답 전용 라우팅을 설계**할 수 있었다면\
이러한 **브로드캐스팅-필터링 구조는 피할 수 있지 않았을까** 하는 의문이 듭니다.
