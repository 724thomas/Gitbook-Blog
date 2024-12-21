---
description: CAS - 동기화와 원자적 연산
---

# CAS (Compare and Set)

## 1. CAS (Compare and Set)

락을 사용하지 않고 원자적인 연산을 수행할 수 있는 방법. Lock-free 기법이라고도 함. CAS는 내부 연산 속도가 빠를 수록 락에 비해 성능이 우수하지만, 내부 연산 속도가 느리면 락에 비해 성능이 더 느려질 수 있음

```java
package thread.cas;

import java.util.concurrent.atomic.AtomicInteger;

public class CasMainV1 {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(0);
        System.out.println("start value = " + atomicInteger.get());

        boolean result1 = atomicInteger.compareAndSet(0, 1);
        System.out.println("result1 = " + result1 + ", value = " + atomicInteger.get());

        boolean result2 = atomicInteger.compareAndSet(0, 1);
        System.out.println("result2 = " + result2 + ", value = " + atomicInteger.get());
    }
}

```

```
start value = 0
result1 = true, value = 1
result2 = false, value = 1
```

* CompareAndSet을 사용하는 경우, 내부적으로 원자적으로 계산하게 됨
  * 원래는 조회하고 값을 갱신하는 작업은 원자적이지 않지만, CompareAndSet은 이를 원자적으로 수행. 이게 가능한 이유는 하드웨어에서 제공하는 기능 덕분
  * CompareAndSet(0, 1) -> value 값이 0 이면 1로 변경하는 것. 그렇기 때문에 1번째 연산은 성공, 2번째 연산은 실패를 리턴.

```java
package thread.cas;

import java.util.concurrent.atomic.AtomicInteger;

import static util.MyLogger.log;

public class CasMainV2 {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(0);
        System.out.println("start value = " + atomicInteger.get());

        int resultValue1 = incrementAndGet(atomicInteger);
        System.out.println("resultValue1 = " + resultValue1 + ", value = " + atomicInteger.get());

        int resultValue2 = incrementAndGet(atomicInteger);
        System.out.println("resultValue2 = " + resultValue2 + ", value = " + atomicInteger.get());

    }

    private static int incrementAndGet(AtomicInteger atomicInteger) {
        int getValue;
        boolean result;
        do {
            getValue = atomicInteger.get();
            log("getValue = " + getValue);
            result = atomicInteger.compareAndSet(getValue, getValue + 1);
            log("result = " + result);
        } while (!result);
        return getValue + 1;
    }
}

```

```
start value = 0
21:40:33:603 [     main] getValue = 0
21:40:33:605 [     main] result = true
resultValue1 = 1, value = 1
21:40:33:609 [     main] getValue = 1
21:40:33:610 [     main] result = true
resultValue2 = 2, value = 2
```

* 둘다 성공할 수 있는 이유는, atomicInteger.get()을 사용하여 값을 읽고, getValue+1을 사용하여 값을 메모리에 갱신. 만약 실패하면 성공할때까지 계속 시도.



### 1.1. 멀티스레드 상황

```java
package thread.cas;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class CasMainV3 {

    private static final int THREAD_COUNT = 2;
    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicInteger = new AtomicInteger(0);
        System.out.println("start value = " + atomicInteger.get());

        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                incrementAndGet(atomicInteger);
            }
        };

        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < THREAD_COUNT; i++) {
            Thread thread = new Thread(runnable);
            threads.add(thread);
            thread.start();
        }

        for (Thread thread : threads) {
            thread.join();
        }

        int result = atomicInteger.get();
        System.out.println(atomicInteger.getClass().getSimpleName() + " result = " + result);
    }

    private static int incrementAndGet(AtomicInteger atomicInteger) {
        int getValue;
        boolean result;
        do {
            getValue = atomicInteger.get();
            sleep(100);
            log("getValue = " + getValue);
            result = atomicInteger.compareAndSet(getValue, getValue + 1);
            log("result = " + result);
        } while (!result);
        return getValue + 1;
    }
}

```

```
start value = 0
21:46:43:093 [ Thread-1] getValue = 0
21:46:43:093 [ Thread-0] getValue = 0
21:46:43:096 [ Thread-1] result = true
21:46:43:096 [ Thread-0] result = false
21:46:43:209 [ Thread-0] getValue = 1
21:46:43:209 [ Thread-0] result = true
AtomicInteger result = 2
```

* 정상적으로 2로 증가한 것을 확인
* AtomicInteger에서 제공하는 incrementAndGet() 코드는 직접 작성한 incrementAndGet()코드와 똑같이 CAS를 활용하도록 작성되어있음.
* CAS는 락 충돌이 자주 발생하지 않는 (낙관적 락) 상황에서 락을 획득, 반납하고 WAITING, RUNNABLE이 되는 시간이 없어서 오버헤드가 줄어듬.

## 2. Lock vs CAS

### 2.1. Lock

* 비관적 접근
* 데이터가 접근하기 전에 항상 락을 획득
* 다른 스레드의 접근을 막음

### 2.2. CAS

* 낙관적 접근
* 락을 사용하지 않고 데이터에 바로 접근
* 충돌이 발생하면 그때 재시도



## 3. CAS 구현

### 3.1. 락 구현1 (bad)

```java
package thread.cas.spinlock;

import static util.MyLogger.log;

public class SpinLockMain {
    public static void main(String[] args) {
        SpinLockBad spinLock = new SpinLockBad();

        Runnable task = new Runnable() {
            @Override
            public void run() {
                spinLock.lock();
                try {
                    log("비즈니스 로직 실행");
                } finally {
                    spinLock.unlock();
                }
            }
        };

        Thread t1 = new Thread(task, "Thread-1");
        Thread t2 = new Thread(task, "Thread-2");

        t1.start();
        t2.start();
    }
}
```

```java
package thread.cas.spinlock;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class SpinLockBad {
    private volatile boolean lock = false;

    public void lock() {
        log("락 획득 시도");
        while(true) {
            if (!lock) {
                sleep(100);
                lock =true;
                break;
            } else {
                // 락을 획득할 때 까지 스핀 대기
                log("락 획득 실패, 스핀 대기");
            }
        }
        log("락 획득 성공");
    }

    public void unlock() {
        lock = false;
        log("락 반납 완료");
    }
}

```

* 위 상황에서는 당연히 원자적 연산이 아니기 떄문에 문제가 발생.
  * 락 사용 여부 확인
  * 락의 값 변경

만약 이 두 코드를 하나로 묶어서 원자적으로 처리 한다면?

CAS 연산을 사용하면 두 연산을 하나로 묶어서 처리 가능. 락의 사용 여부를 확인하고, 그 값이 기대하는 값과 같다면 변경. CAS 연산이 필요한 시기이다.



### 3.2. 락 구현2 (good)

```java
package thread.cas.spinlock;

import java.util.concurrent.atomic.AtomicBoolean;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class SpinLock {
    private final AtomicBoolean lock = new AtomicBoolean(false);

    public void lock() {
        log("락 획득 시도");
        while(!lock.compareAndSet(false, true)) {
            // 락을 획득할 때 까지 스핀 대기
            log("락 획득 실패, 스핀 대기");
        }
        log("락 획득 성공");
    }

    public void unlock() {
        lock.set(false);
        log("락 반납 완료");
    }
}

```

```
21:56:37:725 [ Thread-1] 락 획득 시도
21:56:37:725 [ Thread-2] 락 획득 시도
21:56:37:729 [ Thread-1] 락 획득 성공
21:56:37:729 [ Thread-2] 락 획득 실패, 스핀 대기
21:56:37:729 [ Thread-1] 비즈니스 로직 실행
21:56:37:729 [ Thread-2] 락 획득 실패, 스핀 대기
21:56:37:730 [ Thread-1] 락 반납 완료
21:56:37:730 [ Thread-2] 락 획득 성공
21:56:37:730 [ Thread-2] 비즈니스 로직 실행
21:56:37:730 [ Thread-2] 락 반납 완료
```

```java
while(
    if (!lock) {
        lock = true;
    }
}

->
while (lock.compareAndSet(false, true)) {}
```

* CAS 연산을 사용하는 AtomicBoolean을 사용했을때, 락 사용 여부확인과 락의 값 변경이 원자적으로 됨
* 결과를 보면 락이 잘 적용된 것을 확인.

## 4. CAS 단점

* 락을 기다리는 스레드가 BLOCKED, WAITING 상태로 빠지지는 않지만, RUNNABLE 상태로 락을 획득할때 까지 스핀락이 발생.
* 결론적으로 스핀락이 발생해도 내부 코드가 빠르게 동작하게 되면 락 획득을 자주 확인하기 때문에 효율적이라는 이론이지만, 아무래도 직접 테스트를 하는게 베스트.
* 데이터베이스의 요청을 기다리는 상황 같은 경우, CAS 요청이 적절하지 않을 가능성이 매우 높다.



