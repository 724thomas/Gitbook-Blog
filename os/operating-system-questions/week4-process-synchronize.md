---
description: 프로세스 동기화
---

# Week4 Process Synchronize

## 병행성에 대해 설명해주세요. (병렬성 X)

"여러 작업이 논리적**으로 동시에** 실행되는 것을 의미"

* 여러 작업이 동시에 실행되는 것처럼 보이도록 작업들을 빠르게 번갈아가며 수행하는 것을 의미
* 특히 **단일 코어** 시스템에서 여러 스레드가 동시에 실행되는 것처럼 보이지만 실제로는 교대로 실행
* CPU코어 < 스레드
* 멀티코어 CPU라면, 여러 코어가 동시에 여러 스레드를 병렬로 처리할 수 있지만, 코어가 부족한 경우, 여러 스레드가 **교대로** 실행됩니다.

**(**&#xC5EC;러 프로세스가 있을 경우, **CPU 스케줄러**는 **프로세스 단위로 CPU 시간을 할당. 한 프로세스**가 CPU를 배정받으면, 그 프로세스 내에서 **스레드들 간에도 스케줄링**이 이루어짐.)



## 병렬성에 대해 설명해주세요.

"여러 작업이 **물리적으로 동시에** 실행되는 것을 의미"

* 멀티프로세싱 시스템이나 멀티스레드 환경(+멀티 프로세싱 시스템) 에서 발생하며, **여러 개의 CPU 코어** 또는 **프로세서**가 동시에 각기 다른 작업을 실행할 때 병렬 처리가 이루어짐
* 작업이 실제로 동시에 진행되기 때문에 성능 향상에 유리
* 병렬 컴퓨팅에서 중요한 역할(**하나의 큰 작업을 여러 개의 작은 작업으로 나누어 동시에 처리**하는 방식)



## 프로세스 동기화가 뭔가요?

* **프로세스 동기화**란 여러 프로세스가 동시에 **공유 자원**에 접근할 때, 데이터의 **일관성**과 **무결성**을 유지하는 것을 의미
* 다수의 프로세스가 같은 자원에 접근할 때 **경쟁 상태(Race Condition)** 가 발생하지 않도록 **프로세스 간의 실행 순서를 제어**
* 뮤텍스, 세마포어 같은 동기화 메커니즘을 사용해 구현됩니다.

<details>

<summary>Binary Semaphore vs Mutex</summary>

소유권에 대한 차이.



이진 세마포어는 특정 이벤트를 기다리는 시스템에서 사용. Pub/Sub. Pub이 가득찼다는 0으로 변경하고 Sub이 비어있다는 1로 변경 하는 상황

OR

하나의 프로세스가 I/O 작업을 시작하고, 그 작업이 완료되면 다른 프로세스나 시스템이 자원 해제를 트리거하는 경우.&#x20;

예를 들어, 프로세스 A가 파일을 읽고 쓰기 작업을 시작하고, 프로세스 B가 작업 완료 후 자원 해제를 처리.&#x20;

예시 동작: 프로세스 A는 파일에 접근하여 데이터를 쓰기 시작하고, 이진 세마포어를 사용해 자원을 잠급니다(세마포어 값 0). 데이터가 다 쓰여지면 프로세스 B가 이를 감지하고, 이진 세마포어 값을 1로 변경하여 자원을 해제합니다. 이진 세마포어의 역할: 프로세스 A가 파일 쓰기를 잠근 후, 다른 프로세스가 그 작업이 완료된 것을 확인하고 자원을 해제할 수 있습니다.

여기에서 프로세스 B는 파일에 접근하려는 목적이 아니고 그냥 세마포어를 관리

</details>



## Critical Section(임계 영역)에 대해 설명해주세요.

* 여러 프로세스나 스레드가 **공유 자원**에 접근하는 코드의 특정 부분
* 동시에 여러 프로세스나 스레드가 이 영역에 진입하면 데이터 일관성 문제가 발생
* 하나의 프로세스 또는 스레드만이 접근할 수 있도록 **상호 배제(Mutual Exclusion)**&#xAC00; 필요



## 경쟁 상태(Race Condition)이 뭔가요?

* 두 개 이상의 프로세스나 스레드가 **동시에 공유 자원**에 접근할 때, 실행 순서에 따라 **결과가 달라지거나 예기치 않은 결과**가 발생하는 상태
* 두 스레드가 동일한 변수를 동시에 읽고 수정할 때, 한 스레드의 변경이 다른 스레드에 의해 덮어쓰여 **원치 않는 결과가 발생**
* 상호 배제를 통해 해결 -> 뮤텍스, 세마포어 같은 동기화 메커니즘



## Race Condition을 어떻게 해결할 수 있나요?

RaceCondition: 두개 이상의 프로세스/스레드가 공유 리소스에 동시에 접근할때 실행 순서에 따라 결과값이 달라지는 상황

Race Condition을 해결하기 위해서는 상호보안 (Mutual Exclusive)이 보장되어야합니다.

락 메커니즘

* 뮤텍스:\
  한 번에 하나의 스레드 또는 프로세스만이 **임계 영역**에 진입\
  **락의 소유권**을 가지며, 작업이 끝난 후 **락을 반환**\
  **스핀락 등을 통해 재시도(But CPU 자원 사용)**
* 이진 세마포어:\
  0과 1의 값을 가지며, 뮤텍스와 유사하지만, **락을 설정한 스레드와 해제하는 스레드가 다를 수 있다는 차이**
* **카운팅 세마포어:**\
  동시에 접근할 수 있는 프로세스나 스레드의 수를 **N개로 제한**

<details>

<summary>Spin Lock vs Blocking Wait</summary>

**스핀락(Spinlock)**

* **스핀락**은 자원이 사용 중일 때, 스레드가 **잠금(락)이 해제될 때까지 지속적으로 확인하는 방식**입니다.
* 스핀락은 **바쁜 대기(Busy Waiting)** 상태에 놓이며, 자원을 사용할 수 있을 때까지 **계속 CPU 자원을 사용**하면서 락을 시도합니다.
* **특징**: CPU 자원을 소비하면서 계속해서 락 상태를 확인하므로, 락이 짧은 시간 안에 해제될 가능성이 높다면 스핀락이 효율적일 수 있습니다. 하지만 락이 오랫동안 유지된다면 **CPU 자원 낭비**가 발생할 수 있습니다.

**2. 블로킹 대기(Blocking Wait) - 대기 상태**

* 스레드가 자원을 얻지 못했을 때 **스케줄러에 의해 블로킹 상태로 전환**됩니다.
* **스레드 대기 큐에 들어가 대기**하며, CPU를 사용하지 않고 **이벤트**나 **신호**를 기다립니다.
* **이벤트(Event) 대기**: 락을 가진 스레드가 작업을 끝내고 락을 해제할 때, **이벤트 신호**(예: 조건 변수, 세마포어, 뮤텍스)를 통해 다른 스레드에게 자원이 사용 가능하다는 신호를 보냅니다. 대기 중인 스레드는 이 신호를 받아서 실행 상태로 전환되고, 자원에 접근할 수 있게 됩니다.

</details>

## Mutual Exclusion(상호 배제)에 대해 설명해주세요.

* 임계 영역에 동시에 여러 프로세스 또는 스레드가 접근하지 못하도록 하는 것
* 여러 프로세스나 스레드가 한 번에 하나만 임계 영역에 진입하도록 제한하여 **Race Condition**을 방지하는 것이 목적
* **뮤텍스는 주로 상호 배제를 보장하기 위해** 사용되고, 세마포어는 여러 스레드나 프로세스의 **동시 접근을 조절**



## 뮤텍스에 대해 설명해주세요.

* **뮤텍스(Mutex)**&#xB294; **상호 배제(Mutual Exclusion)**&#xB97C; 보장하기 위해 사용되는 락 메커니즘으로, 여러 스레드 또는 프로세스가 **동시에 공유 자원에 접근하지 못하도록** 하는 역할
* **임계 영역(Critical Section)**&#xC5D0;서 발생할 수 있는 **Race Condition(경쟁 상태)**&#xB97C; 예방하기 위해 사용되며, **한 번에 하나의 스레드 또는 프로세스만이 자원에 접근할 수 있도록** 제한
* **뮤텍스는 락을 획득한 스레드만이 락을 해제할 수 있습니다**. 이는 **이진 세마포어**와의 차이점으로, 이진 세마포어는 다른 스레드가 락을 해제할 수 있지만, 뮤텍스는 반드시 락을 건 스레드가 해제해야 합니다.



## 세마포어에 대해 설명해주세요.

* **세마포어(Semaphore)**&#xB294; **여러 프로세스나 스레드가 공유 자원에 안전하게 접근**할 수 있도록 자원의 **사용을 제어**하는 동기화 메커니즘
* **이진 세마포어(Binary Semaphore)**:\
  **뮤텍스와의 차이**는, **이진 세마포어는 락을 획득한 스레드가 아닌 다른 스레드나 프로세스도 락을 해제할 수 있다는 것**
* **카운팅 세마포어:** \
  동시에 접근할 수 있는 **스레드 또는 프로세스의 수를 제한**



## 뮤텍스와 이진 세마포어의 차이에 대해 설명해주세요.

* 뮤텍스:\
  **뮤텍스를 잠근 스레드나 프로세스만이 락을 해제할 수 있습니다.** 즉, 락을 획득한 **소유권**이 존재
* 이진 세마포어:\
  **다른 스레드나 프로세스도 락을 해제할 수 있습니다.** 즉, 자원을 획득한 스레드가 아닌 **다른 스레드**도 락을 해제



## 모니터(Monitor)에 대해 설명해주세요

* **프로그래밍 언어 수준**에서 제공되는 고수준의 동기화 메커니즘
* **락(Lock)**&#xACFC; **조건 변수(Condition Variable)**&#xB97C; 포함하는 구조로, 이를 통해 **스레드 간의 동기화**를 쉽게 관리 (락 메커니즘 이상의 역할)
* 객체 내의 공유 자원에 접근할 수 있는 메커니즘을 자동으로 제어.
* 특징:\
  **자동 동기화**: 프로그래머가 직접 락을 관리하지 않아도, 모니터는 **상호 배제를 보장**하며 동기화 문제를 해결합니다.\
  **상호 배제**: 모니터는 **한 번에 하나의 스레드만** 임계 영역에 진입할 수 있도록 제어하여, **Race Condition**을 방지합니다.\
  **고수준 동기화**: 모니터는 자원 관리와 스레드 간 동기화를 **자동으로 처리**해 프로그래머가 직접 락과 조건 변수를 다룰 필요가 없습니다.



## 데드락이 무엇인가요?

* **데드락(Deadlock)**&#xC740; 두 개 이상의 프로세스 또는 스레드가 서로가 가진 자원을 기다리며 **무한 대기 상태**에 빠지는 현상
* **상호 배제, 점유 대기, 비선점, 순환 대기** 조건이 모두 만족하면 데드락이 발생합니다.



## 데드락 발생 조건 4가지를 설명해주세요.

* 상호 배제: 한번에 하나의 스레드 또는 프로세스가 접근할 수 있는 조건
* 비선점:다른 스레드나 프로세스가 강제로 점유하고 있는 리소스의 소유권을 뻇을 수 없는 조건
* 점유 대기: **프로세스가 이미 하나 이상의 자원을 점유하고** 있는 상태
* 순환 구조: **여러 프로세스가 순환적으로 자원을 요청하며 서로를 대기**하는 상태



## 데드락을 막는 방법에 대해 설명해주세요.

데드락 예방

* 상호 배제 조건 완화: 동시 접근 가능하게 합니다. (읽기 작업 등)
* 점유 대기 조건 완화: 요청할때 모든 자원의 락을 한꺼번에 (순차적으로)요청. 만약 중간에 획득이 불가능하면 다 반환을 한다.
* 비선점 조건 완화: 다른 프로세스가 자원을 강제로 가져갈 수 있도록
* 순환 대기 조건 완화: 자원을 고정된 순서로만 요청

데드락 회피

* 은행원 알고리즘: 은행원이 고객에게 대출을 해주는 방식. **자원의 상태를 미리 검사**하여 데드락이 발생할 수 있는지 판단한 후, 안전한 자원 할당만을 허용(매니저가 관리해주는 것 처럼)

데드락 무시

* 관리자가 직접 해결
