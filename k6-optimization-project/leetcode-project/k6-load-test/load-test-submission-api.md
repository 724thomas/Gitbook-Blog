# Load Test: Submission API

비기능 요구사항의 고확장성(최대 10,000명의 동시 제출 처리를 지원)을 만족하기 위해 테스트를 해봤습니다.



## Case1. 동시 10,000 요청

문제가 발생하는지 확인하기 위해 k6를 통해 간단히 10,000개의 요청을 보냈습니다. CPU, 메모리, 커넥션 풀을 확인했지만 두가지 문제가 발생했습니다.

* Dial I/O Time-out
  * 클라이언트가 서버에 연결을 시도하는 **TCP 커넥션 맺기 단계**에서 일정 시간 안에 연결이 되지 않으면 발생하는 에러.
  * 커넥션을 받을 수 없는 상황
* Request Time-Out
  * 연결은 성공했지만, 서버가 응답을 **지정 시간 안에 보내지 않으면** 발생하는 에러.
  * 앞의 요청이 밀려서, 뒤에 요청이 처리가 안되는 상황



## Dial I/O Time-out해결 시도

### 시도1: 톰캣 설정 수정 (채택O)

톰캣의 기본 설정은 아래와 같습니다.

```yaml
server:
  tomcat:
    threads:
      max: 200 # 최대 스레드 개수
      min-spare: 10 # 최소 유휴 스레드 개수 
    max-connections: 8192 # 톰캣이 수락할 수 있는 최대 요청 연결 개수
    accept-count: 100 # 운영체제에서 톰캣에게 전달되기 전에 대기시키는 최대 연결 개수
```

위 설정을 기반으로 예상되는 시나리오는:

1. **초기 요청 유입 (0 \~ 10개 요청) → `server.tomcat.threads.min-spare`**
   * 즉시 존재하는 스레드로 처리 후 응답한다.
2. **스레드 풀 확장 (11 \~ 200개 요청) → `server.tomcat.threads.max`**
   * 10개를 초과하는 요청이 들어오면, 톰캣은 200개까지 새로운 스레드를 생성하여 요청을 처리한다.
   * 1번과 비교하여 스레드 생성 비용만 차이날 것이다.
3. **최대 스레드 도달 및 연결 대기 (201 \~ 8392개 동시 요청) → `server.tomcat.max-connections`**
   * 스레드 개수가 200개에 도달했으므로, 더 이상 스레드를 생성할 수 없다.
   * 톰캣은 이 요청에 대한 연결을 8192개 까지 수락하고 해당 요청은 Blocking 되도록 한다.
4. **최대 커넥션 도달 및 OS 대기 큐 활용 (8393 \~ 8492개 동시 요청) → `server.tomcat.accept-count`**
   * 이 요청은 운영체제에 100개 까지 대기 큐에 쌓이게 된다.
   * 이 큐에 있는 요청들은 톰캣에 수락된 요청 중 하나가 비워질 때까지 기다린다.
5. **대기 큐 가득 참 및 연결 거부 (8493개 이상 동시 요청) → 처리 불가**
   * 더 이상 요청을 수락할 수 없기에 즉시 예외를 발생시키고 요청은 처리하지 않는다.



예외 발생 가능 시나리오는



* 2, 3번) 로직 처리 중에 설정된 timeout 시간을 넘어서면 `read-timeout` 예외가 발생한다.
* 3, 4번) 요청이 설정된 timeout 시간만큼 대기하게 되면 `connection-timeout` 예외가 발생한다.
* 5번) 더 이상 이 요청을 수락할 수 없기에 즉시 `connection-refused` 예외가 발생한다.

위 3가지 시나리오 모두 sandbox 코드 제출 로직 때문에 고려되는 엣지 케이스임을 알 수 있습니다.

이를 해결하기 위해 max-connection을 10,000으로 늘렸습니다. 10,000으로 늘린 후 부터는 `Dial I/O Time-out` 문제를 해결하였습니다.



**하지만**, 단순히 max-connection을 늘리면 아래와 같은 문제가 발생합니다.

#### **커넥션 수가 많아질수록 스레드 부족, 병목, CPU 상승**

* `max-connections`는 **동시 TCP 연결 수**입니다.\
  하지만 연결 수가 늘어도 **요청 처리 스레드(`max-threads`)는 제한**되어 있습니다.

즉,

* 커넥션은 많지만 → 처리할 스레드가 없음 → **응답 지연 or 타임아웃 증가**
* 연결은 계속 유지됨 (keep-alive 등) → **리소스(메모리, FD) 고갈**

이젠, 응답지연 및 타임아웃 증가 문제와 메모리 관리가 중요해졌습니다.

***

## Request Time-Out 시도

### 시도1: 서버 확장 (채택X)

단순히 스프링 서버 대수를 늘려서 가용 스레드를 늘리는 방안입니다.

<figure><img src="/broken/files/IMc6GH74MTEZ1zJKN56I" alt=""><figcaption></figcaption></figure>

**장점**

* 높은 처리량: 앞의 요청들이 빠르게 처리된다면, time-out이 발생하기전에 요청을 처리할 수 있고, 처리량도 높아집니다.

**단점**

* 요청에 비례하여 서버 대수를 계속 늘려야합니다. 비용과 Auto Scaling 설정 등 복잡도가 증가합니다.&#x20;
* 서버 한대로 몇백 TPS를 견디기 힘든 상황인 만큼, 많은 서버가 필요하게 됩니다.

지금 당장은 문제를 단순하고 빠르게 해결할 수 있지만, **확실한 해결책은 아니고, 기술부채가 늘어나기때문에** 다른 방안을 고려하기로 했습니다.



### 시도2: 톰캣 Connection Time-out 조정 (채택X)

두번째 방안은 Connection Time-out을 조정하는 것이었습니다. 앞에 처리가 늦어지고, 응답을 '시간'내에 못받아서 문제가 발생하고 있다면, 단순하게 '시간'을 늘리자 생각해봤습니다.

**장점**

* 간단 설정: 설정 값에 대해 단순한 계산 방식과 최소한의 노력으로 효과를 볼 수 있습니다.

**단점**

* 최적화가 아닌, 당장의 문제를 회피하는 방법입니다.
* 부하 상황이 계속되는 상황에서는 좋은 해결책이 아닙니다.

당장은 해결할 수 있지만, **신뢰성있는 서버가 되지는 않을것** 같아서 다른 방안을 고려하기로 했습니다.



### 시도3: 메시지 큐 도입 (채택O)

코드 실행(sandbox) 담당 로직을 비동기로 처리하고, polling / SSE로 결과를 확인하는 방법을 생각했습니다.

<figure><img src="/broken/files/7mGZhJ5s2t1L1hhGysI2" alt=""><figcaption></figcaption></figure>

**장점**

* API의 속도가 빨라지며, '빠르게 처리하는 것처럼' 보이기 떄문에, 더 많은 요청을 받을 수 있습니다.
* 요청이 Queue에 적재되어서 Time-out이 발생하지는 않습니다.

단점

* MQ 처리를 위한 설정이 중요해집니다.
* 서버를 분리해야하는 작업이 필요합니다.
* Edge case 핸들링이 복잡해집니다.

불안 요소가 많기는 하지만, 이전 방안들 보다는 얻을 수 있는 이점이 많다고 판단하였고, 3번 방안을 선택했습니다.





## Case2. 동시 10,000 요청 재시도

모든 요청이 성공적으로 처리되었습니다. 또한 지표를 봤을때,

**Java app**

<figure><img src="/broken/files/jx0KVvFyPE0YcVx6IylW" alt=""><figcaption></figcaption></figure>

* CPU: 최대 81%로, 위험한 수준입니다.
* Mem: 최대 52.3%로 메모리는 널널한 수준입니다.
* Heap: 78.2%로 위험한 수준입니다. 하지만 수초(약 20초) 내에 60% 아래로 회복됨이 확인됩니다.

<figure><img src="/broken/files/6yqtc5Lt6MdUfA6fPnDd" alt=""><figcaption></figcaption></figure>

**RDS**

<figure><img src="/broken/files/JOCk0ZEf8Yr5nJr7rTbi" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/3HeD2vCl3MYPzshHq1Ke" alt=""><figcaption></figcaption></figure>

* CPU: 33%으로 널널한 편입니다.
* FreeableMemory(103M)이라서 Ram 가용공간이 많은 편입니다.
* Connection Pool: **최대 설정한 커넥션수를 모두 사용하고 있습니다.**

**K6**

<figure><img src="/broken/files/CNCrCxmVlF1qDeZbTT8O" alt=""><figcaption></figcaption></figure>

요청 처리 시간:

* **avg (평균 응답 시간)**: 약 **35\~37초**
* **p90 (90% 이하 요청의 응답 시간)**: 약 **38초**
* **p95**: 약 **39초**
* **p99**: 약 **40초 이상**
* **전반적으로 모든 지표가 선형 상승 중**
  * 요청이 많아질수록 대기열(queue)이 쌓이며 점진적 지연 발생
  * 전체 요청의 성능 저하가 발생 중 (일부 요청만 느린 것이 아님)



**API**

요청이 처리되는 최대 시간: 5분 52초

<figure><img src="/broken/files/utcg2qtcQdxFFYDJ6cin" alt=""><figcaption><p>첫번째 요청: 6초 (15분 11초 - 15분 17초)</p></figcaption></figure>

<figure><img src="/broken/files/6wwcYfGLxrSeStnKofcw" alt=""><figcaption><p>10000번째 요청(마지막): <strong>5분 52초.</strong> (15분 57초 - 21분 49초)</p></figcaption></figure>

요청이 잘 처리가되는것을 확인했습니다. 다만, 요청 처리 속도가 효율적이지 않습니다.

<figure><img src="/broken/files/zZFu1uGOqW8ESQ9fQNtg" alt=""><figcaption><p>Api 서버 -> Sandbox 서버</p></figcaption></figure>

<figure><img src="/broken/files/pbcXuCWubVWYWTfOHiNP" alt=""><figcaption><p>Sandbox 서버 -> Api 서비</p></figcaption></figure>



MQ에 publish 속도와 consume 속도를 비교했었을때, **publish 속도(340/s)**&#xAC00; 월등히 빠른것을 확인했습니다. Consume되고 다시 publish되는 **속도가 부족(20/s)**&#xD558;다는 결론이 나왔고, Consumer 스레드 갯수를 증가시켜봤습니다.



## Case3. 컨슈머 결과 처리 서버 분리 + Consumer 스레드 증가

pi 서버에서는 Consumer 스레드 갯수를 증가시키면, 메모리 고갈이 발생할 수 있어서 소비하는 역할을 하는 서버를 따로 만들었습니다.

그 후에, Consumer 스레드의 갯수를 5배로 늘려줬습니다.

AWS에서는 CPU credit 한계가 존재하기 때문에, CPU Burst를 무제한으로 일시적으로 바꿔줬습니다 \
(T3.small기준 시간당 $0.05)

```java
...
factory.setPrefetchCount(1);
factory.setConcurrentConsumers(100); // 100으로 늘려줬습니다.
factory.setMaxConcurrentConsumers(200); // 200으로 늘려줬습니다.
...
```

<div align="left"><figure><img src="/broken/files/AAhy9XAVjEYboHk6hig9" alt=""><figcaption><p>첫번째 요청: 2초 (31분 01초 - 31분 03초)</p></figcaption></figure></div>

<div align="left"><figure><img src="/broken/files/6JOAYgbV9FlD4Wv1vaUy" alt=""><figcaption><p>10000번째 요청(마지막): <strong>2분 50초.</strong> (31분 31초 - 34분 21초)</p></figcaption></figure></div>

마지막 요청 처리 시간이 2분 50초로, 훨씬 빨라진것을 확인했습니다. (처리속도가 2배 증가)



**그러면 스레드를 2배로 늘리면 더 빨라질까? 라는 의문이 들었습니다.**

```java
...
factory.setPrefetchCount(1);
factory.setConcurrentConsumers(250); // 250으로 늘려줬습니다.
factory.setMaxConcurrentConsumers(250); // 250으로 늘려줬습니다.
...
```

<figure><img src="/broken/files/OOIlNBhlA261Sm5b0QdI" alt=""><figcaption><p>Api 서버 -> Sandbox 서버</p></figcaption></figure>

<figure><img src="/broken/files/CDwALyUZlzjQA2XAwyvF" alt=""><figcaption><p>Sandbox 서버 -> Api 서비</p></figcaption></figure>

<div align="left"><figure><img src="/broken/files/oTlC87av1vPM7BBl083v" alt=""><figcaption><p>첫번째 요청: 3초 (52분 27초 - 52분 30초)</p></figcaption></figure></div>

<div align="left"><figure><img src="/broken/files/dqMC1e9LtLrhLRnEu917" alt=""><figcaption><p>10000번째 요청(마지막): 1분 7초. (52분 39초 - 53분 48초)</p></figcaption></figure></div>

훨씬 빨라진것을 확인했습니다. 처리속도가 또 2배로 증가했습니다. \
(그 이상도 테스트를 해봤지만, 빨라지진 않았습니다.)

***

## 결과

### **변경된 점**

* Max-Connections: 8142 -> 10000으로 변경
* RabbitMQ 도입
* sandbox 서버 분리
* Consumer Thread 갯수 증가 -> 250으로 증가



### 장점

* **사용자 경험 개선**: 코드 실행을 외부 샌드박스에서 처리하도록 분리해, 응답 지연을 줄임
* 기존 코드 실행 로직을 위임 했기에 사용자 경험 측면과 코드 제출 API 성능 측면에서 향상됨
* 메시지를 처리하는 스레드가 늘어나서 처리량이 높아짐 (CPU 를 사용하지 않는 메서드라서 가능)

### 단점

* **아키텍처 복잡도 증가**:
  * 기존의 단일 API 흐름 → 샌드박스 + MQ + 폴링 API로 다단계 처리
  * 장애 지점(샌드박스, MQ, 폴링 API) 늘어남
  * 모니터링 지점 증가
  * Edge Case 복잡도 증가
* CPU를 사용하지 않는 Time.sleep()을 사용했기 때문에, CPU 부하가 발생하지 않음.
  * 사실상 "무한대"의 sandbox 서버가 있는 것.



### **시스템 동작 흐름**

1. API 요청:
   * 사용자가 소스코드를 담아 채점 API(`POST /api/submissions`)를 호출합니다.
2. 메시지 발행(Publish):
   * `Controller`는 요청 데이터를 `SubmissionRequest` DTO로 변환하여 RabbitMQ의 `submission.request.queue`로 메시지를 전송합니다.
   * 그리고 즉시 submissionId를 사용자에게 응답을 보냅니다.
3. 메시지 소비(Consume):
   * `SubmissionConsumer`가 `request` 큐를 구독하고 있다가 메시지를 수신합니다.
4. 비즈니스 로직 처리:
   * `SubmissionConsumer`는 수신한 메시지를 `SubmissionProcessor`에게 전달하여 실제 코드 채점 로직을 수행하도록 위임합니다.
5. 결과 반환:
   * `SubmissionProcessor`는 채점을 완료하고, 성공/실패 여부가 담긴 `SubmissionResult` 객체를 `SubmissionConsumer`에게 반환합니다.
6. 결과 메시지 발행(Publish):
   * `SubmissionConsumer`는 전달받은 `SubmissionResult`를 RabbitMQ의 `submission.result.queue`로 전송합니다.
7. 최종 결과 통보: 별도의 결과 처리 리스너가 `result` 큐의 메시지를 받아, `Polling`을 통해 사용자에게 실시간으로 채점 결과를 알려줍니다.'



### **단일 호출 테스트**

Submit API

```java
ProblemController.submitCode(..) args=[1, SubmissionRequest@4286f192]
|-->ProblemService.submitProblem(..) args=[1, SubmissionRequest@4286f192]
|-->CrudRepository.findById(..) args=[1]
|<--CrudRepository.findById(..) args=[1] time=7ms
|-->CrudRepository.findById(..) args=[2]
|<--CrudRepository.findById(..) args=[2] time=2ms
|-->CrudRepository.save(..) args=[Submission@20f83daa]
|<--CrudRepository.save(..) args=[Submission@20f83daa] time=28ms
ProblemController.submitCode(..) args=[1, SubmissionRequest@4286f192] time=83ms
// 83ms 처리속도.
// 코드 실행 역할을 Thread.sleep에서 샌드박스로 위임하였기에 준수한 성능을 보임
```

Polling API

```java
SubmissionController.getSubmissionStatus(..) args=[122252]
|-->SubmissionService.getSubmissionStatus(..) args=[122252]
|-->CrudRepository.findById(..) args=[122252]
|<--CrudRepository.findById(..) args=[122252] time=39ms
|<--SubmissionService.getSubmissionStatus(..) args=[122252] time=40ms
SubmissionController.getSubmissionStatus(..) args=[122252] time=53ms
// 53ms 처리속도
```

### 10000번 호출 테스트

* 10000번째 요청(마지막): 1분 7초. (52분 39초 - 53분 48초)
