---
description: API 엔드포인트
---

# Api Endpoints

## 회원

* 회원가입
* 로그인
* 비밀번호 재설정
* Oauth
* 소셜 로그인



## 상품 관리

* 상품 이미지 등록 - POST /api/v1/product-image

```json
Req
POST /api/v1/product-image
Content-type: multipart/form-data
{
    "fileName": "image1.jpg",
    "contentType": "image/jpeg"
}

Res
HTTP/1.1 201 Created
{
    "imageId": "randumUUID",
    "imageKey": "key",
    "imageUrl": "https://image.s3/product/randomUUID"
}
```



* 상품 조회 - GET /api/v1/products?categoryId={categoryId}\&minPrice={minprice}\&maxPrice={maxprice}\&sort={sortType},{sortBy}\&page={page}\&size={size}

{% code fullWidth="true" %}
```json
Req
GET /api/v1/products?categoryId=3&minPrice=5000&maxPrice=20000&sort=price,asc&page=0&size=10

Res
Http/1.1 200 OK
{    
    "totalPages": 5,
    "currentPage": 0,
    "totalElements" 42,
    "currentSize": 10,
    "productSearchResponse": [
        {
            "id": 1,
            "productName": "무선 이어폰",
            "price": 129000,
            "thumbnailUrl": "https://example.com/images/1.jpg",
            "categoryName": "electronics"
        }, ...
    ]
}
```
{% endcode %}



* 상품 등록 (Admin) - POST /api/v1/products

```json
Req
POST /api/v1/products
Content-type: application/json
{
    "name": "무선 이어폰",
    "description": "고품질 블루투스 아이폰",
    "price": 129000,
    "stockQuantity": 100,
    "categoryId":1,
    "imageIds": ["UUID", "UUID2",..],
    "imageUrls": ["url", "url2",...],
    "imageKeys": ["key", "key2",...],
    "isThumbnail": ["true", "false",...],
    "imageSortOrder": [-1, 1,...]
}

Res
Http/1.1 201 Created
{
    "id": 1,
    "createdAt": "2025-04-11T14:00:00"
}
```

* 상품 수정 (Admin) - PUT /api/v1/products/{productId}

```json
Req
PUT /api/v1/products/1
Content-type: application/json
{
    "name": "무선 이어폰",
    "description": "고품질 블루투스 아이폰",
    "price": 129000,
    "stockQuantity": 100,
    "categoryId":1
}

Res
HTTP/1.1 200 OK
{
    "id": 1,
    "updatedAt": "2025-04-11T17:00:00"
}
```

* 상품 삭제 (Admin) - DELETE /api/v1/products/{productId}

```json
Req
DELETE /api/v1/products/{productId}

Res
HTTP/1.1 204 No Content / 404 Not Found
```



## 카테고리 관리

* 카테고리 생성 (Admin)
* 카테고리 수정 (Admin)



## 상품 주문

* 상품 주문하기
* 장바구니 상품 주문하기



## 장바구니

* 장바구니 조회

```json
REQ
GET /api/v1/cart

RES
HTTP/1.1 200 OK
{
  "dateTime": "2025-05-09",
  "status": {
    "code": "0000",
    "message": "정상 처리되었습니다."
  },
  "data": {
    "cartProductList": [
      {
        "productSummary": {
          "id": 1,
          "productName": "testName",
          "price": 50000,
          "thumbnailUrl": "http://localhost:9000/images/products/9b1a2ca7-b978-4ced-a11a-b86cc4386772.png",
          "soldOut": false
        },
        "quantity": 5,
        "status": "AVAILABLE",
        "productTotalPrice": 250000
      }
    ],
    "cartTotalPrice": 250000
  }
}
```

* 상품 추가

```json
Req
POST /api/v1/cart/products
Content-type: application/json
{
  "productId": 1,
  "addQuantity": 1
}


Res
HTTP/1.1 200 OK
{
  "dateTime": "2025-05-09",
  "status": {
    "code": "0000",
    "message": "정상 처리되었습니다."
  },
  "data": {
    "cartProductList": [
      {
        "productSummary": {
          "id": 1,
          "productName": "testName",
          "price": 50000,
          "thumbnailUrl": "http://localhost:9000/images/products/9b1a2ca7-b978-4ced-a11a-b86cc4386772.png",
          "soldOut": false
        },
        "quantity": 7,
        "status": "AVAILABLE",
        "productTotalPrice": 350000
      }
    ],
    "cartTotalPrice": 350000
  }
}
```

* 상품 수량 수정

```json
Req
PUT /api/v1/cart/products/1
Content-type: application/json
{
  "quantity": 1
}


Res
HTTP/1.1 200 OK
{
  "dateTime": "2025-05-09",
  "status": {
    "code": "0000",
    "message": "정상 처리되었습니다."
  },
  "data": {
    "cartProductList": [
      {
        "productSummary": {
          "id": 1,
          "productName": "testName",
          "price": 50000,
          "thumbnailUrl": "http://localhost:9000/images/products/9b1a2ca7-b978-4ced-a11a-b86cc4386772.png",
          "soldOut": false
        },
        "quantity": 1,
        "status": "AVAILABLE",
        "productTotalPrice": 50000
      }
    ],
    "cartTotalPrice": 50000
  }
}
```

* 상품 제거

```
Req
DELETE /api/v1/cart/products/1

Res
HTTP/1.1 200 OK
{
    "dateTime": "2025-05-09",
    "status": {
        "code": "0000",
        "message": "정상 처리되었습니다."
    },
    "data": {
        "cartProductList": [],
        "cartTotalPrice": 0
    }
}
```



## 배송

* 배송 조회
* 배송 생성
* 배송 업데이트



## 리뷰&#x20;

* 리뷰 전체 조회  (Admin)
* 회원 리뷰 조회  (Admin)
* 내 리뷰 전체조회
* 내 리뷰 조회
* 리뷰 등록
* 리뷰 수정
* 리뷰 삭제



## 고객센터

* 문의 전체 조회 (Admin)
* 회원 문의 조회 (Admin)
* 내 문의 전체 조회
* 내 문의 조회



## 프로모션

* 프로모션 조회
* 프로모션 등록 (Admin)
* 프로모션 수정 (Admin)
* 프로모션 삭제 (Admin)



## 쿠폰

* 쿠폰 전체 조회 (Admin)
* 쿠폰 조회 (Admin)
* 내 쿠폰 전체조회
* 내 쿠폰 조회
* 쿠폰 생성
* 쿠폰 수정
* 쿠폰 삭제



## 관리자

* 회원 전체 조회
* 회원 조회
* 통계 조회
* 공지 전체 조회
* 공지 조회
* 공지 등록
* 공지 수정
* 공지 삭제





