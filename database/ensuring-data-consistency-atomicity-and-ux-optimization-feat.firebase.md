---
description: 'Firebase 구독 관리: 데이터 일관성, 원자성 및 UX 최적화'
---

# Ensuring Data Consistency, Atomicity and UX Optimization (feat.Firebase)

## 개요

팔로우한 작가의 새 게시물을 푸시 알림으로 받기 위해, 사용자는 Firebase Cloud Messaging(FCM)을 통해 작가를 구독(subscribe)합니다. 이 과정에서 사용자 행동을 DB에 반영하는 동시에 FCM에도 요청을 보내야 하는데, 이 두 작업은 하나의 트랜잭션처럼 항상 함께 성공하거나 실패해야 이상적인 사용자 경험(UX)을 보장할 수 있습니다.

서비스 운영 초반에는 장애나 문제 없이 잘 동작하고 있었지만, 외부 API인 FCM 요청이 실패하거나 네트워크 장애가 발생할 경우,

* **구독은 되어 있는데 알림이 오지 않거나**,
* **구독을 취소했는데 알림이 계속 오는** 문제가 발생할 수 있음을 사전에 인지했습니다.

이러한 문제 가능성을 방지하고자 다음과 같은 구조적 개선을 진행했습니다:

* FCM 요청 실패 시를 대비한 **트랜잭션 처리 순서의 재설계**
* **재시도 로직** 도입과 그로 인한 **스케줄러 과부하 리스크 분석**
* 재시도 대상을 구분하고, **구독은 기록만, 구독 취소는 재시도**로 UX 신뢰성 최적화
* 불가항력적 장애(Firebase 문제 등)에 대비한 **타임아웃 설정과 예외 대응 전략 수립**

단순히 실패를 막기 위한 처리가 아닌, “**실패 가능성이 있어 보이는 상황에 어떻게 구조적으로 대응할 것인가**”를 고민한 결과였습니다. 실제 문제는 발생하지 않았지만, **예방적 리팩토링과 설계 판단**을 통해 사용자 경험의 신뢰도를 미리 확보할 수 있었습니다.

***

## 간단한 로직 구현

아래는 처음 간단하게 작성한 코드의 일부분입니다.

{% code fullWidth="true" %}
```java
//구독
@Transactional
public void createSubscription(long userId, String topicName) {
    /* 흐름 시각화
    1. 유저 정보를 조회힙니다.
    └─ 2.존재하지 않으면 Exception을 터트립니다.
    3.유저의 FCM토큰을 가져옵니다.
    4.구독 정보를 조회합니다.
    └─ 5. 이미 구독 중이면 종료합니다.
    6. DB에 구독 정보를 저장합니다.
    7. FCM 토큰이 존재하는지 확인합니다.
    └─ 8. FCM에 구독 요청을 보냅니다.
       ├─ 9. 실패하면 Exception을 터트립니다.
       └─ 10. DB에 반영되지 않고 롤백됩니다.
    */
    var user = userApiRepository.findById(userId).orElseThrow(NotFoundException::new);
    var fcmToken = user.getFcmToken();

    var topicSubsOpt = topicSubscriptionRepository.findByTopicNameAndUserId(topicName, userId);
    if(topicSubsOpt.isPresent()) return;

    topicSubscriptionRepository.save(TopicSubscription.of(topicName, userId, fcmToken, true));

    if(fcmToken != null) {
        try {
            FirebaseMessaging.getInstance().subscribeToTopicAsync(List.of(fcmToken), topicName).get();
        } catch (Exception e) {
            throw new RuntimeException("FCM subscribe error", e);
        }
    }
    
    
}

//구독 취소
@Transactional
public void deleteSubscription(long userId, String topicName) {
    /* 흐름 시각화
    1.구독 정보를 조회합니다.
    └─ 2. 구독 중이지 않으면 종료합니다.
    3. DB에 구독 정보를 삭제합니다.
    4. FCM 토큰을 확인합니다.
    ├─ 5. 존재하면 FCM에 구독 취소 요청을 보냅니다.
    └─ 6. 존재하지 않으면 Exception을 터트립니다.
       └─ 7. DB에 반영되지 않고 롤백됩니다.
    */
    var topicSubOpt = topicSubscriptionRepository.findByTopicNameAndUserId(topicName, userId);
    if(topicSubOpt.isEmpty()) return;
    var topicSub = topicSubOpt.get();

    topicSubscriptionRepository.deleteByTopicNameAndUserId(topicName, userId);

    if(topicSub.getFcmToken() != null) {
        try {
            FirebaseMessaging.getInstance().unsubscribeFromTopicAsync(List.of(topicSub.getFcmToken()), topicName).get();
        } catch (Exception e) {
            throw new RuntimeException("FCM unsubscribe error", e);
        }
    }
}
```
{% endcode %}

해당 코드를 작성할때, DB에 먼저 반영을 해야할지, FCM 요청을 먼저 보내야할지 고민을 했고, DB에 저장 로직이 먼저되도록 했습니다. <mark style="color:yellow;">그 이유는 FCM에 먼저 요청을 보내게 됐을때, 문제가 발생하여 롤백시 복잡해지기 때문입니다.</mark>

**DB -> FCM 순서:**

* DB 요청 실패 시:
  * API 실패가 되고, DB와 FCM의 일관성은 보장
* FCM 요청 실패 시:
  * `RuntimeException`을 던지면서 `@Transactional` 어노테이션으로 인해 롤백이 된다. 이로 인해 **일관성**을 유지할 수 있고, 롤백을 통해 **원자성**을 보장할 수 있습니다.

**FCM -> DB 순서:**

* FCM요청이 실패 시:
  * API 실패가 되고, DB와 FCM의 일관성은 보장
* DB 요청 실패 시:
  * API 실패가 되고, DB에 반영이 되지 않지만 FCM에는 반영이됨
  * 롤백이 **불가능**하므로, Firebase에 취소 요청(구독의 경우, 구독 취소)을 한 번 더 보내야 합니다. 이는 데이터베이스와 Firebase 간의 일관성을 유지하기 위함이지만, Firebase 취소 요청의 성공 여부에 따라 항상 일관성이 보장되지 않을 수 있는 단점이 있습니다.

***

## 재시도 로직

이후에 성공 가능성을 높이기 위해서 롤백이 아니라 재시도 로직을 추가했습니다. 실패한 요청들을 실패 테이블에 저장하였고, 스케줄러를 통해 실패 요청들을 재시도하였습니다.

{% code fullWidth="true" %}
```java
@RequiredArgsConstructor
@Service
public class RetryService {

    private final SubscriptionFailureRepository subscriptionFailureRepository;
    private final UnsubscriptionFailureRepository unsubscriptionFailureRepository;

    @Scheduled(fixedDelay = 60000) // 1분마다 실행
    public void retrySubscriptionFailures() {
        List<SubscriptionFailure> failures = subscriptionFailureRepository.findAll();

        for (SubscriptionFailure failure : failures) {
            try { // 비동기지만 블로킹
                FirebaseMessaging.getInstance().subscribeToTopicAsync(List.of(failure.getFcmToken()), failure.getTopicName()).get();
                subscriptionFailureRepository.delete(failure);
            } catch (Exception e) {
                // 로그 기록 및 다음 재시도 대기
                log.error("Failed to retry subscription: " + failure.getTopicName() + " for user: " + failure.getUserId(), e);
            }
        }
    }

    @Scheduled(fixedDelay = 60000) // 1분마다 실행
    public void retryUnsubscriptionFailures() {
        List<UnsubscriptionFailure> failures = unsubscriptionFailureRepository.findAll();

        for (UnsubscriptionFailure failure : failures) {
            try { // 비동기지만 블로킹
                FirebaseMessaging.getInstance().unsubscribeFromTopicAsync(List.of(failure.getFcmToken()), failure.getTopicName()).get();
                unsubscriptionFailureRepository.delete(failure);
            } catch (Exception e) {
                // 로그 기록 및 다음 재시도 대기
                log.error("Failed to retry unsubscription: " + failure.getTopicName() + " for user: " + failure.getUserId(), e);
            }
        }
    }
}

```
{% endcode %}

**재시도 로직을 도입하여 실패한 요청들에 대해 계속해서 FCM 요청을 보내게됩니다. 장단점은 아래와 같습니다.**

장점:

* 구독/구독취소 기능이 항상(다른 문제가 없다면) 성공합니다.
* 알림이 안오더라도, 유저의 action은 성공함으로 서비스 신뢰도를 높일 수 있습니다.

단점:

* 재시도 로직이 성공할때까지, DB와 FCM의 일관성이 깨집니다.
* FCM 또는 네트워크 문제 발생시, 요청들이 쌓이게 됩니다. 내부적으로 요청들은 블로킹 호출이고, 스케줄러는 1분 간격으로 실행되는데, 1분안에 요청들이 처리가 안되면 다음과 같은 문제점들이 발생합니다.
  * 1분이라는 간격이 의미가 없어집니다.
  * **요청들이 너무 많이 쌓이면 해당 재시도 로직 메서드가 pool의 모든 스레드를 할당받게 되며 스레드가 고갈됩니다.**
  * **다른 작업들이 늦게 실행되거나, 큐를 넘어서는 작업들은 오류가 발생합니다(RejectedExecutionException).**



그러면 재시도 FCM요청들을 비동기로 하면 해결이 되는지 고민을 해봤습니다만, 새로운 문제들이 발생할 수 있어서 골치아파집니다.

* **비동기 작업은 ExecutorService의 스레드에서 실행됩니다. 결국 비동기 스레드 풀도 고갈될 수 있습니다.**
* **이미 요청을 보냈다고 간주하고, 성공했는지는 알 수 없습니다. 이 문제를 해결하기 위해서는 다시 또 DB에 반영해야합니다. 이때 Race Condition이 발생할 수 있습니다.**



그러면 어떻게 해결해야할까? 고민을 했고, 다르게 접근해봤습니다.

**"재시도 로직이 실행시간이 늘어나더라도, 결국 1분안에 실행이 되면 문제가 없을 거다. 그러면 재시도 로직에 실패 요청들이 최대한 덜 포함되도록 할 수 있지 않을까?"**

재시도 로직이 꼭 필요한 경우와, 그렇지 않은 경우를 나눠서 처리하면 가능하겠다고 생각했습니다. 몇 가지 가능성이 있는 경우들로 나눠봤습니다.

* 재시도 횟수를 제한한다.
* 중요한 요청만 재시도를 한다.



첫번째로, **재시도 횟수를 제한**하면 어떤 결과가 나올지를 고민해봤습니다.

* 주기적으로 개발자가 모니터링을 하고, 직접 처리를 한다.
* 모니터링은 최종 실패한 요청들에 대해 매일 특정시간에 알림을 받습니다.

서비스 비정상 종료보단 훨씬 나은 방향이라고 생각했습니다.



다음으로, **중요한 요청만 재시도**를 하면, 어떤 결과가 나올지를 고민해봤습니다.

* api를 실패처리하는 방법 -> 서비스 신뢰도 하락
* api가 성공한것처럼 처리하고 재시도는 하지않는 방법 -> **서비스 신뢰도 하락 최소화**

***

## 유저 속이기(api가 성공한것처럼 처리하기)

**성공한것처럼 보이기 위해서는, 유저가 보기에는 성공했지만, 실제로는 실패 / 재시도 로직 대기 중으로 처리하면 되겠다고 생각했습니다.** 결과적으로 어떤 결과가 발생할 수 있을지 총 4가지 상황을 나열했습니다.

1. DB에만 반영하고 FCM에 재시도 요청을 보내지 않는다.
   1. "구독을 했었고 구독이 되어있는데 왜 알림이 안오지?" -> "유저 활동이 없나?"
   2. "구독 취소를 했었고 구독 취소되어있는데 왜 알림이 계속 오지?" -> <mark style="color:red;">**"서비스가 왜 이래?"**</mark>
2. DB에 반영하지 않고 FCM에 재시도 요청을 보내지 않는다.
   1. "구독을 했었는데 왜 구독이 안되있지?" -> "내가 구독을 안했었나?"
   2. 구독 취소를 했었는데 왜 구독 취소가 안되어 있지?" -> "내가 구독 취소를 안했었나?"



구독의 경우에는 크게 문제가 되지 않지만, 구독취소의 1b 상황에서는 서비스 신뢰도가 크게 하락할 수 있습니다. 차라리 2b 상황이 훨씬 나아 보입니다.

그러면 구독 취소의 경우, FCM이 실패하면 DB도 실패하게끔 설계를 하였습니다. 해결 방법은 단순히 로직 실행 순서를 역전시키는 것입니다.

{% code fullWidth="true" %}
```java
// 구독 취소
@Transactional
public void deleteSubscription(long userId, String topicName) {
    var topicSubOpt = topicSubscriptionRepository.findByTopicNameAndUserId(topicName, userId);

    if(topicSubOpt.isEmpty()) return; // 구독 정보가 없으면 return

    var topicSub = topicSubOpt.get();

    if(topicSub.getFcmToken() != null) {
        try {
            // Firebase 구독 취소 요청을 먼저 처리
            FirebaseMessaging.getInstance().unsubscribeFromTopicAsync(List.of(topicSub.getFcmToken()), topicName).get();
        } catch (Exception e) {
            // 구독 취소 실패 시, 실패 테이블에 저장
            unsubscriptionFailureRepository.save(new UnsubscriptionFailure(userId, topicName, topicSub.getFcmToken()));
            throw new RuntimeException("FCM unsubscribe error", e);
        }
    }

    // 데이터베이스 트랜잭션 처리
    topicSubscriptionRepository.deleteByTopicNameAndUserId(topicName, userId);
}
```
{% endcode %}

이제 구독취소 결과를 다시 정리해보면, 결과적으로 **동일한 상황이 발생하도록 유도**할 수 있게 됩니다.

* FCM 요청 실패 또는 DB 요청 실패시:
  1. 구독 취소를 했었는데 왜 구독 취소가 안되어 있지?" -> "내가 구독 취소를 안했었나?"
* DB 요청 실패시:
  1. 구독 취소를 했었는데 왜 구독 취소가 안되어 있지?" -> "내가 구독 취소를 안했었나?"



다만, 로직을 봤을때, 실패한 요청에 대해서는 재시도를 하게 되고, 이때 FCM 요청과 DB 반영이 같이 되도록 해야합니다.

전체적으로 봤을때,

* 재시도 요청을 최소화한다.
  * 구독 취소 재시도 횟수 제한
  * 구독의 경우 재시도 하지 않음
  * DB와 FCM의 일관성이 깨지더라도:
    * "구독을 했었고 구독이 되어있는데 왜 알림이 안오지?" -> "유저 활동이 없나?"로 판단.
* 구독 취소의 경우 항상 원자적으로 처리되게 한다.
  * 요청이 실패하더라도:
    * 구독 취소를 했었는데 왜 구독 취소가 안되어 있지?" -> "내가 구독 취소를 안했었나?" 로 판단.

***

## 불가항력적인 문제점

**재시도 로직의 문제와 UX 문제를 최소화하게 됐습니다. 하지만, FCM자체에 문제가 발생할 가능성도 생각해봤습니다.**

FCM 자체의 문제 또는 네트워크 문제 같은 불가항력 문제가 발생했을 때, 요청을 계속 기다리게 됩니다. 쓰레드가 계속 할당되는 상태가 되기때문에 **timeout을 명시적으로 설정함으로서 스레드를 반납할 수 있도록 하였습니다.**

스레드를 반납했지만, 구독취소 로직의 순서를 바꿨기때문에, 처리가 늦게라도 되는 경우 결과를 알 수 없기 때문에 문제가 발생할 수 있습니다.&#x20;

* 늦었지만 결국 성공 처리.
* 늦었지만 실패 처리.

늦었지만 성공 처리되는 경우:

* 구독하려는 상황
  * DB처리가 되고(구독이됨), FCM도 결국 처리가 됨(알림이 옴).
* 구독 취소하는 상황
  * DB처리가 되지 않고(구독취소가 안됨), FCM은 결국 처리가 됨(알림이 안옴).

늦었지만 실패 처리되는 경우:

* 구독하려는 상황
  * DB처리가 되고(구독이됨), FCM은 처리가 안됨(알림이 안옴).
* 구독 취소하는 상황
  * DB처리가 되지 않고(구독취소가 안됨), FCM도 결국 처리가 안됨(알림이 옴).

다행히도, 로직을 바꿔서 구독이 안되어 있는데 알림이 오는 상황은 오지 않았습니다.

***

## 결론

이 프로젝트에서는 Firebase와 데이터베이스 간의 일관성을 유지하고, 사용자 경험(UX)을 최적화하기 위해 신중하게 설계된 구독 및 구독 취소 로직을 구현하였습니다.

이로서, 정리하자면:

* 재시도 스캐줄링 간격보다 오래 실행됨으로서 발생할 수 있는 문제:
  * 1분이라는 스케줄링 간격의 의미가 없어짐.
  * 요청들이 쌓여서 스레드 고갈
  * 다른 작업들에 영향
  * **구독 요청의 경우, 재시도 로직을 수행하지 않고, 별도로 기록하여 개발자가 직접 처리하고,**
  * **구독 취소 요청의 경우 재시도 횟수를 제한하여 스케줄링 수행 시간을 최소화.**
* 구독이 안되어있는데 알림이 오는 UX 문제, 제3의 서비스(FCM)의 문제 및 네트워크 문제 같은 불가항력 상황:
  * 구독 취소의 경우 해당 문제가 발생하고, 유저의 서비스 신뢰도 하락.
  * **DB반영->FCM요청 로직의 순서를 FCM요청->DB반영을 하게끔 수정하여 서비스 신뢰도 하락 최소화.**



## 느낀점

"머리가 좋으면 몸이 편하다" 라는 말에는 반대로 "몸이 좋으면 머리가 편하다" 라는 얘기도 있습니다. 이 문제를 기술적으로 해결할 수 있는 방법이 있음에도 불구하고, 생각을 달리하여, 조금 더 편한 길로 갔다는 것은 편법을 쓴것 같은 느낌입니다. 만약 기술적으로 능력이 뛰어났다면, 더욱 쉽고 빠르게 해결할 수 있는 문제였지 않을까 죄책감이 들기도 했습니다. 만약, 언젠가 비슷한 문제를 맞딱뜨린다면, 그때는 기술적으로 능숙하여 쉽게 문제를 해결할 수 있으면 좋겠다는 바람입니다.
