# JPA - OSIV

## 개요

Leetcode 프로젝트의 **submitProblem(유저코드 제출) API**에 부하테스트를 진행하고 있었습니다. 비즈니스 로직 내부에 sleep() 이 있기때문에, 스레드 고갈 상황을 기대했었지만, 커넥션풀 고갈이라는 예상과는 다른 결과를 얻었습니다.

우선 아래 코드는 문제 제출했을때의 실행되는 코드입니다.

```java
// Problem Service
public SubmissionResponse submitProblem(Long problemId, SubmissionRequest request) {
    Problem problem = problemRepository.findById(problemId)
            .orElseThrow(() -> new IllegalArgumentException("문제가 존재하지 않습니다."));
            
    //중간 코드 생략...
    
    sleep((int)runtime*1000); // 유저 제출 코드 실행시 걸리는 시간

    //중간 코드 생략... 

    //아래 메서드는 @Transactional
    submissionService.saveSubmissionAndLeaderboard(submission, request, user);

    return SubmissionResponse.of(testResults);
}
```

```java
//Submission Service
@Transactional
public void saveSubmissionAndLeaderboard(Submission submission, SubmissionRequest request, User user) {
    submissionRepository.save(submission);
    
    // 중간 코드 생략...
    
}
```

* 서비스내에서 호출되는 sleep()은 해당 스레드를 2초간 블로킹합니다.
  * 스레드는 아무 일도 하지 않으면서 풀 안에서 점유 상태로 대기합니다.
* Tomcat은 기본적으로 최대 200개의 요청을 동시에 처리 가능합니다.
  * 과정: 요청 들어옴 -> 요청 큐 -> \[워커 스레드] 처리 -> 처리 끝 -> 스레드 반환

***



### 부하 테스트: 커넥션풀 고갈

아래는 부하테스트를 위한 k6 스크립트입니다.

<details>

<summary>K6 스크립트</summary>

```javascript
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { Trend, Counter } from 'k6/metrics';

// 1) 사용자 정의 메트릭 선언
let rdbTrend      = new Trend('rdb_duration_ms');
let listTrend     = new Trend('list_duration_ms');
let detailTrend   = new Trend('detail_duration_ms');
let submitTrend   = new Trend('submit_duration_ms');

let errorCount    = new Counter('errors_total');

// ─── stages 설정 ───
export let options = {
  stages: [
    { duration: '2s', target: 1000 },  // 0 → 1000 VUs over 2s
    { duration: '10s', target: 1000 },  // 1000 VUs over next 10s
  ],
};

const BASE_URL = 'http://172.31.6.93:8080';

function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

// 공통 로깅 & 메트릭 헬퍼
function logAndMetric(res, trend, label) {
  const ok = check(res, { [`${label} is 200`]: r => r.status === 200 });
  if (!ok) {
    console.error(
      `❌  [${label}] status ${res.status}\n` +
      `URL: ${res.url}\n` +
      `Body:\n${res.body}\n`
    );
    errorCount.add(1);
  }
  trend.add(res.timings.duration);
}

export default function () {
  group('Submit Problem', () => {
    let pid     = getRandomInt(1, 50);
    let userId  = getRandomInt(1, 500);
    let payload = JSON.stringify({
      userId: userId,
      code: 'System.out.println("Hi");',
      language: 'java'
    });
    let params  = { headers: { 'Content-Type': 'application/json' } };
    let res     = http.post(`${BASE_URL}/problems/${pid}/submission`, payload, params);
    logAndMetric(res, submitTrend, `Submit problem ${pid}`);
  });
  sleep(1);
}
```

</details>

* 요약: 약 12초동안 1000명의 가상 유저를 생성하고, /submission 요청을 보내는 부하 상황을 시뮬레이션 했습니다.



**예상 시나리오: 부하테스트 중, 만약 1000명이 동시에 요청하면?**

* **0 \~ 199번째 요청**
  * Tomcat의 스레드 풀에서 스레드를 할당받음
  * 각 스레드는 `sleep().` 2초 동안 점유되며 **대기 중**
  * 응답 못하고 있으므로 **반납도 안 됨**
* **200 \~ 1000번째 요청**
  * 스레드가 모두 점유 중이기 때문에 할당 불가
  * Tomcat은 요청을 대기 큐에 쌓음 (`acceptCount`, 기본 100)
* **대기 큐가 가득 차면**
  * **503 Service Unavailable** 또는 연결 자체가 거부됨
  * 여기서 **스레드풀 고갈** 또는 **Thread starvation**

이러한 이유로,  ThreadPool 고갈을 예상했었습니다. 하지만 실제 결과는 아래와 같았습니다.

{% code fullWidth="true" %}
```

         /\      Grafana   /‾‾/  
    /\  /  \     |\  __   /  /   
   /  \/    \    | |/ /  /   ‾‾\ 
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/ 

     execution: local
        script: load-test.js
        output: -

ERRO[0033] ❌  [Submit problem 6] status 500
URL: http://172.31.6.93:8080/problems/6/submission
Body:
{"success":false,"message":"An unexpected error occurred: Could not open JPA EntityManager for transaction","data":null}  source=console

..생략..

ERRO[0033] ❌  [Submit problem 37] status 500
URL: http://172.31.6.93:8080/problems/37/submission
Body:
{"success":false,"message":"An unexpected error occurred: Could not open JPA EntityManager for transaction","data":null}  source=console
```
{% endcode %}

*   요약: `Could not open JPA EntityManager for transaction` 예외 발생



예외가 발생한 이유는 **DB 커넥션 풀이 고갈되어 JPA의 트랜잭션을 시작하는 데 실패했기 때문**입니다.

DB 커넥션풀이 고갈되었다는 뜻은, \
-> **DB 작업이 오래걸려서** 커넥션을 반환을 못함으로 인해,\
-> 다른 요청에서 **커넥션을 할당 받지 못해서 발생**하는 문제입니다.&#x20;



하지만, Transactional 어노테이션이 존재하는 메서드는 오래 걸리지 않습니다. 그러면 어떠한 이유로 오래 걸리는 걸까? 라고 생각을 했을때, **가장 의심되는 것은 sleep() 메서드**였습니다. 그래서 확인을 해봤습니다.



### 부하테스트2: sleep() 주석처리

다시 코드에서, sleep() 을 주석처리 해봤습니다.

```java
// Problem Service
public SubmissionResponse submitProblem(Long problemId, SubmissionRequest request) {
    Problem problem = problemRepository.findById(problemId)
            .orElseThrow(() -> new IllegalArgumentException("문제가 존재하지 않습니다."));
            
    //중간 코드 생략...
    
    //sleep((int)runtime*1000); // 주석처리!!!!!!

    //중간 코드 생략... 
    
    submissionService.saveSubmissionAndLeaderboard(submission, request, user);

    return SubmissionResponse.of(testResults);
}
```

* 요약: 문제 없이 부하를 견뎠습니다.



정리를 하자면,

* sleep() 생략 -> 부하를 버팀
* sleep() 존재 -> 커넥션풀이 고갈됨



이로서, **sleep() 이 호출될때 커넥션풀을 반납하지 않은 상태이다.** 라는 것을 알게 됩니다. 그러면 **커넥션풀을 problem가져오는 시점에 할당 받는건 맞나?** 라는 의문이 생겼습니다.



### 부하테스트3: sleep() 위치 이동

이를 확인하기 위해, sleep() 메서드를 메서드 호출 되자마자 실행되도록 위치를 옮겼습니다.

```java
// Problem Service
public SubmissionResponse submitProblem(Long problemId, SubmissionRequest request) {
    sleep((int)runtime*1000); // 위치 이동!!! 메서드 호출되자마자 실행
    
    Problem problem = problemRepository.findById(problemId)
            .orElseThrow(() -> new IllegalArgumentException("문제가 존재하지 않습니다."));
            
    //중간 코드 생략...
    
    submissionService.saveSubmissionAndLeaderboard(submission, request, user);

    return SubmissionResponse.of(testResults);
}
```

* 요약: 문제 없이 부하를 견뎠습니다.



실험을 통해 **커넥션 풀을 할당받는 시점은 submitProblem 호출 이후, 즉 problem 객체를 처음 가져오고 난 후** 라는 것을 알게 됬습니다.



submitProblem 메서드에는 @Transactional 이 없기때문에, db 작업이 끝나면 커넥션을 반납하고 있다고 생각을 했지만,  실제로는 계속 가지고 있다는 것을 확인했습니다.

```java
// Problem Service
public SubmissionResponse submitProblem(Long problemId, SubmissionRequest request) {
    // 커넥션 풀 할당!!
    Problem problem = problemRepository.findById(problemId)
            .orElseThrow(() -> new IllegalArgumentException("문제가 존재하지 않습니다."));
    
    //중간 코드 생략...
    
    sleep((int)runtime*1000); // 커넥션풀을 할당 받은 상태에서 sleep() 메서드가 실행!

    //중간 코드 생략... 
		
    // saveSubmissionAndLeaderboard에는 @Transactional 어노테이션이 존재!!
    submissionService.saveSubmissionAndLeaderboard(submission, request, user);    

    return SubmissionResponse.of(testResults);
    // 커넥션 풀 반납!!
}
```

* problem 객체를 가져오는 과정에서 커넥션풀을 사용하고, 이때 반납을 하지 않게됩니다.



찾아본 결과, Open Session In View로 인해(기본값 true), @transactional 어노테이션이 없어도, EntityManger는 요청이 끝날 때까지 살아있습니다. 커넥션도 이 EntityManger안에 붙어서 계속 유지됨으로 sleep()까지 포함됩니다.



### 결론

그러면 왜 커넥션이 고갈됐을까?

1. `problemRepository.findById()` → **DB 접근 → 커넥션 할당됨**
2. 트랜잭션이 없기 때문에 커넥션은 즉시 반납되지 않음
3. OSIV가 true이므로 **EntityManager가 요청 전체에 살아 있음**
4. Hibernate 내부적으로 **커넥션을 EntityManager에 붙인 채 유지**
5. `sleep()` 동안 커넥션은 점유된 채로 놀고 있음
6. 동시에 많은 요청이 들어오면 → 커넥션 풀 고갈 발생





## OSIV

### OSIV란?

OSIV는 "Open Session In View"의 줄임말로, JPA의 영속성 컨텍스트(Session)를 **View 렌더링 시점까지 열어두는 방식**입니다. Spring Boot에서는 `spring.jpa.open-in-view=true`가 기본 설정입니다. 기본값으로, 요청 시작 시 영속성 컨텍스트를 열고, 응답이 끝날 때 닫습니다.

```yaml
spring:
  jpa:
    open-in-view: true
```



### 왜 OSIV가 존재할까?

기존에는 View 렌더링 단계(예: Thymeleaf)에서 엔티티의 Lazy 필드에 접근할 수 있게 하려는 목적이 컸습니다.\
예를 들어:

```java
@GetMapping("/users/{id}")
public String getUser(@PathVariable Long id, Model model) {
    User user = userService.findById(id); // 트랜잭션 종료
    model.addAttribute("user", user);
    return "user"; // 여기서 user.getPosts() 하면 Lazy 필드 초기화 (쿼리 발생)
}
```

이 경우, `user.getPosts()`는 Lazy 필드이며, Controller 이후에도 영속성 컨텍스트가 살아 있어야 쿼리를 실행할 수 있습니다.\
→ 그래서 OSIV를 통해 View까지 EntityManager를 유지합니다.



하지만 API 서버에서는 전혀 다른 문제가 생깁니다.

```java
public SubmissionResponse submitProblem(...) {
    Problem problem = problemRepository.findById(...); // DB 커넥션 사용
    Thread.sleep(2000); // 커넥션 반납 안 된 상태로 대기
    submissionService.saveSubmission(...); // @Transactional
}
```

* `@Transactional`이 없는 `submitProblem()`에서 DB 접근 발생
* 이때 커넥션이 할당되며, OSIV=true이기 때문에 EntityManager가 요청 끝날 때까지 살아 있음
* → 커넥션도 그 안에 붙어 있어 **반납되지 않고 sleep() 동안 점유됨**
* 동시에 많은 요청이 들어오면 커넥션 풀 고갈 발생



### 그럼 언제 false를 써야할까?

False

* API 서버 / REST: 커넥션 풀 고갈 방지, DTO 중심 설계 유도
* 대규모 트래픽 처리: 요청 단위 커넥션 점유 제거

True

* View 기반 웹 (Thymeleaf 등): Lazy 필드 View 접근 필요
* 내부 관리자 페이지(낮은 트래픽): 빠른 개발/유연한 템플릿 처리



### 결론

OSIV는 View에서 Lazy 필드 접근을 위한 기능이지만,\
**API 서버 환경에서는 커넥션을 불필요하게 오래 점유하는 부작용**을 유발할 수 있습니다.\
특히 `@Transactional` 없이 DB에 접근하고 `sleep()` 같은 작업이 있다면,\
**커넥션 풀 고갈로 이어질 수 있습니다.**

