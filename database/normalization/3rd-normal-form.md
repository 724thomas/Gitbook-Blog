---
description: 3차 정규화
---

# 3rd Normal Form

3NF는 테이블이 2NF를 만족하고, 모든 비주요 속성이 기본 키에만 종속되는 것을 목표로 합니다. 즉, 어떤 비주요 속성도 다른 비주요 속성에 종속되지 않아야 합니다. 이는 '이행적 종속성'을 제거함으로써 달성됩니다.

**예시:**

* '주문' 테이블에서 '고객ID'에 의해 '배송 주소'가 결정되고, '고객ID'는 '고객 이름'에 종속될 때, '고객 이름'과 '배송 주소'는 서로 이행적 종속 관계에 있으므로, '고객' 테이블로 분리해야 합니다.