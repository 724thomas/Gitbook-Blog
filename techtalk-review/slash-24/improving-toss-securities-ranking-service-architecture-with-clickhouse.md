---
description: "15주차:\_ClickHouse로 토스증권 랭킹 서비스 구조 개선하기"
---

# Improving Toss Securities Ranking Service Architecture with ClickHouse

## 🚀 토스증권 랭킹 서비스, ClickHouse로 구조 개선하기

### 📝 들어가며

토스증권은 주식/ETF/채권 등 다양한 종목에 대해 **실시간 랭킹 서비스**를 제공합니다.

사용자는 앱에서 "거래량 TOP10", "급상승 종목", "업종별 인기" 등을 확인할 수 있고, 이 기능은 사용자 경험에 큰 영향을 미칩니다.

하지만 기존 구조는 **분 단위 배치 처리**, **복잡한 파이프라인**, **DW(데이터 웨어하우스)와 리소스 충돌** 때문에 한계에 부딪혔습니다.

이 글에서는 토스증권 데이터 엔지니어링 팀이 **ClickHouse를 도입해 실시간 아키텍처를 구축한 과정**을 정리합니다.

***

### 🚨 문제 상황: 실시간성 요구 증가

* 기존 업데이트 주기: **1분 단위**
* 집계 시간: **평균 10초 이상**
* 사용자 니즈: **초 단위 실시간 랭킹 반영**

즉, **배치 기반 DW 파이프라인** 구조에서는 한계가 뚜렷했습니다.

특히 거래량이 폭증하는 장중에는 **랭킹 반영이 늦어지거나 실패**하는 사례도 발생했습니다.

👉 정리:

* **병목 원인** = 배치 잡 + DW 쿼리 경합
* **결과** = 초단위 업데이트 불가, 운영/개발 난이도 상승

***

### 🔍 원인 분석: 기존 아키텍처의 구조적 한계

실시간 이벤트(체결·시세·클라이언트 로그)를 \*\*데이터 레이크(HDFS/S3)\*\*에 쌓고, \*\*DW(Impala/Presto)\*\*로 **주기적 집계**를 돌린 뒤, 결과를 **서비스용 저장소**(RDB/캐시)에 밀어 넣어 앱이 읽게 하는 구조.

#### 1. Airflow 기반 잡 확장성 한계

랭킹별로 Airflow DAG(Job)이 선형적으로 늘어났습니다.

옵션이 추가될수록 잡이 기하급수적으로 늘어나 관리 난이도가 폭발.

#### 2. DW 자원 경합

* 분석가들이 DW에 무거운 쿼리를 날림 → 랭킹 집계 지연/실패
* **서비스와 분석 리소스를 분리하지 못한 구조**

#### 3. 데이터 처리 지연

* **로우 데이터 → DW 적재 → 집계 → 결과 추출**
* 전체 파이프라인이 무겁고 지연에 취약

***

### 🏗️ 새로운 접근: 실시간 데이터 플랫폼 필요

토스증권 팀은 “**랭킹에 필요한 데이터는 한곳에 모으고, 실시간 집계가 가능한 플랫폼이 필요하다**”는 결론을 냈습니다.

👉 후보 기술을 리서치한 결과, **ClickHouse**가 선택되었습니다.

***

## 🏗️ ClickHouse란?

### 📌 정의

**ClickHouse**는 \*\*오픈소스 컬럼 지향(column-oriented) 데이터베이스 관리 시스템(DBMS)\*\*이에요.

특히 \*\*대규모 데이터의 집계/분석(OLAP)\*\*에 특화된 데이터베이스입니다.

***

### 📊 OLAP vs OLTP (왜 ClickHouse가 필요한가?)

* **OLTP (Online Transaction Processing)**
  * 은행 계좌 이체, 주문 처리 같은 트랜잭션 처리용 DB (예: MySQL, PostgreSQL)
  * 초당 수천\~수만 건의 "작은" 트랜잭션 처리에 강점
  * 데이터는 보통 `행(row)` 단위 저장
* **OLAP (Online Analytical Processing)**
  * 대규모 데이터 집계·분석 (예: “지난 1년간 매출 TOP 10 상품”)
  * 수억 건 데이터를 빠르게 SUM, COUNT, GROUP BY 해야 함
  * 데이터는 `열(column)` 단위 저장 → 집계 시 필요한 컬럼만 읽으므로 빠름

👉 ClickHouse = **OLAP 전문 DB**

즉, 대규모 데이터를 **빠르게 집계하고 실시간 분석**하는 데 강합니다.

***

### ⚡ ClickHouse의 주요 특징

#### 1. 컬럼 지향 저장

* 보통 RDB(MySQL)는 **행 단위** 저장 → 한 레코드 전체가 디스크에 저장
* ClickHouse는 **열 단위** 저장 → 특정 컬럼만 빠르게 읽을 수 있음
*   예:

    ```sql
    SELECT SUM(volume) FROM trades;

    ```

    → `volume` 컬럼만 읽으면 되므로 수십 배 빠름

***

#### 2. MergeTree 엔진

* ClickHouse의 기본 테이블 엔진
* 데이터를 파티션 단위로 쌓고, 백그라운드에서 자동 병합(merge)
* 대량 데이터(수십억 row)를 효율적으로 저장하고 조회 가능
* 파생 엔진:
  * `ReplacingMergeTree`: 중복 제거 + 최신 레코드 유지
  * `AggregatingMergeTree`: 집계 함수 결과를 미리 저장
  * `SummingMergeTree`: 합계 미리 저장

👉 **랭킹 서비스 최적화 포인트**: “최신 데이터만 필요” or “중간 집계 필요”에 맞는 엔진 선택 가능

***

#### 3. Materialized View

* **실시간 ETL(Extract-Transform-Load) 지원**
* Kafka에서 들어오는 데이터를 받아, 필터링/집계/조인을 한 뒤 다른 테이블에 저장 가능
* 장점:
  * Airflow 잡 같은 배치 불필요
  * 데이터 인입 순간 실시간 반영

***

#### 4. 초고속 집계 성능

* 페이스북, Uber, Yandex, Cloudflare 등 대규모 트래픽 기업들이 사용
* 수십억 건 데이터도 수백 ms 내에 집계 가능

***

### ⚡ ClickHouse가 선택된 이유

* **OLAP(Online Analytical Processing)** 컬럼 기반 DB
* **빠른 집계 성능** (대용량 데이터 집계에 최적화)
* **단순한 운영 구조** (복잡한 Hadoop/Hive 대비 가벼움)
* **특화 기능**:
  * 다양한 **Table Engine** (MergeTree, AggregatingMergeTree, ReplacingMergeTree 등)
  * **Materialized View** (실시간 ETL/집계 가능)

***

### 📡 실시간 데이터 수집: Kafka Engine vs Sink Connector

* **Kafka Engine**: ClickHouse 안에 `ENGINE=Kafka` 테이블을 만들어 **CH 프로세스가 직접** 토픽을 consume. 보통 **Materialized View(MV)** 로 변환·적재를 트리거해 **타겟 MergeTree 테이블**에 넣는 구조.
* **Sink Connector**: **Kafka Connect**(외부 워커)가 토픽을 consume 해서 **바로 ClickHouse MergeTree** 테이블에 INSERT. 변환/스키마/에러 처리/재시도 등을 **커넥터 레이어**에서 담당.
* 전자: Kafka (topic) → \[ENGINE=Kafka 테이블] → (MV 트리거) → MergeTree 타겟
* 후자: Kafka (topic) → Kafka Connect (Sink Connector) → ClickHouse MergeTree

#### Kafka Engine

* ClickHouse 자체에서 Kafka 토픽을 직접 consume 가능
* 장점: 외부 의존성 없음
* 단점:
  * ClickHouse 서버와 Kafka consumer 리소스 공유 → 성능 불리
  * 테이블 세트가 많아 관리 복잡

#### Clickhouse Sink Connector

* Kafka Connect 기반의 **ClickHouse Sink Connector** 활용
* 독립적으로 관리 가능, 리소스 분리
* 운영 편의성 ↑, 확장성 ↑

👉 결론: **Kafka Engine 대신 Sink Connector를 도입**

***

### 🔄 CDC(Change Data Capture) 도입

**CDC** = 원본 DB(MySQL/PG)에서 일어나는 **Insert/Update/Delete**를 binlog/Logical decoding으로 캡처해 다운스트림(ClickHouse 등)으로 흘려보내는 기술.

* **Append-only**: 기존 행을 직접 고치지 않고, **같은 키**(예: `symbol`)에 대해 **새 버전 행**을 한 줄 더 INSERT 합니다.
* 엔진이 `ReplacingMergeTree(version)`이면, **논리적으로 최신 버전만** 보이도록 설계합니다

랭킹에는 **메타데이터 변경 반영**도 필요합니다.

예:

* 종목 상장/상폐
* 관심 종목 설정/해제

ClickHouse는 **MaterializedMySQL Engine**을 통해 CDC 기능을 제공합니다.

* MySQL binlog를 읽어 ClickHouse에 반영
* 테이블 단위 복제 가능

단점: 아직 일부 기능 제약 존재(실험적 기능) → Sink Connector와 혼합 사용

***

### 🗂️ ReplacingMergeTree로 최신 데이터만 유지

특히 **급상승/급하락 랭킹**은 “가장 최근 가격”만 필요합니다.

* 기존: 중복된 체결 데이터 다 저장 → 집계 시 리소스 낭비
* 개선: `ReplacingMergeTree` 엔진 사용
  * Key + Version 컬럼 지정
  * 최신 데이터만 유지 → 디스크 공간 절약 + 빠른 집계

***

### 🧮 AggregatingMergeTree로 집계 최적화

**인기 랭킹**은 클라이언트 로그 기반인데, 시간당 **1억 건 이상** 발생합니다.

* 로우 데이터 직접 집계: 수백 ms \~ 1초 소요
* 해결책:
  * **중간 집계 테이블** 생성
  * `AggregatingMergeTree` + `Materialized View` 사용
  * 초/분/시간 단위로 미리 집계 저장
  * 최종 쿼리 시 로우 데이터를 읽을 필요 없음

👉 성능 개선:

* 기존: 600\~700ms
* 개선: 200\~300ms

***

### 🔑 핵심 아키텍처 설계

1. **데이터 인입**
   * Kafka → ClickHouse (Sink Connector)
   * MySQL CDC → ClickHouse (MaterializedMySQL Engine)
2. **실시간 집계**
   * Materialized View로 필터링/조인/집계
   * ReplacingMergeTree / AggregatingMergeTree 엔진 활용
3. **결과 제공**
   * 애플리케이션에서 초단위 랭킹 쿼리
   * 캐시 계층 병행 활용

***

### 📊 성능 검증

* 테스트 토픽: 초당 70만 건 데이터 유입
* 클러스터: 서버 5대 구성
* 결과:
  * **Kafka → ClickHouse 인입: 초당 60만 건 안정 처리**
  * **집계 쿼리 응답 속도: 평균 200\~300ms**
  * **장중 트래픽 폭증에도 안정적 동작**

***

### 🛡️ 운영 측면 개선

*   **분석 파이프라인과 서비스 파이프라인 분리**

    → 분석 쿼리 경합 제거
*   **Airflow DAG 폭발 문제 해소**

    → 옵션별 잡 생성 불필요, Materialized View 자동 처리
*   **운영 단순화**

    → 관리 포인트 감소, 장애 원인 추적 용이

***

### 🎯 교훈과 인사이트

1.  **배치 기반 DW 한계 극복**

    → 실시간 서비스에는 별도의 플랫폼 필요
2. **Table Engine 전략적 활용**
   * ReplacingMergeTree: 키값으로 최신값 유지
   * AggregatingMergeTree: 컬럼별로 중간 집계
   * MergeTree: 대량 인서트/쿼리 처리
3. **실시간성과 운영 편의성의 균형**
   * Kafka Engine 대신 Sink Connector 선택
4. **분석 vs 서비스 자원 분리**
   * 서비스 안정성을 위해 필수

***

### 🧑‍💻 개발자를 위한 용어 주석

* **DW(Data Warehouse)**: 기업의 데이터 분석을 위한 중앙 저장소
* **Airflow DAG**: 워크플로우를 정의하는 그래프 구조 (잡 실행 순서 정의)
* **OLAP**: 대용량 데이터를 다차원으로 분석하는 방식
* **CDC**: 원본 DB의 변경을 실시간으로 다른 시스템에 반영하는 기법
* **Materialized View**: 쿼리 결과를 미리 저장해두는 뷰 (실시간 ETL 가능)
* **MergeTree**: ClickHouse의 기본 스토리지 엔진, 대량 데이터 처리에 최적화

***

### 🔮 앞으로의 확장 가능성

* **다른 서비스(알림/추천)에도 ClickHouse 적용 가능**
* **타임시리즈 분석** (분 단위 시세 변동 등) 최적화
* **서빙용 + 분석용 ClickHouse 이중화**

***

### ✅ 마무리

토스증권은 **ClickHouse 기반 실시간 데이터 플랫폼**으로 전환하면서:

* 랭킹 집계 속도: **10초 → 200ms**
* 운영 안정성: **분석/서비스 분리**
* 개발 생산성: **잡 관리 단순화**

👉 교훈: **실시간 서비스는 “분석 중심 DW”가 아니라 “서비스 중심 OLAP DB” 위에서 설계해야 한다.**

***

## 배운점

* ClickHouse는 append-only 구조에 다양한 엔진(Replacing, Aggregating)을 조합해 업데이트/집계 문제를 효율적으로 해결할 수 있다는 걸 배웠습니다.
* Kafka Engine과 Sink Connector는 각각 장단이 있지만, 대규모 실서비스에서는 Sink Connector가 더 안정적이라는 점이 인상적이었습니다.
* Redis는 단독으론 지연 이벤트·가변 윈도우 처리에 한계가 있어, ClickHouse 결과를 캐싱하는 보완재로 쓰는 게 적합하다는것을 배웠습니다.
* 결국 핵심 교훈은 “데이터는 쉽게 쌓고, 읽기·집계 최적화는 DB와 캐시를 적절히 결합해 푼다”는 설계 철학이라는 것을 배웠습니다.
