# Elastic Search

## Elasticsearch란?

Lucene 기반의 분산 검색 엔진으로, 데이터를 인덱스에 저장하고, 각 인덱스는 primary shard와 replica로 나뉘어 분산됩니다.\
문서가 저장되면 near real-time(즉시 아님)으로 검색이 가능해지며, 일반적인 검색 가시성은 1초 미만입니다.\
ElasticSearch는 트랜잭션 처리의 RDBMS가 아닌, 검색과 필터링과 집계를 빠르게 수행하기 위한 별도 검색 계층에 가깝습니다.

이게 중요한 이유는, 기존 병목이 "정확한 거리 계산을 OLTP DB가 떠안고 있었다" 라는 데 있었습니다. Elasticsearch를 도입하면 위치 기반 조회를 메인 DB에서 분리해 별도 검색 계층으로 넘길 수 있습니다. 구조적으로는 DB의 트랜잭션 부하와 검색 부하를 분리하는 방향입니다.



## Elasticsearch의 공간 검색은 어떻게 동작하는가

Elasticsearch는 geo\_point와 geo\_shape를 기본 타입으로 제공합니다.

* geo\_point는 위경도 점 하나를 저장할때 사용하고, bounding box, distance, polygon 기반 검색과 거리 집계, grid 집계에 활용할 수 있습니다.
* geo\_shape는 polygon, rectangle, line 같은 임의의 도형을 저장할 때 사용합니다.

내부적으로 Lucece의 point 계열 데이터는 일반 텍스트 inverted index가 아니라 KD-tree 계열 구조에 인덱싱되며, range, distance, nearest-neighbor, point-in-polygon 같은 연산에 최적화되어 있습니다.\
특히 Elasticsearch의 geo\_shape는 도형을 삼각형 mesh로 분해한 뒤, 각 triangle을 7차원 point로 BKD tree에 저장합니다.

즉, geo\_shape는 "도형을 잘게 쪼개서 검색 가능한 벡터 형태로 인덱싱"하는 방식으로 동작합니다.

위 구조 덕분에 Elasticsearch는 지리 검색을 1급 기능으로 제공하고, MYSQL에서 ST\_Distance\_Sphere()같은 함수를 써서 반경 계산을 직접 수행하는 방식과 달리, ElasticSearch는 애초에 검색 엔진 관점에서 geo\_query를 위한 자료구조를 가지고 시작한다. MYSQL의 ST\_Distance\_Sphere()는 구면 거리 계산 함수이며, 좌표가 geographic SRS의 point/MultiPoint일때 최단 거리를 계산합니다.



## 현재 문제에 적용한다면,

{% content-ref url="../../trustay/documentation/avoiding-spatial-index-for-trade-off.md" %}
[avoiding-spatial-index-for-trade-off.md](../../trustay/documentation/avoiding-spatial-index-for-trade-off.md)
{% endcontent-ref %}

Elasticsearch를 적용할 수 있는 방법은 크게 두가지 입니다.

### 유저 문서를 직접 geo 검색하는 방식

각 user document에 location: geo\_point를 넣고, 질문글이 등록되면 geo\_distance filter로 반경 내 유저를 찾습니다.\
여기에 pushEnabled, isActive, blocked 같은 조건을 bool.filter에 함께 넣으면 점수 계산 없이 필터링 중심으로 동작합니다.

### 현재의 하이브리드 전략을 Elasticsearch에 옮기는 방식

region 인덱스에는 geo\_shape로 지역 경계를 저장하고, users 인덱스에는 region\_id를 중복 저장합니다.&#x20;

먼저 질문글의 검색 범위와 교차하는 region들을 geo\_shape query로 찾고,\
그 다음 terms query로 해당 region\_id를 가진 user들을 조회합니다.

geo\_shape query는 intersects, within, contains, disjoint 같은 공간 관계를 지원하고,\
terms query는 여러 exact value를 한번에 조회할 수 있습니다.



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

대상이 많아서 한번에 모두 가져올 수 없다면, Elasticsearch에서는 scroll보다 search\_after + PIT 가 현재 권장 방식입니다. (다만 트레이드 오프 존재. PIT은 오래된 segment를 계속 살려두기 때문에 disk, file handle, heap 사용량이 늘어납니다)

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

