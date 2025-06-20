# NOT Submission API

## 최초 테스트 요약

단일 서버 환경에서 70 VU 부하로 초기 부하 테스트를 수행한 결과, 전체 API 응답의 p(95)가 약 **30초에 도달**하며 시스템 전반적으로 병목이 발생한 것을 확인했습니다. 특히 **`SubmitProblem` API는 평균 및 중앙값 응답 시간도 다른 API보다 높게 나타났으며**, 오류가 42회 발생(총 요청 대비 7.9%)하였습니다.

비록 `Problem List`, `Problem Detail`, `RDB Leaderboard` API의 평균 응답 시간과 중앙값은 낮았지만, 직접적인 병목은 `Submission` API에 집중되어 있었으며, 나머지 API는 그 **간접적인 영향으로 지연된 것**으로 분석됩니다.



## Submit API 제외

### 유저 5,000명 동시 요청 시나리오

* 문제 리스트 조회 API
* 문제 조회 API
* 실시간 리더보드 조회 API



<details>

<summary>K6 Script</summary>

```sql
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
  scenarios: {
    burst_0m:  { executor: 'per-vu-iterations', vus: 5000, iterations: 1, startTime: '0s' },
    burst_1m:  { executor: 'per-vu-iterations', vus: 5000, iterations: 1, startTime: '60s' },
    burst_2m:  { executor: 'per-vu-iterations', vus: 5000, iterations: 1, startTime: '120s' },
  },
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
  group('RDB Leaderboard', () => {
    for (let i = 0; i < 1; i++) {
      let id  = getRandomInt(1, 10);
      let res = http.get(`${BASE_URL}/v1/contests/${id}/leaderboard`);
      logAndMetric(res, rdbTrend, `RDB Leaderboard ${id}`);
    }
  });

  group('Problem List', () => {
    for (let i = 0; i < 1; i++) {
      let start = getRandomInt(1, 40);
      let end   = start + getRandomInt(1, 10);
      let res   = http.get(`${BASE_URL}/problems?start=${start}&end=${end}`);
      logAndMetric(res, listTrend, `Problem list ${start}-${end}`);
    }
  });

  group('Problem Detail', () => {
    for (let i = 0; i < 1; i++) {
      let pid = getRandomInt(1, 50);
      let res = http.get(`${BASE_URL}/problems/${pid}`);
      logAndMetric(res, detailTrend, `Problem detail ${pid}`);
    }
  });
}  
```

</details>

각 VU는 문제 리스트 1회, 문제 상세 1회, 리더보드 1회 호출

→ **매분, 5000VU마다 총 3회 요청 = 15,000건**

→ **3분간 총 HTTP 요청 수: 15,000 x 3 = 45,000건**



### 결과 지표

* k6 실행 결과

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

전체적으로 **지연이 다소 긴 편** (2초 내외), 일부 요청은 **10초 이상**

* Problem Detail
  * 평균: 2.2초
  * 95% 응답: 3.9초
  * 최대: 5.8초
* Problem List
  * 평균: 1.9초
  * 95% 응답: 3.6초
  * 최대: 5.1초
* Leaderboard
  * 평균: 0.9초
  * 95% 응답: 2.2초
  * 최대: 14.3초

<mark style="color:yellow;">리더보드는 평균 응답은 빠르지만,</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">**최대 14초로 outlier 심합니다.**</mark>



그럼 시스템 리소스는 어떤 상황인지 보겠습니다.

<details>

<summary>Java Application</summary>

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

</details>

<details>

<summary>RDS</summary>

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

</details>

<details>

<summary>Redis, MQ</summary>

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

</details>

* RDS: CPU 및 메모리 여유 충분
* Redis/MQ: CPU 및 메모리 여유 충분
* <mark style="color:yellow;">Java App:</mark> <mark style="color:yellow;"></mark><mark style="color:yellow;">**부하 시 CPU 사용률 100% 도달**</mark>





### 결과 요약

부하 테스트는 총 15,000명의 가상 사용자(VU)가 0분, 1분, 2분에 각각 5,000명씩 동시에 접속하여 문제 리스트, 문제 상세, 리더보드 API를 각 2회씩 호출하는 방식으로 수행되었으며, 총 90,000건의 HTTP 요청이 발생했습니다.

요청 성공률은 100%로 모든 요청이 정상 처리되었으며, 평균 응답 시간은 1.7초, 95퍼센타일 응답 시간은 3.7초로 측정되었습니다. 다만, 리더보드 API에서 최대 14초의 응답 지연이 발생하여 일부 outlier가 관찰되었습니다.

서비스 내부 구성 요소별로는 다음과 같은 최적화가 적용된 상태입니다:

* **문제 상세 API**는 캐싱 처리되어 Redis에서 빠르게 조회됨
* **문제 리스트 API**는 DB 인덱스를 기반으로 효율적인 범위 쿼리를 수행
* **리더보드 API**는 Redis를 통해 실시간 랭킹 데이터를 제공함

시스템 리소스 사용 측면에서는 RDS, Redis, MQ 모두 CPU 및 메모리 자원이 여유 있는 상태를 유지했으나, **Java 애플리케이션이 배포된 EC2 인스턴스에서만 CPU 사용률이 부하 시점에 100%까지 상승**하는 현상이 관찰되었습니다. 이는 전체적인 시스템 안정성에는 문제가 없으나, **애플리케이션 레이어에서의 처리 병렬성 또는 계산 처리 한계로 인해 일부 요청 응답 시간이 증가하는 경향**이 있는 것으로 분석됩니다.



### 해결 방안

#### 1. **수평 확장 (Scale Out)**

동일한 Java 애플리케이션 서버 인스턴스를 여러 대로 늘리는 것

* AWS EC2 Auto Scaling Group, ELB 적용
* API Gateway or Load Balancer(L7) 앞단 구성
* 사용자 요청을 여러 인스턴스로 분산 → CPU 부하 완화

#### 2. **JVM 튜닝**

CPU를 많이 사용하는 Java 앱이라면 JVM 옵션도 중요

* GC 알고리즘 변경 (`UseG1GC`, `ZGC`, `Shenandoah`)
* `Xms`, `Xmx`를 동일하게 고정하여 Full GC 방지
* `XX:+AlwaysPreTouch` 등으로 메모리 미리 할당

수평적 확장이 가장 확실하고 기본적인 해결책이고, 현재 캐시/DB 병목이 없으므로 바로 확장 효과가 있을 것으로 기대합니다. 다만, 비용이 들기때문에, 내부적으로 최적화를 할 수 있는지 확인해봅니다.



### JVM (Micrometer)

Prometeus로 수집하고, Grafana로 시각화한 지표를 바탕으로 해석해보겠습니다.

#### JVM Memory

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

* Heap 메모리 여유 있음.
* Non-Heap 안정적
* 전체 메모리 안정적

#### JVM Misc

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:yellow;">**Java 프로세스 CPU 100% 도달**</mark>
* <mark style="color:yellow;">**시스템 Load 급증: Max 8.8**</mark>
* Thread 수: 동시 처리는 많지만 정상 범위
* <mark style="color:yellow;">**Runnable 스레드 수 과다: CPU를 기다리는 스레드 많음**</mark>

#### Garbage Collection

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

* Minor GC 횟수 적절
* GC Pause 시간 짧음
* <mark style="color:yellow;">**객체 생성량 많음**</mark>

#### **JVM Memory Pools (Heap)**

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

* Eden 공간에서 주기적 GC 발생
* Old Gen 여유 있음
* Survivor 영역 안정



종합적으로 봤을때 병목 원인은 JVM이 아니라 순수 애플리케이션 레벨의 CPU 연산 과부하. 그러므로, JVM 튜닝으로 얻는 이점이 많이 없습니다.

이유는 아래와 같습니다.

1. GC 알고리즘 변경X -> GC 시간 짧고 안정적
   1. G1GC에서 저지연 GC 등으로 변경하여 GC Pause 감소와 Throughput 증가를 기대&#x20;
      1. [chapter-3.-garbage-collector-and-memory-allocation-strategy-1-2.md](../../../books/digging-deep-into-jvm/chapter-3.-garbage-collector-and-memory-allocation-strategy-1-2.md "mention")
      2. [chapter-3.-garbage-collector-and-memory-allocation-strategy-2-2.md](../../../books/digging-deep-into-jvm/chapter-3.-garbage-collector-and-memory-allocation-strategy-2-2.md "mention")
   2. 하지만 GC Pause 자체가 이미 짧고 안정적
2. Heap 사이즈 조정X -> Heap/Old Gen 여유 있음
   1. -Xms, Xmx 조정하여 Full GC를 회피
      1. [chapter-2.-java-memory-area-and-memory-overflow.md](../../../books/digging-deep-into-jvm/chapter-2.-java-memory-area-and-memory-overflow.md "mention")
   2. 현재 Heap 사용량도 안정적이라서 크게 의미가 없습니다.
