# Cache

## 1.개념

자주 접근하는 데이터를 빠른 장치(보통 메모리)에 임시로 저장해 두고 필요할 때마다 즉시 참조할 수 있도록 하는 기법 또는 저장 공간을 가리킵니다. 쓰레드 캐시, JPA 영속성 컨텍스트, 브라우저 캐시, CDN, ETag 등.

캐시가 중요한 이유는 처리 속도이고, 상대적으로 느린 장치에 저장된 데이터를 가져오지 않고, 한번 읽어온 데이터를 미리 저장해 두고 재사용함으로써 응답 지연을 줄이고 처리량을 향상 시킵니다.



## 2. 캐시의 필요성

* 성능 향상: 응답시간 단축으로 UX 향상 및 더 많은 트래픽 수용
* 비용 절감: db 또는 api 호출횟수를 줄임
* 확장성: 트래픽 증가시 서버 추가보다 캐시를 통해 서버의 효율을 높이는 것이 더 경제적이고 빠른 방법



## 3. 캐시 쓰기전략

### 3.1. Write-Through 캐시

데이터를 DB에 쓰는 순간 캐시도 갱신

<figure><img src="/broken/files/YPWpYj1bihHqwvLv8WFQ" alt=""><figcaption></figcaption></figure>

* 캐시와 DB간의 데이터 정합성이 좋습니다
* 쓰기 연산이 느려짐

### 3.2. Write-Around 캐시

DB에 먼저 데이터를 기록하고, 캐시에는 기록하지 않습니다. 데이터가 읽힐때 캐시에 저장

<figure><img src="/broken/files/3p1FCCrC8qpaiA3raJhD" alt=""><figcaption></figcaption></figure>

* 모든 데이터는 DB에 저장하고 캐시를 갱신하지 않음
* 캐시 미스가 발생하는 경우에만 DB와 캐시에도 데이터를 저장

### 3.3. Write-Back 캐시

캐시가 우선적으로 업데이트 되고, 특정 주기에 따라 DB에 기록

<figure><img src="/broken/files/wmuLpEJKBF9FhQwR43Jw" alt=""><figcaption></figcaption></figure>

* 쓰기 속도가 매우 빠름
* DB에 반영이 되지 않은 상태에서 문제가 발생하거나 데이터 유실 가능성



## 4. 캐시 읽기 전략&#x20;

### 4.1. Look-Aside

데이터를 찾을때 우선 캐시에 저장되어 있는지 먼저 확인. 캐시에 데이터가 없으면 DB조회

<figure><img src="/broken/files/hwbgmO1V45J13WKTPHXV" alt=""><figcaption></figcaption></figure>

### 4.2. Read-Through

캐시에서만 데이터를 읽어오는 전략.

<figure><img src="/broken/files/LZ0RYvN921HAzx5kRIE7" alt=""><figcaption></figcaption></figure>



## 5. 캐시 전략 조합

### 5.1. Look Aside + Write Around 조합(일반적)

<figure><img src="/broken/files/iI7vetUF7ES4qBI9TiwK" alt=""><figcaption></figcaption></figure>

* 항상 DB에 씁니다
* 캐시에서 읽을때 항상 DB에서 먼저 읽어옴.

### 5.2. Read Through + Write Through 조합

<figure><img src="/broken/files/MTAsXJQ3rhvgG2lAIw5R" alt=""><figcaption></figcaption></figure>

* 데이터를 쓸때 항상 캐시에 먼저 씀.
* 데이터를 쓸때 항상 캐시에서 DB로 보내기때문에 정합성 보장.



## 6. 캐시 만료 정책

* LRU: 가장 오랫동안 사용되지 않은 항목 부터 제거
* LFU: 특정 기간 동안 사용 횟수가 가장 적은 항목부터 제거
* FIFO: 가장 먼저 들어온 항목부터 제거
* TTL: 일정 시간이 지나면 해당 데이터를 만료 처리하는 방식
