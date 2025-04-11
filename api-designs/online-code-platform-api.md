---
description: 온라인 코드 실행 및 공유 플랫폼 API 설계
---

# Online Code platform API

## 개요

개발자들이 코드를 작성하고 실행할 수 있는 플랫폼을 설계해야 합니다. 사용자는 다양한 프로그래밍 언어로 코드를 실행할 수 있으며, 실행 결과를 저장하고 공유할 수 있습니다.



## 요구사항

**1. 사용자(User) 기능**

* 코드 작성 및 실행
* 실행 결과 저장 및 조회
* 실행된 코드 공유 (비공개/공개 설정 가능)
* 특정 사용자의 코드 실행 기록 조회

**2. 관리자(Admin) 기능**

* 실행 환경(지원하는 프로그래밍 언어 및 버전) 관리
* 악성 코드 탐지 및 차단
* 과부하 방지를 위한 실행 제한 정책 설정



## 1. 엔드포인트 설계

* 코드 실행 및 저장 (POST) - /codes/execute
* 코드 조회(GET) - /codes/{codeId}
* 코드 실행 결과 조회(GET) - /codes/{codeId}/results
* 사용자의 코드 실행 이력 조회(GET) - /users/{userId}/codes
* 코드 삭제(DELETE) - /codes/{codeId}
* 코드 공유 URL 생성(POST) - /codes/{codeId}/share
* 공유된 코드 조회(GET) - /codes/share/{shortenUrl}



## 2. 응답/요청 설계

* 코드 실행 및 저장 (POST) - /codes/execute

```json
Req
POST /codes/execute
Content-Type: application/json
{
  "language": "Java",
  "code": "System.out.println('Hello World');",
  "isTemporary": false
}

Res (success)
HTTP/1.1 201 CREATED
Content-Type: application/json
{
  "resultId": 1,
  "language": "Java",
  "code": "System.out.println('Hello World');",
  "result": "Hello World",
  "executionTime": 120,
  "memoryUsage": 5.3,
  "status": "success",
  "createdAt": "2025-03-18T12:00:00Z"
}

Res (fail)
HTTP/1.1 400 BAD REQUEST
Content-Type: application/json
{
  "message": "Syntax error in code",
  "status": "error"
}

```



* 코드 조회(GET) - /codes/{codeId}

```json
Req
GET /codes/1

Res (success)
HTTP/1.1 200 OK
Content-Type: application/json
{
  "resultId": 1,
  "language": "Java",
  "code": "System.out.println('Hello World');",
  "createdAt": "2025-03-18T12:00:00Z"
}


Res (fail)
HTTP/1.1 404 NOT FOUND
Content-Type: application/json
{
  "message": "Code not found"
}

```



* 코드 실행 결과 조회(GET) - /codes/{codeId}/results

```json
Req
GET /codes/1/results

Res
HTTP/1.1 200 OK
Content-Type: application/json
{
  "resultId": 1,
  "language": "Java",
  "code": "System.out.println('Hello World');",
  "result": "Hello World",
  "executionTime": 120,
  "memoryUsage": 5.3,
  "status": "success",
  "createdAt": "2025-03-18T12:00:00Z"
}

```



* 사용자의 코드 실행 이력 조회(GET) - /users/{userId}/codes

```json
Req
GET /users/1/codes

Res (success)
HTTP/1.1 200 OK
Content-Type: application/json
{
  "userId": 1,
  "codes": [
    {
      "codeId": 1,
      "language": "Java",
      "code": "System.out.println('Hello World');",
      "createdAt": "2025-03-18T12:00:00Z"
    },
    {
      "codeId": 2,
      "language": "Python",
      "code": "print('Hello World')",
      "createdAt": "2025-03-18T12:05:00Z"
    }
  ]
}

Res (fail)
HTTP/1.1 404 NOT FOUND
Content-Type: application/json
{
  "message": "User not found"
}

```



* 코드 삭제(DELETE) - /codes/{codeId}

```json
Req
DELETE /codes/1

Res (success)
HTTP/1.1 204 NO CONTENT

Res (fail)
HTTP/1.1 404 NOT FOUND
Content-Type: application/json
{
  "message": "Code not found"
}

```



* 코드 공유 URL 생성(POST) - /codes/{codeId}/share

```json
Req
POST /codes/1/share

Res (success)
HTTP/1.1 201 CREATED
Content-Type: application/json
{
  "shortenUrl": "www.example.com/shorten/B842"
}

Res (fail)
HTTP/1.1 404 NOT FOUND
Content-Type: application/json
{
  "message": "Code not found"
}

```



* 공유된 코드 조회(GET) - /codes/share/{shortenUrl}

```json
Req
GET /codes/share/B842

Res (success)
HTTP/1.1 200 OK
Content-Type: application/json

{
  "resultId": 1,
  "language": "Java",
  "code": "System.out.println('Hello World');",
  "createdAt": "2025-03-18T12:00:00Z"
}


Res (Fail)
HTTP/1.1 404 NOT FOUND
Content-Type: application/json

{
  "message": "Shorten URL not found"
}

```



## 3. DB 모델링

* user
  * id. BIGINT, PK. 유저 id
* languages
  * id BIGINT PK
  * name VARCHAR(50)
  * version VARCHAR(20)
* codes
  * id BIGINT PK
  * user\_id BIGINT FK(users.id)
  * language\_id BIGINT FK(languages.id)
  * code TEXT
  * createdAt DATETIME
  * modifiedAt DATETIME
* execution\_results
  * id BIGINT PK
  * code\_id BIGINT FK(codes.id)
  * result TEXT
  * executionTime INT
  * memoryUsage FLOAT
  * status ENUM('success', 'error')
  * createdAt DATETIME
* shared\_codes
  * id BIGINT PK
  * code\_id BIGINT FK(codes.id)
  * shortenUrl TEXT
  * createdAt DATETIME



## 아키텍처 고려

* **API 서버 (Spring Boot)**
  * 사용자 요청 처리 (코드 실행, 저장, 조회, 공유)
  * RESTful API 제공
  * DB와 Redis 캐싱 연동
* **코드 실행 서버 (Sandboxed Execution)**
  * 언어별로 컨테이너(Docker) 실행 환경 제공
  * 실행 요청을 받아 격리된 환경에서 실행
  * 실행 결과를 API 서버에 반환
* **데이터베이스 (MySQL)**
  * 코드, 실행 결과, 사용자 정보 저장
* **캐싱 서버 (Redis)**
  * 단축 URL 캐싱
  * 자주 조회되는 코드 실행 결과 캐싱
