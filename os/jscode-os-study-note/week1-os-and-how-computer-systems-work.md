---
description: 운영체제 개요 & 컴퓨터 시스템 동작원리
---

# Week1 OS & How Computer Systems Work



## 운영체제는 뭐고, 뭘 하나요?

* 컴퓨터 하드웨어와 사용자 간의 중간 계층 역할을 하는 소프트웨어
* 컴퓨터 자원을 효율적으로 관리하고 제어
* CPU 스케줄링, 파일 시스템 관리, IO 제어 등



## 시분할 시스템이 뭔가요?

* 여러 사용자나 프로그램이 CPU 자원을 공유하여 동시에 실행되는 것처럼 보이게 하는 멀티태스킹 환경
* 각 작업이 고정된 시간 단위를 할당받아 교대로 실행되는 방식
* 키워드: 멀티태스킹, 컨텍스트 스위칭, CPU 스케줄링 알고리즘



## 다중 프로그래밍 시스템에 대해서 설명해주세요

* 메모리에 여러 프로그램을 동시에 적재하여 CPU 자원을 최대한 효율적으로 사용하는 방식
* 목적은 CPU가 항상 바쁘게 동작하도록 하여 유휴 시간을 줄이는 것
* 여러 작업을 번갈아가면서 처리(컨텍스트 스위칭, CPU 스케줄링, 페이징, 동기화)



## 대화형 시스템에 대해 설명해주세요

* 사용자와 컴퓨터 간의 **즉각적인 상호작용**을 가능하게 하는 시스템
* 지연없이 실시간으로 작업이 이루어지도록하여 사용자는 프로그램을 더 직관적으로 제어 가능
* 즉시 응답
* 연속적인 상호작용: 연속적으로 명령을 내리거나 데이터 입력 가능
* 사용자 친화적: GUI (Graphic User Interface)
* 실시간 처리



## 다중 처리기 시스템

* 두개 이상 CPU가 하나의 컴퓨터 시스템에 물리적으로 존재하는 시스템
* 멀티 코어 시스템과 다르지만, 다중 처리기는 멀티 코어 시스템을 포함하는 개념
* 동시에 작동하여 여러 작업을 병렬로 처리하는 시스템
* 여러 프로세스가 협력하여 이를 동시에 처리하여 성능과 속도를 향상
* 병렬 처리
* 공유 메모리: 각 프로세서는 공통의 메모리 영역을 사용하여 데이터를 읽고 씁니다.
* 확장성: 추가 CPU 장착
* 고가용성: 다른 CPU가 이어서 작업
* 계층: CPU안에 코어가 여러개 있고, 각 코어는 각 프로세스를 담당하며, 각 프로세스내에서는 각 스레드가 존재



## 시스템 콜에 대해 설명해주세요

* 사용자 프로그램이 운영체제의 커널에 있는 기능을 요청할 때 사용하는 인터페이스
* 운영체제와 사용자 프로그램 사이의 안전한 인터페이스를 제공. 자원의 직접적인 접근 제한.
* 파일 시스템 접근, 메모리 할당, 프로세스 생성 등 시스템 자원에 직접 접근을 할 수 없습니다.
* 시스템 콜을 통해 운영체제의 커널모드로 전환되어 이러한 작업을 처리합니다.

시스템 콜은 사용자 프로그램이 운영체제의 커널 기능을 요청할 때 사용하는 인터페이스로, 운영체제가 하드웨어 자원을 안전하고 효율적으로 제어할 수 있도록 합니다. 이를 통해 사용자 프로그램은 시스템 자원을 사용할 수 있으며, 시스템의 안정성을 유지할 수 있습니다.



## 커널에 대해 설명해주세요

* 운영체제의 핵심 구성 요소로, 컴퓨터 시스템의 하드웨어 자원(CPU, 메모리, 디스크 등)을 관리합니다.
* 프로세스와 사용자 프로그램 간의 상호작용을 제어하는 역할을 합니다.
* 시스템 자원을 효율적으로 관리하고 CPU 스케줄링을 수행합니다.
* 커널은 안정성과 보안을 유지하며, 하드웨어 자원에 직접 접근하는 커널 모드에서 동작합니다.



## 커널 모드에 대해 설명해주세요

* 운영체제에서 **하드웨어 자원에 직접 접근**할 수 있는 **최고 권한**의 실행 모드
* 커널 모드에서는 CPU가 **모든 메모리 주소**에 접근할 수 있고, **하드웨어 장치**나 시스템 자원을 제어할 수 있는 모든 명령어를 실행할 수 있습니다.
* 프로그램이 **파일 시스템에 접근**하거나, **네트워크 통신** 또는 **메모리 할당**과 같은 작업을 할 때는 \*\*시스템 콜(System Call)\*\*을 통해 **커널 모드로 전환**
* **컨텍스트 스위칭**이 발생하여 CPU는 유저 모드에서 커널 모드로 전환되며, 커널이 해당 요청을 처리한 후 다시 유저 모드로 복귀

커널 모드는 운영체제가 **시스템 자원에 직접 접근**할 수 있는 모드로, CPU가 모든 명령어와 메모리 영역에 대한 권한을 가지고 실행됩니다. 유저 모드에서 실행되는 응용 프로그램이 시스템 자원에 접근해야 할 때, 시스템 콜을 통해 **커널 모드로 전환**되어 안전하게 자원에 접근할 수 있습니다.



## 유저 모드에 대해 설명해주세요

* 응용 프로그램이 제한된 권한으로 실행되는 모드
* 시스템 자원에 대한 직접적인 접근 불가
* 프로그램은 하드웨어 자원이나 다른 프로세스의 메모리에 접근할 수 없습니다.

유저 모드는 응용 프로그램이 안전하게 실행될 수 있도록 **제한된 권한**을 가지며, 시스템 자원에 직접 접근하지 못하는 실행 환경입니다. 필요할 때는 시스템 콜을 통해 **커널 모드로 전환**하여 하드웨어 자원에 접근한 후 다시 유저 모드로 돌아옵니다.



## 폴링에 대해 설명해주세요

* CPU가 주기적으로 특정 장치나 작업의 상태 변화를 확인하는 방식.
* 상태 변화를 게속해서 확인해야 하기때문에 불필요한 CPU 자원을 소모
* 이를 보완하기 위해 인터럽트가 사용됩니다.



## 인터럽트에 대해 설명해주세요

* 외부 또는 내부의 특정 이벤트가 발생했을때, 현재 CPU가 수행중인 작업을 일시 중단하고, 해당 이벤트를 우선적으로 처리하는 시스템 메커니즘.
* 인터럽트 발생시, CPU는 컨텍스트 스위칭이 발생. 컨텍스트를 저장하고 인터럽트 처리 루틴(인터럽트 핸들러)을 실행하여 해당 이벤트를 처리합니다.
* 각 인터럽트는 고유한 벡터 번호를 갖고 있고, 해당 번호를 인터럽트 벡터 테이블에서 찾아서 맞는 처리 루딘으로 연결됩니다.&#x20;
* 인터럽트 벡터 테이블은 운영체제/하드웨어가 관리하며 미리 정의되어 있는 함수입니다.



## DMA(Direct Memory Access)에 대해 설명해주세요

* 입출력 장치(IO장치)가 CPU의 개입 없이 메모리에 직접 데이터를 읽거나 쓸 수 있게 해주는 데이터 전송 방식.
* CPU부하 감소: CPU를 거치치 않고 장치와 메모리 간에 데이터를 주고 받을 수 있게 하는 기법
* 빠른 데이터 전송: CPU가 일일이 처리하지 않습니다.

DMA(Direct Memory Access)는 **CPU 없이** 장치가 메모리로 직접 데이터를 주고받을 수 있는 방식으로, CPU의 부하를 줄이고 데이터 전송을 효율적으로 처리하는 데 사용됩니다. 이를 통해 시스템의 성능이 향상되고, 특히 대용량 데이터 전송에서 효과적입니다.



## 동기식 I/O가 뭔가요?

* 입출력 작업이 완료될 때까지 프로그램이 기다리는 방식입니다. (블로킹 상태)
* CPU는 대기상태로 변하여 자원을 비효율적으로 사용하게 됩니다.



## 비동기식 I/O가 뭔가요?

* I/O 작업을 요청한 프로그램은 I/O 작업이 끝날 때까지 **기다리지 않고** **다른 작업을 계속 수행(논블로킹)**
* 프로그램이 **I/O 작업**을 요청하면, **즉시 제어권을 반환**받아 다른 작업을 계속 실행
* I/O 작업이 백그라운드에서 진행되고, 작업이 완료되면 **인터럽트** 또는 **콜백**을 통해 프로그램에 작업이 완료되었음을 알립니다.
* CPU 자원을 효율적으로 사용하며, 멀티태스킹에 유리합니다.
* 단점으로는 논블로킹 방식이기때문에 프로그램 흐름이 복잡해집니다. I/O 작업 완료 후에 작업을 콜백 함수나 이벤트 핸들러로 처리해야하기 떄문입니다.