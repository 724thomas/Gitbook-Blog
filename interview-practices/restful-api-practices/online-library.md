# Online Library

## 개요

한 도서관에서 사용자가 책을 대여하고 반납할 수 있는 시스템을 구축하려고 한다.



## 요구사항

* **도서 등록 API**
  * 관리자가 책의 정보를 등록할 수 있어야 해 (제목, 저자, ISBN, 출판일 등).
  * 같은 책이더라도 실물이 다르면 등록할 수 있어야 해.
* **도서 수정 API**
  * 관리자는 책의 정보를 수정할 수 있어야 해.
* **도서 삭제 API**
  * 관리자는 책의 정보를 삭제할 수 있어야 해.
* **도서 목록 조회 API**
  * 사용자가 전체 도서를 검색할 수 있어야 해.
  * 제목, 저자 등으로 필터링 가능해야 해.
* **도서 대여 API**
  * 사용자가 특정 도서를 대여할 수 있어야 해.
  * 한 명의 사용자는 동일한 책을 중복 대여할 수 없어야 해.
  * 재고가 있어야 해
* **도서 반납 API**
  * 사용자가 대여한 책을 반납할 수 있어야 해. (반납 시점 기록)
* **대여 내역 조회 API**
  * 사용자는 본인의 대여 내역을 조회할 수 있어야 해.
  * 관리자는 전체 대여 기록을 조회할 수 있어야 해.



## 엔드포인트

* 도서 정보 등록 (관리자) **-  POST /**&#x61;pi/book-infos
  * 관리자가 책의 정보를 등록할 수 있어야 해 (제목, 저자, ISBN, 출판일 등).
  * 같은 책이더라도 실물이 다르면 등록할 수 있어야 해.

```json
Req
POST /api/book-infos
Content-type: application/json
{
  "isbn": "978-616-90393-3-4",
  "title": "Java의 정석",
  "authors": ["남궁성"],
  "publicationDate": "2024-01-01",
  "bookImageUrl": "https://example.com/image/1"
}

Res
Http/1.1 Created 201
Content-type: application/json
{
  "bookInfoId": 1
}
```

* 도서 정보 수정 (관리자) **- PATCH** api/book-infos/{bookInfoId}
  * 관리자는 책의 정보를 수정할 수 있어야 해.

```json
Req
PATCH api/book-infos/1
Content-type: application/json
{
  "title": "Java의 정석 (개정판)",
  "authors": ["남궁성"],
  "publicationDate": "2024-06-01",
  "bookImageUrl": "https://example.com/image/2"
}


Res
HTTP/1.1 200 OK
Content-type: application/json
{
  "bookInfoId": 1,
  "message": "도서 정보가 수정되었습니다."
}
```

* 도서 정보 삭제 (관리자) **- DELETE** api/book-infos/{bookInfoId}
  * 관리자는 책의 정보를 삭제할 수 있어야 해.

```json
Req
DELETE /api/book-infos/1

Res
HTTP/1.1 200 OK
Content-type: application/json
{
  "message": "도서 정보가 삭제되었습니다."
}
```

* 도서 정보 목록 조회 **- GET /**&#x61;pi/book-infos
  * 사용자가 전체 도서를 검색할 수 있어야 해.
  * 제목, 저자 등으로 필터링 가능해야 해.

{% code fullWidth="false" %}
```json
REQ
GET /api/book-infos?title=java
    &searchType=contains
    &authors=남궁성
    &available=true
    &sort=title,asc
    &page=0
    &size=10
// @GetMapping("/api/book")
// public List<BookResponse> getBooks(
//     @RequestParam(required = false) String bookTitle,
//     @RequestParam(required = false, defaultValue = "contains") String searchType,
//     @RequestParam(required = false) List<String> bookAuthors,
//     @RequestParam(required = false) Boolean available,
//     @PageableDefault(size = 20, sort = "bookTitle") Pageable pageable

RES
HTTP/1.1 200 OK
Content-type: application/json
{
  "totalPages": 2,
  "currentPage": 0,
  "totalElements": 15,
  "bookInfos": [
    {
      "bookInfoId": 1,
      "isbn": "978-616-90393-3-4",
      "title": "Java의 정석",
      "authors": ["남궁성"],
      "publicationDate": "2024-01-01",
      "bookImageUrl": "https://example.com/image/1",
      "totalStock": 5,
      "availableStock": 3
    }
  ]
}

```
{% endcode %}

* 해당 도서에 실물 책 추가 (N권 등록) - POST /api/book-infos/{bookInfoId}/books

```json
Req
POST /api/book-infos/1/books
Content-type: application/json
{
  "quantity": 5
}

Res
HTTP/1.1 201 Created
Content-type: application/json
{
  "createdBookIds": [101, 102, 103, 104, 105]
}

```

* 특정 도서의 실물 재고 조회 - GET /api/book-infos/{bookInfoId}/books

```json
Req
GET /api/book-infos/1/books

Res
HTTP/1.1 200 OK
Content-type: application/json
{
  "bookInfoId": 1,
  "books": [
    {
      "bookId": 101,
      "isAvailable": true
    },
    {
      "bookId": 102,
      "isAvailable": false
    }
  ]
}

```

* 사용자 도서 대여 요청 **- POST** /api/borrows
  * 사용자가 특정 도서를 대여할 수 있어야 해.
  * 한 명의 사용자는 동일한 책을 중복 대여할 수 없어야 해.
  * 재고가 있어야 해

```json
Req
POST /api/borrows
Content-type: application/json
{
  "bookInfoId": 1
}

Res (성공)
HTTP/1.1 200 OK
Content-type: application/json
{
  "borrowId": 2001,
  "bookId": 101,
  "dueDate": "2025-04-21"
}

Res (대여 중복)
HTTP/1.1 400 Bad Request
Content-type: application/json
{
  "code": "ALREADY_BORROWED",
  "message": "이미 해당 도서를 대여 중입니다."
}

Res (재고 없음)
HTTP/1.1 409 Conflict
Content-type: application/json
{
  "code": "OUT_OF_STOCK",
  "message": "해당 도서의 재고가 없습니다."
}
```

* 사용자 도서 반납 요청 **- POST** /api/returns
  * 사용자가 대여한 책을 반납할 수 있어야 해. (반납 시점 기록)

```json
Req
POST /api/returns
Content-type: application/json
{
  "bookInfoId": 1
}

Res
HTTP/1.1 200 OK
Content-type: application/json
{
  "message": "도서가 반납되었습니다.",
  "returnedAt": "2025-04-14T15:22:00"
}
```

* 사용자 or 관리자 대여 기록 조회 - GET /api/borrow-histories?me={me}\&returned={returned}\&page={page}\&size={size}
  * 사용자는 본인의 대여 내역을 조회할 수 있어야 해.
  * 관리자는 전체 대여 기록을 조회할 수 있어야 해.

```json
Req
GET /api/borrow-histories?me=true&returned=false&page=0&size=10

// @GetMapping("/api/borrow-histories")
// public ResponseEntity<BorrowHistoryPageResponse> getBorrowHistories(
//        @AuthenticationPrincipal UserPrincipal user,
//        @RequestParam(defaultValue = "true") boolean me,
//        @RequestParam(required = false) Boolean returned,
//        @PageableDefault(size = 10, sort = "borrowDate", direction = DESC) Pageable pageable)

Res
HTTP/1.1 200 OK
Content-type: application/json
{
  "totalPages": 1,
  "currentPage": 0,
  "borrowHistories": [
    {
      "borrowId": 2001,
      "bookInfoId": 1,
      "title": "Java의 정석",
      "authors": ["남궁성"],
      "borrowDate": "2025-04-14T12:00:00",
      "dueDate": "2025-04-21T12:00:00",
      "returned": false
    }
  ]
}
```



## DB 스키마

user

* id. BIGINT, PK.
* username. Varchar(255)

book\_img

* id.BIGINT, PK.
* url. VARCHAR(255).

book\_info

* id. BIGINT, PK.
* book\_title. Varchar(255)
* book\_bookAuthor. BIGINT
* publication\_date. DateTime
* book\_img. VARCHAR(255)
* total\_stock. BIGINT
* available\_stock. BIGINT

book

* id. BIGINT, PK
* book\_info\_id. BIGINT, FK
* available. BOOLEAN

book\_user

* id. BIGINT, PK
* user\_id. BIGINT, FK
* book\_id. BIGINT, FK



## 아키텍처 고려사항

#### 트랜잭션 처리

* `대여 시`: book row에 `FOR UPDATE` 잠금 → 재고 감소 및 대여 기록 생성
* `반납 시`: 해당 borrow record 조회 후 반납 처리

#### Redis 캐시 활용

* 인기 도서 목록 캐시 (book\_info 기준)
* 도서 상세 페이지 조회 캐싱

#### 인증/인가

* `POST /api/borrows`, `returns`, `borrow-histories`는 로그인 사용자 기준
* 관리자 권한: `/api/book-infos` 등록/수정/삭제 등

#### ⚠예외 처리

* 동일한 `bookInfoId` 중복 대여 제한 (Unique 제약 or validation)
* 동시성 문제 방지를 위한 Optimistic Lock or DB Lock 고려
