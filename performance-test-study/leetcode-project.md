---
description: 리트코드 프로젝트 성능 & 부하 테스트 학습
---

# Leetcode Project

시스템 디자인 인터뷰를 준비하면서 느낀 가장 큰 어려움은 “머리로는 알겠지만, 손으로는 만들어본 적 없는 시스템”이라는 점이었습니다. 읽고 그려보는 것만으로는 설계 원리를 체득하기 어렵다고 판단했고, 실제로 설계, 구현, 성능 개선까지 경험해보는 것이 가장 효과적인 학습이라고 생각했습니다.

직접 LeetCode 같은 코딩 플랫폼을 직접 설계하고, 작은 MVP부터 시작해 성능 병목을 찾아내고, 문제를 구조적으로 해결하는 일련의 과정을 실습해보기로 했습니다.

이 글은 단순히 동작하는 서비스를 만들기 위한 기록이 아니라, **시스템 디자인 인터뷰에서 자주 등장하는 주제들**—예를 들면 **캐싱 전략, 페이지네이션 방식의 선택, 코드 실행 샌드박스 분리, 실시간 리더보드 설계, 메시지 큐와 컨테이너를 활용한 비동기 처리 구조** 등을 실제로 설계하고 실험한 과정입니다.

실무에서도, 인터뷰에서도 자신 있게 설명할 수 있도록 만들기 위한 일종의 **자체 시뮬레이션 기반 학습입니다.**



## 1. 구현

### 1. ✅ MVP 제작

성능 테스트와 부하테스트를 진행하기 위해 필요한 동작하는 MVP 프로젝트를 구현했습니다.

#### 1-1. ✅ 핵심 요구사항

#### 문제 보기 (View Problems)

* 사용자는 문제 설명, 예제, 제약 조건 등을 확인할 수 있어야 한다.
* 문제 목록을 탐색할 수 있어야 한다.

#### 풀이 제출 (Submit Solution)

* 사용자는 코드 문제를 풀기 위해 솔루션 코드를 제출할 수 있어야 한다.
* 제출된 코드는 내장된 테스트 케이스로 실행되어 결과를 제공해야 한다.

#### 코딩 대회 기능 (Coding Contest)

* 사용자는 코딩 대회에 참여할 수 있다.
* 대회는 2시간 동안 진행되며, 4개의 문제로 구성된다.
* 점수는 푼 문제 수와 풀이 시간에 기반해 계산된다.
* 리더보드는 실시간으로 결과를 표시하며, 상위 50명의 사용자(username 및 score)를 보여준다.

#### 1-2. 🚫 제외 범위 (Out of Scope)

* 사용자 인증 (Authentication)
* 권한 관리 (Authorization)
* 사용자 관리 (User Management)
* 과거 대회 기록 조회 (Contest History)



***

### 2. ✅ API 엔드포인트

#### **2-1. GET /problems?start={start}\&end={end}**

* 설명: 시작 페이지 넘버부터 끝 페이지 넘버까지의 문제 리스트를 조회합니다.

```json
Request
GET /problems?start=45&end=59
Content-Type: application/json

Response
HTTP/1.1 200 OK
Content-Type: application/json
{
	"content": [
	  {
	    "problem_id": 675,
	    "title": "Two Sum",
	    "difficulty": "Easy"
	  },....
	  {
	    "problem_id": 884,
	    "title": "Add Two Numbers",
	    "difficulty": "Medium"
	  }
  ]
}
```

#### **2-1. GET /problems/offset**

* 설명: start와 end가 제공되지 않았을 경우, 전체 조회를 페이지네이션으로 가져옵니다.

```json
Request
GET /problems/offset?page=9999&size=100
Content-Type: application/json

Response
HTTP/1.1 200 OK
Content-Type: application/json
{
	"content": [
	  {
	    "problem_id": 675,
	    "title": "Two Sum",
	    "difficulty": "Easy"
	  },....
	  {
	    "problem_id": 884,
	    "title": "Add Two Numbers",
	    "difficulty": "Medium"
	  }
  ],
  "page": 9999,
  "size": 100,
  "totalElements": 1000000,
  "totalPages": 10000
}
```

#### **2-1. GET /problems/cursor**

* 설명: start와 end가 제공되지 않았을 경우, 전체 조회를 페이지네이션으로 가져옵니다.

```json
Request
GET /problems/cursor?cursor=999900&limit=100
Content-Type: application/json

Response
HTTP/1.1 200 OK
Content-Type: application/json
{
	"content": [
	  {
	    "problem_id": 675,
	    "title": "Two Sum",
	    "difficulty": "Easy"
	  },....
	  {
	    "problem_id": 884,
	    "title": "Add Two Numbers",
	    "difficulty": "Medium"
	  }
  ]
}
```

***

#### 2-2. GET /problems/{problem\_id}

* 설명: 문제 설명, 제약 조건, 예시, 스타터 코드를 포함한 문제 상세 정보를 조회합니다.

```json
Request
GET /problems/1
Content-Type: application/json

Response
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,                           // 문제 ID
  "title": "Problem 1",             // 문제 제목
  "difficulty": "HARD",             // 난이도
  "description": "This is the description for problem 1.", // 문제 설명
  "constraints": "Constraints for problem 1 include limits on input size, time complexity, etc.", // 제약 조건
  "examples": [                     // 예시 목록
    {
      "input": "2 7",               // 입력 예시
      "output": "9"                // 출력 예시
    },
    {
      "input": "3 5",
      "output": "8"
    }
  ],
  "startercodes": [                // 스타터 코드 목록
    {
      "language": "JAVA",          // 언어명 (대문자로 표기됨)
      "code": "public class Solution {\\n    public int solution(int[] numbers) {\\n        // Write your code here\\n        return 0;\\n    }\\n}" // Java 코드
    },
    {
      "language": "JAVASCRIPT",
      "code": "function solution(numbers) {\\n    // Write your code here\\n    return 0;\\n}"
    }
  ]
}

```

***

#### 2-3. GET /contests/{contest\_id}/leaderboard

* 설명: 특정 대회의 리더보드를 조회합니다.

```json
Request
GET /contests/3/leaderboard
Content-Type: application/json

Response
HTTP/1.1 200 OK
Content-Type: application/json

{
  "ranking": [               // 리더보드 배열 (점수순 정렬, 최대 50명)
    {
      "userId": 101,         // 사용자 ID
      "score": 1500          // 사용자 점수
    },
    {
      "userId": 102,
      "score": 1450
    },
    {
      "userId": 103,
      "score": 1420
    }
    // ... 최대 50명까지
  ]
}

```

***

#### 2-4. POST /problems/{problem\_id}/submission

* 설명: 사용자가 제출한 풀이 코드를 서버에 전송하여 채점 결과를 받아옵니다. 내부적으로는 각 테스트 케이스별 실행 결과를 반환합니다.

```json
Request
POST /problems/1/submission
Content-Type: application/json

{
  "language": "java",                   // 제출한 코드의 언어 (예: java, python, javascript 등)
  "code": "public class Solution { ... }" // 사용자가 제출한 코드
}

Response
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "SUCCESS",                  // 전체 제출 결과 (예: SUCCESS, FAIL, COMPILE_ERROR 등)
  "testCaseStatus": [                  // 개별 테스트 케이스별 결과 배열
    "SUCCESS",                         // 테스트 케이스 1 통과
    "SUCCESS",                         // 테스트 케이스 2 통과
    "SUCCESS"                          // 테스트 케이스 3 통과
  ]
}

```

***



### 3. ✅ DB 스키마 & 인프라

#### DB 스키마

<div data-full-width="true"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

#### MVP 로컬환경 인프라

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* Java Application: 자바 프로젝트입니다
* MySQL: 데이터베이스입니다.
* SandBox: 유저 코드를 실행하여 정답 여부를 반환합니다.

***



## 2. 📊 성능 테스트

기존에는 단순히 기능이 동작하는 코드에 초점을 맞췄으나, 성능 개선을 위한 실험 및 테스트 항목을 논의한 후 아래 작업들을 수행하기로 결정했습니다.



### 1-1. 📘 문제 목록 가져오기 API

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

따라서 전체 문제 리스트 API에는 cursor 기반 페이지네이션 방식을 적용하기로 결정하였습니다.

***





### 1-2. 📗 문제 상세 정보 가져오기 API

**`GET /problems/{problem_id}`**

* ✅ **캐싱 적용:** 자주 조회되는 상세 정보를 Redis에 캐싱하여, 캐싱된 데이터를 빠르게 가져올 수 있습니다.

#### 가설:

* Redis 캐시 적용 시 API의 응답 속도가 빨라질 것 입니다.
* 자주 조회되는 문제 상세 정보를 Redis에 key로 저장하고 조회하면, DB 접근 빈도가 줄어들 것입니다.

#### 실험/과정:

1. 캐싱 전략 수립:
   * 문제 상세 정보를 Redis에 저장
   * 요청 시 캐시에 데이터가 있으면 바로 반환, 없으면 DB 조회 후 캐시에 저장
2. 성능 측정
   * 캐싱 적용 전/후로 API 응답 시간 측정
   * 동일한 문제를 반복 요청하여 응답 시간 측정
   * 응답 시간 측정 후 개선율 확인
3. 테스트 환경
   * 로컬 개발 환경에서 레디스 캐시와 연동



<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>결과</p></figcaption></figure>

* 캐싱 적용 결과, 문제 상세 정보 조회 API의 응답 속도가 50% 이상 개선되었습니다.
* 캐시 히트가 100%라면 자주 조회되는 문제에 대해서는 DB 접근 빈도가 줄어 서버 부하가 감소함을 예상할 수 있습니다.

#### 평가:

🟩 장점:

* **API 응답 속도 개선**: 실험 결과, API 응답 속도가 **50% 이상 향상**되었습니다.
* **DB 부하 감소**: 캐시 히트율이 100%라면, **자주 조회되는 문제에 대한 데이터베이스 접근 빈도가 크게 줄어듭니다.**
* **서버 자원 효율성 증대**: 불필요한 DB I/O를 줄이고 캐시 서버(Redis)를 활용함으로써, 시스템의 전반적인 자원 활용 효율성을 높일 수 있습니다.

🟥 단점:

* **캐시 무효화 및 일관성 관리 필요**: 데이터가 업데이트될 경우 캐시를 최신 상태로 유지하기 위한 **캐시 무효화(Cache Invalidation) 전략**이 필요합니다. 이를 잘못 관리하면 stale data(오래된 데이터)를 사용자에게 보여줄 위험이 있습니다.
* **캐시 서버 관리 오버헤드**: Redis와 같은 별도의 캐시 서버를 운영해야 하므로, 배포, 모니터링, 유지보수 등 **추가적인 관리 오버헤드**가 발생합니다.

#### 결론:

* 문제 상세 정보 가져오기 API에 대한 캐싱 적용은 매우 효과적이며, 전반적으로 강력히 권장되는 개선 사항입니다.
* 특히, 문제라는 서비스 특성상 데이터가 거의 변경될 가능성이 낮아 캐시 효율을 극대화할 수 있다는 점은 캐싱 도입의 핵심 근거로 내세울 수 있습니다.

***





### 1-3. 📕 풀이 제출 API

**`POST /problems/{problem_id}/submission`**

* ✅ **DB 저장 구조 설계 및 최적화:** 동일한 코드를 다시 제출하였을 때, 캐싱된 데이터를 빠르게 가져올 수 있습니다.

#### 가설:

* 유저의 코드를 DB에 저장하게 되면, 동일한 코드를 제출했을때, 다시 실행하지 않고, 저장되어있는 데이터를 가져올 수 있게 됩니다. 제출 코드를 저장하게 될 경우, 저장공간을 크게 차지하게 됩니다. 이를 해결하기 위해, SHA256으로 인코딩한 결과를 저장하게 되면 최소한의 저장공간을 사용하게 될거라 생각합니다.
* 빠르게 여러번의 요청이 들어온 경우, 이를 식별하여 하나의 요청만 처리되도록 할 수 있을 것 같습니다. 이는 프론트에서 보내주는 idempotency-key를 활용할 수 있습니다.
* 캐시 히트율이 높지 않은 상황에서, 오히려 오버헤드가 될 수 있다고 생각을 하고 있습니다.

#### 실험/과정

**현재:**

```json
요청1
[1fd3bad3] ProblemController.submitProblem(..)
[1fd3bad3] |-->ProblemService.submitProblem(..)
[1fd3bad3] |   |-->CrudRepository.findById(..)
[1fd3bad3] |   |<--CrudRepository.findById(..) time=9ms
[1fd3bad3] |   |-->CrudRepository.findById(..)
[1fd3bad3] |   |<--CrudRepository.findById(..) time=5ms
[1fd3bad3] |   |-->CrudRepository.save(..)
[1fd3bad3] |   |<--CrudRepository.save(..) time=12ms
[1fd3bad3] |<--ProblemService.submitProblem(..) time=1038ms
[1fd3bad3] ProblemController.submitProblem(..) time=1038ms

요청2
[5d67ec83] ProblemController.submitProblem(..)
[5d67ec83] |-->ProblemService.submitProblem(..)
[5d67ec83] |   |-->CrudRepository.findById(..)
[5d67ec83] |   |<--CrudRepository.findById(..) time=9ms
[5d67ec83] |   |-->CrudRepository.findById(..)
[5d67ec83] |   |<--CrudRepository.findById(..) time=6ms
[5d67ec83] |   |-->CrudRepository.save(..)
[5d67ec83] |   |<--CrudRepository.save(..) time=9ms
[5d67ec83] |<--ProblemService.submitProblem(..) time=1033ms
[5d67ec83] ProblemController.submitProblem(..) time=1033ms
```

요청1과 요청2는 동일한 코드를 제출하고 있습니다. 각각 1038, 1033ms가 걸리고 있습니다.

* 멱등성 키를 적용하지 않아서, 빠르게 들어온 중복 요청에 대한 처리가 없고,
* 동일한 코드를 제출했을때도 한번 더 샌드박스에서 실행하고 있습니다.

**개선된 후:**

```json
요청1 (Idempotency-key = "abc")
[01971fe9] ProblemController.submitProblem(..)
[01971fe9] |-->ProblemService.submitProblem(..)
[01971fe9] |   |-->CrudRepository.findById(..) // find problem
[01971fe9] |   |<--CrudRepository.findById(..) time=32ms
[01971fe9] |   |-->CrudRepository.findById(..) // find user
[01971fe9] |   |<--CrudRepository.findById(..) time=5ms
[01971fe9] |   |-->SubmissionApiRepository.findByEncodedCode(..) // find submission
[01971fe9] |   |<--SubmissionApiRepository.findByEncodedCode(..) time=40ms
[01971fe9] |   |-->CrudRepository.save(..)
[01971fe9] |   |<--CrudRepository.save(..) time=53ms
[01971fe9] |<--ProblemService.submitProblem(..) time=1178ms
[01971fe9] ProblemController.submitProblem(..) time=1178ms

요청2 (Idempotency-key = "abc")
[82dbc7c5] ProblemController.submitProblem(..)
[82dbc7c5] |-->ProblemService.submitProblem(..)
[82dbc7c5] |<X-ProblemService.submitProblem(..) time=1ms Exception: 중복된 요청입니다.
[82dbc7c5] ProblemController.submitProblem(..) time=1ms Exception: 중복된 요청입니다.

요청3 (Idempotency-key = "bcd")
[dfc4eade] ProblemController.submitProblem(..)
[dfc4eade] |-->ProblemService.submitProblem(..)
[dfc4eade] |   |-->CrudRepository.findById(..) // find problem
[dfc4eade] |   |<--CrudRepository.findById(..) time=12ms
[dfc4eade] |   |-->CrudRepository.findById(..) // find user
[dfc4eade] |   |<--CrudRepository.findById(..) time=6ms
[dfc4eade] |   |-->SubmissionApiRepository.findByEncodedCode(..) // find submission
[dfc4eade] |   |<--SubmissionApiRepository.findByEncodedCode(..) time=7ms
[dfc4eade] |<--ProblemService.submitProblem(..) time=31ms
[dfc4eade] ProblemController.submitProblem(..) time=31ms

```

전/후 요청 자체 처리는 약 **140ms 차이**가 발생합니다.

`요청1`같은 itempotency-key를 가진 `요청2`은 처리가 되지 않고 빠르게 오류를 반환하고 있습니다.

`요청3`은 동일한 코드에 대해서 코드를 실행시키지 않고, 과거 요청에 대한 결과값을 db에서 가져오고 있습니다.

* 동일한 요청이 들어올 경우: 멱등 키(Idempotency-key) 적용
  * 클라이언트가 요청 시 `Idempotency-Key` 헤더를 첨부하도록 설계. (SHA256 해시 알고리즘)
  * Idempotency-Key를 통해, 특정 상황에 발생할 수 있는 **동일한 요청**에 대해 빠르게 처리할 수 있도록 하였습니다.
* **동일한 코드가 들어올 경우:** 과거 제출된 코드는 DB에서 가져오게 했습니다.
  * 유저의 코드를 DB에 효율적으로 저장하기 위해 SHA256를 적용했습니다.
    * 약 10000줄의 코드를 SHA256 인코딩을 하게 됐을때, 80길이의 String이 생성됩니다.
    * 해싱 결과는 80자 이하로 제한하였고, 이를 초과하면 예외를 반환하도록 처리했습니다.

#### 평가:

🟩 장점:

* 동일한 코드가 재제출될 경우, DB에서 기존 결과를 바로 가져옴으로써 약 1s 이상의 성능 이점을 얻을 수 있습니다.

🟥 단점:

* `SubmissionApiRepository.findByEncodedCode` 호출로 인해 DB I/O 및 네트워크 비용이 한 번 더 발생하게 됩니다.
* 해싱된 유저코드 `EncodedCode Column`을 추가하여 디스크 사용량 증가
  * 디스크 메모리 증가
  * 복잡성 증가
* 인덱싱 오버헤드
  * 인덱스가 설정 되지 않은 경우: 테이블 풀스캔이 발생.
  * 인덱스를 설정한 경우: 코드 제출시 인덱스도 업데이트 되는 오버헤드 발생.

#### 결론:

✅ 현재 서버 리소스와 처리량으로 기본 제출 방식만으로도 충분히 안정적이라면, 해당 기능은 도입을 보류하고 향후 트래픽 증가 시점에 다시 도입을 검토하는 것이 바람직합니다.

반대로,

❗ 실시간 응답 속도에 민감하거나, **동일 코드 제출이 빈번하게 발생하는 서비스 구조**라면, 본 개선안을 적극 도입하여 서버 부하와 처리 지연을 줄이는 전략으로 활용할 수 있습니다.

따라서, **적용하지 않기로 결정**했습니다. _(PR Closed)_

[https://github.com/Collaborative-AI-SystemDesign/design-leetcode-scalable-architecture/pull/31](https://github.com/Collaborative-AI-SystemDesign/design-leetcode-scalable-architecture/pull/31)

***





### 1-4. 📙 리더보드 조회 API

**`GET /contests/{contest_id}/leaderboard`**

* ✅ 1. **DB 인덱스 최적화:** contestId, score 컬럼 인덱스 추가

#### 가설:

* 현재 leaderboard 조회 시 `contest_id`를 조건으로 주고 `score DESC`로 정렬하는 쿼리의 실행 성능이 낮은 것으로 확인되었습니다.
* 기존에는 `contest_id`에 대한 FK 인덱스만 존재하여 데이터 필터링 후 정렬이 발생합니다.
* MySQL의 경우, 정렬(`ORDER BY`)을 효율적으로 처리하기 위해서는 **정렬 컬럼이 포함된 인덱스**가 필요합니다.
* 따라서 `contest_id`와 `score DESC`를 포함하는 **복합 인덱스**를 생성하여 쿼리가 인덱스만으로 정렬까지 처리하도록 최적화하면 성능이 크게 향상될 것으로 예상됩니다.
* 복합 인덱스 추가로 leaderboard 조회 성능이 크게 개선될 것으로 기대됩니다.

<details>

<summary>MySQL 인덱스 동작 원리</summary>

* 단일 컬럼 인덱스(`contest_id`)만으로는 정렬 조건(`score DESC`)이 인덱스에 포함되지 않아 filesort 단계가 발생.

- `contest_id`와 `score DESC`를 함께 포함한 복합 인덱스를 사용하면 정렬 작업을 인덱스 레벨에서 해결하여 정렬 비용을 줄이고 성능을 향상시킬 수 있습니다.

* 일반적으로 MySQL의 인덱스 정렬 처리 비용은 \*\*O(log n)\*\*로, 인덱스를 사용한 정렬이 디스크 정렬보다 훨씬 빠릅니다.

</details>



#### 실험/과정:

**현재:**

SQL Plan 확인

* FK(`contest_id`) 인덱스를 사용하여 데이터를 필터링한 뒤 정렬 처리를 하고 있습니다.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>SQL plan</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>EXPLAIN ANALYZE 결과</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>Application 단 API 성능</p></figcaption></figure>

* DB 조회 시간: 약 **100ms**
* 총 서비스 호출 시간: **286ms (DB 조회, 네트워크 통신, 애플리케이션 처리시간을 모두 포함)**



**개선 후:**

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption><p>SQL Plan</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>EXPLAIN ANALYZE 결과</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption><p>Application 단 API 성능</p></figcaption></figure>

* Plan 상에서는 FK 인덱스와 유사해 보이나 실제 실행 시 정렬 처리가 인덱스를 통해 최적화됨.
  * FK 인덱스만으로도 Plan은 비슷하게 출력되지만, **복합 인덱스**로 인해 `ORDER BY` 최적화가 이루어져 정렬 비용이 크게 줄어듦.
* DB 조회 시간: 약 **5.45ms**
* 총 서비스 호출 시간: **31ms**

#### 평가:

* 인덱스 추가 전이랑 추가 후 성능 비교 평가
  * 서비스 호출 시간: 인덱스 적용 전 **286ms** → 인덱스 추가 후 **31ms**로 응답 속도 대폭 개선
  * DB 조회 시간: 인덱스 적용 전 **100ms** → 인덱스 추가 후 **5.45ms**로 대폭 단축
  * 복합 인덱스 설계로 인해 `ORDER BY` 최적화 효과 발생 → 불필요한 정렬 연산 감소

***

* ✅ 2. **Redis Sorted Set 적용:** 상위 50명만 유지하는 구조를 사용하여, 빠른 조회 및 갱신이 가능하도록 할 수 있습니다.

#### 가설:

* 리더보드를 Redis ZSet에 유지하면, 반복적인 상위 50명 조회 요청에 대해 빠른 응답을 제공할 수 있을 것 같습니다.
* Redis의 Sorted Set은 Skip List 기반 자료구조로 `O(log n)`의 삽입/삭제 성능을 가지며, `reverseRangeWithScores(0, 49)`를 통해 효율적으로 상위 랭커를 반환할 수 있습니다.
* 상위 50명의 리스트만 Redis에 유지하고, `addScore()` 시점마다 51위 이하를 제거하는 방식으로 메모리 사용량을 제어할 수 있습니다.

<details>

<summary>Skip List</summary>

기본적으로 여러 개의 연결 리스트(linked list) 층이 있으며, **중간 노드들을 가리키는 추가 포인터**들을 포함해 효율적인 순회와 탐색이 가능합니다. 스킵 리스트는 기능적으로 **균형 이진 탐색 트리(Balanced BST)** 와 유사하지만, **연결 리스트와 난수(randomization)** 를 사용해 구현된다는 점이 다릅니다. 탐색, 삽입, 삭제의 평균 시간 복잡도는 **O(log n)** 입니다. 스킵 리스트는 전체 원소 수를 M이라 할 때, **상위 N개 요소를 반환하는 연산은 O(N × log M)** 의 시간 복잡도로 처리할 수 있기 때문에 적합합니다.

</details>

#### 실험/과정:

**현재:**

```json
[66eba4b1] LeaderBoardController.getLeaderBoardFromRdb(..)
[66eba4b1] |-->LeaderBoardService.getLeaderBoardInfo(..)
[66eba4b1] |   |-->LeaderBoardRepository.getTop50LeaderBoardByContestId(..)
[66eba4b1] |   |<--LeaderBoardRepository.getTop50LeaderBoardByContestId(..) time=62ms
[66eba4b1] |<--LeaderBoardService.getLeaderBoardInfo(..) time=64ms
[66eba4b1] LeaderBoardController.getLeaderBoardFromRdb(..) time=71ms

[6ea56b6c] LeaderBoardController.getLeaderBoardFromRdb(..)
[6ea56b6c] |-->LeaderBoardService.getLeaderBoardInfo(..)
[6ea56b6c] |   |-->LeaderBoardRepository.getTop50LeaderBoardByContestId(..)
[6ea56b6c] |   |<--LeaderBoardRepository.getTop50LeaderBoardByContestId(..) time=24ms
[6ea56b6c] |<--LeaderBoardService.getLeaderBoardInfo(..) time=24ms
[6ea56b6c] LeaderBoardController.getLeaderBoardFromRdb(..) time=29ms
```

`요청1`은 71ms, `요청2`는 29ms가 걸렸습니다.

두번째 요청은 빠르게 처리 되고 있는데, 애플리케이션에서 캐시 관련 로직은 존재하지 않지만, DB 자체의 쿼리 캐시 효과가 있습니다.

* `요청1`: 디스크 → DB 읽기 → 응답
* `요청2`: InnoDB buffer Pool → 메모리 히트 → 응답

**개선된 후:**

```json
[da6bae17] |-->LeaderBoardService.getLeaderBoardWithCache(..) args=[1]
[da6bae17] |   |-->LeaderBoardRedisService.getTopN(..) 
[da6bae17] |   |<--LeaderBoardRedisService.getTopN(..) time=26ms
[da6bae17] |   |-->LeaderBoardRepository.getTop50LeaderBoardByContestId(..) args=[1]
[da6bae17] |   |<--LeaderBoardRepository.getTop50LeaderBoardByContestId(..) args=[1] time=61ms
[da6bae17] |   |-->LeaderBoardRedisService.addScore(..) 
[da6bae17] |   |<--LeaderBoardRedisService.addScore(..) time=2ms
[da6bae17] |   |-->LeaderBoardRedisService.addScore(..) 
[da6bae17] |   |<--LeaderBoardRedisService.addScore(..) time=0ms
...
[da6bae17] |   |<--LeaderBoardRedisService.addScore(..) time=0ms
[da6bae17] |<--LeaderBoardService.getLeaderBoardWithCache(..) time=95ms
[da6bae17] LeaderBoardController.getLeaderBoardWithCache(..) time=102ms
[f6134134] LeaderBoardController.getLeaderBoardWithCache(..)
[f6134134] |-->LeaderBoardService.getLeaderBoardWithCache(..)
[f6134134] |   |-->LeaderBoardRedisService.getTopN(..)
[f6134134] |   |<--LeaderBoardRedisService.getTopN(..) time=2ms
[f6134134] |<--LeaderBoardService.getLeaderBoardWithCache(..) time=3ms
[f6134134] LeaderBoardController.getLeaderBoardWithCache(..) time=17ms

```

Redis의 ZSet를 적용했을때 결과입니다.

요청1은 102ms, 요청2는 17ms가 걸렸습니다.

* `요청1`: 애플리케이션 → 레디스 (캐시 미스) → 디스크 → DB 읽기 → 레디스 저장 → 응답
* `요청2`: 애플리케이션 → 레디스 (캐시 히트) → 응답

#### 평가:

* **기존 RDB 단독 조회 방식**
  * 첫 번째 요청은 InnoDB 버퍼 풀에 적재되지 않은 상태이므로 디스크 I/O가 발생 → **71ms 소요**
  * 두 번째 요청부터는 DB 메모리 캐시(InnoDB Buffer Pool)에 의해 **29ms**로 응답 속도 개선
  * 그러나 여전히 요청마다 **DB I/O 부하가 발생**하므로, 트래픽 증가 시 병목 가능성 존재
* **Redis Sorted Set 캐시 적용 방식**
  * 첫 요청(캐시 미스)은 Redis + DB를 모두 거쳐 **102ms**로 약간 지연됨
  * 이후 요청(캐시 히트)은 Redis에서 즉시 응답 → **17ms**로 대폭 단축
  * Redis는 ZSet + Skip List 기반 구조로 `reverseRangeWithScores`를 통해 **상위 N개 데이터를 O(N) 수준으로 매우 빠르게 조회** 가능
  * **ZSet 유지 전략**도 적절히 설계

#### 결론:

* 시스템의 초기 단계에서는 데이터베이스에 인덱스를 추가하여 검색 성능을 최적화합니다.
* 인덱스를 통해 일정 수준까지는 데이터베이스 부하를 효과적으로 줄일 수 있겠지만, 사용자 수가 증가하면 인덱스만으로는 부족할 수 있습니다. 이에 따라 Redis를 활용해 데이터를 캐싱하고, 자주 조회되는 요청을 빠르게 처리하도록 시스템을 개선했습니다.
* 이러한 단계적 접근 방식은 초기 단계에서 불필요한 오버엔지니어링을 피하면서도, 사용량 증가에 따라 적시에 캐시 도입을 통해 성능을 보장할 수 있는 구조를 마련할 수 있었습니다.

