---
description: 리뷰 시스템 API
---

# Review System API

## 개요

어떤 서비스(예: 쇼핑몰, 레스토랑 예약, 콘텐츠 플랫폼 등)에서 사용자들이 **상품/콘텐츠에 대한 리뷰를 작성, 조회, 수정, 삭제**할 수 있는 API를 설계해보자. 리뷰에는 평점과 텍스트가 포함되며, 리뷰에 대한 **댓글 기능**도 존재한다.



## 요구사항

* **리뷰 작성 API**
  * 사용자 ID, 상품 ID, 평점(1\~5), 리뷰 내용 입력
* **리뷰 조회 API**
  * 특정 상품에 달린 리뷰 목록 조회 (페이징 지원)
  * 정렬 방식(최신순/평점순) 지원
* **리뷰 수정 및 삭제 API**
  * 작성자 본인만 수정/삭제 가능
* **댓글 작성/조회 API**
  * 리뷰에 달린 댓글 작성/조회 기능
  * 댓글은 대댓글을 지원하지 않음 (1 depth)
* **리뷰 통계 API**
  * 상품에 대한 평균 평점, 총 리뷰 수 조회

asdfasdf

## 1. 엔드포인트 설계

* 리뷰 작성(POST) - /posts/{postId}/reviews
* 리뷰 조회(GET) - /posts/{postId}/reviews?page={page}\&size={size}\&sort={sort}
* 리뷰 수정(PUT) - /reviews/{reviewId}
* 리뷰 삭제(DELETE) - /reviews/{reviewId}
* 댓글 작성(POST) - /reviews/{reviewId}/comments
* 댓글 조회(GET) - /reviews/{reviewId}/comments?page={page}\&size={size}
* 리뷰 통계(GET) - /posts/{postId}/stats



## 2. 요청/응답 설계

*   리뷰 작성(POST) - /posts/{postId}/reviews

    ```json
    Req
    POST /posts/1/reviews
    Content-type: application/json
    {
        "score": 1,
        "content": "useful"
    }

    Res
    HTTP/1.1 201 CREATED
    Content-type: application/json
    {
        "reviewId": 1
    }
    ```
*   리뷰 조회(GET) - /posts/{postId}/reviews?page={page}\&size={size}\&sort={sort}

    ```json
    Req
    GET /posts/1/reviews?page=1&size=10&sort=latest

    Res
    HTTP/1.1 200 OK
    Content-type: application/json
    {
        "postId": 1,
        "page": 1,
        "size": 10,
        "totalPage": 1,
        "totalElements": 2,
        "sortOrder": "latest"
        "reviews": [
            {
                "userId": 1,
                "reviewId": 1,
                "score": 1,
                "content": "useful"
            }, ...
        ]
    }
    ```


*   리뷰 수정(PUT) - /reviews/{reviewId}

    ```json
    Req
    PUT /reviews/1
    Content-type: application/json
    {
        "score": 1,
        "content": "useful"
    }

    Res
    HTTP/1.1 200 OK
    Content-type: application/json
    {
        "reviewId": 1,
        "score": 1,
        "content": "useful"
    }
    ```
*   리뷰 삭제(DELETE) - /reviews/{reviewId}

    ```json
    Req
    DELETE reviews/1

    Res
    HTTP/1.1 200 OK
    Content-type: application/json
    {
        "reviewId": 1
        "message": "deleted successfully"
    }
    ```


*   댓글 작성(POST) - /reviews/{reviewId}/comments

    ```json
    Req
    POST /reviews/1/comments
    Content-type: application/json
    {
        "content": "good"
    }

    Res
    HTTP/1.1 201 CREATED
    Content-type: application/json
    {
        "commentId": 1
    }
    ```


*   댓글 조회(GET) - /reviews/{reviewId}/comments?page={page}\&size={size}

    ```json
    Req
    GET /reviews/1/comments?page=1&size=10

    Res
    HTTP/1.1 200 OK
    Content-type: application/json
    {
        "reviewId": 1,
        "page": 1,
        "size": 10,
        "totalPage": 1,
        "totalElements": 2,
        "comments": {
            [
                "commentId": 1,
                "userId": 1,
                "content": "useful"
            ], ...
        }
    }
    ```


*   리뷰 통계(GET) - /posts/{postId}/stats

    ```json
    Req
    GET /posts/1/stats

    Res
    HTTP/1.1 200 OK
    Content-type: application/json
    {
        "postId": 1,
        "page": 1,
        "size": 10,
        "totalPage": 1,
        "totalElements": 2,
        "sortOrder": "latest"
        "averageScore": 1,
        "totalReviews": 1,
        "totalComments": 1,
        "reviews": {
            [
                "userId": 1,
                "reviewId": 1,
                "score": 1,
                "content": "useful"
            ], ...
        }
    ```





## 3. DB 스키마 설계

* user
  * user\_id. BIGINT, PK. 유저 id
* post
  * post\_id. BIGINT, PK. 게시글 id
  * user\_id. BIGINT, FK. 유저 id
  * content. TEXT. 내용
* review
  * review\_id. BIGINT, PK. 리뷰 id
  * user\_id. BIGINT, FK. 유저 id
  * post\_id. BIGINT, FK. 게시글 id
  * score. SMALLINT. 점수
  * content. TEXT. 내용
* comment
  * comment\_id. BIGINT, PK. 댓글 id
  * user\_id. BIGINT, FK. 유저 id
  * review\_id. BIGINT, FK. 리뷰 id
  * content. TEXT. 내용



## 4. 아키텍처 설계

* API 서버(SpringBoot)
  * 사용자 요청 처리
  * DB와 Redis 캐시 연동
* 데이터베이스 (MySQL)
  * 사용자, 작업 정보 저장
* 캐시 서버 (Redis)
  * 리뷰, 댓글, 통계 같은 GET 방식의 API 캐싱
