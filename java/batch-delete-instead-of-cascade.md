---
description: Cascade 대신 Batch 삭제를 사용한 이유
---

# Batch delete instead of Cascade

스프링부트 JPA를 사용해 애플리케이션을 개발할때, 연관된 데이터를 삭제하는 방법에 따라 성능에 영향을 미칩니다. 일반적으로 연관된 데이터를 삭제시 CascadeType.ALL 또는 cascadeType.REMOVE를 사용하지만 삭제해야하는 데이터가 많아질수록 성능 차이가 발생합니다.



### 프로젝트 개요

회원 삭제(Hard) 기능이 필요했습니다. 해당 회원 삭제를 할시에 같이 삭제돼야했던 것들은 다음과 같습니다.

#### 삭제

* 회원 관련정보
* 회원 게시물과 게시물 정보
* 회원 게시물의 댓글과 댓글 정보
* 회원 게시물의 댓글의 대댓글과 대댓글 정보
* 회원 플레이리스트(게시물들의 모음)
* 회원 댓글과 댓글 정보
* 회원 댓글의 대댓글과 대댓글 정보
* 팔로잉 정보
* FCM에서의 구독 정보 등



회원 삭제는 빈번히 발생하는 작업이 아닙니다. (제가 알기로는 일반적인 서비스 탈퇴율은 2%정도 인걸로 알고 있습니다.) 하지만 한번의 회원 삭제가 관련된 많은 테이블들에 영향을 주기때문에 성능에 문제가 발생할 것 같았습니다.

실제로 몇몇 회원들의 데이터가 압도적으로 많았기때문에 고민을 하게 됐습니다.



### Cascade를 통한삭제

#### 장점:

* 간편함: 간단한 삭제로직
* 일관성: 모든 데이터가 자동으로 삭제되어서 일관성을 쉽게 유지할 수 있습니다.

#### 단점:

* 성능 저하: JPA가 연관된 모든 엔터티를 순차적으로(재귀적으로) 삭제하기때문에 시간이 오래 걸립니다.
* 연결된특정  엔터티의 정보를 삭제하고 싶지 않을시에는 적절하지 않습니다.

Cascade는 연관된 모든 엔터티를 자동으로 삭제해줘서 편하게 사용할 수 있습니다. 예를 들어, 회원 엔터티에 게시물 엔터티와 댓글 엔터티가 연결되어 있을 때, 회원 엔터티를 삭제하면 연관된 모든 게시물과 댓글도 자동 삭제됩니다.

하지만 성능 저하가 발생할 수 있는데, 그 이유는 대량의 데이터를 삭제할 때, JPA가 연관된 모든 엔터티를 **순차적**으로(재귀적으로)  삭제하며, 모든 연관 엔터티가 삭제됩니다.&#x20;

### Batch를 통한 삭제

#### 장점:

* 성능 향상: 효율적으로 대량의 데이터를 삭제할 수 있습니다.
* 삭제하고 싶은 데이터를 정확하게 제어할 수 있습니다.
* 오류 발생시, Chunk 단위로 롤백할 수 있습니다.

#### 단점:

* 복잡한 구현: 직접 로직을 구현해야하기때문에, 배치 사이즈와 순서를 정하기가 복잡합니다.
* 유지보수 어려움: 새로운 연관관계가 생기면 삭제 로직에 추가되어야합니다.

배치 삭제를사용하면 한번에 여러 레코드를 삭제하여 성능을 최적화할 수 있습니다. 삭제할 데이터를 미리 수집하고, 이를 배치 크기 단위로 나누어 한 번에 삭제하는 방식입니다. 이는 데이터베이스와의 통신 횟수를 줄이고, 성능을 향상시킬 수 있습니다. 만약 오류 발생시, 해당 chunk만 롤백하여 데이터 무결성을 보호할 수 있습니다.

하지만 단점으로는 연관 데이터 삭제 로직을 직접 관리해야하고, 관계가 추가될떄마다 삭제 로직도 수정돼야합니다.



### Cascade 제거 후 배치 로직

#### 데이터 수집

* 삭제할 회원과 연관된 데이터들을 수집합니다. (게시물, 댓글, 대댓글, ... FCM 구독 정보)

#### 삭제 순서 정의

* 데이터 무결성을 유지하기 위해 삭제 순서를 자식 엔터티부터 시작하였습니다.
* 예를 들어, 대댓글 -> 댓글 -> 게시물 순서.

#### 배치 크기 설정

* 배치 크기는 성능과 데이터 일관성에 큰 영향을 미치기때문에 적당한 트랜잭션 크기를 설정했습니다. 트랜잭션이 너무 크면 롤백할 때 성능 저하가 발생할 수 있기때문에 배치 크기를 설정했습니다.

#### 배치 삭제 실행

* 배치 크기 단위로 나누어서 한번에 삭제했습니다.
* 삭제 순서에 따라 배치로 삭제를 진행했습니다

#### 배치 삭제 실패시

* 실패한 데이터의 정보를 저장하고, 재시도 로직과 오류 로그를 기록했습니다.



### 결론

스프링부트 JPA에서 Cascade를 제거하고 배치 삭제를 사용하면, 대량의 데이터를 삭제할 때 성능을 크게 향상시킬 수 있습니다. 그러나 코드의 복잡성과 유지보수의 어려움이 따를 수 있습니다.

회원 삭제 기능에 대해 배치를 사용하지 않아도 당장 의미가큰있지는 않겠지만,  서비스가 계속 운영됨에 따라, 각 회원들의 데이터들이 커지게 됩니다. 이는 추후에 성능적으로 이슈가 될 수도 있는 부분이기떄문에 cascade 삭제가 아닌 batch 삭제를 선택하게 됐습니다.


