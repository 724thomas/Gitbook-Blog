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

