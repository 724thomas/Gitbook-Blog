---
description: 트랜잭션, 회복
---

# Week5 DB Transaction, Recovery

## DB 세션이 뭔가요?

클라이언트가 데이터베이스와 맺는 상태 또는 연결.  클라이언트가 데이터베이스와 상호작용하는 연결단위

* 클라이언트가 DB에 연결 → 세션 시작.
* 클라이언트가 쿼리 전송 (조회, 삽입, 삭제 등).
* DB가 요청에 응답하여 결과 반환.
* 트랜잭션이 끝나면 세션 종료 (commit/rollback).



## Commit에 대해서 설명해주세요

* Commit은 트랜잭션을 성공적으로 마무리하고, 변경된 데이터를 **영구적으로** 데이터베이스에 저장하는 작업



## 롤백에 대해서 설명해주세요

* **Rollback**은 데이터베이스에서 **트랜잭션이 비정상적으로 종료되거나 오류가 발생할 경우**, 그 트랜잭션 내에서 변경된 모든 데이터를 취소하고, **트랜잭션이 시작되기 전의 상태로 되돌리는 작업**
* Rollback이 호출되면 로그 파일을 사용하여 변경 사항을 취소하고, **이전의 상태**로 복구

그럼커밋되기전에는 영구적으로 저장되지 않기때문에 그냥 메모리에 있는 정보를 없애면 되는건데 왜 오버헤드가 발생하는가?

* 트랜젝션중에 발생하는 작업들은 로그에 반영을하는데, 로그를 롤백하는데에 오버헤드가 존재
* 락 해제
* 트랜잭션에 참여한 여러 데이터베이스나 외부 시스템(예: 메시지 큐, 파일 시스템 등)이 있다면, 롤백 시 이 자원들과의 연결을 정리하거나 복구하는 과정에서도 추가적인 리소스가 소모될 수 있습니다.

## Auto Commit이 뭔가요?

* 쿼리 단위.
* 데이터베이스에서 SQL 쿼리를 실행할 때, 해당 쿼리가 자동으로 **트랜잭션을 Commit**하도록 하는 설정
* 여러 작업을 **하나의 트랜잭션**으로 묶어 처리하고자 할 때 적합하지 않습니다.



## 트랜잭션에 대해 설명해주세요.

* 하나 이상의 SQL 쿼리를 **하나의 논리적 단위로 묶어서 처리**
* 모든 작업이 성공적으로 완료되면 **Commit**으로 확정하고, 문제가 발생하면 **Rollback**으로 변경 사항을 취소하여 이전 상태로 되돌립니다.
* 트랜잭션은 **ACID** 성질을 만족
* 동시성과 장애복구에 중요한 역할을 합니다.



## 트랜잭션의 성질 ACID에 대해서 설명해주세요.

**원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)**

* All or Nothing
* 모든 제약 조건을 만족, 트랜잭션 이전과 이후에 데이터베이스는 항상 **정확하고 일관된 상태**
* 각 트랜잭션은 서로 **독립적**으로 실행
* 트랜잭션이 **Commit**된 후에는 그 변경 사항이 **영구적**으로 저장



## 트랜잭션 격리 수준이 뭘까요?

* 데이터베이스 트랜잭션 간의 **동시성 제어**를 관리하기 위한 메커니즘
* 동시성 처리 성능과 데이터 일관성의 trade-off
* Read Uncommitted: 커밋되지 않은 데이터도
* Read Committed: 커밋된 데이터만
* Repeatable Read: 다른 트랜잭션에 의해 중간에 값이 바뀌어도, 처음 본인이 읽은 값을 유지.
* Serializable: 순차적 실행

**InnoDB에서는 Repeatable Read격리 수준이지만, MVCC와 Locking(Next Key Lock)을 통해 팬텀리드를 해결합니다.**

## Read Uncommitted에 대해 설명해주세요

* **트랜잭션이 커밋되지 않은 데이터**를 다른 트랜잭션이 읽을 수 있어 **Dirty Read(더티 리드)**&#xAC00; 발생
* **커밋되지 않은 중간 상태의 데이터**도 다른 트랜잭션에서 읽을 수 있습니다. 롤백시 문제.
* Non-repeatable read, phantome read 도 발생.



## Read committed에 대해 설명해주세요

* 트랜잭션이 커밋된 데이터만 다른 트랜잭션이 읽을 수 있습니다. **Dirty Read(더티 리드)** 문제를 방지.
* **Non-Repeatable Read** 문제가 발생.\
  트랜잭션이 동일한 데이터를 여러 번 읽을 때, 그 사이에 **다른 트랜잭션이 데이터를 수정**하고 커밋하면, 처음 읽은 값과 다시 읽은 값이 **다를 수 있습니다**.



## Repeatable Read에 대해 설명해주세요

* 트랜잭션이 **처음 데이터를 읽은 시점의 데이터**를 트랜잭션이 끝날 때까지 계속해서 **일관되게 유지**하는 것을 보장(Lock). **Non-Repeatable Read(논리적 일관성 손실)** 문제를 해결
* Phantom Read 발생.\
  **Phantom Read**는 트랜잭션이 범위 기반 쿼리(예: 특정 조건에 맞는 모든 데이터를 조회)를 실행할 때, **처음에는 없었던 행이 추가되는 문제. (갱신과 삭제는 락이 걸려있기때문에 방지)**
* InnoDB경우는 락이 아닌 스냅샷을 사용하여 Phantom Read도 해결한다.



## Serializable에 대해 설명해주세요

* **완벽한 일관성**을 보장. **모든 동시성 문제**(Dirty Read, Non-Repeatable Read, Phantom Read)를 완벽하게 방지
* 모든 트랜잭션이 마치 순차적으로 실행되는 것처럼 동작



## DB 동시성 제어에 대해서 설명해주세요

* 여러 트랜잭션이 동시에 데이터베이스에 접근할 때, **데이터의 무결성과 일관성을 유지**하기 위해 사용되는 메커니즘

**락(Lock) 메커니즘**:

* **읽기 락(Shared Lock)**: 여러 트랜잭션이 동시에 데이터를 읽을 수 있지만, 수정은 불가
* **쓰기 락(Exclusive Lock)**: 한 트랜잭션만 데이터를 읽고 수정할 수 있으며, 다른 트랜잭션은 대기

트랜잭션 격리 수준:

* Read uncommitted, read committed, repeatable read, serializable

MVCC (다중 버전 동시성 제어):

**락을 사용하지 않고** 동시성을 제어하는 방법으로, 데이터베이스에서 **여러 버전의 데이터를 관리**하여 트랜잭션 간 충돌을 최소화합

예시)

* **트랜잭션 A**가 데이터 `x`를 읽습니다. 이때, `x`는 버전 1입니다.
* **트랜잭션 B**가 같은 데이터 `x`를 수정하고, **버전 2**로 새로운 버전을 생성합니다.
* **트랜잭션 A**가 다시 데이터를 조회할 때는 **버전 1**의 데이터를 읽습니다(처음에 읽었던 스냅샷을 유지). 반면에, 새로운 트랜잭션이 시작되면 **버전 2**의 데이터를 읽습니다.
* **트랜잭션 B**가 완료되면, 이제 다른 트랜잭션은 **버전 2**를 읽게 됩니다.

InnoDB Locking 전략

* 레코드 잠금 (Record Lock): 특정 행(row)에만 적용되는 잠금으로, 트랜잭션이 실행 중인 레코드에 대해서만 잠금을 거는 방식
* 갭 잠금 (Gap Lock): **레코드 사이의 범위(갭)**&#xC5D0; 잠금을 거는 방식입니다. 갭 잠금은 주로 **팬텀 리드(Phantom Read)** 문제를 방지하기 위해 사용
* 넥스트 키 잠금 (Next-Key Lock): **레코드 잠금과 갭 잠금이 결합된 형태**로, 특정 레코드와 그 레코드 **주변의 범위(갭)**&#xC5D0; 대해 동시에 잠금을 거는 방식입니다. 이 전략은 InnoDB의 **Repeatable Read** 격리 수준에서 팬텀 리드를 방지하기 위해 사용



## 갱신 손실 문제가 뭔가요?

* 두 개 이상의 트랜잭션이 동시에 동일한 데이터를 수정 하나의 트랜잭션의 수정 결과가 사라져버리는 것
* Lock 사용 / Repeatable Read 이상의 트랜잭션 적용



## DB 락에 대해서 설명해주세요.

* 읽기 락(Shared Lock): 동시에 읽기 락을 걸 수 있지만 수정 삭제는 불가
* 쓰기 락(Exclusive Lock): 다른 트랜잭션이 읽기 쓰기 모두 불가
* 데드락 발생 가능



## DB의 데드락에 대해 설명해주세요.

* 서로 **자원을 선점한 상태,** 상대방이 보유한 자원을 기다리며 **무한 대기 상태**에 빠지는 현상
* 상호 배제: 자원은 하나의 트랜잭션에만 할당
* 비선점: 다른 트랜잭션이 강제로 자원을 가져갈 수 없음.
* 점유와 대기: 자원을 점유하면서 동시에기다리는 상태
* 순환 구조: 서로 기다리는 상태



## DB 회복에 대해서 설명해주세요

* 장애나 시스템 오류로 인해 데이터가 손상되거나 트랜잭션이 비정상적으로 종료되었을 때, 회복 기법을 통해 데이터를 복구하여 **일관된 상태로 되돌리는 것**.
* 목적: 데이터 일관성 유지, 손실된 트랜잭션 복구, 커밋되지 않은 데이터 취소
* 로그 기반 회복(Log-Based Recovery)\
  **장애가 발생했을 때** 이 로그를 사용하여 **커밋된 트랜잭션을 복구**하거나 **커밋되지 않은 트랜잭션을 롤백. InnoDB에서는 주로 로그 기반 회복을 사용.**
* 체크포인트(Checkpoint) 회복 기법\
  주기적으로 체크포인트를 찍어서 체크 포인트 이후의 로그만 검사하여 복구
* 그림자 페이징(Shadow Paging)\
  트랜잭션 발생시 원래 데이터를 복사하여 별도의 공간에 저장. 장애시 복사본을 사용하여 복구.



## Redo, Undo에 대해서 설명해주세요

데이터베이스 회복 기법에서 사용되는 두가지 중요한 작업.

* Undo = 롤백\
  **커밋되지 않은 트랜잭션**은 **Undo**로 처리
* Redo = 재실행\
  **커밋된 트랜잭션**은 **Redo**로 처리



## MySQL InnoDB 스토리지 엔진의 기본 트랜잭션 수준은 뭔가요?

Repeatable Read

* Non-Repeatable Read 방지
* Phantom Read 해결
