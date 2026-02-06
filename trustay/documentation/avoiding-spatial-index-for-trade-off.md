---
description: '대량 위치 기반 조회에서 Spatial Index를 쓰지 않은 이유: 정확도 vs 성능 트레이드 오프'
---

# Avoiding spatial index for trade-off

(사내 보안 규정에 따라 실제 서비스·테이블·코드 식별자는 모두 일반화되어 있습니다)

## 문제 배경

거리 기반 대상 추출(예: 반경 10km)은 알림 대상을 식별하기 위한 필터로 사용됩니다.\
이 필터는 "누가 알림을 받을 수 있는가"를 결정합니다.

## 기존 접근 방식

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

#### Spatial Index + MBR + 최종 거리 계산

1. 반경 원을 포함하는 외접 사각형(Minimum Bounding Rectangle) 계산
2. Spatial Index(R-Tree)를 이용해 사각형 범위에 포함되는 좌표를 빠르게 조회
3. 조회된 좌표들에 대해 ST\_Distance\_Sphere 기반 실제 거리 게산으로 반경 내 필터링

이 방식의 장점으로는:

* 공간 인덱스로 후보를 빠르게 좁힐 수 있고,
* 최종 거리 계산으로 정확도 100%를 보장합니다.

하지만, 실제 운영 환경, 특히 대량 배치 처리에서는 예상치 못한 병목이 발생했습니다.



## Spatial Index는 왜 부담이 되었는가

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

#### &#x20;Spatial Index는 "거리"를 모른다.

MySQL(InnoDB)의 Spatial Index는 R-Tree 기반입니다.

* 관리 대상: 좌표가 아닌 MBR
* 거리 개념 없음
* 노드 간 중복 영역 허용



**검색 과정**

1. 검색 영역 생성: 반경 원 또는 감싸는 사각형 (MBR)
2. 루트 노드부터 시작: 검색 영역과 겹치는 MBR 노드들만 탐색
3. 겹치는 MBR 노드들을 찾으면: 그 노드의 자식 노드로 내려감. (계층 구조)
4. 리프 노드 도달: 실제 좌표 후보들 획득
5. 최종 거리 계산: 진짜 반경 내인지 판별



" 이 사각형A가 이 사각형B와 겹치는가? " 를 판단합니다. \
(좌표 밀도가 높은 경우에 하나의 MBR 노드에는 수천\~수만 개의 좌표가 포함되고, 이후 단계에서 수행되는 연산이 문제가 됩니다.



#### Distance 함수가 붙는 순간 Index-Only Scan은 불가능

```sql
SELECT *
FROM user_location
WHERE MBRContains(
  ST_GeomFromText(:mbr),
  location
)
AND ST_Distance_Sphere(location, :center) <= 10000;

```

#### 동작 원리

`MBRContains( ST_GeomFromText(:mbr), location)`

* MBRContains: R-Tree는 루트(최상위) MBR(사각형)들로 구성되어 있습니다.
* 각 MBR들은 내부 노드(MBR)들을 포함하고 있고,
* 계층 아래 쪽에 있는 MBR들은 결국 리프 노드(좌표)들을 가지고 있습니다.
* 전체 좌표들 중 후보들을 빠르게 추리기 위한 필터링 방법입니다.
* **이 과정을 생략하게 되면, 존재하는 모든 좌표들에 대해 거리 계산을 하게 됩니다.**

`ST-Distance_Sphere`

* 거리 계산을 위해 사용되며 기준 좌표와 대상 좌표의 거리를 나타냅니다.
* 내부 MBR이 어떻게 생겼는지 알기가 어렵기 때문에, 이 과정을 생략하면 정확도가 많이 떨어집니다.



1. 좌표와 반지름이 주어졌을때, 해당 원을 포함하는 최소한의 x각형을 구성하고, 좌표들을 만들어냅니다.\
   (예를 들어, 사각형이라면 4개의 좌표들이 생성됩니다.)
2. 4개의 좌표들로, 루트 MBR 들을 순회하면서 도형이 교차하는지 검증하고, 교차하지 않으면 후보에서 제외합니다.
3. 후보 루트 MBR들안에 포함되어있는 내부 MBR들을 같은 방식으로 검증하게됩니다.
4. 마지막 MBR 내부의 좌표들을 가져옵니다.



해당 쿼리의 핵심 문제는:

* ST\_Distance\_Sphere:
  * 삼각함수 기반
  * CPU 연산 비용이 높습니다.
* Spatial Index는:
  * 거리 조건을 최적화하지 못합니다.
  * **후보를 찾은 뒤, 모든 row에 대해 거리 계산 수행**

결과적으로, Spatial Index는 후보를 줄여줄 수는 있지만, 거리 계산 비용 자체를 줄여주지는 못합니다.



#### 예시  EXPLAIN ANALYZE 분석

```sql
EXPLAIN ANALYZE
SELECT *
FROM user_location
WHERE MBRContains(
  ST_GeomFromText(:mbr),
  location
)
AND ST_Distance_Sphere(location, :center) <= 10000;
```

```
-> Filter: (st_distance_sphere(...) <= 10000)
   -> Index range scan on user_location using idx_location_spatial
      rows=120000
```

Spatial Index로 12만건 후보를 추출하게 되고, 이후 12만번 거리를 계산합니다.\
DB CPU 사용률이 급증하고, Batch 작업에 따라 DB 전체 성능이 저하됩니다.



#### 사각형 근사 (Distance 제거)

```sql
EXPLAIN ANALYZE
SELECT *
FROM user_location
WHERE latitude BETWEEN :minLat AND :maxLat
AND longitude BETWEEN :minLng AND :maxLng;
```

```
-> Index range scan using idx_lat_lng
   rows=135000
```

#### 동작 원리

1. 위, 경도 컬럼에 인덱스가 존재하는 상황에서, Between minLat, maxLat을 통해 1차 필터링을 하게됩니다.
2. Between minLng, maxLng을 통해 2차 필터링을 하여 결과를 빠르게 가져옵니다.



차이점:

* **거리 계산 함수 제거**
* CPU 연산이 없습니다.
* 단순 B-Tree Range Scan 발생



#### O(n)이 문제가 된 이유

단건 거리 계산들의 **대상이 너무 많고**, **삼각함수 기반 거리 게산**이 되며, **row-by-row 계산**이 됩니다.\
(이론적으로는 O(n) 이지만, 더 최악인 DB CPU를 직접 소모하는 O(n)입니다.)



#### 트레이드 오프 결정

운영 장애 상황에서 "DB가 안정적으로 죽지 않고 끝까지 처리할 것"에 따라,\
&#xNAN;**"정확도 100%" 보다 "안정적으로 끝나는 처리량"** 으로 트레이드 오프를 결정했습니다.



#### 이에, 선택한 트레이드 오프는 "원" 대신 "사각형"으로 결정(정확도 일부 포기)

* &#x20;실제 거리 단건 계산을 생략하고
* &#x20;사각형(MBR) 범위 기준으로 "반경 근사" 필터링으로 처리하게 됐습니다.
* minLon, maxLon, minLat, maxLat 사용

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

원(Circle) -> 사각형(Rectangle) 근사

* 실제 거리 게산 제거
* Bounding Box 기반 근사(위경도 범위 기반 사각형) 반경 필터링

내부적으로 경계 구간 소수 대상 포함을 허용해도 괜찮다고 논의를 하였습니다.



#### 이 결정에 대한 영향도는,

* 정확도 감소: 원형 반경이 아니라 사각형 이준이므로, 경계 구간에서 "반경 밖"이 포함될 수 있습니다.
* 성능 향상: 단건 거리 계산 제거로 DB CPU 부하를 줄이고, 대량 배치 작업의 안정성 확보



#### 왜 "Spatial Index를 쓰지 말아야 할 때"가 있는가

부적합한 경우:

* 대량 Batch 처리
* 동일 쿼리 반복 실행
* 거리 계산 함수 결합
* CPU 사용량이 높을 때

적합한 경우:

* 실시간 근접 검색
* 소량
* 정확도가 핵심



## R-Tree(Rectangle Tree)란 무엇인가

R-Tree는 공간 객체(spatial object)를 빠르게 찾기 위한 자료구조 입니다.

* B-Tree(Balanced Tree, 균형 트리): 정렬된 1차원 값에 최적화된 균형 트리
* R-Tree: 2차원 이상 영역을 기반으로 탐색하는 트리 구조.



R-Tree의 구조

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

```
[Root MBR]
   |
   +-- [Internal Node MBR]
   |       |
   |       +-- [Leaf Node MBR] -> 좌표들
   |       +-- [Leaf Node MBR] -> 좌표들
   |
   +-- [Internal Node MBR]
           |
           +-- [Leaf Node MBR] -> 좌표들

```

* 모든 사각형은 MBR 노드를 나타냅니다.
* 리프 노드: 실제 좌표(Point)
* 리프 노드 MBR: 해당 리프 노드에 들어 있는 좌표 전체를 감싸는 사각형
* 내부 노드: 여러 리프/하위 노드를 가리킴
* 내부 노드 MBR: 자식 노드들의 MBR을 다시 감싸는 사각형
* 루트 MBR: 트리 전체 영역
* 모든 노드에는 MBR이 있습니다.



#### 리프 노드 MBR

여러 좌표(point)를 감쌈

```
[Leaf Node MBR]
 ├─ (x1, y1)
 ├─ (x2, y2)
 └─ (x3, y3)

```

#### 내부 노드 MBR

여러 하위 노드의 MBR을 감쌈

```
[Internal Node MBR]
 ├─ [Leaf MBR A]
 └─ [Leaf MBR B]
```

#### 검색은 이 계층을 위에서 아래로 내려간다.

**검색 흐름 (반경 검색 예시)**

1. 검색 영역 생성 (원 또는 사각형)
2. **루트 MBR과 겹치는지 확인**
3. 겹치면:
   * 자식 노드들의 MBR과 비교
4. 겹치는 노드만 계속 내려감
5. 리프 노드 도달 → 좌표 후보 확보

```
검색 영역
   ↓
Root MBR
   ↓
Internal MBR (여러 개 가능)
   ↓
Leaf MBR (여러 개 가능)
   ↓
좌표 후보
```

#### 왜 R-Tree는 이런 구조를 선택했는가

R-Tree는:

* 동적 삽입/삭제 지원
* 공간 데이터의 유연한 분포 대응
* 균형 유지

를 위해:

* 영역을 **딱 맞게 나누지 않는다**
* overlap을 허용한다

**정확한 분할보다 유연성 우선**
