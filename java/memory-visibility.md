---
description: 메모리 가시성
---

# Memory Visibility

## 1. Volatile

```java
package thread.volatile1;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class VolatileFlagMain {
    public static void main(String[] args) {
        MyTask task = new MyTask();
        Thread t = new Thread(task, "work");
        log("runFalg = " + task.runFlag);
        t.start();

        sleep(1000);
        log("runFalg를 flase로 변경 시도");
        task.runFlag = false;
        log("runFalg = " + task.runFlag);
        log("main 종료");

    }

    static class MyTask implements Runnable {
        boolean runFlag = true;
        //volatile boolean runFalg = true;

        @Override
        public void run() {
            log("task 시작");
            while (runFlag) {
                // runFlag가 false로 변하면 탈출
            }
            log("task 종료");
        }
    }
}

```

* runFlag를 사용해서 스레드의 작업을 종료한다.
* work 스레드는 MyTask를 실행한다. 여기에는 runFlag를 체크하는 무한 루프가 있다.
* runFlag 값이 false가 되면 무한 루프를 탈출하며 작업을 종료한다.
* 이후에 main 스레드가 runFlag의 값을 false 로 변경한다.
* runFlag의 값이 false가 되었으므로 flag 스레드는 무한 루프를 탈출하며, 작업을 종료한다.

<figure><img src="/broken/files/Ajffc9o6OSPlcKQGSIKF" alt=""><figcaption></figcaption></figure>

* main, work 스레드 모두 MyTask 인스턴스(x001)에 있는 runFlag를 사용
* 이 값을 false로 변경하면 work 스레드의 작업을 종료.

```
15:39:59.830 [ main] runFlag = true
15:39:59.830 [ work] task 시작
15:40:00.837 [ main] runFlag를 false로 변경 시도
15:40:00.838 [ main] runFlag = false
15:40:00.838 [ work] task 종료
15:40:00.838 [ main] main 종료
```

```
// 실제실행 결과
10:31:32:485 [     main] runFlag = true
10:31:32:489 [     work] task 시작
10:31:33:496 [     main] runFlag 를 flase로 변경 시도
10:31:33:496 [     main] runFlag = true
10:31:33:496 [     main] main 종료
... 계속 실행중
```

실행 결과를 보면 기대와 다르게 동작한다.  조건이 변경되어도 while문을 빠져나오지 못함.



### 1.1. 메모리 가시성 문제

일반적으로 생각하는 메모리 접근 방식은 아래와 같다.

<figure><img src="/broken/files/CY304NNMzHPtqihActqK" alt=""><figcaption></figcaption></figure>

하지만 실제로는,

<figure><img src="/broken/files/20yCHb9BXvqSC3VqvK1G" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/KzQhAj761PGGQ2wL8aO6" alt=""><figcaption></figcaption></figure>

* 그림에서 보듯이, 스레드는 캐시를 가지고 있다. 그래서 변수가 변경되어도 바로 메인 메모리에 반영되지 않고 캐시메모리만 갱신된다.
* 메인 메모리와 work 스레드의 캐시 메모리의 runFlag는 여전히 true가 된다.

### 1.2. 메모리 가시성

위 상황 처럼, 한 스레드가 변경한 값이 다른 스레드에서 언제 보이는지에 대한 문제를 메모리 가시성이라고 한다. 즉,메모리에 보이냐, 보이지 않냐의 문제이다.

스레드의 캐시 메모리로 인해 성능이 향상 되지만, 정확한 데이터를 보기 어렵다. 이를 해결하기 위해서는 값을 읽거나 쓸때 모두 메인 메모리에 직접 접근 하면된다. 이때 volatile 키워드를 사용한다.

그러면 스레드 캐시 메모리와 메인 메모리가 동기화가 되는 시점은 언제일까?  정답은 알 수 없다.

CPU 설계 방식과 실행 환경에 따라 다름. 하지만 보통 컨텍스트 스위칭이 발생할때 캐시 메모리도 같이 갱신됨.

### 1.3. Happens-Before

happens-before 관계는 자바 메모리 모델에서 스레드 간의 작업 순서를 정의하는 개념. 만약 A작업이 B작업보다 happens-before 관계에 있으면, A작업에서의 모든 메모리 변경 사항은 B 작업에서 볼 수 있다. 즉, A작업에서 변경된 내용은 B작업이 시작되기 전에 모두 메모리에 반영된다.

* happens-before 관계는 한 동작이 다른 동작보다 먼저 발생함을 보장.
* happens-before 관계는 스레드 간의 메모리 가시성을 보장하는 규칙
* happens-before 관계가 성립하면, 한 스레드의 작업을 다른 스레드에서 볼 수 있게 된다.
* 한 스레드에서 수행한 작업을 다른 스레드가 참조할때 최신 상태가 보장되는 것.

```java
int a = 1;
int b = 2; //일때 a는 b보다 먼저 실행.
```

### 1.4. Java Memory Model(JMM)의 Happens-before 규칙

**1. Volatile 변수 규칙**

* **설명**: 특정 `volatile` 변수에 대한 **쓰기 작업**은 다른 스레드에서 해당 변수에 대해 **읽기 작업**을 수행하기 전에 반드시 메모리에 반영되며, 두 작업 사이에는 happens-before 관계가 형성됩니다.
* **의미**: volatile 변수에 값을 쓰면, 해당 변수의 변경 사항이 모든 스레드에서 가시적(visible)이어야 하며, 메모리 가시성 문제를 해결합니다.
*   **예제**:

    ```java
    java코드 복사class Example {
        private volatile boolean flag = false;

        public void writer() {
            flag = true; // volatile 변수 쓰기
        }

        public void reader() {
            if (flag) { 
                // reader()는 writer()의 쓰기 작업 이후 값을 읽음.
                System.out.println("Flag is true!");
            }
        }
    }
    ```

***

**2. Thread 시작 규칙**

* **설명**: 한 스레드에서 `Thread.start()`를 호출하면, 해당 스레드 내에서 실행되는 모든 작업은 `start()` 호출 이전의 작업보다 happens-before 관계를 형성합니다.
* **의미**: 새로 생성된 스레드는 `start()` 이전에 완료된 작업의 결과를 반드시 볼 수 있습니다.
*   **예제**:

    ```java
    class Example {
        private int value = 0;

        public void mainThread() {
            value = 42; // start() 이전의 작업
            Thread t = new Thread(() -> {
                // 새 스레드는 value 값을 볼 수 있음.
                System.out.println(value); // 출력: 42
            });
            t.start();
        }
    }
    ```

***

**3. Thread 인터럽트 규칙**

* **설명**: 한 스레드에서 `Thread.interrupt()`를 호출하는 작업은, 인터럽트된 스레드가 인터럽트를 감지하는 시점의 작업보다 happens-before 관계를 형성합니다.
* **의미**: `interrupt()` 호출 후, 해당 스레드가 인터럽트 상태를 확인하거나 `InterruptedException`을 처리하는 작업이 반드시 그 뒤에 실행됩니다.
*   **예제**:

    ```java
    class Example {
        public static void main(String[] args) {
            Thread t = new Thread(() -> {
                while (!Thread.currentThread().isInterrupted()) {
                    System.out.println("Running...");
                }
            });

            t.start();
            t.interrupt(); // interrupt() 호출
            // interrupt() 이후에 스레드가 인터럽트를 감지
        }
    }
    ```

***

**4. 객체 생성 규칙**

* **설명**: 객체의 생성자는 객체가 완전히 생성된 후에만 다른 스레드에 의해 참조될 수 있도록 보장합니다.
* **의미**: 객체의 생성자에서 초기화된 필드는 생성자가 완료된 이후 다른 스레드에서 참조될 때 happens-before 관계를 형성합니다.
*   **예제**:

    ```java
    class Example {
        private final int value;

        public Example(int value) {
            this.value = value; // 생성자 초기화
        }

        public int getValue() {
            return value; // 생성자가 완료된 이후에 참조 가능
        }
    }
    ```

***

**5. 모니터 락 규칙**

* **설명**: 한 스레드에서 `synchronized` 블록을 종료한 후, 그 모니터 락을 얻는 모든 스레드는 해당 블록 내의 모든 작업을 볼 수 있습니다.
* **의미**: 예를 들어, `synchronized(lock) { ... }` 블록 내에서의 작업은 블록을 나가는 시점에 happens-before 관계를 형성합니다.
  * **ReentrantLock**과 같은 다른 락 구현에서도 동일하게 적용됩니다.
*   **예제**:

    ```java
    class Example {
        private int value;

        public synchronized void writer() {
            value = 42; // synchronized 블록 내 작업
        }

        public synchronized void reader() {
            System.out.println(value); // writer() 작업 이후 실행됨
        }
    }
    ```

***

**6. 전이 규칙 (Transitivity Rule)**

* **설명**: 만약 A가 B보다 happens-before 관계에 있고, B가 C보다 happens-before 관계에 있다면, A는 C보다 happens-before 관계에 있습니다.
* **의미**: 이는 happens-before 관계의 \*\*전이성(transitivity)\*\*을 설명하며, 모든 규칙의 근간이 됩니다.
