---
description: 생산자와 소비자
---

# Producer & Consumer

## 1. 소개

생산자와 소비자 문제는 멀티스레드 프로그래밍에서 자주 등장하는 동시성 문제중 하나이다. 여러 스레드가 동시에 데이터를 생산하고 소비하는 상황.

프린터 예제:

<figure><img src="../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

* 생산자: 데이터를 생산하는 역할. 파일에서 데이터를 읽어오거나 네트워크에서 데이터를 받아오는 스레드. 위 예제에서는 사용자의 입력을 프린터 큐에 전달하는 스레드가 생산자.
* 소비자: 생성된 데이터를 사용하는 역할. 데이터를 처리하거나 저장하는 스레드가 소비자. 위 예제에서는 큐에 전달된 데이터를 받아서 출력하는 스레드가 소비자.
* 버퍼: 생산자가 생성한 데이터를 일시적으로 저장하는 공간. 버퍼는 한정된 크기를 가지고, 생산자와 소비자가 이 버퍼를 통해 데이터를 주고 받음. 위 예제에서는 프린터 큐가 버퍼.

문제 상황 - 생산자 소비자 문제(producer-consumer problem) / 한정된 버퍼 문제(bounded-buffer prob)

* 생산자가 너무 빠를때: 버퍼가 가득 차서 더 이상 데이터를 넣을 수 없을때까지 생산자가 데이터를 생성. 버퍼가 가득 찬 경우 생산자는 버퍼에 빈 공간이 생길 때까지 기다려야 한다.
* 소비자가 너무 빠를때: 버퍼가 비어서 더 이상 소비할 데이터가 없을 때까지 소비자가 데이터를 처리. 버퍼가 비어있을 때 소비자는 버퍼에 새로운 데이터라 들어올 때까지 기다려야함.

### 1.1. 예제 코드

```java
package thread.bounded;

public interface BoundedQueue {
    void put(String data);
    String take();
}
```

* BoundedQueue: 버퍼 역할을 하는 큐의 인터페이스
* put(data): 버퍼에 데이터를 보관 (생산자 스레드가 호출하고 데이터 생산)
* take(): 버퍼에 보관된 데이터를 가져간다 (소비자 스레드가 호출하고, 데이터 소비)

```java
package thread.bounded;

import java.util.ArrayDeque;
import java.util.Queue;

import static util.MyLogger.log;

public class BoundedQueueV1 implements BoundedQueue{
    private final Queue<String> queue = new ArrayDeque<>();
    private final int max;

    public BoundedQueueV1(int max) {
        this.max = max;
    }

    @Override
    public void put(String data) {
        if (queue.size() == max) {
            log("[put] 큐가 가득 참, 버림: " + data);
            return;
        }
        queue.offer(data);
    }

    @Override
    public String take() {
        if (queue.isEmpty()) {
            log("[take] 큐가 비어 있음");
            return null;
        }
        return null;
    }

    @Override
    public String toString() { // Synchronized가 원래는 필요하다.
        return queue.toString();
    }
}

```

임계 영역:

Queue(ArrayDeque)가 임계영역이다. 여러 스레드가 접근하기 때문에 synchronized를 사용해야한다.

```java
package thread.bounded;

import static util.MyLogger.log;

public class ProducerTask implements Runnable{
    private BoundedQueue queue;
    private String request;
    
    public ProducerTask(BoundedQueue queue, String request) {
        this.queue = queue;
        this.request = request;
    }
    
    @Override
    public void run() {
        log("[생산 시도] " + request + " -> " + queue);
        queue.put(request);
        log("[생산 완료] " + request + " -> " + queue);
    }
}
```

* ProducerTask: 데이터를 생선하는 스레드가 실행하는 클래스.

```java
package thread.bounded;

import static util.MyLogger.log;

public class ConsumerTask implements Runnable {
    private BoundedQueue queue;

    public ConsumerTask(BoundedQueue queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        log("[소비 시도]     ? <- " + queue);
        String data = queue.take();
        log("[소비 완료] " + data + " <- " + queue);
    }
}
```

* ConsumerTask: 데이터를 소비하는 소비자 스레드가 실행하는 클래스, Runnable을 구현

```java
package thread.bounded;

import java.util.ArrayList;
import java.util.List;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BoundedMain {
    public static void main(String[] args) {
        // 1. BoundedQueue 선택
        BoundedQueue queue = new BoundedQueueV1(2);

        // 2. 생산자, 소비자 실행 순서 선택, 반드시 하나만 선택!
        producerFirst(queue);
        // consumerFirst(queue);
    }

    private static void producerFirst(BoundedQueue queue) {
        log("==[생산자 먼저 실행] 시작, " + queue.getClass().getSimpleName() + "==");
        List<Thread> threads = new ArrayList<>();
        startProducer(queue, threads);
        printAllState(queue, threads);
        startConsumer(queue, threads);
        printAllState(queue, threads);
        log("==[생산자 먼저 실행] 종료, " + queue.getClass().getSimpleName() + "==");
    }

    private static void consumerFirst(BoundedQueue queue) {
        log("==[소비자 먼저 실행] 시작, " + queue.getClass().getSimpleName() + "==");
        List<Thread> threads = new ArrayList<>();
        startConsumer(queue, threads);
        printAllState(queue, threads);
        startProducer(queue, threads);
        printAllState(queue, threads);
        log("==[소비자 먼저 실행] 종료, " + queue.getClass().getSimpleName() + "==");
    }

    private static void startProducer(BoundedQueue queue, List<Thread> threads) {
        System.out.println();
        log("생산자 시작");
        for (int i = 1; i <= 3; i++) {
            Thread producer = new Thread(new ProducerTask(queue, "data" + i));
            threads.add(producer);
            producer.start();
            sleep(100);
        }
    }

    private static void startConsumer(BoundedQueue queue, List<Thread> threads) {
        System.out.println();
        log("소비자 시작");
        for (int i = 1; i <= 3; i++) {
            Thread consumer = new Thread(new ConsumerTask(queue));
            threads.add(consumer);
            consumer.start();
            sleep(100);
        }
    }

    private static void printAllState(BoundedQueue queue, List<Thread> threads) {
        System.out.println();
        log("현재 상태 출력, 큐 데이터: " + queue);
        for (Thread thread : threads) {
            log(thread.getName() + ": " + thread.getState());
        }
    }
}
```

* startProducer: 생산자 스레드를 3개 만들어서 실행. producer1->producer2->producer3 실행
* startConsumer: 소비자 스레드를 3개 만들어서 실행. consumer1->consumer2->consumer3 실행



### 1.2. producerFirst()

```
10:57:11:810 [     main] ==[생산자 먼저 실행] 시작, BoundedQueueV1==

10:57:11:813 [     main] 생산자 시작
10:57:11:836 [ Thread-0] [생산 시도] data1 -> []
10:57:11:836 [ Thread-0] [생산 완료] data1 -> [data1]
10:57:11:938 [ Thread-1] [생산 시도] data2 -> [data1]
10:57:11:938 [ Thread-1] [생산 완료] data2 -> [data1, data2]
10:57:12:043 [ Thread-2] [생산 시도] data3 -> [data1, data2]
10:57:12:044 [ Thread-2] [put] 큐가 가득 참, 버림: data3
10:57:12:044 [ Thread-2] [생산 완료] data3 -> [data1, data2]

10:57:12:148 [     main] 현재 상태 출력, 큐 데이터: [data1, data2]
10:57:12:149 [     main] Thread-0: TERMINATED
10:57:12:149 [     main] Thread-1: TERMINATED
10:57:12:149 [     main] Thread-2: TERMINATED

10:57:12:149 [     main] 소비자 시작
10:57:12:153 [ Thread-3] [소비 시도]     ? <- [data1, data2]
10:57:12:153 [ Thread-3] [소비 완료] null <- [data1, data2]
10:57:12:253 [ Thread-4] [소비 시도]     ? <- [data1, data2]
10:57:12:254 [ Thread-4] [소비 완료] null <- [data1, data2]
10:57:12:358 [ Thread-5] [소비 시도]     ? <- [data1, data2]
10:57:12:358 [ Thread-5] [소비 완료] null <- [data1, data2]

10:57:12:463 [     main] 현재 상태 출력, 큐 데이터: [data1, data2]
10:57:12:463 [     main] Thread-0: TERMINATED
10:57:12:464 [     main] Thread-1: TERMINATED
10:57:12:464 [     main] Thread-2: TERMINATED
10:57:12:464 [     main] Thread-3: TERMINATED
10:57:12:465 [     main] Thread-4: TERMINATED
10:57:12:465 [     main] Thread-5: TERMINATED
10:57:12:466 [     main] ==[생산자 먼저 실행] 종료, BoundedQueueV1==

Process finished with exit code 0
```

<figure><img src="../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

세번째 생산자 스레드가 버퍼에 데이터를 저장하려고 하지만 꽉차서 데이터를 버리게 됨. 데이터를 버리지 않기 위해서는 큐에 빈 공간이 생길때까지 기다리는 방법이 있다.

<figure><img src="../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

소비자 입장에서 큐에 데이터가 없으니 null을 반환. 하지만 기다리는 것도 대안.

<figure><img src="../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

결과적으로 p3스레드가 데이터를 버리게 되므로 c3가 조회하는 시점에 버퍼는 비어있어서 데이터를 받지 못하고 null값을 받는다.

### 1.3. consumerFirst() 실행

```
11:00:13:248 [     main] ==[소비자 먼저 실행] 시작, BoundedQueueV1==

11:00:13:252 [     main] 소비자 시작
11:00:13:255 [ Thread-0] [소비 시도]     ? <- []
11:00:13:255 [ Thread-0] [take] 큐가 비어 있음
11:00:13:263 [ Thread-0] [소비 완료] null <- []
11:00:13:362 [ Thread-1] [소비 시도]     ? <- []
11:00:13:363 [ Thread-1] [take] 큐가 비어 있음
11:00:13:363 [ Thread-1] [소비 완료] null <- []
11:00:13:467 [ Thread-2] [소비 시도]     ? <- []
11:00:13:468 [ Thread-2] [take] 큐가 비어 있음
11:00:13:468 [ Thread-2] [소비 완료] null <- []

11:00:13:572 [     main] 현재 상태 출력, 큐 데이터: []
11:00:13:573 [     main] Thread-0: TERMINATED
11:00:13:573 [     main] Thread-1: TERMINATED
11:00:13:574 [     main] Thread-2: TERMINATED

11:00:13:574 [     main] 생산자 시작
11:00:13:581 [ Thread-3] [생산 시도] data1 -> []
11:00:13:581 [ Thread-3] [생산 완료] data1 -> [data1]
11:00:13:692 [ Thread-4] [생산 시도] data2 -> [data1]
11:00:13:693 [ Thread-4] [생산 완료] data2 -> [data1, data2]
11:00:13:797 [ Thread-5] [생산 시도] data3 -> [data1, data2]
11:00:13:798 [ Thread-5] [put] 큐가 가득 참, 버림: data3
11:00:13:798 [ Thread-5] [생산 완료] data3 -> [data1, data2]

11:00:13:902 [     main] 현재 상태 출력, 큐 데이터: [data1, data2]
11:00:13:902 [     main] Thread-0: TERMINATED
11:00:13:903 [     main] Thread-1: TERMINATED
11:00:13:903 [     main] Thread-2: TERMINATED
11:00:13:903 [     main] Thread-3: TERMINATED
11:00:13:903 [     main] Thread-4: TERMINATED
11:00:13:904 [     main] Thread-5: TERMINATED
11:00:13:904 [     main] ==[소비자 먼저 실행] 종료, BoundedQueueV1==

Process finished with exit code 0
```

<figure><img src="../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

consumer가 먼저 실행이 되었을때, 소비자 스레드들은 모두 null값을 반환하게 되고, 큐에는 생산자 스레드 값이 2개만 들어간다.



위 예제에서는 스레드가 3개로만 했지만, 계속해서 생산되고 소비되는 환경에서는 문제가 발생할 수 있다.

* 버퍼가 가득 찬 경우: 생산자 입장에서 버퍼에 여유가 생실 때 까지 기다린다.
* 버퍼가 빈 경우: 소비자 입장에서 버퍼에 데이터가 채워질 때 까지 기다리면 null값을 얻지 않을 것 같다.



**하지만 단순히 WAIT을 하게 되면 어떨까?**

생산자 우선:

<figure><img src="../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

소비자 우선:

<figure><img src="../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

위 그림과 같이 임계영역 안에서 **락을 가지고 있는 TIMED\_WAITING 상태가 되어서, 다른 스레드가 접근하지 못해서 프로그램이 멈추게 됨.**



## 2. Object - wait, notify

Synchronized를 사용했을때 문제점은 (락을 가지고 무한 대기) Object 클래스로 해결이 가능하다. Wait(), notify() 메서드를 사용하며, Object는 모든 자바 객체의 부모이기 때문에 사용 가능하다.

* Object.wait():
  * 현재 스레드가 가진 락을 반납하고 WAITING 한다.
  * 현재 스레드를 WAITING 상태로 전환. 현재 스레드가 synchronized 블록이나 메서드에서 락을 소유하고 있을 때만 호출 가능.&#x20;
  * 호출한 스레드는 락을 반납하고, 다른 스레드가 해당 락을 획득할 수 있게 한다. 이때 WAITING 상태로 전환된 스레드는 다른 스레드가 notify() 또는 notifyAll()을 호출할때 까지 WAITING 상태를 유지.
* Object.notify():
  * 대기 중인 스레드 중 하나를 꺠운다 (랜덤)
  * 이 메서드는 synchronized 블록 또는 메서드에서 호출되어야한다.
  * 깨운 스레드는 락을 다시 획득할 기회를 얻게 된다.
* Object.notifyAll():
  * 대기 중인 모든 스레드를 꺠운다
  * Synchronized 블록이나 메서드에서 호출되어야한다.
  * 모든 대기 중인 스레드가 락을 획득할 수 있는 기회를 얻게 된다.

### 2.1. 예제 코드 1

```java
package thread.bounded;

import java.util.ArrayDeque;
import java.util.Queue;

import static util.MyLogger.log;

public class BoundedQueueV3 implements BoundedQueue {
    private final Queue<String> queue = new ArrayDeque<>();
    private final int max;

    public BoundedQueueV3(int max) {
        this.max = max;
    }

    public synchronized void put(String data) {
        while (queue.size() == max) {
            log("[put] 큐가 가득 참, 생산자 대기");
            try {
                wait(); // RUNNABLE -> WAITING, 락 반납
                log("[put] 생산자 깨어남");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        queue.offer(data);
        log("[put] 생산자 데이터 저장, notify() 호출");
        notify(); // 대기 스레드, WAIT -> BLOCKED
        //notifyAll(); // 모든 대기 스레드, WAIT -> BLOCKED
    }

    public synchronized String take() {
        while (queue.isEmpty()) {
            log("[take] 큐에 데이터가 없음, 소비자 대기");
            try {
                wait();
                log("[take] 소비자 깨어남");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        String data = queue.poll();
        log("[take] 소비자 데이터 획득, notify() 호출");
        notify(); // 대기 스레드, WAIT -> BLOCKED
        //notifyAll(); // 모든 대기 스레드, WAIT -> BLOCKED
        return data;
    }

    @Override
    public String toString() {
        return queue.toString();
    }
}
```

생산자 먼저 실행:

```
18:50:36:344 [     main] ==[생산자 먼저 실행] 시작, BoundedQueueV3==

18:50:36:348 [     main] 생산자 시작
18:50:36:361 [ Thread-0] [생산 시도] data1 -> []
18:50:36:362 [ Thread-0] [put] 생산자 데이터 저장, notify() 호출
18:50:36:362 [ Thread-0] [생산 완료] data1 -> [data1]
18:50:36:468 [ Thread-1] [생산 시도] data2 -> [data1]
18:50:36:468 [ Thread-1] [put] 생산자 데이터 저장, notify() 호출
18:50:36:469 [ Thread-1] [생산 완료] data2 -> [data1, data2]
18:50:36:573 [ Thread-2] [생산 시도] data3 -> [data1, data2]
18:50:36:573 [ Thread-2] [put] 큐가 가득 참, 생산자 대기

18:50:36:678 [     main] 현재 상태 출력, 큐 데이터: [data1, data2]
18:50:36:679 [     main] Thread-0: TERMINATED
18:50:36:679 [     main] Thread-1: TERMINATED
18:50:36:679 [     main] Thread-2: WAITING

18:50:36:680 [     main] 소비자 시작
18:50:36:682 [ Thread-3] [소비 시도]     ? <- [data1, data2]
18:50:36:682 [ Thread-3] [take] 소비자 데이터 획득, notify() 호출
18:50:36:682 [ Thread-2] [put] 생산자 깨어남
18:50:36:683 [ Thread-2] [put] 생산자 데이터 저장, notify() 호출
18:50:36:683 [ Thread-3] [소비 완료] data1 <- [data2]
18:50:36:683 [ Thread-2] [생산 완료] data3 -> [data2, data3]
18:50:36:783 [ Thread-4] [소비 시도]     ? <- [data2, data3]
18:50:36:784 [ Thread-4] [take] 소비자 데이터 획득, notify() 호출
18:50:36:784 [ Thread-4] [소비 완료] data2 <- [data3]
18:50:36:888 [ Thread-5] [소비 시도]     ? <- [data3]
18:50:36:889 [ Thread-5] [take] 소비자 데이터 획득, notify() 호출
18:50:36:889 [ Thread-5] [소비 완료] data3 <- []

18:50:36:993 [     main] 현재 상태 출력, 큐 데이터: []
18:50:36:993 [     main] Thread-0: TERMINATED
18:50:36:994 [     main] Thread-1: TERMINATED
18:50:36:994 [     main] Thread-2: TERMINATED
18:50:36:994 [     main] Thread-3: TERMINATED
18:50:36:995 [     main] Thread-4: TERMINATED
18:50:36:995 [     main] Thread-5: TERMINATED
18:50:36:995 [     main] ==[생산자 먼저 실행] 종료, BoundedQueueV3==

Process finished with exit code 0

```

<figure><img src="../.gitbook/assets/image (300).png" alt=""><figcaption></figcaption></figure>

* p3는 자리가 없으므로 wait()을 호출하고 스레드 대기 집합에 들어간다.
* c1이 실행되면서 데이터를 큐에서 빼고, 스레드 대기 집합에 있는 스레드를 하나 꺠운다.
* 해당 상황에서 결과적으로 봤을때는 잘 작동한다.

소비자 먼저 실행:

```
18:53:03:732 [     main] ==[소비자 먼저 실행] 시작, BoundedQueueV3==

18:53:03:737 [     main] 소비자 시작
18:53:03:740 [ Thread-0] [소비 시도]     ? <- []
18:53:03:740 [ Thread-0] [take] 큐에 데이터가 없음, 소비자 대기
18:53:03:855 [ Thread-1] [소비 시도]     ? <- []
18:53:03:856 [ Thread-1] [take] 큐에 데이터가 없음, 소비자 대기
18:53:03:960 [ Thread-2] [소비 시도]     ? <- []
18:53:03:961 [ Thread-2] [take] 큐에 데이터가 없음, 소비자 대기

18:53:04:065 [     main] 현재 상태 출력, 큐 데이터: []
18:53:04:072 [     main] Thread-0: WAITING
18:53:04:072 [     main] Thread-1: WAITING
18:53:04:073 [     main] Thread-2: WAITING

18:53:04:073 [     main] 생산자 시작
18:53:04:079 [ Thread-3] [생산 시도] data1 -> []
18:53:04:079 [ Thread-3] [put] 생산자 데이터 저장, notify() 호출
18:53:04:079 [ Thread-0] [take] 소비자 깨어남
18:53:04:080 [ Thread-0] [take] 소비자 데이터 획득, notify() 호출
18:53:04:080 [ Thread-1] [take] 소비자 깨어남
18:53:04:080 [ Thread-3] [생산 완료] data1 -> [data1]
18:53:04:080 [ Thread-0] [소비 완료] data1 <- []
18:53:04:080 [ Thread-1] [take] 큐에 데이터가 없음, 소비자 대기
18:53:04:185 [ Thread-4] [생산 시도] data2 -> []
18:53:04:186 [ Thread-4] [put] 생산자 데이터 저장, notify() 호출
18:53:04:186 [ Thread-2] [take] 소비자 깨어남
18:53:04:186 [ Thread-4] [생산 완료] data2 -> [data2]
18:53:04:186 [ Thread-2] [take] 소비자 데이터 획득, notify() 호출
18:53:04:187 [ Thread-1] [take] 소비자 깨어남
18:53:04:187 [ Thread-2] [소비 완료] data2 <- []
18:53:04:187 [ Thread-1] [take] 큐에 데이터가 없음, 소비자 대기
18:53:04:290 [ Thread-5] [생산 시도] data3 -> []
18:53:04:291 [ Thread-5] [put] 생산자 데이터 저장, notify() 호출
18:53:04:291 [ Thread-1] [take] 소비자 깨어남
18:53:04:291 [ Thread-5] [생산 완료] data3 -> [data3]
18:53:04:291 [ Thread-1] [take] 소비자 데이터 획득, notify() 호출
18:53:04:292 [ Thread-1] [소비 완료] data3 <- []

18:53:04:395 [     main] 현재 상태 출력, 큐 데이터: []
18:53:04:395 [     main] Thread-0: TERMINATED
18:53:04:396 [     main] Thread-1: TERMINATED
18:53:04:396 [     main] Thread-2: TERMINATED
18:53:04:396 [     main] Thread-3: TERMINATED
18:53:04:396 [     main] Thread-4: TERMINATED
18:53:04:397 [     main] Thread-5: TERMINATED
18:53:04:397 [     main] ==[소비자 먼저 실행] 종료, BoundedQueueV3==

Process finished with exit code 0

```

<figure><img src="../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

* 소비자 스레드들이 모두 실행되지만 큐에 데이터가 없으므로 모두 대기 집합에서 WAITING 상태.
* p1이 데이터를 큐에 넣고 대기 집합에서 스레드를 하나 깨운다.
* c1이 꺠어나고 데이터를 뺀 다음 대기 집합에서 스레드를 하나 깨운다.
* 이때, c2, c3 스레드들이 꺠어나지만 데이터가 없어서 다시 들어가게된다. 이부분이 매우 비효율적이다.
* c2, c3가 다시 WAITING 상태로 들어가게되면 p2가 데이터를 저장한다.
* 결과적으로는 잘 돌아간다.

### 2.2. wait, notify의 한계

위 예시에서는 wait()을 통해 스레드를 WAITING 상태로 대기 집합에 넣을 수 있었다.  몇가지 단점이 있다.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* notify() / notifyAll() 을 통해서 스레드를 깨우게 되는데, 랜덤으로 스레드를 깨우게 된다.&#x20;
* 이때, 상황에 따라 Thread Starvation이 발생할 수 있다. 왜냐하면 랜덤으로 스레드가 깨어나서 락을 획득하기 떄문에, 락을 획득 못하는 스레드가 발생할 수 있다.



## 3. Lock & ReentrantLock

### 3.1. Condition

Lock 인터페이스와 Reentrant 락을 사용하게 되면 대기 집합을 분리할 수 있다.

```java
package thread.bounded;

import java.util.ArrayDeque;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

import static util.MyLogger.log;

public class BoundedQueueV4 implements BoundedQueue {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private final Queue<String> queue = new ArrayDeque<>();
    private final int max;

    public BoundedQueueV4(int max) {
        this.max = max;
    }

    public void put(String data) {
        lock.lock(); // 수정부분
        try {
            while (queue.size() == max) {
                log("[put] 큐가 가득 참, 생산자 대기");
                try {
                    condition.await(); // 수정부분
                    log("[put] 생산자 깨어남");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            queue.offer(data);
            log("[put] 생산자 데이터 저장, notify() 호출");
            condition.signal(); // 수정부분
        } finally {
            lock.unlock(); // 수정부분
        }
    }

    public String take() {
        lock.lock(); // 수정부분
        try {
            while (queue.isEmpty()) {
                log("[take] 큐에 데이터가 없음, 소비자 대기");
                try {
                    condition.await(); // 수정부분
                    log("[take] 소비자 깨어남");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            String data = queue.poll();
            log("[take] 소비자 데이터 획득, notify() 호출");
            condition.signal(); // 수정부분
            return data;
        } finally {
            lock.unlock(); // 수정부분
        }
    }

    @Override
    public String toString() {
        return queue.toString();
    }
}
```

* Condition (Condition condition = lock.newCondition() )
  * Condition은 ReentrantLock을 사용하는 스레드가 대기하는 스레드 공간이다.
  * lock.newCondition() 메서드를 호출하면 스레드 대기 공간이 만들어진다. Lock(ReentrantLock)의 스레드 대기 공간은 이렇게 만들 수 있다.
  * Object.wait() 에서 사용한 스레드 대기 공간은 모든 객체 인스턴스가 내부에 기본으로 가지고 있는 반면, Lock(ReentrantLock)을 사용하는 경우 스레드 대기 공간을 집접 만들어서 사용해야함.
* condition.await()
  * Object.wait()과 유사.
  * 지정한 condition에 현재 스레드를 WAITING 상태로 보관. 이때 ReentrantLock에서 획득한 락을 반납.
* condition.signal()
  * Object.notify()와 유사한 기능. 지정한 condition에서 대기중인 스레드를 하나 꺠우고, 해당 스레드를 condition에서 빼온다.



생산자 먼저:

```
20:26:25:191 [     main] ==[생산자 먼저 실행] 시작, BoundedQueueV4==

20:26:25:195 [     main] 생산자 시작
20:26:25:209 [ Thread-0] [생산 시도] data1 -> []
20:26:25:209 [ Thread-0] [put] 생산자 데이터 저장, notify() 호출
20:26:25:210 [ Thread-0] [생산 완료] data1 -> [data1]
20:26:25:311 [ Thread-1] [생산 시도] data2 -> [data1]
20:26:25:311 [ Thread-1] [put] 생산자 데이터 저장, notify() 호출
20:26:25:312 [ Thread-1] [생산 완료] data2 -> [data1, data2]
20:26:25:416 [ Thread-2] [생산 시도] data3 -> [data1, data2]
20:26:25:417 [ Thread-2] [put] 큐가 가득 참, 생산자 대기

20:26:25:521 [     main] 현재 상태 출력, 큐 데이터: [data1, data2]
20:26:25:522 [     main] Thread-0: TERMINATED
20:26:25:522 [     main] Thread-1: TERMINATED
20:26:25:523 [     main] Thread-2: WAITING

20:26:25:523 [     main] 소비자 시작
20:26:25:526 [ Thread-3] [소비 시도]     ? <- [data1, data2]
20:26:25:526 [ Thread-3] [take] 소비자 데이터 획득, notify() 호출
20:26:25:527 [ Thread-2] [put] 생산자 깨어남
20:26:25:527 [ Thread-3] [소비 완료] data1 <- [data2]
20:26:25:527 [ Thread-2] [put] 생산자 데이터 저장, notify() 호출
20:26:25:528 [ Thread-2] [생산 완료] data3 -> [data2, data3]
20:26:25:626 [ Thread-4] [소비 시도]     ? <- [data2, data3]
20:26:25:627 [ Thread-4] [take] 소비자 데이터 획득, notify() 호출
20:26:25:627 [ Thread-4] [소비 완료] data2 <- [data3]
20:26:25:731 [ Thread-5] [소비 시도]     ? <- [data3]
20:26:25:732 [ Thread-5] [take] 소비자 데이터 획득, notify() 호출
20:26:25:732 [ Thread-5] [소비 완료] data3 <- []

20:26:25:837 [     main] 현재 상태 출력, 큐 데이터: []
20:26:25:837 [     main] Thread-0: TERMINATED
20:26:25:838 [     main] Thread-1: TERMINATED
20:26:25:838 [     main] Thread-2: TERMINATED
20:26:25:838 [     main] Thread-3: TERMINATED
20:26:25:839 [     main] Thread-4: TERMINATED
20:26:25:839 [     main] Thread-5: TERMINATED
20:26:25:840 [     main] ==[생산자 먼저 실행] 종료, BoundedQueueV4==

Process finished with exit code 0
```



소비자 먼저:

```
20:27:18:867 [     main] ==[소비자 먼저 실행] 시작, BoundedQueueV4==

20:27:18:870 [     main] 소비자 시작
20:27:18:873 [ Thread-0] [소비 시도]     ? <- []
20:27:18:873 [ Thread-0] [take] 큐에 데이터가 없음, 소비자 대기
20:27:18:978 [ Thread-1] [소비 시도]     ? <- []
20:27:18:978 [ Thread-1] [take] 큐에 데이터가 없음, 소비자 대기
20:27:19:083 [ Thread-2] [소비 시도]     ? <- []
20:27:19:083 [ Thread-2] [take] 큐에 데이터가 없음, 소비자 대기

20:27:19:188 [     main] 현재 상태 출력, 큐 데이터: []
20:27:19:195 [     main] Thread-0: WAITING
20:27:19:196 [     main] Thread-1: WAITING
20:27:19:196 [     main] Thread-2: WAITING

20:27:19:197 [     main] 생산자 시작
20:27:19:203 [ Thread-3] [생산 시도] data1 -> []
20:27:19:203 [ Thread-3] [put] 생산자 데이터 저장, notify() 호출
20:27:19:204 [ Thread-0] [take] 소비자 깨어남
20:27:19:204 [ Thread-3] [생산 완료] data1 -> [data1]
20:27:19:204 [ Thread-0] [take] 소비자 데이터 획득, notify() 호출
20:27:19:205 [ Thread-1] [take] 소비자 깨어남
20:27:19:205 [ Thread-0] [소비 완료] data1 <- []
20:27:19:205 [ Thread-1] [take] 큐에 데이터가 없음, 소비자 대기
20:27:19:308 [ Thread-4] [생산 시도] data2 -> []
20:27:19:309 [ Thread-4] [put] 생산자 데이터 저장, notify() 호출
20:27:19:309 [ Thread-2] [take] 소비자 깨어남
20:27:19:309 [ Thread-4] [생산 완료] data2 -> [data2]
20:27:19:309 [ Thread-2] [take] 소비자 데이터 획득, notify() 호출
20:27:19:310 [ Thread-1] [take] 소비자 깨어남
20:27:19:310 [ Thread-2] [소비 완료] data2 <- []
20:27:19:310 [ Thread-1] [take] 큐에 데이터가 없음, 소비자 대기
20:27:19:413 [ Thread-5] [생산 시도] data3 -> []
20:27:19:414 [ Thread-5] [put] 생산자 데이터 저장, notify() 호출
20:27:19:414 [ Thread-1] [take] 소비자 깨어남
20:27:19:414 [ Thread-5] [생산 완료] data3 -> [data3]
20:27:19:414 [ Thread-1] [take] 소비자 데이터 획득, notify() 호출
20:27:19:415 [ Thread-1] [소비 완료] data3 <- []

20:27:19:518 [     main] 현재 상태 출력, 큐 데이터: []
20:27:19:518 [     main] Thread-0: TERMINATED
20:27:19:519 [     main] Thread-1: TERMINATED
20:27:19:519 [     main] Thread-2: TERMINATED
20:27:19:519 [     main] Thread-3: TERMINATED
20:27:19:519 [     main] Thread-4: TERMINATED
20:27:19:520 [     main] Thread-5: TERMINATED
20:27:19:520 [     main] ==[소비자 먼저 실행] 종료, BoundedQueueV4==
```



### 3.2. Condition 활용

Condition을 여러개를 만들어서 대기 공간을 분리할 수 있다.

```java
package thread.bounded;

import java.util.ArrayDeque;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

import static util.MyLogger.log;

public class BoundedQueueV5 implements BoundedQueue {
    private final Lock lock = new ReentrantLock();
    private final Condition producerCondition = lock.newCondition(); // 수정부분
    private final Condition consumerCondition = lock.newCondition(); // 수정부분
    private final Queue<String> queue = new ArrayDeque<>();
    private final int max;

    public BoundedQueueV5(int max) {
        this.max = max;
    }

    public void put(String data) {
        lock.lock();
        try {
            while (queue.size() == max) {
                log("[put] 큐가 가득 참, 생산자 대기");
                try {
                    producerCondition.await(); // 수정부분
                    log("[put] 생산자 깨어남");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            queue.offer(data);
            log("[put] 생산자 데이터 저장, notify() 호출");
            consumerCondition.signal(); // 수정부분
        } finally {
            lock.unlock();
        }
    }

    public String take() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                log("[take] 큐에 데이터가 없음, 소비자 대기");
                try {
                    consumerCondition.await(); // 수정부분
                    log("[take] 소비자 깨어남");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            String data = queue.poll();
            log("[take] 소비자 데이터 획득, notify() 호출");
            producerCondition.signal(); // 수정부분
            return data;
        } finally {
            lock.unlock();
        }
    }

    @Override
    public String toString() {
        return queue.toString();
    }
}
```

생산자 먼저 & 소비자 먼저

```
22:59:18:700 [     main] ==[생산자 먼저 실행] 시작, BoundedQueueV5==

22:59:18:703 [     main] 생산자 시작
22:59:18:714 [ Thread-0] [생산 시도] data1 -> []
22:59:18:714 [ Thread-0] [put] 생산자 데이터 저장, notify() 호출
22:59:18:714 [ Thread-0] [생산 완료] data1 -> [data1]
22:59:18:825 [ Thread-1] [생산 시도] data2 -> [data1]
22:59:18:825 [ Thread-1] [put] 생산자 데이터 저장, notify() 호출
22:59:18:826 [ Thread-1] [생산 완료] data2 -> [data1, data2]
22:59:18:930 [ Thread-2] [생산 시도] data3 -> [data1, data2]
22:59:18:931 [ Thread-2] [put] 큐가 가득 참, 생산자 대기

22:59:19:034 [     main] 현재 상태 출력, 큐 데이터: [data1, data2]
22:59:19:035 [     main] Thread-0: TERMINATED
22:59:19:035 [     main] Thread-1: TERMINATED
22:59:19:035 [     main] Thread-2: WAITING

22:59:19:036 [     main] 소비자 시작
22:59:19:038 [ Thread-3] [소비 시도]     ? <- [data1, data2]
22:59:19:038 [ Thread-3] [take] 소비자 데이터 획득, notify() 호출
22:59:19:039 [ Thread-2] [put] 생산자 깨어남
22:59:19:039 [ Thread-3] [소비 완료] data1 <- [data2]
22:59:19:039 [ Thread-2] [put] 생산자 데이터 저장, notify() 호출
22:59:19:040 [ Thread-2] [생산 완료] data3 -> [data2, data3]
22:59:19:139 [ Thread-4] [소비 시도]     ? <- [data2, data3]
22:59:19:139 [ Thread-4] [take] 소비자 데이터 획득, notify() 호출
22:59:19:140 [ Thread-4] [소비 완료] data2 <- [data3]
22:59:19:244 [ Thread-5] [소비 시도]     ? <- [data3]
22:59:19:245 [ Thread-5] [take] 소비자 데이터 획득, notify() 호출
22:59:19:245 [ Thread-5] [소비 완료] data3 <- []

22:59:19:349 [     main] 현재 상태 출력, 큐 데이터: []
22:59:19:349 [     main] Thread-0: TERMINATED
22:59:19:350 [     main] Thread-1: TERMINATED
22:59:19:350 [     main] Thread-2: TERMINATED
22:59:19:350 [     main] Thread-3: TERMINATED
22:59:19:351 [     main] Thread-4: TERMINATED
22:59:19:351 [     main] Thread-5: TERMINATED
22:59:19:351 [     main] ==[생산자 먼저 실행] 종료, BoundedQueueV5==

ㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡ23:00:33:986 [     main] ==[소비자 먼저 실행] 시작, BoundedQueueV5==

23:00:33:990 [     main] 소비자 시작
23:00:33:993 [ Thread-0] [소비 시도]     ? <- []
23:00:33:993 [ Thread-0] [take] 큐에 데이터가 없음, 소비자 대기
23:00:34:100 [ Thread-1] [소비 시도]     ? <- []
23:00:34:100 [ Thread-1] [take] 큐에 데이터가 없음, 소비자 대기
23:00:34:205 [ Thread-2] [소비 시도]     ? <- []
23:00:34:205 [ Thread-2] [take] 큐에 데이터가 없음, 소비자 대기

23:00:34:310 [     main] 현재 상태 출력, 큐 데이터: []
23:00:34:317 [     main] Thread-0: WAITING
23:00:34:318 [     main] Thread-1: WAITING
23:00:34:318 [     main] Thread-2: WAITING

23:00:34:319 [     main] 생산자 시작
23:00:34:324 [ Thread-3] [생산 시도] data1 -> []
23:00:34:325 [ Thread-3] [put] 생산자 데이터 저장, notify() 호출
23:00:34:326 [ Thread-0] [take] 소비자 깨어남
23:00:34:326 [ Thread-3] [생산 완료] data1 -> [data1]
23:00:34:326 [ Thread-0] [take] 소비자 데이터 획득, notify() 호출
23:00:34:327 [ Thread-0] [소비 완료] data1 <- []
23:00:34:430 [ Thread-4] [생산 시도] data2 -> []
23:00:34:431 [ Thread-4] [put] 생산자 데이터 저장, notify() 호출
23:00:34:431 [ Thread-1] [take] 소비자 깨어남
23:00:34:431 [ Thread-4] [생산 완료] data2 -> [data2]
23:00:34:431 [ Thread-1] [take] 소비자 데이터 획득, notify() 호출
23:00:34:432 [ Thread-1] [소비 완료] data2 <- []
23:00:34:535 [ Thread-5] [생산 시도] data3 -> []
23:00:34:536 [ Thread-5] [put] 생산자 데이터 저장, notify() 호출
23:00:34:536 [ Thread-2] [take] 소비자 깨어남
23:00:34:536 [ Thread-5] [생산 완료] data3 -> [data3]
23:00:34:536 [ Thread-2] [take] 소비자 데이터 획득, notify() 호출
23:00:34:537 [ Thread-2] [소비 완료] data3 <- []

23:00:34:640 [     main] 현재 상태 출력, 큐 데이터: []
23:00:34:640 [     main] Thread-0: TERMINATED
23:00:34:641 [     main] Thread-1: TERMINATED
23:00:34:641 [     main] Thread-2: TERMINATED
23:00:34:641 [     main] Thread-3: TERMINATED
23:00:34:642 [     main] Thread-4: TERMINATED
23:00:34:642 [     main] Thread-5: TERMINATED
23:00:34:642 [     main] ==[소비자 먼저 실행] 종료, BoundedQueueV5==
```

* 기존 하나였던 대기 공간을 producerCondition과 consumerCondition으로 나눠서 관리하게 됐다.
* 나눠진 대기 공간으로 인해, 원하는 스레드를 꺠울 수 있게됨.



## 4. 락 대기 집합

### 4.1. synchronized vs ReentrantLock

#### Synchronized

자바의 모든 객체 인스턴스는 멀티스레드와 임계 영역을 다루기 위해 내부에 3가지 기본 요소를 가진다.

* 모니터 락
* 락 대기 집합(모니터 락 대기 집합)
* 스레드 대기 집합

여기서 락 대기 집합이 1차 대기소이고, 스레드 대기 집합이 2차 대기소이다. 2차 대기소에 들어간 스레드는 2차, 1차 대기소를 모두 빠져나와야 임계영역을 수행.

이 3가지 요소들은 서로 맞물려 돌아간다.

1. synchronized를 사용한 임계 영역에 들어가려면 모니터 락이 필요하다.
2. 모니터 락이 없으면 락 대기 집합에 들어가서 BLOCKED 상태로 대기
3. 모니터 락을 반납하면 락 대기 집합에 있는 스레드 중 하나가 락을 획득하고 BLOCKED -> RUNNABLE 상태변경

#### ReentrantLock

Lock(ReentrantLock)도 2가지 단계의 대기 상태 존재하며 3가지 기본 요소를 가진다.

* 락 (ReentrantLock의 락)
* 락 대기 집합 (ReentrantLock의 락 대기 집합)
* 스레드 대기 집합

1. 임계 영역에 들어가려면 락이 필요하다.
2. lock.lock()을 호출했을때 락이 없으면 락 대기 집합에 들어가서 WAITING 상태로 대기
3. lock.unlock()을 호출하면 락을 반납하고, 락 대기 집합에 있는 스레드 중 하나가 락을 획득하고 WAITING->RUNNABLE 상태변경



## 5. BlockingQueue

자바는 생산자 소비자 문제를 해결하기 위해 java.util.concurrent.BlockingQueue 라는 특별한 멀티스레드 자료 구조를 제공. 이름 그대로 스레드를 차단(blocking)할 수 있는 큐이다.

* 데이터 추가 차단: 큐가 가득 차면 데이터 추가 작업을 시도하는 스레드는 공간이 생길 때까지 차단된다.
* 데이터 획득 차단: 큐가 비어 있으면 획득 작업을 시도하는 스레드는 큐에 데이터가 들어올 때까지 차단.

### 5.1. BlockingQueue 인터페이스

구현체

* ArrayBlockingQueue: 배열 기반. 버퍼 크기 고정
* LinkedBlockingQueue: 링크 기반. 버퍼 크기 고정 또는 무한으로 사용 가능



제공 메서드

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### **Throws Exception - 대기 시 예외 발생**

1. **add(e)**
   * 큐에 지정된 요소를 추가합니다.
   * 큐가 가득 차면 `IllegalStateException` 예외를 던집니다.
2. **remove()**
   * 큐에서 요소를 제거하고 반환합니다.
   * 큐가 비어 있으면 `NoSuchElementException` 예외를 던집니다.
3. **element()**
   * 큐의 머리 요소를 반환하지만, 요소를 제거하지 않습니다.
   * 큐가 비어 있으면 `NoSuchElementException` 예외를 던집니다.

#### **Special Value - 즉시 반환값 제공**

1. **offer(e)**
   * 큐에 지정된 요소를 추가하려고 시도합니다.
   * 큐가 가득 차면 `false`를 반환합니다.
2. **poll()**
   * 큐에서 요소를 제거하고 반환합니다.
   * 큐가 비어 있으면 `null`을 반환합니다.
3. **peek()**
   * 큐의 머리 요소를 반환하지만, 요소를 제거하지 않습니다.
   * 큐가 비어 있으면 `null`을 반환합니다.

#### **Blocks - 대기**

1. **put(e)**
   * 지정된 요소를 큐에 추가할 수 있을 때까지 대기합니다.
   * 큐가 가득 차면 공간이 생길 때까지 대기합니다.
2. **take()**
   * 큐에서 요소를 제거하고 반환합니다.
   * 큐가 비어 있으면 요소가 준비될 때까지 대기합니다.

#### **Times Out - 시간 대기**

1. **offer(e, time, unit)**
   * 큐에 지정된 요소를 추가하려고 시도합니다.
   * 큐가 가득 차면 지정된 시간 동안 대기하다가 시간이 초과되면 `false`를 반환합니다.
2. **poll(time, unit)**
   * 큐에서 요소를 제거하고 반환합니다.
   * 큐에 요소가 없으면 지정된 시간 동안 대기하다가 시간이 초과되면 `null`을 반환합니다.

이 메서드들을 활용하여 큐가 가득찼을때 다양한 상황에 대처할 수 있다.

* 예외를 던진다. 예외를 받아서 처리한다.
* 대기하지 않는다. 즉시 false를 반환한다.
* 대기한다.
* 특정 시간 만큼만 대기한다.



### 5.2. put(), take() 예시

기존 BoundedQueueV5에서 BlockingQueue, put(), take() 를 사용한 예시

```java
package thread.bounded;

import java.util.ArrayDeque;
import java.util.Queue;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

import static util.MyLogger.log;

public class BoundedQueueV6_1 implements BoundedQueue {
    private BlockingQueue<String> queue;
    
    public BoundedQueueV6_1(int max) {
        queue = new ArrayBlockingQueue<>(max);
    }
    
    public void put(String data) {
        try {
            queue.put(data);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
    /*
    public void put(String data) {
        lock.lock();
        try {
            while (queue.size() == max) {
                log("[put] 큐가 가득 참, 생산자 대기");
                try {
                    producerCondition.await();
                    log("[put] 생산자 깨어남");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            queue.offer(data);
            log("[put] 생산자 데이터 저장, notify() 호출");
            consumerCondition.signal();
        } finally {
            lock.unlock();
        }
    }
    */
    // V5의 put과 동일하게 동작
    
    
    public String take() {
        try {
            return queue.take();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
    /*
    public String take() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                log("[take] 큐에 데이터가 없음, 소비자 대기");
                try {
                    consumerCondition.await();
                    log("[take] 소비자 깨어남");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            String data = queue.poll();
            log("[take] 소비자 데이터 획득, notify() 호출");
            producerCondition.signal();
            return data;
        } finally {
            lock.unlock();
        }
    }
    */
    // V5의 take와 동일하게 동작

    @Override
    public String toString() {
        return queue.toString();
    }
}
```

```
18:51:19:418 [     main] ==[생산자 먼저 실행] 시작, BoundedQueueV6_1==

18:51:19:421 [     main] 생산자 시작
18:51:19:430 [ Thread-0] [생산 시도] data1 -> []
18:51:19:431 [ Thread-0] [생산 완료] data1 -> [data1]
18:51:19:536 [ Thread-1] [생산 시도] data2 -> [data1]
18:51:19:536 [ Thread-1] [생산 완료] data2 -> [data1, data2]
18:51:19:641 [ Thread-2] [생산 시도] data3 -> [data1, data2]

18:51:19:746 [     main] 현재 상태 출력, 큐 데이터: [data1, data2]
18:51:19:746 [     main] Thread-0: TERMINATED
18:51:19:747 [     main] Thread-1: TERMINATED
18:51:19:747 [     main] Thread-2: WAITING

18:51:19:747 [     main] 소비자 시작
18:51:19:749 [ Thread-3] [소비 시도]     ? <- [data1, data2]
18:51:19:750 [ Thread-2] [생산 완료] data3 -> [data2, data3]
18:51:19:750 [ Thread-3] [소비 완료] data1 <- [data2]
18:51:19:851 [ Thread-4] [소비 시도]     ? <- [data2, data3]
18:51:19:851 [ Thread-4] [소비 완료] data2 <- [data3]
18:51:19:956 [ Thread-5] [소비 시도]     ? <- [data3]
18:51:19:957 [ Thread-5] [소비 완료] data3 <- []

18:51:20:061 [     main] 현재 상태 출력, 큐 데이터: []
18:51:20:061 [     main] Thread-0: TERMINATED
18:51:20:061 [     main] Thread-1: TERMINATED
18:51:20:062 [     main] Thread-2: TERMINATED
18:51:20:062 [     main] Thread-3: TERMINATED
18:51:20:062 [     main] Thread-4: TERMINATED
18:51:20:062 [     main] Thread-5: TERMINATED
18:51:20:062 [     main] ==[생산자 먼저 실행] 종료, BoundedQueueV6_1==
```

* 기존 V5와 동일한 결과.



### 5.3. offer(), poll() 예시

기존 BoundedQueueV5에서 BlockingQueue, offer, poll() 를 사용한 예시

```java
package thread.bounded;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

import static util.MyLogger.log;

public class BoundedQueueV6_2 implements BoundedQueue {
    private BlockingQueue<String> queue;

    public BoundedQueueV6_2(int max) {
        queue = new ArrayBlockingQueue<>(max);
    }

    public void put(String data) {
        boolean result = queue.offer(data);
        log("[put] 생산자 데이터 저장, offer() 호출 결과: " + result);
    }

    public String take() {
        return queue.poll();
    }

    @Override
    public String toString() {
        return queue.toString();
    }
}
```

```
18:49:18:344 [     main] ==[생산자 먼저 실행] 시작, BoundedQueueV6_2==

18:49:18:346 [     main] 생산자 시작
18:49:18:355 [ Thread-0] [생산 시도] data1 -> []
18:49:18:355 [ Thread-0] [put] 생산자 데이터 저장, offer() 호출 결과: true
18:49:18:356 [ Thread-0] [생산 완료] data1 -> [data1]
18:49:18:464 [ Thread-1] [생산 시도] data2 -> [data1]
18:49:18:464 [ Thread-1] [put] 생산자 데이터 저장, offer() 호출 결과: true
18:49:18:464 [ Thread-1] [생산 완료] data2 -> [data1, data2]
18:49:18:569 [ Thread-2] [생산 시도] data3 -> [data1, data2]
18:49:18:570 [ Thread-2] [put] 생산자 데이터 저장, offer() 호출 결과: false
18:49:18:570 [ Thread-2] [생산 완료] data3 -> [data1, data2]

18:49:18:674 [     main] 현재 상태 출력, 큐 데이터: [data1, data2]
18:49:18:674 [     main] Thread-0: TERMINATED
18:49:18:674 [     main] Thread-1: TERMINATED
18:49:18:675 [     main] Thread-2: TERMINATED

18:49:18:675 [     main] 소비자 시작
18:49:18:677 [ Thread-3] [소비 시도]     ? <- [data1, data2]
18:49:18:677 [ Thread-3] [소비 완료] data1 <- [data2]
18:49:18:780 [ Thread-4] [소비 시도]     ? <- [data2]
18:49:18:780 [ Thread-4] [소비 완료] data2 <- []
18:49:18:885 [ Thread-5] [소비 시도]     ? <- []
18:49:18:885 [ Thread-5] [소비 완료] null <- []

18:49:18:990 [     main] 현재 상태 출력, 큐 데이터: []
18:49:18:990 [     main] Thread-0: TERMINATED
18:49:18:990 [     main] Thread-1: TERMINATED
18:49:18:991 [     main] Thread-2: TERMINATED
18:49:18:991 [     main] Thread-3: TERMINATED
18:49:18:991 [     main] Thread-4: TERMINATED
18:49:18:991 [     main] Thread-5: TERMINATED
18:49:18:992 [     main] ==[생산자 먼저 실행] 종료, BoundedQueueV6_2==
```

* offer를 사용했기때문에 생산자 3번 스레드는 false를 반환하고 종료됨
* poll()을 사용했기때문에 소비자 3번 스레드는 null을 반환하고 종료됨

