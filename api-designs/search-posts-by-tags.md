---
description: 게시글 목록 조회
---

# Search posts by tags

## 개요

사용자가 특정 태그를 검색하여 관련된 게시글 목록을 조회할 수 있는 API를 설계하세요.

## 요구사항

* 게시글에는 `title`, `content`, 그리고 여러 개의 `tags`가 포함된다.
* 게시글은 생성, 조회가 가능해야 한다.
* 게시글은 여러 개의 태그를 가질 수 있으며, 태그는 중복 없이 관리되어야 한다.
* 사용자는 특정 태그를 기준으로 게시글을 검색할 수 있어야 한다.



## 1. 엔드포인트 설계

* 게시글 생성(POST) - /posts
* 게시글 조회(GET) - /posts/tags?{tags}



## 2. 요청/응답 구조 설계

* 게시글 생성

```json
Req
POST /posts
Content-Type: Application/json
{
    "title": "Java",
    "content": "it's programming language",
    "tags": ["Programming", "Language"],
    "userId": 1
}

Res (Success)
HTTP/1.1 201 Created
{
    "postId": 1,
    "title": "Java",
    "tags": ["Programming", "Language"]
    "content": "it's programming language",
    "createdAt": "2025-03-17",
    "userId" 1
}
```

* 게시글 조회

```json
Req
GET /posts/tags?Programming,Language

Res (Success)
HTTP/1.1 200 OK
[
    {
        "postId": 1,
        "title": "Java",
        "tags": ["Programming", "Language"]
        "content": "it's programming language",
        "createdAt": "2025-03-17",
        "userId" 1
    }
]
```



## 3. DB 모델링

* user
  * id. BIGINT, PK. 유저 id
* posts
  * id. BIGINT, PK, 게시글 id
  * userId. BIGINT, FK. 유저 id
  * title. VARCHAR(255). 게시글 제목
  * content. TEXT. 게시글 내용
  * createdAt. DATETIME. 생성일
  * modifiedAt. DATETIME. 수정일
* postsToTags
  * id. BIGINT, PK. 게시글-태그 id
  * postId. BIGINT, FK. 게시글 id
  * tagId. BIGINT, FK. 태그 id
* Tags
  * id. BIGINT, PK. 태그 id
  * name. VARCHAR(255). 태그 이름



## 4. 아키텍처 고려

redis와 rdb를 사용

redis는 캐싱 용도

* 새로운 게시글이 등록되면 캐싱
* 게시글이 수정되면 캐싱을 지우고 다시 등록

