---
description: 자바의 동시성 이슈
---

# Concurrency Programming in Java

## 1. 개념

동시성은 여러 작업이 동시에 실행되는 환경을 의미합니다. 자바에서는 여러 쓰레드를 이용해 동시성을 구현합니다. 멀티코어 프로세서의 성능을 최대한 활용할 수 있도록 하며, 서버 응답성을 높이고, 여러 사용자 요청을 병렬로 처리할 수 있는 환경을 제공합니다.



## 2. 동시성 구현 방법

* Thread 클래스
* Runnable 인터페이스
* Executor 프레임워크



## 3. 동시성 이슈의 주요 원인

### 3.1. 데이터 레이스

여러 그레드가 공유 자원에 동시에 접근하고 수정하는 과정에서 발생하는 문제입니다. 한 스레드가 데이터를 읽는동안 다른 스레드가 데이터를 수정하면 예상치 못한 결과가 발생할 수 있습니다.

예를 들어, count++ 메서드를 호출 했을때 총 3단계로 나뉩니다.

1. 메모리에서 count값을 불러옵니다
2. 연산을 통해 1을 더합니다
3. 연산 결과를 메모리에 저장합니다.

이 과정에서, 2번과 3번 사이에 다른 스레드가 값을 읽거나 쓰게되면 연산이 완료되지 않는 값을 읽게 됩니다.

### 3.2.  데이터 레이스 해결 방법

#### 3.2.1. Synchronized 키워드

메서드나 블록을 하나의 스레드만 접근할 수 있도록 동기화합니다.

동기화된 영역 내에서 데이터가 안전하게 보호되지만, 동기화 블록이 크면 모든 스레드가 순차적으로 동기화된 코드에 접근하게 되어,  성능 저하가 발생할 수 있습니다.

모든 객체는 암묵적으로 모니터를 가지고 있고, 스레드가 synchronized 블록이나 메서드에 진입하려고 하면, JVM은 해당 객체의 모니터 락을 획득하려고 시도합니다. 다른 스레드가 이미 가지고 있다면, 대기 상태로 전환됩니다.

* JVM은 `synchronized` 블록의 시작 부분에서 **Monitor Enter** 명령어를 실행하고, 종료 부분에서 **Monitor Exit** 명령어를 실행합니다.
* **Monitor Enter**는 스레드가 모니터 락을 획득하려고 시도하며, 성공하면 스레드는 블록 내부의 코드를 실행할 수 있습니다.
* **Monitor Exit**는 스레드가 모니터 락을 해제하고, 다른 대기 중인 스레드가 락을 획득할 수 있도록 합니다.'

#### 3.2.2. Lock 인터페이스

Synchronized보다 더 정교환 동기화 메커니즘을 제공합니다. ReentrantLock이 많이 사용됩니다.

<details>

<summary>Example</summary>

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockCounter {
    private final Lock lock = new ReentrantLock();
    private int count = 0;

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }

    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```

**Lock 인터페이스의 장점:**

</details>

* 공정성: fairness 설정으로 인해 starvation 문제가 발생하지 않습니다.
* 타임아웃: 잠금 시도 중 실패시, 다른 작업을 수행합니다. tryLock(long timeout, TimeUnit unit)
* 다중 조건 대기: Condition 객체를 사용해 동기화된 블록 내에서 여러 조건을 대기할 수 있습니다.

<details>

<summary>ReentrantLock의 공정성 설정</summary>

#### `ReentrantLock`의 공정성 설정

`ReentrantLock`은 \*\*비공정(fairness=false)\*\*과 **공정(fairness=true)** 두 가지 모드를 지원합니다. 공정성을 확보하려면 락을 생성할 때, **공정성 모드**를 활성화하여 대기 중인 스레드들이 먼저 대기한 순서대로 락을 획득할 수 있도록 설정할 수 있습니다.

**공정성과 비공정성 차이**

* **비공정 모드 (기본값)**: `ReentrantLock`의 기본 동작 방식은 비공정 모드입니다. 이 경우, 락을 대기 중인 스레드가 있어도 락이 해제되자마자 락을 요청한 스레드가 락을 획득할 가능성이 큽니다. 즉, 새로 락을 요청한 스레드가 우선 대기 중이던 스레드보다 먼저 락을 획득할 수 있습니다.
* **공정 모드**: `ReentrantLock`을 **공정 모드**로 설정하면, 락을 요청한 스레드들은 **FIFO (First-In, First-Out)** 방식으로 대기열에 들어가며, 먼저 대기한 스레드가 우선적으로 락을 획득하게 됩니다. 즉, 락을 먼저 요청한 스레드가 먼저 락을 얻는 것을 보장합니다.

</details>

#### 3.2.3. Volatile 키워드

변수의 값을 모든 스레드가 항상 최신 상태로 읽을 수 있도록 보장합니다. 주로 읽기와 쓰기 작업이 분리된 간단한 플래그 변수에 사용됩니다.

* 가시성 보장: volatile로 선언된 변수는 스레드 간의 가시성 문제를 해결해줍니다. 한 스레드가 값을 변경하면, 다른 스레드는 즉시 그 변경된 값을 볼 수 있습니다.
* 재배치 방지: JVM의 명령어 재배치를 방지해서 변수의 일관성을 유지합니다.

복합 연산(읽기-수정-쓰기)에 대한 원자성을 보장하지 않기 때문에, 이러한 작업에는 여전히 syncrhoinzed / Lock을 사용해야합니다.

#### 3.2.4. 불변 객체

불변 객체는 생성된 후 상태가 변하지 않는 객체를 의미합니다. 모든 필드를 final로 선언하여 변경이 불가능하게 합니다.

* 스레드 안전성
* 설계 간소화
* 성능 최적화

#### 3.2.5. Atomic 클래스 (CAS)

원자적 연산을 지원하는 클래스를 제공합니다. 동기화 없이 여러 스레드가 안전하게 공유 데이터를 수정할 수 있습니다.

메모리 주소, 예상되는 값, 새로운 값을 사용하여 동작합니다.

1. 특정 메모리 위치에서 현재 값을 읽습니다. (예상되는 값. 스레드가 메모리 위치에서 값을 읽은 순간 존재하는 값)
2. 업데이트를 위한 새로운 값을 연산
3. 기존 값을 업데이트 하기전에 다시 한번 확인하고, 새로운 값으로 업데이트합니다.

* 고성능의 원자적 연산이 가능하며, Synchronized에 비해 성능이 뛰어납니다.

단점으로는 복잡한 연산이나 조건부 연산에서는 다루기 어려울 수 있습니다. 업데이트 전 다시 한번 확인할때, A->B->A의 경우는 인식하지 못합니다.

#### 3.2.6. Concurrent 클래스

Concurrent 클래스는 락 분할을 사용하여 맵 전체가 아닌 개별 버킷에 대한 락을 걸어 동기화를 처리합니다. 이를 통해 여러 스레드가 동시에 다른 버킷에 접근할 수 있으며, 읽기 작업이 비동기적으로 수행되므로 성능이 뛰어나고, 동시성 처리 성능이 크게 향상됩니다.



### 3.3. 데드락

두 개 이상의 스레드가 서로 자원이 해제되기를 기다리면서 무한 대기 상태에 빠지는 상황입니다. 이는 서로 다른 스레드가 다른 자원을 잠금한 상태에서 발생할 수 있다.

### 3.4. 데드락 해결방법

#### 3.4.1. 자원 할당 순서 고정

모든 스레드가 자원에 접근할 때, 항상 동일한 순서로 잠금을 요청하도록 하는 방식으로, 자원 간의 상호 잠금 상황을 피할 수 있습니다.

모든 스레드가 자원에 접근하는 순서를 통일하면, 두 스레드가 서로의 자원을 기다리며 교착 상태에 빠지지 않도록 할 수 있습니다.

쓰레드 A : 자원A -> 자원B

쓰레드 B : 자원B -> 자원A

위 경우, 항상 자원A에 대해 순서를 먼저 설정할 수 있습니다.

#### 3.4.2. 타임아웃 설정

특정 자원에 대한 잠금을 요청할 때, 일정 시간 내에 잠금을 획득하지 못하면 요청을 취소합니다.

스레드가 특정 자원을 기다리다 잠금을 획득하지 못하면 대기를 종료하고, 다른 작업을 수행하거나 재시도 합니다. 단순히 타임아웃으로만 데드락을 완전히 피할 수는 없습니다.

#### 타임아웃 설정의 한계

스레드 1이 작업 순서로 A → B → C를 요청하고, 스레드 2가 B → A → D를 요청할 때, 타임아웃을 설정하면 다음과 같은 상황이 발생할 수 있습니다:

1. 스레드 1이 A를 잠금하고 B를 잠그려 하지만, 이미 스레드 2가 B를 잠금하고 있기 때문에 잠금 시도가 실패합니다.
2. 타임아웃 시간이 지나면 스레드 1은 잠금 요청을 취소하고 C 작업을 먼저 수행합니다.
3. 스레드 2도 마찬가지로 B를 잠금하고, A를 잠그려 하지만, 이미 스레드 1이 A를 잠금하고 있기 때문에 타임아웃 시간이 지나고 D 작업을 먼저 수행합니다.
4. 이후 두 스레드가 다시 A와 B에 대해 잠금을 시도할 때, 앞선 상황과 동일하게 서로의 자원을 기다리는 상태가 반복될 수 있습니다.

#### 3.4.3. Lock 인터페이스 사용

Synchronized와 비슷한 기능을 제공하지만 동기화 제어에 있어서 더 많은 유연성을 제공합니다. tryLock() 메서드를 사용해 잠금을 시도하고 실패할 경우 다른 작업을 수행할 수 있는 로직을 구현할 수 있습니다. 또한, Synchronized는 블록에 진입하기 위해 항상 락을 획득해야하고, 타임아웃이나 즉시 실패 같은 옵션이 없습니다.

* 공정성 제공: ReentrantLock을 사용하면 대기 시간이 긴 스레드가 먼저 락을 획득할 수 있도록 보장합니다.
* 조건 대기: Condition 객체를 사용하여 특정 조건에서 스레드를 대기시키고 조건이 만족되면 스레드를 깨울 수 있습니다.

Lock 인터페이스는복잡한 동기화 시나리오 또는 여러 스레드가 락을 요구할때 사용할때 적합합니다.



## 4. 자바 메모리 모델 (JMM)

자바 메모리 모델은 자바에서 다중 스레드 프로그램이 메모리와 상호 작용하는 방식을 정의하는 규칙의 집합입니다. JMM은 자바 프로그램에서 발생할 수 있는 가시성과 순서화 문제를 해결하기 위한 중요한 역할을 합니다.

JMM은 **동시성 문제**를 해결하기 위한 추상적인 모델로, 다음과 같은 내용을 포함합니다:

* **가시성**: 한 스레드에서 변경된 데이터가 다른 스레드에 언제 보이는지 규정합니다.
* **순서화**: 프로그램의 명령어가 실제 실행되는 순서를 규정합니다.
* **동기화 규칙**: `volatile`, `synchronized`, `Lock` 등 동기화 메커니즘이 어떻게 동작해야 하는지 정의합니다.

JMM은 스레드 간의 메모리 상호작용을 정의하는 모델이며, JVM이 이러한 동작을 보장하도록 하는 역할을 합니다.

(JMM은 동시성 제어와 관련된 추상적인 모델로, 스레드 간의 메모리 상호작용을 규정합니다. 반면, JVM 런타임 데이터 영역은 자바 프로그램이 실행되는 동안 메모리가 어떻게 배치되고 관리되는지를 설명하는 실제 메모리 구조입니다.)

### 4.1. 가시성 문제

가시성 문제는 여러 스레드가 같은 변수를 공유할 때, 한 스레드에서 변경한 값이 다른 스레드에게 즉시 보이지 않을 수 있는 문제입니다. 이는 스레드가 변수 값을 CPU 캐시에 저장하고, 메인 메모리와의 동기화가 제대로 이루어지지 않을 때 발생합니다. 자바에서 volatile/synchronized 키워드를 사용하여 일관성을 유지하여 가시성 문제를 해결할 수 있습니다.

자바에서 스레드는 각자 별도의 **CPU 캐시** 또는 **레지스터**를 사용해 작업을 수행할 수 있습니다. 이때, 스레드가 변수의 값을 변경하더라도 그 값이 메인 메모리(Main Memory)에 즉시 기록되지 않고, 해당 스레드의 캐시에만 반영될 수 있습니다. 다른 스레드가 동일한 변수를 읽을 때, 메인 메모리가 아닌 자신의 캐시나 레지스터에서 값을 가져오므로, 변경 사항을 인식하지 못하는 문제가 발생할 수 있습니다.



### 4.2. 순서화 문제

컴파일러나 CPU가 성능 최적화를 위해 명령어의 실행 순서를 변경하면서 발생하는 문제입니다.

<details>

<summary>Example</summary>

```java
Thread A
x = 1;  // 명령어 1
y = 1;  // 명령어 2

Thread B
if (y == 1) {  // 명령어 3
    System.out.println(x);  // 명령어 4
}
```

위 상황에서 명령어 순서가 재정렬될 수 있습니다.

```java
Thread A
y = 1;
x = 1; 로 재정렬 됐을때,

int x = 0;
int y = 0;

Thread A
y = 1;
<-------Thread B 실행
if (y == 1) {
    System.out.println(x); // x = 0;
}
x = 1;
```

</details>

이 문제를 해결하기 위해 `volatile` 키워드나 `synchronized` 블록을 사용할 수 있습니다. `volatile` 키워드는 가시성 문제를 해결할 뿐만 아니라, JVM이 변수의 읽기 및 쓰기 순서를 재배치하지 않도록 보장합니다. `synchronized` 블록을 사용하면 해당 블록 내의 모든 명령어가 순서대로 실행되도록 강제할 수 있습니다.



### 4.3 JMM의 Happens-Before 원칙

Happens-Before는 동시성 문제를 해결하기 위한 원칙이고, 두 개의 작업 간의 실행 순서를 결정하는 규칙입니다. 만약 작업 A가 작업 B보다 먼저 일어난다면, 작업 B는 작업 A의 결과를 반드시 볼 수 있어야 합니다. 이는 JVM이 어떤 최적화를 수행하더라도 유지되어야 하는 규칙입니다.

* **프로그램 순서 규칙**: 한 스레드 내에서 이전 명령은 항상 후속 명령보다 먼저 발생합니다.
* **모니터 잠금 규칙**: 한 스레드가 락을 해제하는 모든 동작은 그 락을 다음에 획득하는 스레드보다 먼저 발생합니다.
* **volatile 변수 규칙**: 한 스레드가 volatile 변수에 기록한 모든 동작은 그 변수를 다음에 읽는 스레드보다 먼저 발생합니다.
* **스레드 시작 규칙**: `Thread.start()`를 호출한 모든 동작은 시작된 스레드의 모든 동작보다 먼저 발생합니다.
* **스레드 종료 규칙**: 스레드의 모든 동작은 해당 스레드의 `Thread.join()`이 반환되기 전에 발생합니다.

### 4.4. JMM과 동기화 메커니즘

JMM은 다양한 동기화 메커니즘과 함께 작동합니다. 예를 들어, `synchronized` 키워드, `volatile` 키워드, `Lock` 인터페이스 등은 모두 JMM에서 정의된 규칙에 따라 동작합니다. 이 메커니즘을 사용하면, 다중 스레드 환경에서 발생할 수 있는 가시성 문제와 순서화 문제를 효과적으로 해결할 수 있습니다.



## 5.  기타

### 5.1. Vector, HashTable, Collections.synchronizedXXX의 문제점

Vector, HashTable, Collections.synchronized 등은 모두 synchronized 키워드를 사용하여 동기화를 처리합니다. 이로 인해 전체 메서드에 락이 걸리게 되며, 하나의 스레드만 해당 메서드를 실행합니다. 전체 메서드에 대해 동기화를 적용하여, 동기화가 필요하지 않은 경우에도 동기화가 발생할 수 있습니다.

자바 5 이후부터는 ConcurrentHashMap과 같은 더 효율적인 대체 기술들이 등장하여, 기존 동기화 컬렉션들은 잘 사용되지 않습니다. 락 분할을 사용하여 특정 데이터가 저장된 버킷(슬롯)에만 락이 걸립니다.



### 5.2. SynchronizedList와 CopyOnWriteArrayList의 차이

Thread-safe한 리스트를 제공하기 위해 사용됩니다.

SyncrhoinzedList는 내부적으로 synchronized 블록이 사용됩니다.

CopyOnWriteArrayList는 쓰기 작업이 발생할 떄마다 전체 배열의 복사본을 생성합니다. 읽기 작업은 기존 배열을 참조하므로, 동기화 없이도 안전하게 동시 읽기 작업이 가능합니다. 쓰기 작업이 많아지면 메모리 사용량이 많아지는게 단점이라서 읽기 작업이 빈번하고 쓰기 작업은 적은 경우에 적합합니다.



### 5.3. ConcurrentHashMap의 동작 과정을 SynchronizedMap과 비교하여 설명해주세요.

**SynchronizedMap**은 모든 작업에 대해 **맵 전체에 단일 락**을 사용하여 동기화를 보장합니다. 이로 인해 병목 현상과 성능 저하가 발생할 수 있으며, 특히 동시성 처리가 중요한 환경에서는 확장성이 제한적입니다.

반면, **ConcurrentHashMap**은 **락 분할(Lock Stripping)** 기법을 사용하여, 맵 전체가 아닌 **개별 버킷에 대한 락**을 걸어 동기화를 처리합니다. 이를 통해 여러 스레드가 동시에 다른 버킷에 접근할 수 있으며, 특히 읽기 작업이 비동기적으로 수행되므로 성능이 뛰어납니다. 이로 인해 **동시성 처리 성능이 크게 향상**되며, 확장성이 뛰어납니다.