---
description: 쓰레드
---

# Thread

## 1. 자바 메모리 구조

<figure><img src="../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

* 메서드 영역: 프로그램을 실행하는데 필요한 공통 데이터를 관리. 모든 영역에서 공유함
  * 클래스 정보: 클래스의 실행 코드(바이트 코드), 필드, 메서드와 생성자 코드등 모든 실행 코드가 존재.
  * static 영역: static 변수들을 보관
  * 런타임 상수 풀: 프로그램을 실행하는데 필요한 공통 리터럴 상수를 보관.
* 스택 영역: 자바 실행 시, 하나의 실행 스택이 생성. 각 스텍 프레임은 지역 변수, 중간 연산 결과, 메서드 호출 정보 등을 포함
  * 스택 프레임: 메서드를 호출할 떄 마다 하나의 스택 프레임이 쌓이고, 메서드가 종료되면 해당 스택 프레임이 제거
  * 스택 영역은 각 스레드별로 하나의 실행 스택이 생성. 따라서 스레드 수 만큼 스택이 생성됨
* 힙 영역: 객체와 배열이 생성되는 영역. GC가 이루어지는 주요 영역.



## 2. 스레드 생성

스레드를 만들 때는 Thread 클래스를 상속 받는 방법과 Runnable 인터페이스를 구현하는 방법이 있다.



### 2.1. Thread 상속

```java
//메인 스레드
package thread.start;

public class HelloThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        HelloThread helloThread = new HelloThread();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 전");
        helloThread.start();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}

```

* 앞서 만든 HelloThread 스레드 객체를 생성하고 start() 메서드를 호출한다.&#x20;
* start() 메서드는 스레드를 실행하는 아주 특별한 메서드이다.&#x20;
* start() 를 호출하면 HelloThread 스레드가 run() 메서드를 실행한다.

```java
// 새로운 스레드
package thread.start;

public class HelloThread  extends Thread{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": run()");
    }
}

```

* Thread 클래스를 상속하고, 스레드가 실행할 코드를 run() 메서드에 재정의한다.&#x20;
* Thread.currentThread()를 호출하면 해당 코드를 실행하는 스레드 객체를 조회할 수 있다.&#x20;
* Thread.currentThread().getName()\` : 실행 중인 스레드의 이름을 조회한다.

```
// 실행결과
main: main() start
main: start() 호출 전
main: start() 호출 후
main: main() end
Thread-0: run()
(main() end가 먼저 실행됐지만, 이건 스케줄링에 의해 순서가 변할 수 있음)
```

* HelloThread 스레드 객체를 생성한 다음에 start() 메서드를 호출하면 자바는 스레드를 위한 별도의 스택 공간을 할당한다.
* 스레드 객체를 생성하고 반드시 start() ( run()이 아님)를 호출해야 스택 공간을 할당 받고 스레드가 작동한다.
* 스레드에 이름을 주지 않으면 자바는 스레드에 Thread-0, Thread-1 같은 임의의 이름 부여
* 새로운 Thread-0 스레드가 사용할 전용 스택 공간 마련됨
* Thread-0 스레드는 run() 메서드의 스택 프레임을 스택에 올리면서 run() 메서드를 시작.



<figure><img src="../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

* main 스레드가 HelloThread 인스턴스를 생성.
* start() 메서드 호출 후 Thread-0 스레드가 run() 메서드 호출. main 스레드가 run()을 호출하는게 아니라 Thread-0 스레드가 run() 메서드를 실행.
* main 스레드는 단지 start() 메서드를 통해 Thread-0 스레드에게 실행을 지시할 뿐.
* main, Thread-0 스레드는 동시에 실행



### 2.1. start() vs run()

```java
// start()가 아니라 run()을 실행
package thread.start;

public class HelloThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        HelloThread helloThread = new HelloThread();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 전");
        helloThread.run();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}

```

```
// 실행 결과
main: main() start
main: start() 호출 전
main: run()
main: start() 호출 후
main: main() end
```

<figure><img src="../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

* 별도의 스레드가 run()을 실행하지 않고, main 스레드가 run() 메서드를 호출

### 2.2. 데몬 스레드 vs 사용자 스레드(non-daemon)

* 사용자 스레드
  * 프로그램의 주요 작업 수행
  * 작업이 완료될 때까지 실행된다.&#x20;
  * 모든 user 스레드가 종료되면 JVM도 종료된다.
* 데몬 스레드
  * 백그라운드에서 보조적인 작업을 수행한다.&#x20;
  * 모든 user 스레드가 종료되면 데몬 스레드는 자동으로 종료된다.

```java
// 데몬 스레드
package thread.start;

public class DaemonThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        DaemonThread daemonThread = new DaemonThread();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 전");
        daemonThread.setDaemon(true);
        daemonThread.start();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }

    static class DaemonThread extends Thread {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + ": run() start");
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println(Thread.currentThread().getName() + ": run() end");
        }
    }
}
// run() 메서드 안에서 Thread.sleep()을 호출할때 
// 체크 예외인 InterruptedException을 밖으로 던질 수 없고 반드시 잡아야한다.
```

* setDaemon(true) 데몬 스레드로 설정
* 데몬 스레드 여부는 start() 실행전에 결정.
* 기본 값은 false (user 스레드가 기본)

```
// 실행 결과
main: main() start
main: start() 호출 전
main: start() 호출 후
main: main() end
Thread-0: run() start
```

* 데몬 스레드가 종료되지 않았는데, 메인 스레드가 종료됨으로서 자동으로 종료됨.



### 2.3. Runnable

Thread 클래스를 상속받지 않고 Runnable 인터페이스를 구현하는 방법

```java
// Runnable Main
package thread.start;

public class HelloRunnableMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");

        HelloRunnable helloRunnable = new HelloRunnable();
        Thread thread = new Thread(helloRunnable);
        System.out.println(Thread.currentThread().getName() + ": start() 호출 전");
        thread.start();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 후");

        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}

```

```java
// Runnable Thread
package thread.start;

public class HelloRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + ": run()");
    }
}

```

```
// 실행 결과
main: main() start
main: start() 호출 전
main: start() 호출 후
main: main() end
Thread-0: run()
```

* 실행 결과는 같음
* 다른 점은, 스레드와 해당 스레드가 실행할 작업이 서로 분리.

**핵심 차이**

1. **`Thread` 상속**:
   * 실행 작업(run 메서드)과 스레드 제어(start, join 등)가 **하나의 클래스**에 결합됩니다.
   * 즉, 작업(비즈니스 로직)과 **스레드 제어 로직**이 혼재합니다.
   * 실행 작업을 다른 스레드에서 재사용하거나 독립적으로 분리하기 어렵습니다.
2. **Runnable 구현**:
   * 실행 작업(run 메서드)은 **Runnable 인터페이스**에 의해 정의되고, **스레드 제어는 별도**로 Thread 객체가 담당합니다.
   * 실행 작업과 스레드 제어가 **완전히 분리**되어 더 유연한 설계가 가능합니다.

#### **작업이 분리되어 있다는 의미**

**`Runnable` 구현의 장점: 작업과 스레드의 분리**

1.  **작업 로직의 재사용 가능**:

    * `Runnable` 객체는 여러 `Thread` 객체에서 재사용할 수 있습니다.
    * 동일한 작업(`HelloRunnable`)을 여러 스레드에서 실행하거나, 다른 컨텍스트에서 실행할 때 유용합니다.

    **예시**:

    ```java
    HelloRunnable runnable = new HelloRunnable();
    Thread thread1 = new Thread(runnable, "thread-1");
    Thread thread2 = new Thread(runnable, "thread-2");

    thread1.start();
    thread2.start();
    ```
2. **작업과 스레드 관리가 독립적**:
   * `Runnable` 구현은 스레드와 작업 로직을 명확히 분리합니다.
   * 스레드 제어(`start`, `join`, `interrupt`)는 `Thread` 클래스에서, 작업 로직은 `Runnable` 객체에서 관리합니다.
   * 작업 로직을 다른 스레드 관리 메커니즘(ex. `ExecutorService`)에서도 쉽게 사용할 수 있습니다.

***

#### **실행 흐름 비교**

**`HelloThread` 상속 방식**

```java
HelloThread helloThread = new HelloThread();
helloThread.start();
```

* **결합 구조**:
  * `HelloThread` 객체는 **스레드의 실행 로직**과 **스레드 관리 로직**을 모두 포함합니다.
  * `run` 메서드를 실행하려면 반드시 `HelloThread`를 `start()`로 시작해야 합니다.

***

**`Runnable` 구현 방식**

```java
HelloRunnable helloRunnable = new HelloRunnable();
Thread thread = new Thread(helloRunnable);
thread.start();
```

* **분리 구조**:
  * `Runnable` 객체는 실행 작업만 정의하고, 스레드 제어는 `Thread`가 담당합니다.
  * 실행 작업은 재사용 가능하며, 스레드 관리와 별도로 설계할 수 있습니다.



### 2.4. 로거 만들기

{% code fullWidth="true" %}
```java
package util;

import java.time.LocalTime;
import java.time.format.DateTimeFormatter;

public class MyLogger {
    private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("HH:mm:ss:SSS");
    public static void log(Object obj) {
        String time = LocalTime.now().format(formatter);
        System.out.printf("%s [%9s] %s\n", time, Thread.currentThread().getName(), obj);
    }
}

```
{% endcode %}

```java
package util;

import static util.MyLogger.log;

public class MyLoggerMain {
    public static void main(String[] args) {
        log("Hello, MyLogger");
        log(123);
    }
}

```

```
05:53:14:395 [     main] Hello, MyLogger
05:53:14:397 [     main] 123
```



### 2.5. 여러 스레드 만들기

```java
package thread.start;

import static util.MyLogger.log;

public class ManyThreadMainV1 {
    public static void main(String[] args) {
        log("main() start");

        HelloRunnable helloRunnable = new HelloRunnable();
        Thread thread1 = new Thread(helloRunnable);
        thread1.start();
        Thread thread2 = new Thread(helloRunnable);
        thread2.start();
        Thread thread3 = new Thread(helloRunnable);
        thread3.start();

        log("main() end");
    }
}

```

```
05:54:59:467 [     main] main() start
05:54:59:471 [     main] main() end
Thread-2: run()
Thread-0: run()
Thread-1: run()
```

<figure><img src="../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

* 스레드 3개를 생성할 때 모두 같은 HelloRunnable 인스턴스(x001)를 스레드의 실행 작업으로 전달.
* Thread-0, 1, 2는 모두 HelloRunnable 인스턴스에 있는 run() 메서드를 실행.

### 2.6. Runnable을 만드는 여러 방법들

* 중첩 클래스 사용

```java
package thread.start;

import static util.MyLogger.log;

public class InnerRunnableMainV1 {
    public static void main(String[] args) {
        log("main() start");

        Runnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();

        log("main() end");
    }

    static class MyRunnable implements Runnable {
        @Override
        public void run() {
            log("run()");
        }
    }
}

```

* 익명 클래스 사용

```java
package thread.start;

import static util.MyLogger.log;

public class InnerRunnableMainV2 {
    public static void main(String[] args) {
        log("main() start");

        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                log("run()");
            }
        };
        Thread thread = new Thread(runnable);
        thread.start();

        log("main() end");
    }
}

```

* 익명 클래스 변수 없이 직접 전달

```java
package thread.start;

import static util.MyLogger.log;

public class InnerRunnableMainV3 {
    public static void main(String[] args) {
        log("main() start");

        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                log("run()");
            }
        });
        thread.start();

        log("main() end");
    }
}
```

* 람다

```java
package thread.start;

import static util.MyLogger.log;

public class InnerRunnableMainV4 {
    public static void main(String[] args) {
        log("main() start");

        Thread thread = new Thread(() -> log("run()"));
        thread.start();

        log("main() end");
    }
}

```



## 3. 스레드 제어와 생명 주기

### 3.1. 스레드 제어

#### 3.1.1. 스레드 생성

```java
 Thread myThread = new Thread(new HelloRunnable(), "myThread")
```

* Runnable 인터페이스: 실행할 작업을 포함하는 인터페이스. HelloRunnable 클래스는 Runnable 인터페이스를 구현한 클래스.
* 스레드 이름: "myThread" 라는 이름으로 스레드 생성. 디버깅/로깅 목적. (default : Thread-0, -1, ...)

#### 3.1.2. 스레드 객체 정보

```java
log("myThread = " + myThread);
```

* myThread 객체를 문자열로 변환하여 출력.
* toString() -> 스레드ID, 스레드 이름, 우선순위, 스레드 그룹을 포함 (Thread\[#21,myThread,5,main])

#### 3.1.3. 스레드ID

```java
log("myThread.threadId() = " + myThread.threadId());
```

* threadId(): 스레드 고유 식별자를 반환. ID는 JVM내에서 유일. 직접 지정할 수 없으며 생성시 할당.

#### 3.1.4. 스레드 이름

```java
 log("myThread.getName() = " + myThread.getName());
```

* getName(): 스레드 이름 반환. 스레드 이름은 중복 가능

#### 3.1.5. 스레드 우선순위

```java
 log("myThread.getPriority() = " + myThread.getPriority());
```

* getPriority(): 스레드의 우선순위를 반환하는 메서드. 1(가장 낮음)\~ 10(가장 높음). 기본값은 5. setPriority()를 통해 우선순위 변경 가능.
* 스레드 스케줄러가 어떤 스레드를 우선 실행할지 결정하는데 사용. JVM구성과 OS따라 달라질 수 있음

#### 3.1.6. 스레드 그룹 (잘 안쓰임)

```java
 log("myThread.getThreadGroup() = " + myThread.getThreadGroup());
```

* getThreadGroup(): 스레드가 속한 그룹을 반환. 스레드 그룹은 스레드를 그룹화하여 관리할 수 있는 기능 제공. 여러 스레드를 묶어서 특정 작업 수행가능(일괄 종료, 우선순위 설정 등)
* 기본적으로 모든 스레드는 부모 스레드와 동일한 스레드 그룹에 속한다.
* 부모 스레드: 새로운 스레드를 생성하는 스레드를 의미. 스레드는 다른 스레드에 의해 생성되는데, 이러한 생성 관계에서 새로 생성된 스레드는 생성한 스레드를 부모로 간주.

#### 3.1.7. 스레드 상태

```java
 log("myThread.getState() = " + myThread.getState());
```

* getState(): 스레드의 현재 상태를 반환.
  * NEW: 아직 시작되지 않은 상태
  * RUNNABLE: 스레드가 실행중 / 실행될 준비가 된 상태
  * BLOCKED: 동기화 락을 기다리는 상태
  * WAITING: 다른 스레드의 특정 작업이 완료되기를 기다리는 상태
  * TIMED\_WAITING: 일정 시간 동안 기다리는 상태
  * TERMINATED: 스레드가 실행을 마친 상태

### 3.2. 스레드 생명주기

<figure><img src="../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

* NEW: 아직 시작되지 않은 상태
  * 스레드가 생성되고 아직 시작되지 않은 상태
  * Thread 객체가 생성되지만 start() 메서드가 아직 호출되지 않은 상태.
* RUNNABLE: 스레드가 실행중 / 실행될 준비가 된 상태
  * 스레드가 실행될 준비가 된 상태. 실제 CPU에서 실행될 수 있음. start() 메서드가 호출되면 이 상태로 됨.
  * RUNNABLE은 실행되고 있는 뜻이 아니라, OS의 스케줄러가 각 스레드에 CPU 시간을 할당하여 실행하기 떄문에, Runnable 상태에 있는 스레드는 스케줄러의 실행 대기열에 포함되어 있다가 차례로 CPU에서 실행.
* BLOCKED: 동기화 락을 기다리는 상태
  * synchronized (lock) 같은 코드 블록에 진입하려고 할때, 다른 스레드가 이미 lock을 가지고 있는 경우
* WAITING: 다른 스레드의 특정 작업이 완료되기를 기다리는 상태
  * wait(), join() 메서드가 호출될때 이 상태로 됨. 다른 스레드가 notify() / notifyAll() 메서드를 호출하거나 join()이 완료될때까지 기다림.
* TIMED\_WAITING: 일정 시간 동안 기다리는 상태
  * sleep(XXms) / wait(long timeout), join(long millis) 메서드가 호출될때 이 상태로 ㅗ딤.
  * 주어진 시간이 경과 / 다른 스레드가 해당 스레드를 꺠우면 상태에서 벗어남.
* TERMINATED: 스레드가 실행을 마친 상태
  * 정상 종료 / 예외가 발생하여 종료된 경우. 스레드는 한번 종료되면 재시작 불가.



상태 전이 과정:

* NEW -> RUNNABLE: start() 메서드를 호출
* RUNNABLE -> BLOCKED / WAITING / TIMED\_WAITING: 락을 얻지 못하거나 wait() / sleep() 메서드를 호출
* BLOCKED / WAITING / TIMED\_WAITING -> RUNNABLE: 스레드가 락을 얻음 / 기다림이 완료
* RUNNABLE -> TERMINATED: run() 메서드가 완료됨

```java
import static util.MyLogger.log;

public class ThreadStateMain {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new MyRunnable(), "myThread");
        log("myThread.state1 = " + thread.getState()); // NEW
        log("myThread.start()");
        thread.start();
        Thread.sleep(1000);
        log("myThread.state3 = " + thread.getState()); // TIMED_WAITING
        Thread.sleep(4000);
        log("myThread.state5 = " + thread.getState()); // TERMINATED
        log("end");
    }

    static class MyRunnable implements Runnable {
        @Override
        public void run() {
            try {
                log("start");
                log("myThread.state2 = " +
                        Thread.currentThread().getState()); // RUNNABLE
                log("sleep() start");
                Thread.sleep(3000);
                log("sleep() end");
                log("myThread.state4 = " +
                        Thread.currentThread().getState()); // RUNNABLE
                log("end");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

```
// 실행 결과
 11:40:31.503 [     main] myThread.state1 = NEW
 11:40:31.505 [     main] myThread.start()
 11:40:31.505 [ myThread] start
 11:40:31.505 [ myThread] myThread.state2 = RUNNABLE
 11:40:31.505 [ myThread] sleep() start
 11:40:32.507 [     main] myThread.state3 = TIMED_WAITING
 11:40:34.510 [ myThread] sleep() end
 11:40:34.512 [ myThread] myThread.state4 = RUNNABLE
 11:40:34.512 [ myThread] end
 11:40:36.511 [     main] myThread.state5 = TERMINATED
 11:40:36.512 [     main] end
```

* Thread.currentThread() 를 호출하면 해당 코드를 실행하는 스레드 객체를 조회할 수 있다.
* Thread.sleep(): 해당 코드를 호출한 스레드는 TIMED\_WAITING 상태가 되면서 특정 시간 만큼 대기한다. 시간은 밀리초(ms) 단위이다. 1밀리초 = 1/1000 초, 1000밀리초 = 1초이다.&#x20;
* Thread.sleep(): InterruptedException 이라는 체크 예외를 던진다. 따라서 체크 예외를 잡아서 처리하거나 던져야 한다
* InterruptedException: 은 인터럽트가 걸릴 때 발생.

<figure><img src="../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

### 3.3. 체크 예외 재정의

```java
public interface Runnable {
    void run();
}
```

Runnable 인터페이스는 위와 같이 정의 되어있다.

체크 예외는:

* 부모 메서드가 체크 예외를 던지지 않는 경우, 재정의된 자식 메서드도 체크 예외를 던질 수 없다.
* 자식 메서드는 부모 메서드가 던질 수 있는 체크 예외의 하위 타입만 던질 수 있다.

위 규칙에 따라, runnable 인터페이스의 run() 메서드를 재정의하는 곳에서는 체크 예외를 밖으로 던질 수 없음.

체크 예외를 run() 메서드에서 던질 수 없게 강제하고 있는데, 이는 개발자가 반드시 예외를 try-cahtch 블록 내에서 처리해야된다. (프로그램의 비정상 종료 상황 방지). 특히 멀티스레딩 환경에서는 예외 처리를 강제함으로써 스레드의 안정성과 일관성을 유지할 수 있음.

### 3.4. Sleep 유틸리티

Thread.sleep() 메서드는 InterruptedException 체크 예외를 발생시킨다. 이를 매번 try-catch로 감싸주지 않는 방식으로 변경할 수 있음.

```java
package util;

import static util.MyLogger.log;

public abstract class ThreadUtils {
    public static void sleep(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            log("인터럽트 발생, " + e.getMessage());
            throw new RuntimeException(e);
        }
    }
}
```

### 3.5. join

Join은 특정 시간 만큼만 대기함.

* join(): 호출 스레드는 대상 스레드가 완료될 때까지 무한정 대기
* join(ms): 호출 스레드는 특정 시간 동안 만큼만 대기. 호출 스레드는 지정한 시간이 지나면 다시 RUNNABLE 상태가됨.

```java
package thread.control.join;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class JoinMainV4 {
    public static void main(String[] args) throws InterruptedException {
        log("Start");
        SumTask task1 = new SumTask(1, 50);
        Thread thread1 = new Thread(task1, "thread-1");

        thread1.start();

        //스레드가 종료될 때 까지 대기
        log("join(1000) - main 스레드가 thread1 종료까지 1초 대기");
        thread1.join(1000);
        log("main 스레드 대기 완료");

        log("task1.result = " + task1.result);
    }

    static class SumTask implements Runnable {
        int startValue;
        int endValue;
        int result = 0;

        public SumTask(int startValue, int endValue) {
            this.startValue = startValue;
            this.endValue = endValue;
        }

        @Override
        public void run() {
            log("작업 시작");
            sleep(2000);
            int sum = 0;
            for (int i = startValue; i <= endValue; i++) {
                sum += i;
            }
            result = sum;
            log("작업 완료 result = " + result);
        }
    }
}
```

* 별도의 스레드에서 1-50까지 더하고, 그 결과를 조회
* join(1000)을 사용해서 1초만 대기

```
// 실행 결과
17:34:54.572 [     main] Start
17:34:54.575 [     main] join(1000) - main 스레드가 thread1 종료까지 1초 대기
17:34:54.575 [ thread-1] 작업 시작
17:34:55.580 [     main] main 스레드 대기 완료
17:34:55.585 [     main] task1.result = 0
17:34:56.580 [ thread-1] 작업 완료 result = 1275
```

<figure><img src="../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

* main 스레드는 join(1000)을 사용해서 thread-1을 1초간 기다린다. 이때 main 스레드의 생태는 TIMED\_WAITING이다.
* thread-1의 작업에는 2초가 걸린다.
* 1초가 지나도 thread-1의 작업이 완료되지 않으므로, main 스레드는 대기를 중단. 그리고 main 스레드는 다시 RUNNABLE 상태로 바뀌면서 다음 코드를 수행. 이때 thread-1의 작업이 아직 완료되지 않았기 때문에 task1.result=0이 출력.
* main 스레드가 종료된 이후에 thread-1이 계산을 끝낸다. result=1275가 출력.



```java
package thread.control.test;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class JoinTest1Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new MyTask(), "t1");
        Thread t2 = new Thread(new MyTask(), "t2");
        Thread t3 = new Thread(new MyTask(), "t3");
        t1.start();
        t1.join();
        t2.start();
        t2.join();
        t3.start();
        t3.join();
        System.out.println("모든 스레드 실행 완료");
    }

    static class MyTask implements Runnable {
        @Override
        public void run() {
            for (int i = 1; i <= 3; i++) {
                log(i);
                sleep(1000);
            }
        }
    }
}
 
```

* 위 코드는 총 9초의 실행이 걸림. join()이 될때까지 3초를 기다림

```java
package thread.start;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class JoinTest2Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new MyTask(), "t1");
        Thread t2 = new Thread(new MyTask(), "t2");
        Thread t3 = new Thread(new MyTask(), "t3");
        t1.start();
        t2.start();
        t3.start();
        t1.join();
        t2.join();
        t3.join();
        System.out.println("모든 스레드 실행 완료");
    }

    static class MyTask implements Runnable {
        @Override
        public void run() {
            for (int i = 1; i <= 3; i++) {
                log(i);
                sleep(1000);
            }
        }
    }
}
 
```

* 위 코드는 효율적으로 스레드를 사용하고 있다.
* 총 걸리는 시간은 3초.



## 4. 스레드 제어와 생명 주기2

### 4.1. 인터럽트

```java
package thread.control.interrupt;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class ThreadStopMainV1 {
    public static void main(String[] args) {
        MyTask task = new MyTask();
        Thread thread = new Thread(task, "work");
        thread.start();
        sleep(4000);
        log("작업 중단 지시 runFlag=false");
        task.runFlag = false;
    }

    static class MyTask implements Runnable {
        volatile boolean runFlag = true; // 제어 변수

        @Override
        public void run() {
            while (runFlag) {
                log("작업 중");
                sleep(3000);
            }
            log("자원 정리");
            log("작업 종료");
        }
    }
}
```

```
// 실행 결과
14:58:27.520 [     work] 작업 중
14:58:30.525 [     work] 작업 중
14:58:31.510 [     main] 작업 중단 지시 runFlag=false
14:58:33.532 [     work] 자원 정리  <- 2초
14:58:33.533 [     work] 작업 종료
```

* 특정 스레드의 작업을 중단하는 가장 쉬운 방법은 변수를 사용. 하지만, 변수를 사용하더라도 스레드가 즉각 반응을 하지 않음.
* sleep(3000)으로 인해 3초동안 잠들어 있기때문에, 변수가 변경되어도 자원 정리까지 대기 시간 발생



이를 해결하기 위해 인터럽트를 사용할 수 있다. 인터럽트를 사용하면 WAITING, TIMED\_WAITING같은 대기 상태의 스레드를 직접 꺠워서, RUNNABLE 상태로 만들 수 있음.

```java
package thread.control.interrupt;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class ThreadStopMainV2 {
    public static void main(String[] args) {
        MyTask task = new MyTask();
        Thread thread = new Thread(task, "work");
        thread.start();
        sleep(4000);
        log("작업 중단 지시 thread.interrupt()");
        thread.interrupt();
        log("work 스레드 인터럽트 상태1 = " + thread.isInterrupted());
    }

    static class MyTask implements Runnable {
        @Override
        public void run() {
            try {
                while (true) {
                    log("작업 중");
                    Thread.sleep(3000);
                }
            } catch (InterruptedException e) {
                log("work 스레드 인터럽트 상태2 = " +
                        Thread.currentThread().isInterrupted());
                log("interrupt message=" + e.getMessage());
                log("state=" + Thread.currentThread().getState());
            }
            log("자원 정리");
            log("작업 종료");
        }
    }
}
```

* 특정 스레드의 인스턴스에 interrupt() 메서드가 호출되면, 해당 스레드에 인터럽트가 발생.
* 인터럽트가 발생하면 해당 스레드에 InterruptedException 발생. 이로 인해 RUNNABLE 상태로 전환.
* 다만, 즉각적으로 InterruptedException이 발생하지는 않는다.

```
// 실행 결과
18:10:40.024 [     work] 작업 중
18:10:43.026 [     work] 작업 중
18:10:44.011 [     main] 작업 중단 지시 thread.interrupt()
18:10:44.021 [     main] work 스레드 인터럽트 상태1 = true
18:10:44.021 [     work] work 스레드 인터럽트 상태2 = false
18:10:44.022 [     work] interrupt message=sleep interrupted
18:10:44.022 [     work] state=RUNNABLE
18:10:44.022 [     work] 자원 정리
18:10:44.023 [     work] 작업 종료
```

* Thread.interrupt를 통해 작업을 중단하면 거의 즉각적으로 인터럽트가 발생
* 이떄 work 스레드는 TIMED\_WAITING -> RUNNABLE 상태로 변경.
* 하지만, while (true) 부분을 체크하지 않기때문에 인터럽트가 발생해도 이 부분은 항상 true 이기 떄문에 다음 코드로 넘어감. 그리고 sleep()을 호출하고 나서야 인터럽트가 발생.



while( isInterrupted() )으로 변경하면 조금 더 빨리 반응할 수 있음. 하지만 이렇게 하게되면 work 상태가 계속 true로 유지되는게 문제.

이를 해결하기 위해, Thread.interrupted() 메서드를 사용.

* 스레드가 인터럽트 상태라면 true를 반환하고 해당 스레드의 인터럽트 상태를 false로 변경
* 스레드가 인터럽트 상태가 아니라면 false를 반환하고, 해당 스레드의 인터럽트 상태를 변경하지 않음.

```java
package thread.control.interrupt;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class ThreadStopMainV4 {
    public static void main(String[] args) {
        MyTask task = new MyTask();
        Thread thread = new Thread(task, "work");
        thread.start();
        sleep(100); //시간을 줄임
        log("작업 중단 지시 - thread.interrupt()");
        thread.interrupt();
        log("work 스레드 인터럽트 상태1 = " + thread.isInterrupted());
    }

    static class MyTask implements Runnable {
        @Override
        public void run() {
        
            // 중요
            while (!Thread.interrupted()) { //인터럽트 상태 변경O
                log("작업 중"); 
            }
            
            
            log("work 스레드 인터럽트 상태2 = " +
                    Thread.currentThread().isInterrupted());
            try {
                log("자원 정리 시도");
                Thread.sleep(1000);
                log("자원 정리 완료");
            } catch (InterruptedException e) {
                log("자원 정리 실패 - 자원 정리 중 인터럽트 발생");
                log("work 스레드 인터럽트 상태3 = " +
                        Thread.currentThread().isInterrupted());
            }
            log("작업 종료");
        }
    }
}
```

```
// 실행 결과
...
15:40:45:356 [     work] 작업 중
15:40:45:356 [     work] 작업 중
15:40:45:357 [     work] 작업 중
15:40:45:357 [     work] 작업 중
15:40:45:357 [     main] 작업 중단 지시 - thread.interrupt()
15:40:45:357 [     work] 작업 중
15:40:45:364 [     work] work 스레드 인터럽트 상태2 = false
15:40:45:364 [     main] work 스레드 인터럽트 상태1 = false
15:40:45:364 [     work] 자원 정리 시도
15:40:46:365 [     work] 자원 정리 완료
15:40:46:365 [     work] 작업 종료
```

* 결과적으로 while문을 탈출하는 시점에, 스레드의 인터럽트 상태도 false로 변경.
* 자바는 인터럽트 예외가 한번 발생하면, 스레드의 인터럽트 상태를 다시 정상(false)로 되돌림. **스레드의 인터럽트 상태를 정상으로 돌리지 않으면 이후에도 계속 인터럽트가 발생.**



### **4.2. Yield**

어떤 스레드를 얼마나 실행할지는 운영체제가 스케줄링을 통해 결정한다. 그런데 특정 스레드가 크게 바쁘지 않은 상황 이어서 다른 스레드에 CPU 실행 기회를 양보하고 싶을 수 있다. 이렇게 양보하면 스케줄링 큐에 대기 중인 다른 스레드 가 CPU 실행 기회를 더 빨리 얻을 수 있다

자바의 스레드가 RUNNABLE 상태일때, 운영체제의 스케줄링은 아래의 상태를 가질 수 있다:

* Running: CPU에서 실행중
* Ready: CPU에서 실행되길 기다리며 큐에 대기중



yield()는 현재 실행 중인 스레드가 자발적으로 CPU를 양보하여 다른 스레드가 실행될 수 있도록한다. 메서드를 호출한 스레드는 RUNNABLE 상태를 유지하면서 CPU를 양보한다. 즉, 이 스레드는 다시 스케줄링 큐에 들어가면서 다른 스레드에게 CPU 사용 기회를 넘긴다. 다만, CPU가 비어있다면 큐에서 다시 꺼내서 실행된다.

```java
package thread.control.printer;

import java.util.Queue;
import java.util.Scanner;
import java.util.concurrent.ConcurrentLinkedQueue;

import static util.MyLogger.log;

public class MyPrinterV4 {
    public static void main(String[] args) throws InterruptedException {
        Printer printer = new Printer();
        Thread printerThread = new Thread(printer, "printer");
        printerThread.start();
        Scanner userInput = new Scanner(System.in);
        while (true) {
            System.out.println("프린터할 문서를 입력하세요. 종료 (q): ");
            String input = userInput.nextLine();
            if (input.equals("q")) {
                printerThread.interrupt();
                break;
            }
            printer.addJob(input);
        }
    }

    static class Printer implements Runnable {
        Queue<String> jobQueue = new ConcurrentLinkedQueue<>();

        @Override
        public void run() {
            while (!Thread.interrupted()) {
                if (jobQueue.isEmpty()) {
                    Thread.yield(); //추가
                    continue;
                }
                try {
                    String job = jobQueue.poll();
                    log("출력 시작: " + job + ", 대기 문서: " + jobQueue);
                    Thread.sleep(3000); //출력에 걸리는 시간
                    log("출력 완료: " + job);
                } catch (InterruptedException e) {
                    log("인터럽트!");
                    break;
                }
            }
            log("프린터 종료");
        }

        public void addJob(String input) {
            jobQueue.offer(input);
        }
    }
}
```

```
// 입력과 실행 결과
프린터할 문서를 입력하세요. 종료 (q): 
a
프린터할 문서를 입력하세요. 종료 (q): 
15:46:30:670 [  printer] 출력 시작: a, 대기 문서: []
b
프린터할 문서를 입력하세요. 종료 (q): 
c
프린터할 문서를 입력하세요. 종료 (q): 
d
프린터할 문서를 입력하세요. 종료 (q): 
15:46:33:675 [  printer] 출력 완료: a
15:46:33:675 [  printer] 출력 시작: b, 대기 문서: [c, d]
15:46:36:690 [  printer] 출력 완료: b
15:46:36:690 [  printer] 출력 시작: c, 대기 문서: [d]
q
15:46:37:343 [  printer] 인터럽트!
15:46:37:343 [  printer] 프린터 종료

Process finished with exit code 0

```



