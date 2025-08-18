---
description: 서브 쿼리
---

# Subquery

## 요약

* **스칼라**: 1행 1값 보장. 다중 행이면 집계로 단일화.
* **다중 행**: `IN/EXISTS`(존재), `ANY/ALL`(최소/최대 비교)
* **다중 컬럼**: `(a,b) = (subquery)`(1행) / `(a,b) IN (subquery)`(다중행)
* **EXISTS**: 큰 하위집합에 유리. 첫 일치 발견 즉시 TRUE.
* **NOT IN**: NULL 함정 → `NOT EXISTS` 권장.
* **FROM 서브쿼리**: alias 필수, 물질화/머지 전략 확인.
* **리라이트**: JOIN/윈도우 고려, `EXPLAIN`으로 계획 확인.
* 서브쿼리는 “쿼리 안의 쿼리”로, **값/목록/테이블** 형태로 상위에 제공한다.
* 위치(SELECT/FROM/WHERE/HAVING)와 의존성(상관/비상관)에 따라 **평가 시점**과 **비용**이 달라진다.
* `IN/EXISTS/ANY/ALL`의 의미를 정확히 이해하고 **리라이트**(JOIN/윈도우)로 성능을 확보한다.
* 안티패턴(`NOT IN`+NULL, 스칼라 다중 행, 무의미 ORDER BY 등)을 피한다.
* 항상 **EXPLAIN/실측**으로 확인하라. “규칙”은 데이터와 버전에 따라 달라진다.



### 서브쿼리

* **문제의 본질**: 단일 쿼리로 풀기 어려운 “두 단계 이상의 사고”를 SQL 내부에서 **원자적으로** 묶고 싶을 때.
* **JOIN**이 수평 결합(넓히기)이라면, **서브쿼리**는 쿼리 안에 쿼리를 넣어 **깊이 파고드는** 방식.
* 실무 이점: 애플리케이션↔DB 간 왕복 감소, 기준값의 **정합성** 확보(중간 변경/경합으로 인한 오차 방지), 가독성 향상.

**대표 상황**

* 전체 평균/최대/최소 등 집계값을 기준으로 필터링
* 존재 여부(Existence) 확인: “주문된 적이 있는/없는 상품”
* 두 속성(튜플) 조합 비교: `(user_id, status)`와 같은 다중 컬럼 동시 비교
* 각 행 맥락에 의존한 비교: “해당 행이 속한 그룹의 평균 이상” 등 (**상관 서브쿼리**)

***

### 1. 분류

#### 1.1 반환 형태 기준

1. **단일 행·단일 컬럼** (= _스칼라 서브쿼리_)
   1. 결과가 **정확히 하나의 값**. `=, <, >, <=, >=, <>` 등의 단일 비교 연산자와 함께 사용.
   2. **반드시 1행**이어야 하며, 2행 이상이면 오류 (예: MySQL `Error 1242: Subquery returns more than 1 row`).
2. **다중 행·단일 컬럼**
   1. 결과가 **여러 값(목록)**. `IN, ANY/SOME, ALL`과 결합.
3. **다중 컬럼(튜플)**
   1. 결과가 **(c1, c2, ...)** 형태. 메인쿼리에서 **튜플 비교** `(a, b) IN (SELECT x, y ...)` 또는 `(a, b) = (SELECT x, y ...)`.
   2. `=`와 함께 쓰려면 **반드시 1행**만 반환해야 함.
   3. 여러 행이라면 `IN`으로 전환.

#### 1.2 외부 쿼리 의존성 기준

1. **비상관 서브쿼리(Non-correlated)**: 독립 실행 가능. 먼저 1회 실행→결과를 상위에서 재사용.
2. **상관 서브쿼리(Correlated)**: 서브쿼리 내부가 외부 컬럼을 참조(예: `p2.category = p1.category`). **외부의 각 행마다 반복 실행**.

***

### 2. 위치별 활용과 의미

#### 2.1 WHERE / HAVING 절

* **필터** 역할. “기준값”이나 “존재 여부”로 행을 걸러냄.
* 단일 값 비교(스칼라), 목록 비교(IN/ANY/ALL), 존재 확인(EXISTS/NOT EXISTS) 모두 여기서 빈번.

**예시 — 평균보다 비싼 상품**

```sql
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

**예시 — 전자기기 카테고리 상품이 포함된 주문**

```sql
SELECT *
FROM orders
WHERE product_id IN (
  SELECT product_id FROM products WHERE category = '전자기기'
);
```

**예시 — 한 번이라도 주문된 상품만** (EXISTS, 상관)

```sql
SELECT p.product_id, p.name, p.price
FROM products p
WHERE EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.product_id = p.product_id
);
```

**예시 — 한 번도 주문되지 않은 상품** (NOT EXISTS, 안티세미조인)

```sql
SELECT p.product_id, p.name, p.price, p.stock_quantity
FROM products p
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.product_id = p.product_id
);
```

> **IN vs EXISTS**
>
> * `IN`: 서브쿼리 결과 **목록**을 만들고 비교. 목록이 아주 크면 부담.
> * `EXISTS`: 각 외부 행마다 **“첫 일치 발견 즉시 TRUE”**. 큰 테이블에 유리.

#### 2.2 SELECT 절 (스칼라 서브쿼리)

* **계산된 컬럼**을 만들 때. 단, **반드시 단일 값** 반환.

**예시 — 모든 행에 전체 평균 열 추가(비상관)**

```sql
SELECT name, price,
       (SELECT AVG(price) FROM products) AS avg_price
FROM products;
```

**예시 — 각 상품별 주문 수(상관)**

```sql
SELECT p.product_id, p.name, p.price,
       (SELECT COUNT(*) FROM orders o WHERE o.product_id = p.product_id) AS order_count
FROM products p;
```

> **주의**: 상관 스칼라 서브쿼리는 각 행마다 실행되므로 **N+1** 형태가 된다. 수십만 행이면 비용 급증. 가능하면 **JOIN + GROUP BY**로 대체 고려.

#### 2.3 FROM 절 (파생 테이블, 인라인 뷰)

* 서브쿼리 결과를 **테이블처럼** 다룸. 반드시 **별칭(alias)** 필요.
* 전처리/집계를 먼저 만든 후 상위에서 필터/조인.

**예시 — 부서별 평균 급여>5000 부서**

```sql
SELECT t.dept_id, t.avg_sal
FROM (
  SELECT dept_id, AVG(salary) AS avg_sal
  FROM employees
  GROUP BY dept_id
) AS t
WHERE t.avg_sal > 5000;
```

> **MySQL**: 상황에 따라 파생 테이블을 **물리적 임시 테이블로 물질화**하거나, **머지**할 수 있다. 인덱스 부재/중복 계산에 주의. 항상 `EXPLAIN`으로 확인.

#### 2.4 HAVING 절

* 그룹 집계 이후의 **그룹 필터**에 사용.

```sql
SELECT dept_id, AVG(salary) AS avg_sal
FROM employees
GROUP BY dept_id
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```

***

### 3. 연산자 의미 — IN / ANY(SOME) / ALL / EXISTS

#### 3.1 IN / NOT IN

* “목록 안에 **포함**/미포함” 여부.
* **주의: `NOT IN`과 NULL**
  * 서브쿼리 결과에 **NULL이 하나라도** 섞이면, 3치 논리(UN`K`NOWN)로 인해 **전부 FALSE**가 되어 빈 결과가 되는 사고가 잦다.
  * 해결: `NOT EXISTS`로 전환하거나, 서브쿼리에서 `WHERE col IS NOT NULL`로 NULL을 제거.

#### 3.2 ANY(SOME) / ALL

* 비교 연산자와 결합해 **목록의 최소/최대와의 비교**로 해석 가능.
  * `> ANY (list)` ≡ `> MIN(list)`
  * `> ALL (list)` ≡ `> MAX(list)`
* 가독성 관점에선 `MIN/MAX` 기반 스칼라 서브쿼리가 더 선호됨.

#### 3.3 EXISTS / NOT EXISTS

* 결과 **존재성**만 판단. 값은 무시(慣: `SELECT 1`).
* 상관 서브쿼리와 찰떡궁합. **세미조인/안티조인**으로 옵티마이저가 변환 가능.

***

### 4. 상관 서브쿼리 깊게 보기

#### 4.1 동작 메커니즘

1. 외부 쿼리가 한 행을 읽음 → 2) 해당 행의 값을 서브쿼리에 **주입** → 3) 서브쿼리 실행 및 TRUE/FALSE 산출 → 4) 반복.

#### 4.2 대표 예시 — 카테고리 평균 이상인 상품

```sql
SELECT p1.product_id, p1.name, p1.category, p1.price
FROM products p1
WHERE p1.price >= (
  SELECT AVG(p2.price)
  FROM products p2
  WHERE p2.category = p1.category
);
```

* 각 행의 `category`가 달라서 **매번 다른 집계**와 비교해야 함 → 상관 필요.
* 결과: 각 카테고리별 평균 이상인 상품만.

#### 4.3 EXISTS 패턴 — “적어도 하나”

* “주문이 하나라도 있는 상품” / “팔린 적이 없는 상품” 같은 질의에 최적.
* 대규모 테이블에서 **첫 일치 발견 즉시 중단**되어 유리.

#### 4.4 성능 리라이팅

* 상관 서브쿼리는 종종 **JOIN + GROUP BY/QUALIFY**로 치환 가능.
* 예: “각 카테고리 평균 이상”은 윈도우 `AVG() OVER (PARTITION BY category)`로 1패스 계산.

```sql
SELECT product_id, name, category, price
FROM (
  SELECT p.*, AVG(price) OVER (PARTITION BY category) AS avg_in_cat
  FROM products p
) t
WHERE price >= avg_in_cat;
```

> 윈도우 함수를 지원하지 않는 DB라면 원문의 상관 서브쿼리/SELF JOIN을 사용.

***

### 5. 다중 컬럼(튜플) 서브쿼리

#### 5.1 동시 비교의 필요

* “`user_id`와 `status`가 모두 같은 주문” / “(user\_id, order\_date) 조합이 각 사용자 **최초 주문**과 일치” 등.

**예시 — (user\_id, status) 1:1 비교 (단일 행)**

```sql
SELECT order_id, user_id, status, order_date
FROM orders
WHERE (user_id, status) = (
  SELECT user_id, status
  FROM orders
  WHERE order_id = 3  -- PK → 1행 보장
);
```

**예시 — (user\_id, order\_date) 다중 행 비교(IN)**

```sql
SELECT o.order_id, o.user_id, o.order_date
FROM orders o
WHERE (o.user_id, o.order_date) IN (
  SELECT user_id, MIN(order_date)
  FROM orders
  GROUP BY user_id
);
```

**확장 — 상세 조인**

```sql
SELECT o.order_id, o.user_id, u.name, p.name AS product_name, o.order_date
FROM orders o
JOIN users u ON u.user_id = o.user_id
JOIN products p ON p.product_id = o.product_id
WHERE (o.user_id, o.order_date) IN (
  SELECT user_id, MIN(order_date)
  FROM orders
  GROUP BY user_id
);
```

> **제약**: `=` 사용 시 서브쿼리는 반드시 1행. 여러 행이면 `IN`으로 전환.

***

### 6. SELECT/FROM/WHERE/HAVING 별 패턴 모음

#### 6.1 SELECT (스칼라)

* 전역 상수성 값(전체 평균 등): 비상관으로 1회 실행 후 재사용.
* 행별 파생 값(개별 주문수 등): 상관으로 계산하되, 대량이면 **JOIN/집계**로 치환.

#### 6.2 FROM (파생 테이블)

* 복잡한 집계를 먼저 만들어 상위에서 필터/조인.
* 인덱스 부재/재계산 비용 → 필요시 **CTE**로 분리해 가독성/재사용성 확보.

#### 6.3 WHERE / HAVING

* **IN/EXISTS/ANY/ALL**의 의미적 차이를 정확히 이해하고 사용.
* `NOT IN`+NULL 함정 회피.

***

### 7. 실행 흐름 & 옵티마이저 변환

#### 7.1 논리적 처리 순서

`FROM → ON → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`

* 서브쿼리 위치에 따라 **언제 평가되는지**가 달라짐.
* OUTER JOIN과 함께 쓸 때는 **ON vs WHERE** 위치 차이로 결과가 달라질 수 있음(외부행 NULL 보존 여부).

#### 7.2 대표적 변환들(개념)

* `IN` → **세미조인**
* `NOT IN` → **안티조인**(단, NULL 처리 이슈 시 변환 제한)
* `EXISTS`/`NOT EXISTS` → (안티)세미조인
* 상관 서브쿼리 → **블록 네스티드 루프** / **인덱스** 활용 탐색
* 파생 테이블 → **물질화** 또는 **머지**

> 항상 `EXPLAIN`으로 실제 계획 확인. DB/버전에 따라 상이.

***

### 8. 성능 최적화 체크리스트

1. **인덱스**: 상관 서브쿼리 조인열(`o.product_id = p.product_id`) 반드시 인덱스.
2. **리라이트**: 상관 → JOIN으로 바꾸기.
3. **`EXISTS` vs `IN`**: 큰 하위 집합에는 `EXISTS` 선호, 소수/작은 목록엔 `IN` 무난.
4. **`NOT IN` 회피**: NULL 섞이면 전부 탈락. `NOT EXISTS`로 전환.
5. **스칼라 1행 보장**: `=` 비교 시 PK/UNIQUE 조건으로 1행 보장하거나 집계(`MAX/MIN`)로 단일화.
6. **ORDER BY/LIMIT in Subquery**: IN/EXISTS 문맥에서 내부 `ORDER BY`는 **의미 없음**(존재/포함만 판단). LIMIT은 결과 크기 제한 용도로만 의미.
7. **파생 테이블 재사용**: 동일 서브쿼리를 여러번 쓰면 비용 중복. CTE/파생테이블로 묶어 1회 계산 유도(엔진별 최적화 상이).

***

### 9. 안티패턴

* **`NOT IN` + NULL**: 의도와 달리 결과 **빈 집합**(3치 논리). `NOT EXISTS`로 치환.
* **무의미한 ORDER BY**: IN/EXISTS 문맥에서 내부 정렬은 비용만 증가.
* **SELECT-상관 서브쿼리 남발**: N+1 폭증. JOIN/윈도우 고려.
* **파생 테이블 alias 누락**: SQL 표준 및 MySQL 모두 **별칭 필수**.

***

### 10. 조인으로의 리라이트(요리책)

#### 10.1 각 상품별 주문 수 (SELECT-상관 → JOIN 집계)

```sql
SELECT p.product_id, p.name, p.price, COALESCE(cnt.order_count, 0) AS order_count
FROM products p
LEFT JOIN (
  SELECT product_id, COUNT(*) AS order_count
  FROM orders
  GROUP BY product_id
) cnt ON cnt.product_id = p.product_id;
```

#### 10.2 카테고리 평균 이상 (상관 → 윈도우)

```sql
SELECT product_id, name, category, price
FROM (
  SELECT p.*, AVG(price) OVER (PARTITION BY category) AS avg_in_cat
  FROM products p
) t
WHERE price >= avg_in_cat;
```

#### 10.3 “주문된 적이 없는 상품” (NOT EXISTS → LEFT JOIN IS NULL)

```sql
SELECT p.product_id, p.name, p.price
FROM products p
LEFT JOIN orders o ON o.product_id = p.product_id
WHERE o.product_id IS NULL;
```

> 데이터 정합성/NULL 의미 차로 결과가 다를 수도 있다(특히 조인 조건/중복에 유의). 두 방식의 **동치성**은 전제 조건에 따라 달라진다.

***

### 11. 실무 패턴

* **세미조인(ANY 존재)**: `EXISTS` / `IN`(소수) / `JOIN DISTINCT`
* **안티조인(부재)**: `NOT EXISTS` / `LEFT JOIN ... IS NULL`
* **그룹 내 첫/마지막 행**: 튜플 IN + `MIN/MAX(order_date)` / 윈도우 `ROW_NUMBER()`
* **값 기준 동적 필터**: 스칼라 `(SELECT MAX(...) ...)`
* **상태 동시 비교**: 다중 컬럼 튜플 비교 `(a,b) IN (...)`

***

### 12. 질문

* **Q. 스칼라 서브쿼리는 왜 1행만?**
  * 단일 비교/표현식 자리에 **하나의 값**만 들어갈 수 있기 때문. 2행이 되면 비교가 정의되지 않는다.
* **Q. ANY/ALL은 언제 쓰나?**
  * 의미상 최소/최대와 비교하는 문제. 가독성 위해 `MIN/MAX`가 더 선호.
* **Q. EXISTS 안에서 `SELECT 1`을 쓰는 이유?**
  * 값 자체는 무의미. **존재성**만 체크하기 때문(최적화 힌트 성격은 아님).
* **Q. 파생 테이블과 CTE 차이?**
  * 기능은 유사. CTE는 이름부여/가독성/재사용 용이. 엔진/버전에 따른 **머지/물질화 전략**은 다르다.

***
