---
description: 'Firebase 구독 관리: 데이터 일관성, 원자성 및 UX 최적화'
---

# Ensuring Data Consistency, Atomicity and UX Optimization (feat.Firebase)

앱 서비스에서는 팔로잉한 작가의 새로운 게시물이 등록되면, 팔로워들에게 알림을 보낸다. 알림을 보내는 방식은 ActionType을 통해 적절한 메시지를 작성한 후, Firebase Cloud Messaging(FCM)을 사용하여 작가를 구독(subscribe) 중인 사용자들에게 메시지를 전송하고 있다. 이를 위해 Firebase 외부 API를 통해 요청을 보내고 있다.

유저가 작가를 팔로잉했을 때, 유저가 작가를 구독하도록 Firebase에 저장해놔야 한다. 하지만 단일 책임 원칙에 의해 FCM의 목적은 메시지를 보내는 것이므로, 구독을 하고 있다는 정보를 데이터베이스에 따로 저장하고 있다.



아래는 처음 간단하게 작성한 코드의 일부분이다.

```java
//구독
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void createSubscription(long userId, String topicName) {
    var user = userApiRepository.findById(userId).orElseThrow(NotFoundException::new);
    var fcmToken = user.getFcmToken();

    var topicSubsOpt = topicSubscriptionRepository.findByTopicNameAndUserId(topicName, userId);

    if(topicSubsOpt.isPresent()) return;

    topicSubscriptionRepository.save(TopicSubscription.of(topicName, userId, fcmToken, true));

    if(fcmToken != null) {
        try {
            FirebaseMessaging.getInstance().subscribeToTopicAsync(List.of(fcmToken), topicName);
        } catch (Exception e) {
            throw new RuntimeException("FCM subscribe error", e);
        }
    }
}

//구독 취소
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void deleteSubscription(long userId, String topicName) {
    var topicSubOpt = topicSubscriptionRepository.findByTopicNameAndUserId(topicName, userId);

    if(topicSubOpt.isEmpty()) return; // 구독 정보가 없으면 return

    var topicSub = topicSubOpt.get();

    topicSubscriptionRepository.deleteByTopicNameAndUserId(topicName, userId);

    if(topicSub.getFcmToken() != null) {
        try {
            FirebaseMessaging.getInstance().unsubscribeFromTopicAsync(List.of(topicSub.getFcmToken()), topicName);
        } catch (Exception e) {
            throw new RuntimeException("FCM unsubscribe error", e);
        }
    }
}
```

처음 이 코드를 작성하고 고민을 많이했었다. <mark style="background-color:yellow;">**DB에 먼저 저장한 후에 Firebase에 요청을 보내야하나? Firebase에 요청을 보낸 후에 DB에 저장해야하나?**</mark>

당시 고민했던 이유는 아래와 같다:

1. **DB에 먼저 저장한 후 Firebase에 실패한 경우**:
   * `RuntimeException`을 던지면서 `@Transactional` 어노테이션으로 인해 롤백이 된다. 이로 인해 **일관성**을 유지할 수 있고, 롤백을 통해 **원자성**을 보장할 수 있다.
2. **Firebase에 먼저 요청을 보낸 후 DB 저장에 실패 경우**:
   * 롤백이 **불가능**하므로, Firebase에 롤백 요청(구독의 경우, 구독 취소)을 한 번 더 보내야 한다. 이는 데이터베이스와 Firebase 간의 일관성을 유지하기 위함이지만, Firebase 롤백 요청의 성공 여부에 따라 일관성이 보장되지 않을 수 있는 단점이 있다.

직관적으로 봤을때는 당연히 1번이 적절한 것 같다. 하지만 요청에 대해서 신뢰성이 부족했다. 최대한 성공 가능성을 높이기 위해서 롤백이 아니라 재시도 로직을 추가했다. 해결방법은 간단했다.

(실패한 요청들을 테이블에 저장하고, 스케줄러를 통해 실패 요청들을 재시도한다.)

```java
@Service
public class RetryService {

    @Autowired
    private SubscriptionFailureRepository subscriptionFailureRepository;

    @Autowired
    private UnsubscriptionFailureRepository unsubscriptionFailureRepository;

    @Scheduled(fixedDelay = 60000) // 1분마다 실행
    public void retrySubscriptionFailures() {
        List<SubscriptionFailure> failures = subscriptionFailureRepository.findAll();

        for (SubscriptionFailure failure : failures) {
            try {
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
            try {
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

**여기서 끝인가? 아니다!**



과연 <mark style="background-color:yellow;">**UX적인 측면으로 봤을때 적절한가?**</mark> 의문이 들었다. 구독을 취소하는 요청의 경우는 로직이 반대로 되야했다. 이유는 아래와 같다:

사용자가 팔로잉을 하는데 알림이 오지 않는 경우보다는 사용자가 팔로잉 취소했는데 알림이 오는 상황이 **훨씬 더 큰 문제**다. 따라서 이런 상황을 피하기 위해 팔로잉 취소의 경우에는 Firebase 요청을 먼저 처리하고, 데이터베이스 저장이 실패할 경우 Firebase 구독을 취소하는 방법이 더 적절할 수 있다.

수정된 코드는 아래와 같다.

```java
// 구독 취소
@Transactional(propagation = Propagation.REQUIRES_NEW)
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



**결론:**

이 프로젝트에서는 Firebase와 데이터베이스 간의 일관성을 유지하고, 사용자 경험(UX)을 최적화하기 위해 신중하게 설계된 구독 및 구독 취소 로직을 구현하였다.

먼저, 구독의 경우 데이터베이스에 정보를 먼저 저장하고 Firebase에 요청을 보내는 방식으로 **일관성과 원자성을 보장**하였다. 이는 트랜잭션 롤백을 통해 데이터베이스와 Firebase 간의 상태 불일치를 방지할 수 있다. 그러나 Firebase 요청이 실패할 가능성도 고려하여, 실패한 요청을 테이블에 저장하고 스케줄러를 통해 재시도하는 로직을 추가하여 **신뢰성을 높였다.**

반면, 구독 취소의 경우에는 UX적인 측면을 고려하여 로직을 반대로 구성하였다. 사용자가 팔로잉을 취소했는데도 알림이 계속 오는 상황을 방지하기 위해, Firebase 요청을 먼저 처리하고 데이터베이스 작업을 나중에 수행하도록 하였다. 만약 데이터베이스 저장이 실패하더라도 Firebase 구독 취소 요청이 성공적으로 처리되도록 하여 사용자에게 부정적인 영향을 최소화하였다.

**결론적으로**, 이와 같은 재시도 로직을 도입함으로써 시스템의 신뢰성과 안정성을 높일 수 있었으며, **만에 하나 일관성이 깨지더라도 UX적인 측면에서 문제를 최소화할 수 있는 방법을 구현하였다.** 이는 사용자 경험을 최우선으로 고려한 설계 결정이었으며, 지속적인 재시도를 통해 최종적으로 모든 요청이 성공할 수 있도록 하였다.



