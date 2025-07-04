# API endpoints

### ✅ API 엔드포인트

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
