---
description: 수강 관리
---

# Course Management API

## 개요

온라인 강의 플랫폼을 위한 수강 관리 API를 설계하세요. 사용자는 강의를 수강할 수 있으며, 강의에는 여러 개의 강의 영상이 포함됩니다. 수강을 완료한 강의는 완료 상태로 변경할 수 있으며, 사용자는 수강한 강의를 조회할 수 있습니다.



## 요구사항

* **강의 등록**: 관리자가 새로운 강의를 등록할 수 있어야 합니다.
* **강의 영상 등록**: 등록된 강의에 여러 개의 강의 영상을 추가할 수 있어야 합니다.
* **강의 목록 조회**: 사용자는 수강 가능한 강의 목록을 조회할 수 있어야 합니다.
* **강의 수강 신청**: 사용자는 특정 강의를 수강 신청할 수 있어야 합니다.
* **수강 중인 강의 목록 조회**: 사용자는 자신이 수강 중인 강의 목록을 조회할 수 있어야 합니다.
* **수강 완료 처리**: 사용자는 자신이 수강한 강의를 완료 상태로 변경할 수 있어야 합니다.
* **완료한 강의 목록 조회**: 사용자는 자신이 완료한 강의 목록을 조회할 수 있어야 합니다.



## 1. 엔드포인트 설계

* 강의 등록(POST) - /courses
* 강의 영상 등록(POST) - /courses/{courseId}/videos
* 강의 목록 조회(GET) - /courses
* 강의 수강 신청(POST) - /courses/{courseId}/subscriptions
* 수강 중인 강의 목록 조회(GET) - /users/{userId}courses
* 수강 완료 처리(PATCH) - /users/{userId}/courses/{courseId}/status
* 완료한 강의 목록 조회(GET) - /user/{userId}courses?status=completed



## 2. 요청/응답 구조 설계

* 강의 등록

```json
Req
POST /courses
Content-Type: application/json
{
    "courseName": "Spring",
    "lecturerId": "1"
}

Res (success)
HTTP/1.1 201 Created
Content-Type: application/json
{
    "courseId": 1
}

Res (fail)
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
    "message": "이미 존재하는 강의입니다."
}

```



* 강의 영상 등록(POST) - /courses/{courseId}/clips

```json
Req
POST /courses/1/clips
Content-Type: application/json
{
    "videoUrl": "http://clips/B312"
}

Res
HTTP/1.1 201 Created
Content-Type: application/json
{
    "videoId": 1
}
```



* 강의 목록 조회(GET) - /courses

```json
Req
GET /courses

Res
HTTP/1.1 200 OK
Content-Type: application/json
{
    "courses": [
        {
            "courseId": 1,
            "courseName": "Spring",
            "lecturer": "Alice"
        }, ...
    ]
}
```



* 강의 수강 신청(POST) - users/{userId}/courses/{courseId}/subscriptions

```json
Req
POST users/1/courses/1/subscribe
Content-Type: application/json


Res (Success)
HTTP/1.1 200 OK
{
    "userId": 1,
    "courseId": 1,
    "status": "COMPLETED"
}

Res (Fail)
HTTP/1.1 400 Bad Reqeust
{
    "message": "course does not exist" / "already subscribed"
}
```



* 수강 중인 강의 목록 조회(GET) - /users/{userId}/courses

```json
Req
GET /users/1/courses

Res
HTTP/1.1 200 OK
[
    {
        "courseId": 1,
        "courseName": "Spring",
        "lecturer": "Alice"
        "status": "In progress"
    }, ...
]

```



* 수강 완료 처리(PATCH) - /users/{userId}/courses/{courseId}/status

```json
Req
PATCH /users/1/courses/1/status
Content-Type: application/json
{
    "courseId": 1
    "status": "Completed"
}

Res (success)
HTTP/1.1 200 OK
{
    "message": "Status changed to completed"
}

Res (fail)
HTTP/1.1 400 Bad Request
{
    "message": "Already completed" / "Course does not exist"
}
```



* 완료한 강의 목록 조회(GET) - /user/{userId}courses?status=completed

```json
Req
GET /user/1/courses/completed

Res
HTTP/1.1 200 OK
Content-Type: application/json
[
    {
        "courseId": 1,
        "courseName": "Spring",
        "lecturer": "Alice"
        "status": "Completed"
    }, ...
]
```



## 3. DB 모델링

* user
  * id. BIGINT, PK. 유저 id
  * role. VARCHAR. 역할 (user, lecturer)
* user\_courses
  * id. BIGINT, PK. 유저-강의 id
  * user\_id. BIGINT, FK. 유저 id
  * course\_id. BIGINT, FK. 강의 id
  * status. VARCHAT(255). 상태
  * created\_at. DATETIME. 생성일
  * modified\_at. DATETIME. 수정일
* courses
  * id. BIGINT, PK. 강의 id
  * user\_id. BIGINT, FK. 유저 id
  * course\_name. VARCHAR(255). 강의 이름



## 4. 아키텍처 고려

* `GET /courses`는 자주 조회되는 API이므로 **Redis 캐싱**을 고려해야 함.
* 강사 정보도 자주 조회된다면 캐싱 필요.

**캐싱 적용 가능 부분**

1. 강의 목록 캐싱 (`GET /courses`)
2. 강사 정보 캐싱 (`GET /users/{userId}`)
3. 사용자 수강 강의 캐싱 (`GET /users/{userId}/courses`)
