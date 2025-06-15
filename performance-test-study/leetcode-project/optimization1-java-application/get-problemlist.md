---
description: 문제 목록 가져오기 API 최적화
---

# GET ProblemList

### 📘 문제 목록 가져오기 API

**`GET /problems?start={start}&end={end}` (start, end를 제공하지 않았을 때)**

✅ **Cursor 기반 페이징**

start와 end를 사용하지 않았을 때 즉, 전체 문제리스트 조회시, 페이징기법을 도입합니다.

페이지네이션 방법은 커서 기반과 offset방식이 있으며 offset/limit 방식과 cusor기반 페이징 방식의 가장 큰 차이점은 offset 즉 건너뛸 데이터들`(page-1)*page size`을 스캔하는 비용을 계산하기에 성능 이슈가 있을 수 있습니다.

따라서, 문제 전체 목록 가져오기 API는 offset/limit 방식의 페이지네이션 방식에서, cursor 기반 페이지네이션 방식을 도입할 수 있을 거라고 생각했었습니다.

#### 가설:

* Offset/limit 방식에서 cursor 방식으로 변경했을때, 속도가 빠를 것으로 기대됩니다.

#### 실험/과정:

* 서비스 단위 테스트
* 총 데이터 수 100만건
* 페이지 사이즈 100건



**현재 Offset/limit 방식**

```json
[0ff0efbb] ProblemService.getProblemPage(..) args=[Page request [number: 9999, size 100, sort: UNSORTED]]
[0ff0efbb] |-->PagingAndSortingRepository.findAll(..) args=[Page request [number: 9999, size 100, sort: UNSORTED]]
[0ff0efbb] |<--PagingAndSortingRepository.findAll(..) args=[Page request [number: 9999, size 100, sort: UNSORTED]] time=555ms
[0ff0efbb] ProblemService.getProblemPage(..) args=[Page request [number: 9999, size 100, sort: UNSORTED]] time=560ms
페이지 9999에서 10000 페이지까지 조회: 100건
✅ 오프셋 기반 페이지 전체 조회 시간: 563ms, 총 조회된 개수: 100건

```

* 999900번째부터 1000000번째 페이지를 검색했을때, 기존 563ms이 걸렸습니다.



**Cursor 방식을 도입한 후**

```json
[59338349] ProblemService.getProblemsByCursor(..) args=[999900, 100]
[59338349] |-->ProblemApiRepository.findByIdGreaterThan(..) args=[999900, Page request [number: 0, size 100, sort: id: ASC]]
[59338349] |<--ProblemApiRepository.findByIdGreaterThan(..) args=[999900, Page request [number: 0, size 100, sort: id: ASC]] time=135ms
[59338349] ProblemService.getProblemsByCursor(..) args=[999900, 100] time=137ms
1000000 번째에서 1000100 번째까지 조회: 100건
✅ 커서 기반 페이지 전체 조회 시간: 140ms, 총 조회된 개수: 100건
```

* 동일한 데이터를 cursor 기반 페이지네이션으로 조회 시, 이젠 140ms이 걸립니다.

#### 평가:

**offset방식**

#### 🟩 장점:

* 사용자가 page를 입력하여 특정 페이지로 직접 점프를 하게 됩니다.
* cursor기반과 비교했을 때, 다양한 정렬 방식에 따른 쿼리 작성이 쉽습니다.

#### 🟥 단점:

* `(page - 1) * page size`에 해당하는 데이터를 스캔해야 하므로, 대용량 데이터일수록 성능 저하가 발생합니다.
* 문제 데이터를 추가하거나 삭제 시, 조회하는 문제 데이터가 중복되거나 손실되는 상황이 발생합니다.



**cursor방식**

#### 🟩 장점:

* Offset 방식은 지정된 위치까지의 행을 모두 스캔해야 하는 반면, Cursor 방식은 특정 키 값 이후의 데이터만 탐색하므로 불필요한 스캔 없이 바로 원하는 데이터를 가져올 수 있어 성능이 더 우수합니다. (759ms → 137ms)
* 데이터가 많아져도 페이징 마다 일정한 시간이 걸리기에 안정적인 조회가 가능합니다.

#### 🟥 단점:

* 정렬 방식에 따른 쿼리 작성이 상대적으로 더 복잡해집니다.
* 특정 지점으로 cursor를 이동시켜야 하기에 정렬기준에 중복 값이 존재해서는 안됩니다.
*   Cursor 방식은 순방향 조회에 최적화되어 있어, 이전 페이지로 이동하는 기능이 없으며, 이를 위해서는 이전에 조회된 데이터를 로컬에 저장해야 합니다.

    그러나 리트코드는 문제 리스트 조회기능이 무한스크롤 방식이기에 해당 단점은 현재 고려하지 않아도 됩니다.

#### 결론:

* offset기반 페이지네이션 구현은 offset이 커질수록 **조회 성능이 급격히 저하됩니다**.
* 반면 cursor 방식은 **안정적인 정합성 유지**와 O(1)의 **일관된 성능**을 제공하므로, 이 서비스 특성과 요구사항에 더 적합하다고 판단하였습니다.
* 또한, 문제 리스트는 무한 스크롤 형태로 조회되므로 cursor 방식의 "순방향 탐색" 구조는 오히려 UX에 자연스럽게 부합하며, 단점으로 작용하지 않습니다.

