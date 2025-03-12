---
description: 이벤트 티켓 예약 API 설계
---

# Event Ticket Reservation

## 문제

한 온라인 이벤트 플랫폼에서는 다양한 콘서트와 공연을 진행하며, 사용자는 티켓을 예약할 수 있습니다. 이 플랫폼의 티켓 예약 기능을 위한 API를 설계하세요.

## 요구사항

* 사용자는 특정 이벤트의 티켓을 예약할 수 있어야 합니다.
* 각 이벤트에는 최대 예약 가능한 티켓 수가 존재합니다.
* 예약이 성공하면 예약 ID가 반환되어야 합니다.
* 사용자는 자신의 예약을 조회할 수 있어야 합니다.
* 사용자는 자신의 예약을 취소할 수 있어야 합니다.
* 예약 취소 시, 해당 티켓은 다시 사용 가능해야 합니다.
* 단일 사용자당 하나의 이벤트에 대해 하나의 예약만 가능해야 합니다.





## 1. 기능정의

* 티켓예약 (POST)
  * 사용자가 이벤트의 티켓을 예약할 수 있음
  * 성공하면 예약 ID반환
  * 좌석이 부족하면 실패
* 예약조회 (GET)
  * 특정 사용자의 예약 내역 조회 가능
* 예약 취소 (DELETE)
  * 사용자가 본인의 예약을 취소할 수 있음
  * 취소된 티켓은 다시 사용 가능해야 함
* 이벤트 정보 조회 (GET)
  * 특정 이벤트의 티켓 잔여 수 조회 가능



## 2. 엔드포인트 설계

* 티켓 예약
  * POST
  * /events/{eventId}/reservations
* 예약 조회
  * GET
  * /users/{userId}/reverstaions
* 예약 취소
  * DELETE
  * /reservations/{reservationsId}
* 이벤트 정보 조회
  * GET
  * /events/{eventId}



## 3. 요청/응답 구조 설계

* 티켓 예약

```json
Req
POST /events/{eventId}/reservations
Content-Type: applicaiton/json
{
    "userId": 123
}

Res (Success)
HTTP/1.1 201 Created
Content-Type: application/json
{
    "reservationId": 456,
    "eventId": 1,
    "userId": 123,
    "status": "CONFIRMED"
}

Res (Fail)
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
    "error": "Tickets are sold out."
}
```

* 예약 조회

```json
Req
GET /users/{userId}/reservations

Res (Success)
HTTP/1.1 200 OK
[
    {
        "reservationId": 456,
        "eventId": 1,
        "eventName": "IU Concert",
        "status": "CONFIRMED"
    }, ...
]
```

* 예약 취소

```json
Req
DELETE /reservations/{reservationId}

Res (Success)
HTTP/1.1 200 OK
Content-Type: application/json
{
    "message": "Reservation cancelled successfully."
}

Res (Fail)
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
    "message": "Cannot find reservation"
}
```

* 이벤트 정보 조회

```json
Req
GET /events/{eventId}

Res (Success)
HTTP/1.1 200 OK
Content-Type: application/json
{
    "eventId": 1,
    "eventName": "IU Concert",
    "totalTickets": 500,
    "availableTickets": 120
}

Res (Fail)
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
    "message": "Cannot find event"
}
```



## 4. DB 모델링

* user
  * id. BIGINT, PK. 유저id
  * name. VARCHAR(255). 유저 이름
* event
  * id. BIGINT, PK. 이벤트id
  * name. VARCHAR(255), 이벤트 이름
  * total\_tickets. INT. 전체 티켓 수
  * availble\_tockets. INT. 남은 티켓 수
* reservation
  * id. BIGINT, PK. 예약 ID
  * user\_id. BIGINT, FK. 유저 ID
  * event\_id. BIGINT, FK. 이벤트 ID
  * status. VARCHAR. 예약 상태(CONFIRMED, CANCELLED)
  * created\_at. TIMESTAMP. 예약 생성일
  * modified\_at. TIMESTAMP. 예약 수정일



## 5. 아키텍처 고려

Redis, RDB 활용

redis - 수량을 카운트하고 단일 스레드로 동시성 문제를 피하기 위함

rdb - 수량을 카운트하고, 예약 정보를 저장하기 위함

* 예약 요청
  * 서버에서 요청을 받으면 redis에서 남은 수량 여부를 체크하고 반영합니다.
  * (수량이 남아있지 않으면 실패 처리  & 응답을 하게 됩니다.)
  * 수량이 남아있다면, eventListener로 db에 비동기적으로 reservation 정보와 남은수량을 저장합니다.

DB반영 실패시: fallback & recovery 패턴을 활용하여, 비동기 처리작업이 실패하면 실패 테이블에 데이터를 저장을 한 후에, batch작업을 통해 업데이트합니다. (실시간 일관성이 깨지는 단점)



