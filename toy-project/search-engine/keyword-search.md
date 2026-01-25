---
description: 키워드 검색
---

# Keyword Search

## 키워드 검색

키워드 검색은 사용자가 문자열을 문서에서 찾아 관려도가 높은 순서로 반환하는 검색.

* 문서 전체를 매번 훒지 않고
* 미리 만들어준 역색인(Inverted Index)를 통해
* "이 단어(토큰)이 들어간 문서 ID 목록"을 빠르게 찾고
* 여러 단어의 조합(AND/OR), 필드(제목/본문) 가중치, 랭킹(BM25 등)을 적용하여
* 최종 결과를 정렬하여 반환



## 관련 기술과 원리 (Lucene / ElasticSearch / OpenSearch)

### 전체 파이프라인

#### A. 색인 파이프라인

1. 문서 입력(Json, DB row 등)
2. 필드별 분석 (Analyzer: 토큰화/정규화/형태소/스테링)
3. 토큰들을 역색인 구조로 기록
4. 세그먼트(segment) 단위로 디스크에 저장
5. 주기적으로 merge(compaction)하여 성능 유지

#### B. 질의(Query) 파이프라인

1. 사용자 쿼리 입력
2. 쿼리 분석(동일 Analyzer 또는 Search-time Analyzer)
3. 토큰별 posting list(문서ID 목록) 조회
4. Boolean 조합(AND/OR), 필터 적용
5. 랭킹(BM25 등)으로 점수 계산
6. Top-K 결과만 추출
7. 하이라이트/집계/페이징 등 후처리



### 역색인이 왜 빠른가?

#### 정방향 색인(Forward) vs 역색인(Inverted)

* 정방향: "문서 -> 단어들"
  * Doc1: \[아이폰, 케이스, ...]
  * Doc2: \[갤럭시, ...]
* 역색인: "단어 -> 문서들"
  * 아이폰 -> \[Doc1, Doc7, Doc20,...]
  * 케이스 -> \[Doc1, Doc3, Doc9,...]

키워드 검색의 대부분은 "특정 단어가 들어간 문서만" 보면 되므로,\
전체 문서를 스캔하지 않고 posting list만 보면 됩니다.

### 역색인의 내부 구조(postings와 dictionary)

Lucene 계열은 크게 3가지를 묶어서 생각하면 정확합니다.

#### 1. Term Dictionary(용어 사전)

* 각 필드마다 "등장한 모든 term(도큰)"을 정렬된 형태로 저장
* 빠른 lookup을 위해 보통 FST(Finite State Transducer) 같은 압축 트라이 구조를 사용
  * 메모리 효율 좋고 prefix 탐색에도 강함
  * "단어가 존재하는가 / 다음 단어는 무엇인가"를 빠르게

#### 2. Posting List(문서 목록)

각 term마다 다음 정보가 붙습니다.

* docID 목록: 이 term이 등장한 문서들
* term frequency: 해당 문서에서 term이 몇번 등장하는지
* positions: 문서 내 위치(phrase query에 사용)
* offsets: 원문에서의 시작/끝(하이라이트에 사용)

"아이폰 케이스" 검색시,\
"아이폰" postings와 "케이스" postings를 가져와서 교집합(AND) / 합집합(OR) 등을 계산합니다

#### 3. Stored fields / Doc values

* Stored Fields: 원분(또는 원문 일부)을 결과로 보여주기 위해 저장
* Docs Values: 정렬/집계/스크립트에 최적화된 columnar 저장(필드별로 column 형태)
  * 예: 가격 정렬, 브랜드별 집계, 최신순 정렬 등

즉,

* “검색(매칭)”은 postings,
* “정렬/집계”는 doc values,
* “표시”는 stored fields\
  이렇게 책임이 갈립니다.

### Analyzer가 검색 품질 결정

키워드 검색은 “문서와 쿼리”가 **같은 언어 체계로 분해**되어야 매칭이 됩니다.

#### 1) Analyzer 구성요소

Analyzer는 보통 3단계로 구성됩니다.

1. Character Filter
   * 입력 텍스트 전처리(HTML 제거, 특정 패턴 치환)
2. Tokenizer
   * 문자열을 토큰 단위로 분리(공백/형태소/표준 토크나이저 등)
3. Token Filter
   * 토큰 정규화(소문자, 스테밍, 불용어 제거, 동의어 확장 등)

#### 2) Search-time vs Index-time

* index-time analyzer: 문서를 넣을 때 어떤 토큰을 색인할지 결정
* search-time analyzer: 사용자의 검색어를 어떤 토큰으로 바꿀지 결정

실무에서 흔한 룰:

* **둘을 동일하게 맞추는 게 기본**
* 단, 자동완성/오타/동의어 등은 search-time에 확장을 더 넣기도 함

#### 3) 한국어가 어려운 이유

한국어는 띄어쓰기/조사/활용 때문에 “토크나이저 선택”이 검색 품질에 압도적 영향을 줍니다.

* 예: “먹었다”를 “먹다”로 normalize할지?
* 예: “카페를”에서 조사를 제거할지?
* 예: “현대백화점”을 “현대/백화점”으로 분해할지?

ES/OpenSearch에서는 보통:

* Nori Analyzer(형태소 기반)를 많이 씁니다.



### 랭킹(Score)은 어떻게 계산되나? (BM25 중심)

“키워드 검색”은 단순히 포함 여부가 아니라\
**얼마나 관련이 높은지**를 계산합니다.

Lucene/ES/OpenSearch 기본 랭킹의 대표가 **BM25** 입니다.

#### 1) TF-IDF 직관

* TF(문서 내 빈도): 문서에서 많이 등장하면 관련 있을 가능성↑
* IDF(희소성): 전체 문서에서 흔한 단어는 중요도↓
  * 예: “상품”, “좋아요” 같은 단어는 너무 흔해서 랭킹에 기여를 줄여야 함

#### 2) BM25가 TF-IDF를 개선한 지점

BM25는 TF를 무한히 키우지 않고 “완만하게 포화”시키고,\
문서 길이가 긴 문서가 유리해지는 문제를 “길이 정규화”로 보정합니다.

(개념적으로)

* tf는 커질수록 점수는 증가하지만 증가폭은 점점 줄어듦
* 문서 길이가 길면(단어가 많으면) 점수를 깎아서 공정하게

#### 3) 필드 가중치(Field boost)

실무에선 보통 필드마다 중요도가 다릅니다.

* title 매칭은 강력
* body 매칭은 그 다음
* tags 매칭은 또 다름

그래서:

* title^3, tags^2, body^1 처럼 필드별 가중치를 줍니다.

### Depth 6. Lucene(라이브러리) vs Elasticsearch/OpenSearch(서버)

#### 1) Lucene

* “검색 엔진 코어 라이브러리”
* 인덱싱/검색/세그먼트/쿼리 실행/스코어링을 제공
* 하지만 네트워크 API, 클러스터링, 샤딩/복제, 운영도구는 없음

#### 2) Elasticsearch / OpenSearch

Lucene 위에 “운영 가능한 분산 검색 서버”를 얹은 것.

* REST API 제공
* 샤드(shard)로 인덱스를 분할 저장
* 레플리카(replica)로 내결함성 확보
* 노드 추가로 수평 확장
* 집계(aggregation), 보안, 모니터링, ILM(수명주기) 등 제공

OpenSearch는 ES 계열에서 파생된 별도 프로젝트로,\
실무 관점에서는 “Lucene 기반 분산 검색 서버”라는 공통점이 핵심입니다.



## 이 기술을 “어떻게 적용”하는가

여기서는 “키워드 검색을 제품에 붙인다”를 기준으로 가장 흔한 접근을 정리합니다.

### 적용 Step 1) 검색 대상 정의

* 문서 타입: 상품 / 게시글 / 장소 / 코드 / 사용자 등
* 각 문서의 필드 정의
  * title, description, tags, category, price, location, createdAt …

### 적용 Step 2) 매핑/분석기 설계 (정확도 80%가 여기서 결정)

* 텍스트 필드에 어떤 analyzer를 쓸지
* 정렬/집계를 위해 어떤 필드를 keyword/doc\_values로 둘지
* 검색 품질 위해 multi-field 구성
  * 예: title은 `text` + `keyword` 둘 다 (검색 + 정렬/정확매칭)

### 적용 Step 3) 색인 파이프라인 구축

* 데이터 원천: DB
* 동기화 전략:
  1. 배치 전체 색인(초기 구축)
  2. CDC/Outbox/이벤트로 증분 업데이트(운영)
* 일관성 전략:
  * “DB가 진실(SOT)” + 검색은 “조회 최적화 캐시/인덱스”로 두는 경우가 많음

### 적용 Step 4) 검색 API 설계

* query string 지원
* 필터(카테고리/가격) 지원
* 정렬(최신/인기) 지원
* pagination (search\_after 권장)
* 응답에는 score + highlight + facets 포함 가능

### 적용 Step 5) 튜닝 & 운영

* 샤드 개수, 레플리카 개수
* refresh interval
* merge 정책
* query cache / request cache
* slowlog / profile로 병목 추적



## 예시 코드

### Lucene 예시 (Java) — 인덱싱 + 검색

의존성: lucene-core, lucene-analyzers-common 등

```java
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.*;
import org.apache.lucene.index.*;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.*;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.RAMDirectory;

public class LuceneKeywordSearchDemo {

    public static void main(String[] args) throws Exception {
        Directory dir = new RAMDirectory();
        Analyzer analyzer = new StandardAnalyzer();

        // 1) IndexWriter: 색인 생성
        IndexWriterConfig config = new IndexWriterConfig(analyzer);
        IndexWriter writer = new IndexWriter(dir, config);

        addDoc(writer, "1", "아이폰 13 케이스", "정품 실리콘 케이스", 29000);
        addDoc(writer, "2", "갤럭시 S23 케이스", "투명 TPU 케이스", 9900);
        addDoc(writer, "3", "아이폰 충전기", "20W 고속 충전기", 19000);

        writer.close();

        // 2) Searcher: 검색 실행
        IndexReader reader = DirectoryReader.open(dir);
        IndexSearcher searcher = new IndexSearcher(reader);

        // 3) QueryParser: 키워드 검색(기본은 OR 성향이지만 설정 가능)
        QueryParser parser = new QueryParser("title", analyzer);
        Query query = parser.parse("아이폰 케이스");

        TopDocs topDocs = searcher.search(query, 10);

        System.out.println("총 hits: " + topDocs.totalHits.value());
        for (ScoreDoc sd : topDocs.scoreDocs) {
            Document d = searcher.doc(sd.doc);
            System.out.printf("docId=%s, title=%s, score=%.4f%n",
                    d.get("id"), d.get("title"), sd.score);
        }

        reader.close();
        dir.close();
    }

    private static void addDoc(IndexWriter writer, String id, String title, String body, int price) throws Exception {
        Document doc = new Document();

        // 검색 대상(text): 토큰화되어 postings에 들어감
        doc.add(new TextField("title", title, Field.Store.YES));
        doc.add(new TextField("body", body, Field.Store.YES));

        // 필터/정렬용 숫자: postings가 아니라 doc values / point로 관리
        doc.add(new IntPoint("price", price));
        doc.add(new StoredField("price_store", price));

        // 식별자
        doc.add(new StringField("id", id, Field.Store.YES));

        writer.addDocument(doc);
    }
}
```

#### 이 코드에서 “키워드 검색”이 일어나는 지점

* `TextField("title"...` 가 analyzer로 토큰화되어 \*\*역색인(postings)\*\*에 들어감
* `QueryParser("title", analyzer)` 가 `"아이폰 케이스"`를 토큰화하여\
  postings 조회 → 매칭 문서 후보 생성 → BM25 점수 계산 → TopK 추출



### Elasticsearch/OpenSearch 예시 (REST) — 매핑/색인/검색

#### (1) 인덱스 생성 + 매핑

```bash
curl -X PUT "http://localhost:9200/products" -H "Content-Type: application/json" -d '
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_ko_analyzer": {
          "type": "standard"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": { "type": "text", "analyzer": "my_ko_analyzer" },
      "body":  { "type": "text", "analyzer": "my_ko_analyzer" },
      "price": { "type": "integer" },
      "createdAt": { "type": "date" }
    }
  }
}'
```

#### (2) 문서 색인

```bash
curl -X POST "http://localhost:9200/products/_doc/1" -H "Content-Type: application/json" -d '
{
  "title": "아이폰 13 케이스",
  "body": "정품 실리콘 케이스",
  "price": 29000,
  "createdAt": "2026-01-10T10:00:00+09:00"
}'
```

#### (3) 키워드 검색 (multi\_match + 필드 가중치)

```bash
curl -X GET "http://localhost:9200/products/_search" -H "Content-Type: application/json" -d '
{
  "query": {
    "multi_match": {
      "query": "아이폰 케이스",
      "fields": ["title^3", "body"]
    }
  },
  "size": 10
}'
```

* `title^3` : 제목 매칭을 더 중요하게
* 내부적으로:
  * query analyzer로 “아이폰”, “케이스” 토큰화
  * postings 기반으로 후보 문서 찾기
  * BM25 등으로 점수 계산
  * Top 10 반환
