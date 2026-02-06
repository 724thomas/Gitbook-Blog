# Documentation

#### 알림 푸시 시스템 진화 기록

{% content-ref url="overall-evolution-of-notification-and-push-delivery-system.md" %}
[overall-evolution-of-notification-and-push-delivery-system.md](overall-evolution-of-notification-and-push-delivery-system.md)
{% endcontent-ref %}

#### 푸시 추적성 강화:  다중 대상 푸시를 1Row = 1Target 구조로 재설계하여 에러 추적 용이함과 성능 예측 향상

{% content-ref url="designing-a-traceable-push-delivery-system.md" %}
[designing-a-traceable-push-delivery-system.md](designing-a-traceable-push-delivery-system.md)
{% endcontent-ref %}

#### 도메인 정합성 확보:  알림 수신 주체를 유저/채널 -> 채널 기준으로 통일하여 알림 주체 혼란 제거

{% content-ref url="eliminating-domain-confusion-in-a-notification-system.md" %}
[eliminating-domain-confusion-in-a-notification-system.md](eliminating-domain-confusion-in-a-notification-system.md)
{% endcontent-ref %}

#### 중복 발송 이슈 해결 - Redis 기반 Rate Limit 도입

{% content-ref url="resolving-duplicate-notification-bursts-with-redis-based-rate-limiting.md" %}
[resolving-duplicate-notification-bursts-with-redis-based-rate-limiting.md](resolving-duplicate-notification-bursts-with-redis-based-rate-limiting.md)
{% endcontent-ref %}

#### 대량 알림 생성 파이프라인 재설계

{% content-ref url="overall-redesigning-a-bulk-notification-creation-pipeline-with-an-outbox-based-structure.md" %}
[overall-redesigning-a-bulk-notification-creation-pipeline-with-an-outbox-based-structure.md](overall-redesigning-a-bulk-notification-creation-pipeline-with-an-outbox-based-structure.md)
{% endcontent-ref %}

#### 대량 위치 기반 조회에서 Spatial Index를 쓰지 않은 이유: 정확도 vs 성능 트레이드 오프

{% content-ref url="avoiding-spatial-index-for-trade-off.md" %}
[avoiding-spatial-index-for-trade-off.md](avoiding-spatial-index-for-trade-off.md)
{% endcontent-ref %}
