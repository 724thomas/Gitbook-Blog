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

임계 영역
