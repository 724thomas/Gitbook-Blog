---
description: URL 단축 서비스
---

# Url Shortener API

## 문제

URL 단축 서비스를 제공하는 API를 설계하는 상황을 가정하고, 아래의 요구사항을 만족하는 API를 설계해 주세요.



## 요구사항

1. 사용자가 원본 URL을 입력하면 단축된 URL을 반환해야 한다.
2. 단축된 URL은 일정한 길이를 유지해야 한다.
3. 동일한 원본 URL에 대한 항상 동일한 단축 URL을 반환할지, 매번 새로운 단축 URL을 생성할지 선택할 수 있어야 한다.
4. 사용자가 단축된 URL을 입력하면 원본 URL로 리디렉션해야 한다.
5. 리디렉션 요청이 오면 해당 단축 URL의 조회수를 증가시켜야 한다.
6. 특정 단축 URL에 대한 조회수를 조회할 수 있어야 한다.
7. 사용자는 자신의 단축 URL 목록을 조회할 수 있어야 한다.
8. 단축 URL을 생성할 때 만료 기한을 설정할 수 있어야 한다.
9. 만료된 단축 URL은 더 이상 접근할 수 없어야 한다.



## 1. 기능정의

* 단축 URL 생성 (POST)
  * 사용자가 단축된 URL을 생성할 수 있음
  * 단축된 URL을 새로 생성할지, 기존에 있는걸 반환할지 선택할 수 있음
  * 단축된 URL의 만료 기한을 설정할 수 있다.
  * 성공하면 URL을 반환
* URL 리디렉션 (GET)
  * 입력된 단축된 URL을 입력하면 원본 URL로 리디렉션
  * 조회수 증가
  * 만료된 URL은 접근 불가
* URL 조회수조회 (GET)
  * 단축 URL에 대한 조회가 가능
* 유저의 모든 URL 조회 (GET)
* 유저의 특정 URL 만료 기간 설정(PUT)



## 2. 엔드포인트 설계

* 단축 URL 생성
  * POST
  * /urls
* URL 리디렉션
  * GET
  * /s/{shortenUrl}
* URL 조회
  * GET
  * /urls/{urlId}
* 유저의 모든 URL 조회
  * GET
  * /users/{userId}/urls
* 유저의 특정 URL 만료기간 설정
  * PUT
  * /urls/{urlId}/expire



## 3, 요청/응답 구조 설계

* 단축 URL 생성
*

    URL 리디렉션 (GET)

    * 입력된 단축된 URL을 입력하면 원본 URL로 리디렉션
    * 조회수 증가
    * 만료된 URL은 접근 불가

```json
Req
POST /urls
Content-Type: application/json
{
    "originalUrl": "www.example.com/asdfqwerzxcvasdfqwerzxcv",
    "createNew": true,
    "expireDate": "2026-01-01T00:00:00Z"
}

Res (create new)
HTTP/1.1 201 Created
Content-Type: application/json
{
    "urlId": 2,
    "shortenUrl": "www.example.com/short/B852",
    "expireDate": "2016-01-01T00:00:00Z"
}

Res (get original)
HTTP/1.1 200 OK
Content-Type: application/json
{
    "urlId": 1,
    "shortenUrl": "www.example.com/short/B842",
    "expireDate": "2015-07-01T00:00:00Z"
}
```

* URL 리디렉션

```json
Req
GET /s/B852

Res (success)
HTTP/1.1 302 Found
Location: https://www.example.com/asdfqwerzxcvasdfqwerzxcv

Res (fail)
HTTP/1.1 410 Gone
{
    "error": "This shortened URL has expired."
}
```

* URL 조회

```json
Req
GET /urls/1

Res
HTTP/1.1 200 OK
Content-Type: application/json
{
    "urlId": 1,
    "originalUrl": "www.example.com/asdfqwerzxcvasdfqwerzxcv",
    "shortenUrl": "www.example.com/short/B842",
    "views" 1,
    "expireDate": "2016-01-01T00:00:00Z"
}
```

* 유저의 모든 URL 조회

```json
Req
GET /users/1/urls

Res
HTTP/1.1 200 OK
Content-Type: application/json
[
    {
        "urlId": 1,
        "originalUrl": "www.example.com/asdfqwerzxcvasdfqwerzxcv",
        "shortenUrl": "www.example.com/short/B842",
        "views" 1,
        "expireDate": "2026-01-01T00:00:00Z"
    }, ...
]
```

* 유저의 특정 URL 만료기간 설정

```json
Req
PUT /urls/1/expire
Content-Type: application/json
{
    "expireDate": "2027-01-01T00:00:00Z"
}

Res
HTTP/1.1 200 OK
Content-Type: application/json
{
    "urlId": 1,
    "originalUrl": "www.example.com/123456",
    "shortenUrl": "www.example.com/short/B842",
    "views" 1,
    "expireDate": "2027-01-01T00:00:00Z"
}
```



## 4. DB 모델링

* user
  * id. BIGINT, PK. 유저 id
* url
  * id. BIGINT, PK. url id
  * user\_id. BIGINT, FK, 유저 id
  * original\_url. VARCHAR(255), 기존 url (index)
  * shorten\_url. VARCHAR(255), 수정 url
  * expire\_date. DATETIME. 만료날짜 (batch delete)
  * views. BIGINT. 조회수
  * created\_at. DATETIME. 생성날짜
  * modified\_at. DATETIME. 수정날짜



## 5. 아키텍처 고려

Redis, RDB 활용

redis - originalUrl, shortenUrl에 대한 값을 캐싱, 조회수 카운트

rdb - 유저 정보와 url 관련 정보를 저장하기 위함

* 단축 URL 생성 프로세스
  * originalUrl에 대해 Redis에서 캐시 확인 (존재하면 즉시 반환).
  * 존재하지 않으면 새 URL을 생성하고 RDB에 저장 후 Redis 업데이트.
  * 결과를 반환.
* 리디렉션 요청 처리
  * shortenUrl을 Redis에서 확인 (존재하면 바로 원본 URL 반환).
  * 존재하지 않으면 RDB에서 조회 후 Redis에 캐싱.
  * views 증가 처리 (Redis에서 INCR 후 비동기적으로 DB 반영).

