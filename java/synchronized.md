---
description: 동기화
---

# Synchronized

## 1. 출금 예제

```java
package thread.sync;

public interface BankAccount {
    boolean withdraw(int amount);
    int getBalance();
}
```

```java
package thread.sync;

public class WithdrawTask implements Runnable {
    private BankAccount account;
    private int amount;

    public WithdrawTask(BankAccount account, int amount) {
        this.account = account;
        this.amount = amount;
    }

    @Override
    public void run() {
        account.withdraw(amount);
    }
}

```

```java
package thread.sync;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankMain {
    public static void main(String[] args) throws InterruptedException {
        BankAccount account = new BankAccountV1(1000);

        Thread t1 = new Thread(new WithdrawTask(account, 800), "t1");
        Thread t2 = new Thread(new WithdrawTask(account, 800), "t2");
        t1.start();
        t2.start();

        sleep(500);
        log("t1 state: " + t1.getState());
        log("t2 state: " + t2.getState());

        t1.join();
        t2.join();
        log("최종 잔액: " + account.getBalance());
    }
}

```

(초기 셋업 코드)

### 1.1. 공유 자원에 대한 동시 접근 시나리오

```java
package thread.sync;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV1 implements BankAccount {
    private int balance;

    public BankAccountV1(int balance) {
        this.balance = balance;
    }

    @Override
    public boolean withdraw(int amount) {
        log("거래 시작: " + getClass().getSimpleName());
        log("[검증 시작] 출금액: " + amount + ", 잔액: " + balance);
        if (balance < amount) {
            log("[검증 실패] 출금액: " + amount + ", 잔액: " + balance);
            return false;
        }
        log("[검증 완료] 출금액: " + amount + ", 잔액: " + balance);
        sleep(1000); // 출금에 걸리는 시간으로 가정
        balance = balance - amount;
        log("[출금 완료] 출금액: " + amount + ", 변경 잔액: " + balance);
        log("거래 종료");
        return true;
    }

    @Override
    public int getBalance() {
        return balance;
    }
}
```

```
// 실행 결과
12:16:55:211 [       t1] 거래 시작: BankAccountV1
12:16:55:210 [       t2] 거래 시작: BankAccountV1
12:16:55:221 [       t1] [검증 시작] 출금액: 800, 잔액: 1000
12:16:55:221 [       t2] [검증 시작] 출금액: 800, 잔액: 1000
12:16:55:222 [       t1] [검증 완료] 출금액: 800, 잔액: 1000
12:16:55:222 [       t2] [검증 완료] 출금액: 800, 잔액: 1000
12:16:55:688 [     main] t1 state: TIMED_WAITING
12:16:55:688 [     main] t2 state: TIMED_WAITING
12:16:56:237 [       t1] [출금 완료] 출금액: 800, 변경 잔액: 200
12:16:56:237 [       t2] [출금 완료] 출금액: 800, 변경 잔액: -600
12:16:56:238 [       t1] 거래 종료
12:16:56:238 [       t2] 거래 종료
12:16:56:242 [     main] 최종 잔액: -600
```

위 상황에서는 동시에 2개의 스레드를 통해서 출금을 하고 있음

* t1, t2 스레드가거의 동시의 실행됨

과정:

<div align="left"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

t1: 잔액(1000)이 출금액(800)보다 많으므로 검증 로직 통과

<div align="left"><figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure></div>

t2: 잔액(1000)이 출금액(800)보다 많으므로 검증 로직 통과

<div align="left"><figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure></div>

t1: balance(1000) - amount(800) = 200

<div align="left"><figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure></div>

t2: balance(200) - amount(800) = -600



다른 경우가 발생할 수도 있다.

```
12:31:10.885 [       t2] 거래 시작: BankAccountV1
12:31:10.885 [       t1] 거래 시작: BankAccountV1
12:31:10.892 [       t1] [검증 시작] 출금액: 800, 잔액: 1000
12:31:10.892 [       t2] [검증 시작] 출금액: 800, 잔액: 1000
12:31:10.893 [       t1] [검증 완료] 출금액: 800, 잔액: 1000
12:31:10.893 [       t2] [검증 완료] 출금액: 800, 잔액: 1000
12:31:11.370 [     main] t1 state: TIMED_WAITING
12:31:11.370 [     main] t2 state: TIMED_WAITING
12:31:11.898 [       t2] [출금 완료] 출금액: 800, 변경 잔액: 200
12:31:11.898 [       t1] [출금 완료] 출금액: 800, 변경 잔액: 200
12:31:11.899 [       t2] 거래 종료
12:31:11.899 [       t1] 거래 종료
12:31:11.905 [     main] 최종 잔액: 200
```

이러한 경우에는 balance = balance - amount부분에서 문제가 발생.

* t1: 잔액(1000)을 읽음
* t2:잔액(1000)을 읽음
* t1: 출금(800) 후 잔액(200) 저장
* t2: 출금(800) 후 잔액(200) 저장



정리하자면 두가지 케이스로 나뉜다.

1. t1 쓰레드의 출금 후 잔액 저장 전에, t2 쓰레드의 출금 여부 확인 통과.
2. t1 쓰레드의 출금 후 잔액 저장 전에, t2 쓰레드가 잔액을 불러왔을때.

### 1.2. 임계영역과Synchronized

#### 1.2.1.  임계영역(Critical Section)

* 여러 스레드가 동시에 접근하면 데이터 불일치나 예상치 못한 동작이 발생할 수 있는 코드 부분
* 여러 스레드가 동시에 접근해서는 안되는 공유 자원을 접근하거나 수정하는 부분 (공유 변수나 공유 객체 수정)

#### 1.2.2. Synchronized

위 상황을 피하기 위해서는 synchronized 키워드를 사용할 수 있다.

```java
package thread.sync;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV2 implements BankAccount {
    private int balance;

    public BankAccountV2(int balance) {
        this.balance = balance;
    }

    @Override
    public synchronized boolean withdraw(int amount) {
        log("거래 시작: " + getClass().getSimpleName());
        log("[검증 시작] 출금액: " + amount + ", 잔액: " + balance);
        if (balance < amount) {
            log("[검증 실패] 출금액: " + amount + ", 잔액: " + balance);
            return false;
        }
        log("[검증 완료] 출금액: " + amount + ", 잔액: " + balance);
        sleep(1000); // 출금에 걸리는 시간으로 가정
        balance = balance - amount;
        log("[출금 완료] 출금액: " + amount + ", 변경 잔액: " + balance);
        log("거래 종료");
        return true;
    }

    @Override
    public int getBalance() {
        return balance;
    }
}
```

* 단순히 메서드 앞에 synchronized를 붙여주면 된다.
* withdraw 메서드 전체가 "보호받는 임계영역"이 된다. 한번에 하나의 스레드만 접근 가능.

```
// 실행 결과
13:58:39:633 [       t1] 거래 시작: BankAccountV2
13:58:39:643 [       t1] [검증 시작] 출금액: 800, 잔액: 1000
13:58:39:643 [       t1] [검증 완료] 출금액: 800, 잔액: 1000
13:58:40:108 [     main] t1 state: TIMED_WAITING
13:58:40:108 [     main] t2 state: BLOCKED
13:58:40:645 [       t1] [출금 완료] 출금액: 800, 변경 잔액: 200
13:58:40:645 [       t1] 거래 종료
13:58:40:646 [       t2] 거래 시작: BankAccountV2
13:58:40:646 [       t2] [검증 시작] 출금액: 800, 잔액: 200
13:58:40:647 [       t2] [검증 실패] 출금액: 800, 잔액: 200
13:58:40:650 [     main] 최종 잔액: 200
```

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

* t1이 모두 수행될떄까지 t2는 메서드를 시작할 수 없게 된다.
* t1이 실행될동안 t2는 **BLOCKED** 상태이다.
* 현재는 두개의 스레드만 존재하지만, 여러 스레드가 있을때 락을 획득하는 다음 스레드는 랜덤이다.
* synchronized안에서 접근하는 변수의 메모리 가시성 문제는 volatile이 없어도 해결됨(JMM의 Happens-Before)

### 1.3. Synchronized 코드 블록

위 코드를 자세히 보면, 보호가 필요없는 임계영역이 존재한다. 메서드 전체가 아닌 특정 코드 부분만 보호되는 임계영역으로 지정할 수 있다.

```java
package thread.sync;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV3 implements BankAccount {
    private int balance;

    public BankAccountV3(int balance) {
        this.balance = balance;
    }

    @Override
    public boolean withdraw(int amount) {
        log("거래 시작: " + getClass().getSimpleName());

        synchronized(this) {
            log("[검증 시작] 출금액: " + amount + ", 잔액: " + balance);
            if (balance < amount) {
                log("[검증 실패] 출금액: " + amount + ", 잔액: " + balance);
                return false;
            }
            log("[검증 완료] 출금액: " + amount + ", 잔액: " + balance);
            sleep(1000); // 출금에 걸리는 시간으로 가정
            balance = balance - amount;
            log("[출금 완료] 출금액: " + amount + ", 변경 잔액: " + balance);
        }

        log("거래 종료");
        return true;
    }

    @Override
    public int getBalance() {
        return balance;
    }
}
```

* synchronized(this)를 사용하여 코드 영역을 지정할 수 있다.

```
// 실행 결과
14:17:46:387 [       t1] 거래 시작: BankAccountV3
14:17:46:387 [       t2] 거래 시작: BankAccountV3
14:17:46:397 [       t1] [검증 시작] 출금액: 800, 잔액: 1000
14:17:46:397 [       t1] [검증 완료] 출금액: 800, 잔액: 1000
14:17:46:863 [     main] t1 state: TIMED_WAITING
14:17:46:863 [     main] t2 state: BLOCKED
14:17:47:399 [       t1] [출금 완료] 출금액: 800, 변경 잔액: 200
14:17:47:399 [       t1] 거래 종료
14:17:47:399 [       t2] [검증 시작] 출금액: 800, 잔액: 200
14:17:47:400 [       t2] [검증 실패] 출금액: 800, 잔액: 200
14:17:47:404 [     main] 최종 잔액: 200
```

* "거래 시작" 로그는 동시에 여러 스레드가 접근해도 문제가 없기때문에, 이 부분을 제외했다.
* 로그를 봤을때 t2의 "거래   시작" 까지는 실행되는것을 볼 수 있다.

### 1.4. Synchronized의 단점

* 무한 대기: BLOCKED 상태의 스레드는 락이 풀릴 때 까지 무한 대기
  * 특정 시간까지만 대기하는 타임아웃 불가
  * 중간에 인터럽트 불가
* 공정성: 락이 돌아왔을 때 BLOCKED 상태의 여러 스레드 중에 어떤 스레드가 락을 획득할 지 알 수 없다. 최악의 경우 스레드가 너무 오랜기간 락을 획득하지 못할 수 있다.

조금 더 정교한 설정이 필요하면 java.util.concurrent 동시성 문제 해결을 위한 패키지를 사용한다.



## 2. concurrent.Lock

java.util.concurrent 라이브러리 패키지는 synchronized의 단점들을 보안하기 위해 만들어졌다.

### 2.1. LockSupport

`LockSupport`는 기본적인 스레드 동기화 메커니즘을 제공하는 클래스입니다. 이 클래스는 낮은 레벨의 잠금을 처리하며 스레드를 잠들게 하거나 깨우는 메서드를 제공합니다.&#x20;

`LockSupport`의 주요 기능들입니다:

* `park()`: 현재 스레드를 WAITING 상태로 만듭니다. 다른 스레드에 의해 깨워질 때까지 멈춰 있습니다.
* `unpark(Thread thread)`: 지정된 스레드를 WAITING 상태에서 RUNNABLE로 변경
* `parkNanos(long nanos)`: 주어진 시간(나노 초) 동안 스레드를 TIMED\_WAITING상태로 변경합니다. 시간이 완료되면 자동으로 깨어납니다.
* `parkUntil(long deadline)`: 지정된 시점까지 스레드를 멈춥니다. 이 기능은 주로 타임스탬프 기반으로 스레드를 일시 중지하고 싶을 때 유용합니다.

```java
package thread.sync.lock;

import java.util.concurrent.locks.LockSupport;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class LockSupportMainV1 {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new ParkTask(), "Thread-1");
        thread1.start();

        // 잠시 대기하여 Thread-1이 park 상태에 빠질 시간을 준다.
        sleep(100);
        log("Thread-1 state: " + thread1.getState());

        log("main -> unpark(Thread-1)");
        LockSupport.unpark(thread1); // 1. unpark 사용
        //thread1.interrupt();  // 2. interrupt() 사용
    }

    static class ParkTask implements Runnable {
        @Override
        public void run() {
            log("park 시작");
            LockSupport.park();
            log("park 종료, state: " + Thread.currentThread().getState());
            log("인터럽트 상태: " + Thread.currentThread().isInterrupted());
        }
    }
}

```

```
// 실행 결과
14:29:32:062 [ Thread-1] park 시작
14:29:32:150 [     main] Thread-1 state: WAITING
14:29:32:151 [     main] main -> unpark(Thread-1)
14:29:32:151 [ Thread-1] park 종료, state: RUNNABLE
14:29:32:155 [ Thread-1] 인터럽트 상태: false
```

* main 스레드가 Thread-1을 start() 하면 Thread-1은 RUNNABLE 상태가 된다.
* Thread-1은 Thread-park()를 호출. Thread-1은 RUNNABLE -> WAITING 상태가 된다.
* main 스레드가 Thread-1을 깨운다. Thread-1은 대기 상태에서 실행 가능 상태로 변한다. WAITING -> RUNNABLE
* park()와 unpark()의 차이는, park는 해당 스레드가 실행하고, unpark는 다른 스레드에 의해서 실행되어야한다.

### 2.2. Interrupt 사용

```
14:32:31:506 [ Thread-1] park 시작
14:32:31:580 [     main] Thread-1 state: WAITING
14:32:31:580 [     main] main -> unpark(Thread-1)
14:32:31:581 [ Thread-1] park 종료, state: RUNNABLE
14:32:31:586 [ Thread-1] 인터럽트 상태: true
```

LockSupport.unpark(thread1) 대신, thread1.interrupt()를 사용하게되어도 깨울 수 있지만 인터럽트 상태가 true인걸 확인할 수 있다.

### 2.3. 시간 대기

parkNanos(nanos)를 사용한 경우.

```java
package thread.sync.lock;

import java.util.concurrent.locks.LockSupport;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class LockSupportMainV2 {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new ParkTask(), "Thread-1");
        thread1.start();

        // 잠시 대기하여 thread1이 park 상태에 빠질 시간을 준다.
        sleep(100);
        log("Thread-1 state: " + thread1.getState());
    }

    static class ParkTask implements Runnable {
        @Override
        public void run() {
            log("park 시작, 2초 대기");
            LockSupport.parkNanos(2000_000000); // parkNanos 사용
            log("park 종료, state: " + Thread.currentThread().getState());
            log("인터럽트 상태: " + Thread.currentThread().isInterrupted());
        }
    }
}
```

```
14:34:49:069 [ Thread-1] park 시작, 2초 대기
14:34:49:149 [     main] Thread-1 state: TIMED_WAITING
14:34:51:084 [ Thread-1] park 종료, state: RUNNABLE
14:34:51:088 [ Thread-1] 인터럽트 상태: false
```

* 따로 unpark를 해주지 않아도, 2초후에 꺠어난다.



BLOCKED vs WAITING

WAITING 상태에 특정한 시간까지만 대기하는 기능이 포함된 것이 TIMED\_WAITING. 둘을 묶어서 WAITING 상태라고 가정.

* 인터럽트
  * BLOCKED 상태는 인터럽트가 걸려도 대기 상태를 빠져나오지 못한다. 여전히 BLOCKED 상태
  * WAITING, TIMED\_WAITING 상태는 인터럽트가 걸리면 대기 상태를 빠져 나온다. RUNNABLE 상태로 변경.
* 용도
  * BLOCKED 상태: 자바의 synchronized에서 락을 획득하기 위해 대기할 때 사용.
  * WAITING, TIMED\_WAITING 상태: 스레드가 특정 조건이나 시간 동안 대기할 때 발생하는 상태
  * WAITING 상태: 다양한 상황에서 사용된다. 예를 들어, Thread.join(), LockSupport.park(), Object.wait()과 같은 메서드 호출시 WAITING 상태가 된다.
  * TIMED\_WAITING 상태: Thread.sleep(ms), Object.wait(long timeout), Thread.join(long millis), LockSupport.parkNanos(ns) 등과 같은 시간 제한이 있는 대기 메서드 호출시 발생.
* Thread.join() <--> Thread.join(millis)
* Thread.park() <--> Thread.parkNanos(long millis)
* Object.wait() <--> Object.wait(long timeout)

이러한 기능을 직접 구현하기는 어렵다. 여러 스레드가 대기하고 있는 상태에서 다음 스레드를 선정하는 것, 우선순위를 부여하는 것 등의 세밀한 작업은 LockSupport로 하기에는 synchronized 보다 더 저수준의 기능이다. 이를 해결하기 위해 Lock 인터페이스와 ReentrantLock이 존재한다.



### 2.4. Lock 인터페이스

자바1.0 Synchronized와 BLOCKED 상태를 통한 임계 영역 관리의 한계를 극복하기 위해 자바1.5부터 Lock 인터페이스와 ReentrantLock 구현체가 등장했다.

Lock 인터페이스를 통해서 synchronized의 단점인 **무한 대기 문제 해결**

**여기서의 Lock은 객체 내부에 있는 모니터 락이랑 다른 락이다. Lock 인터페이스와 ReentrantLock이 제공하는 기능이다.**



Lock 인터페이스는 아래와 같다:

```java
package java.util.concurrent.locks;

public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```

* void lock()
  * 락을 획득. 만약 다른 스레드가 이미 락을 획득 했다면, 락이 풀릴때까지 대기(WAITING) 한다. 인터럽트에 응답하지 않음
  * 맛집에 줄을 서면 끝까지 기다린다. 친구가 다른 맛집을 찾았다고 중간에 연락해도 포기하지 않고 기다림.
  * **WAITING 상태의 스레드에 인터럽트가 발생하면 WAITING 상태를 빠져나오는게 정상. 근데 lock()의 경우에는 다시 WAITING으로 변경시킨다. 결국 인터럽트를 무시**
* void lockInterruptibly()
  * 락 획득을 시도하되, 다른 스레드가 인터럽트 할 수 있다. 락을 획득할 수 없다면 획득할때까지 대기한다. 대기중 인터럽트가 발생하면 InterruptException이 발생하며 락 획득을 포기
  * 맛집에 줄을 서서 기다린다. 친구가 다른 맛집을 찾았다고 중간에 연락하면 포기한다.
* boolean tryLock()
  * 락 획득을 시도하고, 즉시 성공 여부 반환. 다른 스레드가 이미 락을 획득하면 false를 반환
  * true 반환 -> 락 획득 -> 임계 구역 코드 실행
  * false 반환 -> 포기
  * 맛집에 대기 줄이 없으면 바로 들어가고, 대기 줄이 있으면 즉시 포기
* boolean tryLock(long time, TimeUnit unit)
  * 주어진 시간동안 락 획득 시도. 주어진 시간 안에 락을 획득하면 true 반환. 주어진 시간이 지나도 락을 획득하지 못한 경우 false를 반환.
  * 인터럽트가 발생하면 InterruptedException이 발생하며 락 획득 포기
  * 맛집에 줄을 서지만 특정 시간 만큼만 기다린다. 특정 시간이 지난 후에도 계속 줄을 서야 하면 포기. 친구가 다른 맛집을 찾았다고 중간에 연락해도 포기
* Condition newCondition()
  * Condition 객체를 생성하여 반환. Condition 객체는 락과 결합되어 사용되며, 스레드가 특정 조건을 기다리거나 신호를 받을 수 있도록 한다. Object 클래스의 wait, notify, notifyAll 메서드와 유사함



### 2.5. ReentrantLock

