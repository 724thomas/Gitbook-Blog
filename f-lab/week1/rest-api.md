# REST API

## REST란

Representational State Transfer의 약자.\
네트워크 상에서 클라이언트와 서버 간의 상호작용을 정의하는 아키텍처 스타일. 웹의 기본 구조와 원칙에 기반하여 설계 되어있고, REST API는 이러한 원칙을 준수하는 api를 의미.



## 특징

* Stateless (무상태성)
* 클라이언트-서버 구조
* Uniform Interface
* 캐싱 가능
* Layered Architecture
* Code on Demand

### Stateless

* 서버는 클라이언트의 상태를 유지하지 않음
* 각 요청들은 독립적으로 처리되며, 필요한 모든 정보는 요청 자체에 포함. (쿠키, JWT)

### Client-Server 구조

* 클라이언트와 서버가 명확히 분리
* 클라이언트는 사용자 인터페이스를 처리, 서버는 데이터 처리를 담당
* 분리되어 있기때문에 독립적으로 확장하거나 변경 가능

### Uniform Interface

일관된 방식으로 자원을 식별.

* URI: 자원을 식별하는 고유 주소
* HTTP 메서드: GET, POST, PUT, PATCH, DELETE

### Cacheable

* 서버 응답은 캐싱이 가능해야 합니다.
* HTTP 헤더를 사용하여 응답이 캐싱 가능한지 명시.
* 응답에 Cache-Control 헤더를 포함(HTTP에서 캐싱 가능 여부와 캐싱 정책을 명시하는 헤더)
* Etag / Last-Modified 같은 헤더를 사용해 응답의 유효성 검증 필요

### Layered System

* 클라이언트와 서버는 직접 상호작용하는지 또는 중간 계층을 통해 상호작용하는지 알 필요가 없음
* 중간 계층을 두어 보안을 강화하거나 서버 확장으로 확장성을 높일 수 있음

### Code on Demand

* 필요에 따라 클라이언트가 서버로부터 실행 가능한 코드를 전달받아 실행할 수 있어야합니다.
* 예: 서버가 클라이언트로 javascript 코드를 전달하여 UI를 동적으로 업데이트.



## RESTful API란

REST의 제약 조건 6가지를 충실히 따르며, 표준 HTTP 사용 원칙을 준수하고 있는 api입니다.

* 무상태성, 클라이언트-서버 구조, 일관된 인터페이스, 캐시 처리 가능, 계층화된 시스템 아키텍처, Code on demand의 제약 조건 6가지를 따르고,
* URI로 자원을 unique하게 식별하며, HTTP 메서드로 자원에 대한 작업을 정의하고, HTTP 상태코드로 요청에 대한 결과를 코드로 전달해야합니다.
