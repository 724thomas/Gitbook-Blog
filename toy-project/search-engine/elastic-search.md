# Elastic Search

## Elasticsearch란?

Lucene 기반의 분산 검색 엔진으로, 데이터를 인덱스에 저장하고, 각 인덱스는 primary shard와 replica로 나뉘어 분산됩니다.

<details>

<summary>각 인덱스는 primary shard와 replica로 나뉘어 분산된다</summary>

Elasticsearch의 `index`는 논리적인 이름이고, 실제 저장은 하나 이상의 **shard**로 쪼개집니다. 각 shard는 Lucene index 하나에 해당하며, 문서는 정확히 하나의 **primary shard**에 소속됩니다. **replica shard**는 그 primary의 복제본입니다. 이렇게 쪼개서 여러 노드에 배치하면, 저장 용량과 검색 부하를 분산할 수 있습니다.

쓰기 요청은 먼저 primary shard가 처리하고, 그 결과를 replica들에게 복제합니다. 반면 읽기 요청은 primary 또는 replica 어느 쪽이든 처리할 수 있어서, replica는 **장애 복구**뿐 아니라 **읽기 처리량 증가**에도 기여합니다. Elasticsearch는 기본적으로 읽기 시 active shard copy를 선택해 요청을 보냅니다.

쉽게 말하면, primary는 “원본 담당”, replica는 “복사본 담당”입니다. 원본은 쓰기를 책임지고, 복사본은 장애 시 대체와 읽기 분산을 돕습니다.

</details>

\
문서가 저장되면 near real-time(즉시 아님)으로 검색이 가능해지며, 일반적인 검색 가시성은 1초 미만입니다.\
ElasticSearch는 트랜잭션 처리의 RDBMS가 아닌, 검색과 필터링과 집계를 빠르게 수행하기 위한 별도 검색 계층에 가깝습니다.

<details>

<summary>near real-time 방식으로 검색 가능</summary>

Elasticsearch는 문서를 저장했다고 해서 **즉시** 검색 가능한 것은 아닙니다. 공식 문서는 이를 **near real-time search**라고 부르며, 일반적으로 문서가 저장된 뒤 **약 1초 이내** 검색 가능해진다고 설명합니다. 이 차이는 Lucene이 데이터를 segment 단위로 다루고, 검색 가능 상태로 여는 **refresh** 과정을 거치기 때문입니다.

즉, DB의 commit 직후 곧바로 조회되는 느낌과는 약간 다릅니다. Elasticsearch에서는 “쓰기 성공”과 “검색에 보이기 시작함” 사이에 refresh 지연이 있을 수 있습니다. 기본 refresh는 보통 1초 주기이며, refresh 자체는 리소스를 쓰는 작업이라 너무 자주 강제하면 성능에 부담이 생깁니다.

그래서 Elasticsearch를 붙일 때는 “최신성” 요구를 꼭 확인해야 합니다. 예를 들어 사용자의 위치가 방금 바뀌었는데, 그 반영이 검색에 몇백 ms\~1초 정도 늦게 보여도 되는지부터 판단해야 합니다. 이게 MySQL과 Elasticsearch를 함께 쓸 때 자주 생기는 설계 포인트입니다.

</details>

이게 중요한 이유는, 기존 병목이 "정확한 거리 계산을 OLTP DB가 떠안고 있었다" 라는 데 있었습니다. Elasticsearch를 도입하면 위치 기반 조회를 메인 DB에서 분리해 별도 검색 계층으로 넘길 수 있습니다. 구조적으로는 DB의 트랜잭션 부하와 검색 부하를 분리하는 방향입니다.

<details>

<summary>OLTP</summary>

OLTP는 **Online Transaction Processing**의 약자입니다. 쉽게 말하면, 많은 사용자가 동시에 짧고 빠른 트랜잭션을 계속 발생시키는 **운영계 데이터 처리**입니다. 온라인 뱅킹, 주문, 결제, 메시지, 사용자 상태 업데이트 같은 작업이 여기에 가깝습니다.

지금 글의 맥락에서 보면, **유저 위치 업데이트**, **알림 대상 추출**, **푸시 발송 대상 기록** 같은 건 전형적인 OLTP 성격입니다. 반대로 “지역별 유저 분포를 장기적으로 분석한다” 같은 건 OLAP 쪽에 더 가깝습니다. 즉, 이 글에서 메인 DB는 검색 엔진이라기보다 **운영 트랜잭션을 안정적으로 처리하는 OLTP 저장소**라고 이해하면 됩니다.

</details>



## Elasticsearch의 공간 검색은 어떻게 동작하는가

Elasticsearch는 geo\_point와 geo\_shape를 기본 타입으로 제공합니다.

* geo\_point는 위경도 점 하나를 저장할때 사용하고, bounding box, distance, polygon 기반 검색과 거리 집계, grid 집계에 활용할 수 있습니다.
* geo\_shape는 polygon, rectangle, line 같은 임의의 도형을 저장할 때 사용합니다.

내부적으로 Lucece의 point 계열 데이터는 일반 텍스트 inverted index가 아니라 KD-tree 계열 구조에 인덱싱되며, range, distance, nearest-neighbor, point-in-polygon 같은 연산에 최적화되어 있습니다.\
특히 Elasticsearch의 geo\_shape는 도형을 삼각형 mesh로 분해한 뒤, 각 triangle을 7차원 point로 BKD tree에 저장합니다.

<details>

<summary>KD-tree, BKD tree</summary>

**KD-tree**는 k-dimensional tree의 약자로, 2차원·3차원·그 이상 **다차원 점(point)** 을 빠르게 찾기 위한 자료구조입니다. Lucene 문서도 일반 텍스트용 inverted index와 달리, 숫자/점 데이터는 KD-tree 같은 구조로 인덱싱된다고 설명합니다.

**BKD tree**는 Lucene의 **Block KD-tree** 구현입니다. Lucene 패키지 문서에는 아예 “Block KD-tree”라고 적혀 있고, Elastic/Lucene 설명에 따르면 이 구조는 디스크 I/O에 효율적이도록 설계되어 있습니다. 핵심 아이디어는 N차원 공간을 더 작은 직사각형 셀들로 재귀적으로 쪼개고, leaf block 단위로 묶어 저장하는 것입니다.

비유하면 KD-tree는 “공간을 계속 반으로 잘라 가며 탐색 범위를 줄이는 트리”이고, BKD tree는 그 방식을 **검색 엔진의 디스크 저장 방식에 맞게 최적화한 Lucene 버전**이라고 이해하면 됩니다. Elasticsearch의 geo와 numeric range가 빠른 이유 중 하나가 바로 이 계열 자료구조를 쓰기 때문입니다.

</details>

<details>

<summary>range, distance, nearest-neighbor, point-in-polygon이 뭐고 이 연산에 어떻게 최적화되어 있는가</summary>

각 용어를 먼저 아주 짧게 풀면 이렇습니다. **range**는 “값이 이 범위 안에 있나?”, **distance**는 “기준점에서 반경 N km 안에 있나?”, **nearest-neighbor**는 “가장 가까운 k개가 누구냐?”, **point-in-polygon**은 “이 점이 어떤 다각형 내부에 있나?”입니다. Lucene은 point data structure가 이런 연산에 최적화되어 있다고 설명합니다.

왜 빠르냐면, BKD tree가 공간을 작은 셀들로 쪼개 놓기 때문입니다. 검색 시에는 “이 쿼리 도형이 왼쪽 subtree와 겹치나? 오른쪽 subtree와 겹치나?”만 먼저 보고, **겹치지 않는 큰 영역 전체를 통째로 건너뜁니다**. Leaf cell이 쿼리 안에 완전히 들어오면, 그 셀 내부의 점들은 개별 정밀 검사 없이 한 번에 수집할 수 있고, 경계에 걸친 셀만 개별 point 검사로 내려갑니다.

즉, 최적화의 핵심은 “모든 점을 다 검사하지 않는다”는 데 있습니다. 먼저 **셀 단위로 대량 가지치기(pruning)** 를 하고, 꼭 필요한 leaf나 경계 부분만 정밀 검사를 합니다. 이게 풀스캔이나 row-by-row 거리 계산과 가장 큰 차이입니다.

</details>

즉, geo\_shape는 "도형을 잘게 쪼개서 검색 가능한 벡터 형태로 인덱싱"하는 방식으로 동작합니다.

<details>

<summary>도형을 잘게 쪼개서 검색 가능한 벡터 형태로 인덱싱"한다는 게 무슨 뜻인가</summary>

"geo\_shape는 도형을 삼각형 메쉬로 분해한 뒤, 이를 BKD tree가 탐색할 수 있는 다차원 point 표현으로 인덱싱한다.\
여기서 말하는 "벡터"는 임베딩 벡터가 아니라, 검색 엔진 내부의 다차원 좌표 표현에 가깝다."

이 표현은 정확히 말하면 “AI 임베딩 벡터” 같은 뜻이 아니라, **도형을 다차원 점 표현으로 바꿔서 BKD tree에 넣는다**는 뜻에 가깝습니다. Elastic의 geo\_shape 관련 설명과 공식 GitHub 이슈를 보면, `geo_shape`는 도형을 **triangular mesh**로 분해하고, 각 triangle을 **7차원 point**로 인덱싱한다고 되어 있습니다.

예를 들어 복잡한 행정구역 polygon이 있으면, 그 polygon 전체를 통째로 저장해서 매번 “이 도형과 저 도형이 겹치냐”를 brute-force로 계산하는 게 아니라, 먼저 삼각형 조각들로 나눈 뒤 그 조각들을 검색 가능한 point 구조로 저장하는 방식입니다. 그래서 쿼리 시에도 BKD tree의 가지치기 혜택을 받을 수 있습니다.

</details>

위 구조 덕분에 Elasticsearch는 지리 검색을 1급 기능으로 제공하고, MYSQL에서 ST\_Distance\_Sphere()같은 함수를 써서 반경 계산을 직접 수행하는 방식과 달리, ElasticSearch는 애초에 검색 엔진 관점에서 geo\_query를 위한 자료구조를 가지고 시작한다. MYSQL의 ST\_Distance\_Sphere()는 구면 거리 계산 함수이며, 좌표가 geographic SRS의 point/MultiPoint일때 최단 거리를 계산합니다.

<details>

<summary>이 구조 덕분에 Elasticsearch는 지리 검색을 1급 기능으로 제공한다”가 왜 성립하는가</summary>

Elasticsearch가 지리 데이터를 “문자열을 억지로 검색하는 식”이 아니라 **전용 field type + 전용 index structure + 전용 query DSL**로 다루기 때문입니다. 공식 문서만 봐도 `geo_point`, `geo_shape`, `geo_distance`, `geo_shape query`, geo sort, geo aggregation이 모두 별도 기능으로 제공됩니다.

즉, 지리 검색은 “사용자가 함수 몇 개를 조합해서 우연히 만든 기능”이 아니라, 엔진이 처음부터 지원하는 핵심 검색 기능입니다. 그래서 저는 “1급 기능”이라고 표현한 것입니다. 엄밀히 말하면 공식 용어는 아니고, **native support가 매우 강하다**는 뜻의 실무적 표현입니다.

MySQL에서 `ST_Distance_Sphere()`를 매 row 계산하는 느낌과 달리, Elasticsearch는 애초에 위치 데이터를 위한 field와 쿼리와 정렬과 집계를 따로 준비해 둔 셈입니다. 앞에서 설명한 BKD 기반 구조가 바로 그 기술적 토대입니다.

</details>



## 현재 문제에 적용한다면,

{% content-ref url="../../trustay/documentation/avoiding-spatial-index-for-trade-off.md" %}
[avoiding-spatial-index-for-trade-off.md](../../trustay/documentation/avoiding-spatial-index-for-trade-off.md)
{% endcontent-ref %}

Elasticsearch를 적용할 수 있는 방법은 크게 두가지 입니다.

### 유저 문서를 직접 geo 검색하는 방식

각 user document에 location: geo\_point를 넣고, 질문글이 등록되면 geo\_distance filter로 반경 내 유저를 찾습니다.\
여기에 pushEnabled, isActive, blocked 같은 조건을 bool.filter에 함께 넣으면 점수 계산 없이 필터링 중심으로 동작합니다.

<details>

<summary>점수 계산(score calculation)이라는 게 뭐냐</summary>

Elasticsearch는 기본적으로 검색 결과를 **relevance score**, 즉 `_score` 순으로 정렬합니다. 이 점수는 “이 문서가 이 검색어와 얼마나 잘 맞는가”를 나타내는 숫자입니다. 공식 문서도 query context에서는 단순히 맞는다/틀리다만 보는 게 아니라, **얼마나 잘 맞는지**를 계산한다고 설명합니다.

예를 들어 검색어가 “질문글”일 때, 제목과 본문에 모두 들어 있는 글이 본문에만 한 번 들어 있는 글보다 더 높은 `_score`를 받을 수 있습니다. 반면 `filter`는 “조건에 맞느냐”만 보고, 점수에는 관여하지 않습니다. 그래서 상태값, 기간, 차단 여부 같은 구조적 조건은 대개 `filter`에 둡니다.

거리도 점수에 섞을 수 있습니다. 공식 문서의 `distance_feature` query는 기준 위치에 가까울수록 relevance score를 더 높여 주며, 계산식도 `boost * pivot / (pivot + distance)` 형태로 공개되어 있습니다. 즉, “가까운 글이 더 관련 있어 보이게” 만들 수 있습니다.

예시는 이렇습니다.

```json
GET posts/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "질문글" } }
      ],
      "should": [
        {
          "distance_feature": {
            "field": "location",
            "origin": [127.0276, 37.4979],
            "pivot": "1km"
          }
        }
      ],
      "filter": [
        { "term": { "status": "PUBLISHED" } }
      ]
    }
  }
}
```

이 경우 텍스트가 기본 relevance를 만들고, 가까운 위치의 글은 추가 점수를 얻습니다. 이것이 “거리와 텍스트를 한 순위 체계 안에서 합친다”는 뜻입니다.

</details>

### 현재의 하이브리드 전략을 Elasticsearch에 옮기는 방식

region 인덱스에는 geo\_shape로 지역 경계를 저장하고, users 인덱스에는 region\_id를 중복 저장합니다.&#x20;

먼저 질문글의 검색 범위와 교차하는 region들을 geo\_shape query로 찾고,\
그 다음 terms query로 해당 region\_id를 가진 user들을 조회합니다.

geo\_shape query는 intersects, within, contains, disjoint 같은 공간 관계를 지원하고,\
terms query는 여러 exact value를 한번에 조회할 수 있습니다.

<details>

<summary>geo_shape query의 intersects / within / contains / disjoint, 그리고 terms query는 어떻게 동작하는가</summary>

`geo_shape` query는 `geo_shape` field뿐 아니라 `geo_point` field에도 적용할 수 있습니다. 그리고 relation은 다음 뜻입니다. **INTERSECTS**는 조금이라도 겹치면 매치, **WITHIN**은 field 값이 query geometry 내부에 있으면 매치, **CONTAINS**는 field 값이 query geometry를 포함하면 매치, **DISJOINT**는 공통 부분이 전혀 없으면 매치입니다.

예를 들어 region polygon이 저장돼 있고, 내가 만든 검색 박스(envelope)가 있을 때 `relation: "intersects"`를 쓰면 “이 박스와 조금이라도 겹치는 region들”을 찾는 것입니다. 반대로 `within`은 “이 region이 검색 도형 안에 완전히 들어오느냐”에 더 가깝습니다.

`terms` query는 훨씬 단순합니다. 특정 field에 대해 **여러 exact value 후보 중 하나라도 정확히 일치**하면 매치합니다. 공식 문서도 값 배열을 넘기면 그중 하나 이상이 field 값과 정확히 일치해야 문서를 반환한다고 설명합니다. 그래서 `regionId` 목록을 먼저 구한 다음, 그 목록으로 유저를 조회할 때 자주 씁니다.

예시는 이런 식입니다.

```json
{
  "query": {
    "terms": {
      "regionId": ["11680101", "11680103", "11680105"]
    }
  }
}
```

이 쿼리는 `regionId`가 저 세 값 중 하나인 문서들을 exact match로 가져옵니다.

</details>



현재 상황에서는 Ealsticsearch를 쓰더라도 direct geo search 보다는 region 하이브리드가 더 현실적이라고 생각합니다.\
이유는, 이기능의 본질은 "정확한 원형 반경 검색" 보다 "근처 동네 사람을 빠뜨리지 않게 찾는 것"에 가깝습니다. Elasticsearch를 도입한다고 해서, 기존의 정확한 원형 반경 검색으로 회귀할 필요는 없다고 생각합니다.



## 코드로 구현한다면,

```json
PUT users_v1
{
  "mappings": {
    "properties": {
      "userId":       { "type": "keyword" },
      "regionId":     { "type": "keyword" },
      "location":     { "type": "geo_point" },
      "pushEnabled":  { "type": "boolean" },
      "isActive":     { "type": "boolean" },
      "updatedAt":    { "type": "date" }
    }
  }
}
```

```json
POST users_v1/_search
{
  "size": 1000,
  "_source": ["userId"],
  "query": {
    "bool": {
      "filter": [
        { "term": { "pushEnabled": true } },
        { "term": { "isActive": true } },
        {
          "geo_distance": { // 반경 필터링
            "distance": "5km",
            "location": {
              "lat": 37.4979,
              "lon": 127.0276
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "_geo_distance": { // 정렬
        "location": {
          "lat": 37.4979,
          "lon": 127.0276
        },
        "order": "asc",
        "unit": "m",
        "distance_type": "arc" // arc 또는 plane
      }
    },
    { "userId": "asc" }
  ]
}
```

* geo\_distance는 반경 필터링이고,
* \_geo\_distance는 정렬입니다.

정렬시 distance\_type은 arc 또는 plane 을 사용할 수 있습니다.\
(공식 문서상 plane은 더 빠르지만, 장거리나 극지방 근처에서는 부정확할 수 있음 -> arc가 안전)

<details>

<summary>geo_distance, _geo_distance, distance_type, arc, plane</summary>

`geo_distance`는 **필터/쿼리**입니다. “기준점에서 반경 N km 안에 있는 문서만 남겨라”라는 뜻입니다. 반면 `_geo_distance`는 **정렬(sort)** 입니다. “이미 매치된 문서들을 기준점으로부터 가까운 순 또는 먼 순으로 정렬해라”라는 뜻입니다. 둘은 이름은 비슷하지만 역할이 다릅니다.

`distance_type`은 거리를 어떤 방식으로 계산할지 정하는 옵션입니다. Elastic 공식 문서는 `arc`를 기본값으로 두고, `plane`은 더 빠르지만 **장거리**이거나 **극지방 근처**에서는 부정확할 수 있다고 설명합니다. 즉, `arc`는 지구 곡률을 반영하는 더 안전한 방식이고, `plane`은 평면 근사입니다.

현재 사례처럼 서울 주변의 상대적으로 짧은 거리에서는 `plane`도 실용상 충분할 수 있지만, 기본값이 `arc`인 이유는 범용성과 정확성 때문입니다. 블로그에서는 “정확도 논쟁을 피하고 설명도 단순하게 하려면 `arc`를 쓴다” 정도로 정리해도 좋습니다.

참고로 `_geo_distance` 정렬에서는 위치 값이 없는 문서는 거리가 `Infinity`로 취급됩니다. 그래서 sort를 쓸 때는 해당 field가 비어 있는 문서 처리도 같이 생각해야 합니다.

</details>

<details>

<summary>plane은 더 빠르지만, 장거리나 극지방 근처에서는 부정확할 수 있음 → arc가 안전”을 더 쉽게 설명하면</summary>

`plane`은 지구를 거의 평평한 판처럼 보고 거리 계산을 단순화하는 방식입니다. 그래서 계산은 빠르지만, 지구가 둥글다는 사실을 충분히 반영하지 못합니다. 거리가 짧고 위도 변화가 크지 않은 도시권 검색에서는 큰 문제가 없을 수 있지만, 범위가 넓어지거나 극지방에 가까워질수록 오차가 커질 수 있습니다.

`arc`는 지구 곡면을 따라 거리를 계산하는 방식이라 더 일반적으로 안전합니다. 공식 문서가 `arc`를 default로 두는 이유도 그 때문입니다. 실무에서 특별한 이유가 없으면 `arc`를 먼저 쓰고, 아주 짧은 거리에서만 성능을 이유로 `plane`을 검토하는 편이 설명하기 쉽습니다.

</details>

대상이 많아서 한번에 모두 가져올 수 없다면, Elasticsearch에서는 scroll보다 search\_after + PIT 가 현재 권장 방식입니다. (다만 트레이드 오프 존재. PIT은 오래된 segment를 계속 살려두기 때문에 disk, file handle, heap 사용량이 늘어납니다)

<details>

<summary>scroll, search_after, PIT, 그리고 PIT이 old segment를 살려두는 이유</summary>

`scroll`은 대량 데이터를 한 번에 끝까지 훑는 배치성 API입니다. 공식 문서는 scroll이 **실시간 사용자 요청용이 아니라**, reindex 같은 대량 처리용이라고 설명합니다. scroll은 처음 검색 시점의 결과 집합 상태를 유지하기 위해 search context를 열어 두고, 다음 batch를 `_scroll_id`로 이어 받아옵니다.

`search_after`는 Elasticsearch판 **cursor paging**에 더 가깝습니다. 이전 페이지의 마지막 문서의 sort 값을 넘겨서 다음 페이지를 가져옵니다. `from + size`는 10,000건 이후 deep pagination에 불리해서, 공식 문서는 그 이후엔 `search_after`를 쓰라고 안내합니다. 그리고 페이지 사이에 refresh가 끼면 순서가 바뀔 수 있으므로, **일관된 스냅샷**이 필요하면 PIT와 함께 쓰라고 설명합니다.

PIT(Point In Time)는 “이 시점의 인덱스 상태를 고정해서 보겠다”는 뷰입니다. 문제는 이렇게 과거 시점의 검색을 계속 이어가려면, merge가 끝나도 예전 segment를 바로 지울 수 없다는 점입니다. Elastic 공식 문서는 open PIT가 오래된 segment 삭제를 막아서 **disk와 file handle 사용량**이 늘고, 삭제/업데이트가 많은 인덱스에서는 해당 시점의 live/deleted 상태를 추적해야 하므로 **heap 사용량**도 늘 수 있다고 설명합니다.

정리하면, scroll은 “배치 전체 순회”, `search_after + PIT`는 “깊은 페이지네이션 + 일관된 보기”에 가깝습니다. 둘 다 내부적으로 검색 상태를 붙잡고 있기 때문에 공짜가 아닙니다.

</details>

하이브리드 방식은 아래처럼 바뀔 수 있습니다.

```json
PUT regions_v1
{
  "mappings": {
    "properties": {
      "regionId": { "type": "keyword" },
      "boundary": { "type": "geo_shape" }
    }
  }
}
```

```json
POST regions_v1/_search
{
  "size": 100,
  "_source": ["regionId"],
  "query": {
    "geo_shape": {
      "boundary": {
        "shape": {
          "type": "envelope",
          "coordinates": [
            [126.97, 37.53],
            [127.05, 37.46]
          ]
        },
        "relation": "intersects"
      }
    }
  }
}
```

이 방식이 더 중요한 이유는 Elasticsearch도 Join형 모델링을 좋아하지 않기 때문입니다.\
(join을 피하고, nested는 몇배, parent-child는 훨씬 느려짐)\
그래서 region과 user를 런타임에 join하는 대신, region\_id를 user document에 같이 저장하는 비정규화 모델이 더 적절합니다.



## Elasticsearch의 장점

가장 큰 장점은 검색 부하를 트랜잭션 DB에서 떼어낼 수 있다는 점입니다. 지금 처럼 "쓰기 source of truth는 DB에 두고, 조회/검색은 별도 검색 계층으로 넘기는 구조"를 만들 수 있습니다.\
그리고 geo\_point, geo\_shape, distance query, geo sorting, grid aggregation을 한 플랫폼 안에서 제공하므로, 단순 알림 대상 조회를 넘어 지도 탐색, 주변 검색, heatmap, 지역 통계까지 자연스럽게 확장할 수 있습니다.

<details>

<summary>geo_point, geo_shape, distance query, geo sorting, grid aggregation</summary>

`geo_point`는 위도/경도 한 점입니다. 공식 문서 기준으로 bounding box 검색, distance 검색, `geo_shape` query 안의 point-in-polygon, distance 기반 집계, grid 집계, 거리 정렬, 거리 기반 relevance에 활용할 수 있습니다. 즉, “사람/가게/유저 현재 위치”처럼 점 데이터에 적합합니다.

`geo_shape`는 point보다 더 복잡한 선·면·다각형 같은 도형입니다. 지역 경계, 서비스 가능 구역, 행정구역 polygon처럼 “영역 자체”를 저장할 때 씁니다. `geo_shape` query는 이런 도형들 사이의 `intersects`, `within`, `contains`, `disjoint` 관계를 검색합니다.

`distance query`는 “반경 N km 안에 있는 문서만 남기는 조건”, `geo sorting`은 “가까운 순으로 정렬”, `grid aggregation`은 “지도를 격자로 쪼개서 각 칸에 몇 개가 있는지 집계”입니다. 특히 `geotile_grid`는 `zoom/x/y` 형태의 지도 타일 단위 bucket을 만들어서, 지도 클러스터나 히트맵 비슷한 UI를 만들 때 유용합니다.

현재 글의 문제는 “알림 대상 ID 추출”이라 grid aggregation까지는 필요 없지만, 나중에 “지역별 질문글 밀집도 시각화” 같은 기능이 붙으면 이런 집계가 자연스럽게 후보가 됩니다.

</details>

또한 검색 조건을 조합하기 쉽습니다. 반경 필터, 상태 필터, 차단 여부, 마지막 활동 시간, 텍스트 검색까지 하나의 bool query 안에서 합칠 수 있습니다.\
필터 중심 쿼리는 점수 계산 없이 동작하므로 현재 상황 같은 "대상 추출" 문제와 잘 맞습니다.

<details>

<summary>bool query로 텍스트 + 거리 + 상태 + 차단 여부 + 마지막 활동 시간 등을 한 번에 조합하는 법</summary>

Elasticsearch의 `bool` query는 여러 조건을 한 요청 안에서 합치는 기본 도구입니다. 공식 문서 기준으로 `must`와 `should`는 점수 계산에 참여하고, `filter`와 `must_not`은 점수에 영향을 주지 않으면서 포함/제외 조건을 담당합니다. 즉, “텍스트 relevance”와 “구조적 필터링”을 한 DSL 안에서 함께 다룰 수 있습니다.

예를 들어 “강남역 근처 질문글” 검색은 아래처럼 만들 수 있습니다.

```json
GET posts/_search
{
  "track_scores": true,
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "질문글"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "status": "PUBLISHED"
          }
        },
        {
          "range": {
            "createdAt": {
              "gte": "now-7d"
            }
          }
        },
        {
          "geo_distance": {
            "distance": "3km",
            "location": {
              "lat": 37.4979,
              "lon": 127.0276
            }
          }
        }
      ],
      "must_not": [
        {
          "terms": {
            "authorId": ["blockedUser1", "blockedUser2"]
          }
        }
      ]
    }
  },
  "sort": [
    { "_score": "desc" },
    {
      "_geo_distance": {
        "location": {
          "lat": 37.4979,
          "lon": 127.0276
        },
        "order": "asc",
        "unit": "m",
        "distance_type": "arc"
      }
    }
  ]
}
```

이 쿼리의 의미는 이렇습니다. `must.match`는 제목 텍스트 relevance를 만들고, `filter.term/range/geo_distance`는 상태·최근성·거리 조건으로 후보를 줄이며, `must_not.terms`는 차단된 작성자를 제외합니다. 그리고 sort에서 `_score`를 먼저, `_geo_distance`를 다음에 두면 “관련도 우선, 가까운 글 우선” 같은 UX를 만들 수 있습니다. 필드 sort를 쓰면 기본적으로 score를 계산하지 않기 때문에, score도 같이 보고 싶다면 `track_scores: true`가 필요합니다.

반대로 현재 글의 “알림 대상 추출”은 보통 텍스트 relevance가 필요 없습니다. 그래서 아래처럼 전부 filter 위주로 가는 편이 더 자연스럽습니다.

```json
GET users/_search
{
  "_source": ["userId"],
  "query": {
    "bool": {
      "filter": [
        {
          "terms": {
            "regionId": ["11680101", "11680103", "11680105"]
          }
        },
        {
          "term": {
            "pushEnabled": true
          }
        },
        {
          "term": {
            "isActive": true
          }
        }
      ]
    }
  }
}
```

이건 “누가 더 관련 있는가”가 아니라 “누가 대상인가”가 핵심이기 때문입니다. 그래서 검색 엔진의 relevance보다, 빠른 후보 추출이 더 중요합니다. 이 부분이 사용자 검색 UI와 배치성 알림 추출의 차이입니다.

</details>



## Elasticsearch의 단점

Elasticsearch는 별도 분산 시스템이기 떄문에 shard 수, replica 수, query 형태, index 설계, filesystem cache, 하드웨어, refresh 정책까지 운영 포인트가 많습니다. 도입한다고 빨라지는 도구는 아닙니다.\
(검색 성능이 shard 전략, 쿼리 비용, 병렬 검색 수, 캐시, 하드웨어의 영향을 크게 받습니다)

또한 Elasticsearch 검색은 near real-time입니다.\
문서가 저장되더라도 보통 1초 이내에 searchable 해질 뿐, DB commit 직후 즉시 검색에 반영된다고 가정하기에는 위험합니다.\
위치 업데이트가 잦은 서비스에서는 이 특성의 의미가 큽니다. (다만, 현재 기능에서는 유저 위치 업데이트가 빈번하진 않아서 고려 대상이 아닙니다) 그러기에 "쓰기 성능"과 "검색 최신성" 사이에 또 다른 trade-off가 생깁니다.



## 왜 도입하지 않았는가

현재 기능에서는 Elasticsearch가 기술적으로는 가능하지만, 비용 대비 이득이 크지 않다고 판단합니다.

이미 MySQL 안에서 연산 대상을 User에서 Region으로 옮기고, 동적인 User는 region\_id 기반 B-Tree 조회로 바꾸면서 핵심 병목을 제거했습니다.

문제는 "정확한 원형 검색 엔진이 없어서"라기보다는 "동적인 대규모 유저 테이블에 비싼 공간 연산을 걸고 있는 상황"에 가깝습니다.

이 상태에서 Elasticsearch를 추가로 도입하면, 검색 인프라 자체보다 동기화 파이프라인과 운영 복잡도가 더 큰 비용이 됩니다. (DB와 ES간 이중 저장 또는 CDC, 인덱스 버전 관리, 재색인, refresh 정책, 장애 복구, 샤드 운영까지)\
현재 기능은 근사치 허용이 가능했고, full-text relevance나 지도 탐색 UI가 핵심도 아니었기 떄문에 단순하게 가져가는 것이 더 적절합니다.

즉, Elasticsearch를 도입하지 않은 이유는 Elasticsearch가 부족해서가 아닌, 현재 문제를 해결하는 데 필요한 최소 복잡도를 이미 기존 DB 구조 안에서 달성했기 때문입니다.



## 언제 도입하면 좋을까

만약, 아래와 같은 요구사항들이 추가된다면 고려할 수 있습니다.

* 특정 위치(강남역) 근처 질문글 같은 텍스트+거리+필터+정렬을 동시에 처리하거나
* 지도 위에서 주변 유저를 탐색하고 cluster/heatmap/grid 같은 공간 집계가 필요할 때
* 메인 DB를 source of truth로 두고, 검색 트래픽을 별도 계층으로 완전히 분리해야하는 상황이거나
* 행정 구역(region\_id) 기반이 아닌, 원형/다각형 기반 탐색이 일반적일때

<details>

<summary>강남역 근처 질문글처럼 텍스트 + 거리 + 필터 + 정렬을 동시에 처리한다”는 게 지금 알림 대상 추출과 어떻게 다른가</summary>

이건 **문제 종류가 다릅니다**. 현재 글의 문제는 “질문글이 올라왔을 때 알림을 보낼 유저 ID를 빠르게 뽑아라”입니다. 즉, 결과의 **순위**보다 **대상 여부**가 중요합니다. 보통은 score가 필요 없고, 정확한 필터링과 대량 추출이 중요합니다. 이건 candidate generation에 가깝습니다.

반면 “강남역 근처 질문글”은 **사용자 검색 UX**에 가깝습니다. 여기서는 “질문글”이라는 텍스트 relevance, “강남역 근처”라는 거리 조건, “삭제되지 않음/공개 상태/최근 7일” 같은 필터, “더 관련 있는 글 먼저 또는 더 가까운 글 먼저” 같은 정렬이 동시에 필요할 수 있습니다. 그래서 Elasticsearch의 `match + geo_distance + filter + sort` 조합이 특히 잘 맞습니다.

즉, 현재 알림 문제는 “누구에게 보내야 하는가”이고, 검색 문제는 “무엇을 먼저 보여줘야 하는가”입니다. Elasticsearch는 후자에서 특히 빛나고, 지금 블로그의 하이브리드 전략은 전자에 더 잘 맞는 선택이라고 정리하면 자연스럽습니다.

</details>
