---
description: 풀이 제출 API 최적화
---

# POST Submit Problem

## 최적화 시도: 캐싱 & 멱등키 도입

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
