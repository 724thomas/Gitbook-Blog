---
description: 차량 관리 API 설계
---

# Car management API

## 개요

한 자동차 판매 관리 시스템의 백엔드 API를 설계하고자 한다. 관리 대상은 차량이며, 각 차량은 다음의 속성을 가진다:

* 브랜드 (`brand`)
* 모델명 (`model`)
* 색상 (`color`)
* 연비 (`gasEconomy`) – km/l
* 추가 장비 목록 (`accessories`) – 문자열 리스트

다음 요구사항을 만족하는 REST API를 설계하라.



## 요구사항

* **차량 목록 조회 API**
  * 모든 차량 데이터를 JSON 형식으로 조회할 수 있어야 한다.
* **차량 필터링 및 정렬 API**
  * 차량 목록을 아래 조건으로 필터링할 수 있어야 한다:
    * 브랜드 (`brand`)
    * 색상 (`color`)
    * 연비 범위 (`gasEconomy[min, max]`)
    * 특정 장비 포함 여부 (`accessories` 리스트 중 일부 포함)
  * 정렬 조건:
    * 연비 기준 오름차순 / 내림차순
    * 브랜드 이름 알파벳 순
  * 필터 조건은 **Query Parameter**로 전달
* **차량 등록 API**
  * 새로운 차량 정보를 등록할 수 있어야 하며, 유효성 검증이 포함되어야 한다.
  * 중복 차량은 등록되지 않도록 처리할 것 (`brand + model + color` 조합)
* **차량 삭제 API**
  * 차량 ID 기준으로 삭제할 수 있어야 하며, 없는 ID 요청 시 적절한 에러를 반환해야 한다.
* **차량 상세 조회 API**
  * 특정 차량의 상세 정보를 차량 ID로 조회할 수 있어야 한다.





## 1. 엔드포인트

* 차량 목록 조회 API (GET) /cars
* 차량 필터링 및 정렬 API (GET) /cars?brand={brand}\&color={color}\&mingaseconomy={min}\&maxgaseconomy={max}\&includeacc={includeacc}\&page={page}\&size={size}\&sort={sort}\&type={type}
* 차량 등록 API (POST) /cars
* 차량 삭제 API (DELETE) /cars/{carsId}
* 차량 상세 조회 API (GET) /cars/{carsId}





* 차량 목록 조회 API (GET) /cars

```json
Req
GET /cars

Res
HTTP/1.1 200 OK
Content-type: application/json
{
    [
        "carId": 1,
        "brand": "A",
        "color": "RED",
        "gasEconomy": 10,
        "accessories": {
            [
                "accessoryId": 1,
                "name": "acc1",
                "description", "description1"
            ], ...
        }
    ], ...
}
```



* 차량 필터링 및 정렬 API (GET) /cars?brand={brand}\&color={color}\&mingaseconomy={min}\&maxgaseconomy={max}\&includeacc={includeacc}\&page={page}\&size={size}\&sort={sort}\&type={type}

```json
Req
GET /cars?brand=A&color=red&mingaseconomy=1&maxgaseconomy=100&includeacc=true&page=1&size=10&sort=asc&type=brand

Res
HTTP/1.1 200 OK
Content-type: application/json
{    
    "total"= 100,
    "page"=1,
    "size"=10
    "cars": {
        [
            "carId": 1,
            "brand": "A",
            "color": "RED",
            "gasEconomy": 10,
            "accessories": {
                [
                    "accessoryId": 1,
                    "name": "acc1",
                    "description", "description1"
                ], ...
            }
        ], ...
    }
}
```



* 차량 등록 API (POST) /cars

```json
Req
POST /cars
Content-type: application/json
{
    "brand": "A",
    "color": "RED",
    "gasEconomy": 10,
    "accessories": {
        [
            "accessoryId": 1,
            "name": "acc1",
            "description", "description1"
        ], ...
    }
}

Res
HTTP/1.1 201 CREATED
Content-type: application/json
{
    "carId": 1,
    "brand": "A",
    "color": "RED",
    "gasEconomy": 10,
    "accessories": {
        [
            "accessoryId": 1,
            "name": "acc1",
            "description", "description1"
        ], ...
    }
} 

또는

{
    "carId": 1
}
```



* 차량 삭제 API (DELETE) /cars/{carsId}

```json
Req
DELETE /cars/1

Res
HTTP/1.1 200 OK
Content-type: application/json
{
    "carId":1,
    "message": "success"
}
```



* 차량 상세 조회 API (GET) /cars/{carsId}

```json
Req
GET /cars/1

Res
HTTP/1.1 200 OK
Content-type: application/json
{
    "carId": 1,
    "brand": "A",
    "color": "RED",
    "gasEconomy": 10,
    "accessories": {
        [
            "accessoryId": 1,
            "name": "acc1",
            "description", "description1"
        ], ...
    }
} 
```



## 2. DB 스키마

* car
  * car\_id. BIGINT, PK.
  * brand\_id. BIGINT, FK.
  * gas\_economy. BIGINT.
  * color. VARCHAR(255)
* brand
  * brand\_id. BIGINT, PK
  * name. TEXT
* accessories
  * accessory\_id. BIGINT, PK
  * name. TEXT
  * description. TEXT
* car\_accessories
  * car\_accessories\_id. BIGINT, PK
  * car\_id. BIGINT FK
  * accessories\_id. BIGINT FK



## 3. 아키텍처

* Server
  * 사용자 요청 처리
  * DB 연동
* MySql
  * 데이터 저장
* Redis
  * 캐싱.
