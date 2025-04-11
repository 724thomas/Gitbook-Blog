---
description: 이벤트 참여 API
---

# Event Participation API

## 개요

어떤 서비스에서 사용자가 다양한 이벤트에 참여할 수 있는 기능을 제공하려고 한다. 예를 들어 "출석체크 이벤트", "댓글 달기 이벤트", "리뷰 작성 이벤트" 등의 미션 기반 이벤트가 존재하며, 사용자는 각 이벤트에 참여하고 보상을 받을 수 있다.



## 요구사항

* **이벤트 등록 API**
  * 관리자가 새로운 이벤트를 생성할 수 있어야 한다.
  * 이벤트는 `제목`, `설명`, `시작일`, `종료일`, `참여 조건 설명`, `보상 내용` 정보를 포함한다.
* **이벤트 목록 조회 API**
  * 사용자 입장에서 현재 참여 가능한 이벤트 리스트를 조회할 수 있어야 한다.
  * 종료된 이벤트는 제외되며, pagination을 고려해야 한다.
* **이벤트 참여 API**
  * 사용자가 특정 이벤트에 참여할 수 있는 API이다.
  * 중복 참여는 불가하다.
  * 참여 시점은 기록되어야 한다.
* **내 참여 이벤트 목록 조회 API**
  * 사용자가 참여한 이벤트들을 조회할 수 있다.
* **관리자용 참여자 목록 조회 API**
  * 이벤트 ID를 기반으로 해당 이벤트에 참여한 사용자 목록을 조회할 수 있어야 한다.



## 1. 엔드포인트 설계

* 이벤트 등록(POST) - /admin/events
* 이벤트 목록 조회(GET) - /events?status={available}\&page={page}\&size={size}
* 이벤트 참여(POST) - /me/events/{eventId}/participate
* 내 이벤트 목록 조회(GET) - /me/events?status={available}\&page={page}\&size={size}
* 관리자용 참여자 목록 조회(GET) - /admin/events/{eventId}



## 2. 요청/응답 설계

* 이벤트 등록(POST) - /events

```json
Req
POST /events
Content-type: application/json
{
    "eventName": "JavaEvent",
    "description": "Event for Java",
    "startDate": "2025-04-24",
    "participateRequirement": "Developer",
    "reward": "Knowledge",
    "active": "true"
}

Res
HTTP/1.1 201 CREATED
Content-type: application/json
{
    "eventId": 1,
    "eventName": "JavaEvent",
    "description": "Event for Java",
    "startDate": "2025-04-24",
    "participateRequirement": "Developer",
    "reward": "Knowledge"
    "status": "Available"
    "createdAt": "2025-03-24",
    "modifiedAt": "2025-03-24"
}
```



* 이벤트 목록 조회(GET) - /events?status={available}\&page={page}\&size={size}

```json
Req
GET /events/status=available&page=1&size=10

Res
HTTP/1.1 200 OK
Content-type: application/json
{
    "status": "available",
    "page": 0,
    "size": 10,
    "totalElements": 2,
    "totalPages": 1,
    "events": [
        {
            "eventId": 1,
            "eventName": "JavaEvent"
            "description": "Event for Java",
            "startDate": "2025-04-24",
            "participateRequirement": "Developer",
            "reward": "Knowledge"
            "status": "Available"
            "createdAt": "2025-03-24",
            "modifiedAt": "2025-03-24"
        }, ...
    ]
}
```



* 이벤트 참여(POST) - /users/events/participate

```json
Req
POST /users/events/1/participate

Res (Success)
HTTP/1.1 200 OK
Content-type: application/json
{
    "eventId": 1,
    "eventName": "JavaEvent",
    "description": "Event for Java",
    "startDate": "2025-04-24",
    "participateRequirement": "Developer",
    "reward": "Knowledge"
    "status": "Available"
    "createdAt": "2025-03-24",
    "modifiedAt": "2025-03-24"
}

Res (Fail)
HTTP/1.1 400 BAD REQUEST
Content-type: application/json
{
    "message": "fully reserved"
}

Res (Fail)
HTTP/1.1 400 BAD REQUEST
Content-type: application/json
{
    "message": "already reserved"
}
```



* 내 이벤트 목록 조회(GET) - /users/events?status={available}\&page={page}\&size={size}

```json
Req
GET /users/events?page=1&size=10

Res
HTTP/1.1 200 OK
Content-type: application/json
{
    "page": 0,
    "size": 10,
    "totalElements": 1,
    "totalPages": 1,
    "events": [
        {
            "eventId": 1,
            "eventName": "JavaEvent"
            "description": "Event for Java",
            "startDate": "2025-04-24",
            "participateRequirement": "Developer",
            "reward": "Knowledge"
            "status": "Available"
            "createdAt": "2025-03-24",
            "modifiedAt": "2025-03-24"
        }, ...
    ]
}
```



* 관리자용 참여자 목록 조회(GET) - /admins/events/{eventId}/participants

```json
Req
GET /admins/events/1/participants

Res
HTTP/1.1 200 OK
Content-type: application/json
{
    "eventId": 1,
    "eventName": "JavaEvent",
    "totalParticipants": 2,
    "participants": 
    [
        {
            "userId": 1,
            ...
        }, ...
    ]
}
```



## 3. DB 스키마 설계

* baseEntity
  * createdAt. DATETIME. 등록일
  * modifiedAt. DATETIME. 수정일
* user
  * user\_id. BIGINT, PK. 유저 id
* event
  * event\_id. BIGINT, PK. 이벤트 id
  * description. TEXT.  설명
  * startDate. DATETIME. 시작일
  * participate\_requirement. TEXT. 참여 조건
  * reward. TEXT. 보상
  * status. VARCHAR(255), ENUM. 상태
* user-event
  * user-event id. BIGINT, PK. 유저-이벤트 id
  * user\_id. BIGINT, FK. 유저 id
  * event\_id. BIGINT, FK. 이벤트 id



## 4. 아키텍처 설계

* API 서버(SpringBoot)
  * 이벤트, 사용자, 관리자 요청 처리
  * DB와 Redis 캐시 연동
* 데이터베이스 (MySQL)
  * 이벤트, 사용자, 관리자 정보 저장
* 캐시 서버 (Redis)
  * 유저 이벤트 목록 조회(GET)
  * 이벤트 목록 조회(GET)
