---
description: k6 부하테스트 요구사항 및 초기 테스트
---

# K6 load test

* Submission API를 제외한, 나머지 API에 대한 부하테스트[load-test-apis.md](load-test-apis.md "mention")
* Submission API 병목 원인 분석 및 구조 개선 [load-test-submission-api.md](load-test-submission-api.md "mention")

***

### 비기능 요구사항

* 고가용성: 웹사이트는 24: 웹사이트는 24시간 365일 항상 접근 가능해야 함
* 고확장성: 최대 10,000명의 동시 제출 처리를 지원해야 함
* 저지연: 대회 중 리더보드는 실시간으로 갱신되어야 함
* 보안: 사용자가 제출한 코드가 서버를 손상시키거나 침해하지 않도록 방지해야 함

### 확장성 요구사항

* **최대 동시 사용자 수**: 최대 10,000명이 동시에 대회에 참가
* **대회 수**: 하루에 최대 1회
* **대회 시간**: 2시간
* **사용자당 평균 제출 횟수**: 20회
* **제출당 평균 테스트 케이스 수**: 20개
* **소스코드 보존 기간**: 제출된 코드는 1개월간 저장된 후 삭제
* **소스코드 크기**: 각 제출은 약 10KB 저장 공간 필요
* **읽기:쓰기 비율**: 약 2:1



### 클라우드 인프라(초기)

부하 테스트를 위한 기초 클라우드 환경입니다.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

* K6
  * 성능 부하 테스트를 수행.&#x20;
  * HTTP 요청을 다수의 Virtual User(VU)로 시뮬레이션하여 실제 트래픽을 유발
* Spring Backend&#x20;
  * Java Spring 기반 백엔드 서버
* Spring Monitor
  * Prometeus: Spring Actuator에서 주기적으로 메트릭을 받아서 저장
  * Grafana: Prometeus에 저장된 메트릭을 대시보트 형태로 시각화
* redis\_and\_mq
  * Redis: 인메모리 데이터베이스
  * RabbitMq: 메시징 큐
* LB
  * Elastic Load Balancer
  * 비용 이슈로 사용하지 않습니다. k6에서 스크립트로 로드밸런싱
* RDS: 데이터 저장을 위한 데이터베이스





### 초기 테스트

본격적인 대규모 부하 테스트에 앞서, 간단한 실험을 통해 **최대 70 VU(가상 사용자)** 기준으로 시스템의 병목 구간을 사전 파악하고자 하였습니다.&#x20;

<table data-full-width="true"><thead><tr><th width="244.3636474609375">메트릭</th><th>평균 (avg)</th><th>중앙값 (med)</th><th>p(90)</th><th>p(95)</th><th>최대 (max)</th></tr></thead><tbody><tr><td>http_req_duration</td><td>3.52 s</td><td>25.28 ms</td><td>29.58 s</td><td>30.00 s</td><td>30.13 s</td></tr><tr><td>rdb_duration_ms (Trend)</td><td>4.49 s</td><td>77.07 ms</td><td>29.90 s</td><td>30.01 s</td><td>30.01 s</td></tr><tr><td>list_duration_ms (Trend)</td><td>2.92 s</td><td>19.02 ms</td><td>0.31 s</td><td>30.01 s</td><td>30.02 s</td></tr><tr><td>detail_duration_ms (Trend)</td><td>2.85 s</td><td>15.04 ms</td><td>0.28 s</td><td>30.01 s</td><td>30.02 s</td></tr><tr><td>submit_duration_ms (Trend)</td><td>3.17 s</td><td>1.02 s</td><td>1.05 s</td><td>30.01 s</td><td>30.13 s</td></tr><tr><td>오류율 (errors_total)</td><td>—</td><td>—</td><td>—</td><td>—</td><td>42회 (7.9%)</td></tr></tbody></table>

이번 실험에서는 문제 목록/상세 조회, 리더보드 조회, 문제 제출 API 등 주요 경로에 대해 동시에 부하를 가한 결과:

**1. 전체 요청 평균 응답시간이 높다**

* `http_req_duration` 평균이 **3.52초**입니다. 이는 상당히 높은 수치로, 대부분의 요청이 밀려 있었음을 의미합니다.

**2. p(90), p(95), max가 모두 30초대**

* 거의 모든 API(`rdb`, `list`, `detail`, `submit`)가 `p(95)`에서 **30초에 가까운 지연**을 보이고 있습니다.
* 이건 단순히 submit API만이 아니라, 전체적으로 **리소스가 고갈되어 응답이 지연되고 있는 현상**입니다.

**3. Submit만 median이 1초 이상**

* `submit_duration_ms`의 **중앙값이 1.02초**로, 다른 API보다 현저히 느림.
* `list`, `detail`, `rdb`는 median이 15\~77ms 수준이지만, `submit`은 기본적으로 느린 작업임을 나타냅니다.
* 여기서 **submit API가 병목의 원임**임 가능성이 높다고 생각하였습니다.



