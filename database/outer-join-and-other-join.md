---
description: Outer Join & 기타 Join
---

# Outer Join & other join

## 요약

* **외부 조인(OUTER JOIN)** 은 한쪽 테이블에는 데이터가 있지만, 다른 쪽에는 없는 데이터도 결과에 포함시킨다.
* 기준이 되는 테이블을 왼쪽으로 두면 `LEFT JOIN`, 오른쪽으로 두면 `RIGHT JOIN`.
* **LEFT JOIN**: 왼쪽 테이블의 모든 행을 포함하고, 오른쪽은 조건에 맞는 데이터만 붙임. 없으면 NULL.
* **RIGHT JOIN**: 오른쪽 테이블의 모든 행을 포함하고, 왼쪽은 조건에 맞는 데이터만 붙임. 없으면 NULL.
* **FULL OUTER JOIN**: 양쪽 모두의 모든 행을 포함. (MySQL은 지원하지 않음.)
* **SELF JOIN**: 같은 테이블을 스스로 조인하여 계층 구조나 관계를 표현.
* **CROSS JOIN**: 두 테이블의 모든 행 조합(Cartesian product)을 반환. 필터가 없으면 결과 행 수가 폭발적으로 증가.
* 실무에서는 `LEFT JOIN`이 압도적으로 많이 사용됨. `RIGHT JOIN`은 테이블 순서만 바꿔도 LEFT로 대체 가능.



## 1. 외부 조인 개념

내부 조인(INNER JOIN)은 양쪽에 모두 존재하는 데이터만 보여줌(교집합). 하지만 때로는 한쪽에 있는 데이터도 결과에 포함하고 싶을때가 있음.

예:

* 가입했지만 주문하지 않은 고객
* 등록했지만 한번도 팔리지 않은 재품

내부 조인으로는 이런 데이터가 아예 누락되어있으므로, 외부 조인을 사용해야 함.



## 1-1. LEFT JOIN

동작 방식

* **FROM 절 왼쪽에 있는 테이블**이 기준.
* 왼쪽의 모든 데이터를 포함하고, `ON` 조건에 맞는 오른쪽 데이터를 붙임.
* 오른쪽에 매칭이 없으면 해당 칼럼은 `NULL`.

```sql
SELECT u.user_id, u.name, o.order_id
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id;
```

* 모든 고객이 출력됨.
* 주문이 없는 고객은 `order_id`가 `NULL`.

#### 사용 예시

* 가입했지만 주문하지 않은 고객 찾기:

```sql
SELECT u.user_id, u.name
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE o.order_id IS NULL;
```



## 1-2. RIGHT JOIN

#### 동작 방식

* **JOIN 절 오른쪽에 있는 테이블**이 기준.
* 오른쪽의 모든 데이터를 포함하고, 왼쪽에서 매칭되는 데이터를 붙임.
* 매칭이 없으면 `NULL`.

```sql
SELECT o.order_id, p.name
FROM orders o
RIGHT JOIN products p ON o.product_id = p.product_id;
```

#### 특징

* LEFT JOIN과 결과는 같지만 **기준 테이블 위치만 다름**.
* 실무에서는 거의 안 쓰고, LEFT JOIN으로 변환하는 경우가 많음.



## 1-3. FULL OUTER JOIN

* 양쪽 테이블의 모든 데이터를 포함.
* 매칭이 없으면 한쪽 컬럼은 NULL.
* MySQL에서는 직접 지원하지 않음 (UNION으로 구현 가능).

```sql
SELECT ... FROM A
LEFT JOIN B ON ...
UNION
SELECT ... FROM A
RIGHT JOIN B ON ...
```



## 1-4. 논리적 개념

* **LEFT JOIN**:\
  FROM 절에 있는 테이블(왼쪽)이 기준.
  1. 왼쪽 테이블의 모든 데이터를 결과에 포함.
  2. ON 조건에 맞는 데이터를 오른쪽 테이블에서 찾아 붙임.
  3. 매칭이 없으면 NULL 채움.
* **RIGHT JOIN**:\
  JOIN 절에 있는 테이블(오른쪽)이 기준.
  1. 오른쪽 테이블의 모든 데이터를 결과에 포함.
  2. ON 조건에 맞는 데이터를 왼쪽 테이블에서 찾아 붙임.
  3. 매칭이 없으면 NULL 채움.
* **FULL OUTER JOIN**: 양쪽 모두의 데이터를 포함. 교집합 + 각 집합의 단독 데이터. (MySQL은 지원하지 않음)



## 1-5. 물리적 개념

* 내부 조인과 동일하게 Nested Loop / Hash Join / Merge Join 방식 사용 가능.
* 차이점은 **조인 후 필터링 단계에서 NULL 값을 허용**하는지 여부.
* 예: LEFT JOIN에서는 왼쪽 테이블이 외부 루프, 오른쪽 테이블이 내부 루프 역할을 하며 NULL 채우기를 수행.



## 1-6. 실행 흐름 예시

**`LEFT JOIN`으로 한 번도 주문하지 않은 고객 찾기**

1. 왼쪽 테이블(users) 전 행 포함.
2. orders에서 user\_id 일치 데이터 검색.
3. 매칭이 없으면 NULL.
4. `WHERE o.order_id IS NULL`로 필터링.

**`LEFT JOIN`으로 한 번도 팔리지 않은 상품 찾기**

* 기준: products
* LEFT JOIN → 주문 없는 상품의 주문 컬럼은 NULL → WHERE 조건으로 필터.

**`RIGHT JOIN` 동일 예시**

* 테이블 순서 바꾸고 RIGHT JOIN으로도 같은 결과 가능.
* 실무에서는 LEFT JOIN이 가독성이 좋아 더 많이 사용됨.



## 2. SELF JOIN

#### 개념

* 동일한 테이블을 자기 자신과 조인.
* 보통 계층 구조(트리)나 상하 관계 표현에 사용.
* 한 테이블을 서로 다른 별칭(A, B)로 두어 부모-자식 관계 연결.

**예시**

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

* 직원 테이블을 두 번 사용.
* 직원의 상사 이름 조회 가능.



## 3. CROSS JOIN

#### 개념

* 두 테이블의 모든 행을 조합 (데카르트 곱).
* ON 조건 없이 실행.
* m × n 개의 결과 생성.
* 상품 옵션 조합, 테스트 데이터 생성 등에 사용.

**예시**

```sql
SELECT s.size, c.color
FROM sizes s
CROSS JOIN colors c;
```

* 모든 사이즈와 색상 조합 반환.



## 4. 조인 시 행 개수 변화 특징

* **자식 → 부모 조인 (FK → PK)**: 행 수 그대로.
* **부모 → 자식 조인 (PK → FK)**: 자식의 수만큼 행 증가.
* **SELF JOIN**: 매칭 수에 따라 증가 가능.
* **CROSS JOIN**: 항상 곱셈 만큼 행 수 폭증.



## 궁금한 점

### **Q. LEFT JOIN과 RIGHT JOIN은 단순히 방향만 다른가?**

A. 옵티마이저가 조인 순서를 자동으로 바꾸기도 하지만, 최선의 선택은 아님. 여러 테이블을 조인할 때는 초기에 건수를 가장 많이 줄일 수 있는 테이블을 먼저 드라이빙 테이블로 잡는게 좋다.

<details>

<summary>Dummy Data</summary>

```sql
-- === DB & 세션 설정 ===
CREATE DATABASE IF NOT EXISTS playground;
USE playground;

SET autocommit = 0;
SET unique_checks = 0;
SET foreign_key_checks = 0;

SET SESSION net_read_timeout  = 600;
SET SESSION net_write_timeout = 600;

-- === 기존 테이블 삭제 ===
DROP TABLE IF EXISTS test_orders;
DROP TABLE IF EXISTS test_products;
DROP TABLE IF EXISTS test_users;

-- === 테이블 생성 ===
CREATE TABLE test_users (
  user_id     BIGINT AUTO_INCREMENT PRIMARY KEY,
  name        VARCHAR(255) NOT NULL,
  email       VARCHAR(255) NOT NULL UNIQUE,
  created_at  DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

CREATE TABLE test_products (
  product_id     BIGINT AUTO_INCREMENT PRIMARY KEY,
  name           VARCHAR(255) NOT NULL,
  category       VARCHAR(100) NOT NULL,
  price          INT          NOT NULL,
  stock_quantity INT          NOT NULL
) ENGINE=InnoDB;

CREATE TABLE test_orders (
  order_id   BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id    BIGINT NOT NULL,
  product_id BIGINT NOT NULL,
  order_date DATETIME NOT NULL,
  quantity   INT      NOT NULL,
  status     VARCHAR(20) NOT NULL,
  KEY idx_orders_user          (user_id),
  KEY idx_orders_product       (product_id),
  KEY idx_orders_status_date   (status, order_date),
  CONSTRAINT fk_orders_user    FOREIGN KEY (user_id)    REFERENCES test_users(user_id),
  CONSTRAINT fk_orders_product FOREIGN KEY (product_id) REFERENCES test_products(product_id)
) ENGINE=InnoDB;

-- === 시퀀스 테이블 ===
DROP TABLE IF EXISTS seq_10;
CREATE TABLE seq_10 (
  n TINYINT UNSIGNED NOT NULL,
  PRIMARY KEY (n)
) ENGINE=InnoDB;

INSERT INTO seq_10(n) VALUES (0),(1),(2),(3),(4),(5),(6),(7),(8),(9);

-- 1..100,000 (10^5) 생성
DROP TABLE IF EXISTS seq_1e5;
CREATE TABLE seq_1e5 (
  id INT UNSIGNED NOT NULL,
  PRIMARY KEY (id)
) ENGINE=InnoDB;

INSERT /*+ SET_VAR(max_execution_time=0) */ INTO seq_1e5 (id)
SELECT a.n + b.n*10 + c.n*100 + d.n*1000 + e.n*10000 + 1 AS id
FROM seq_10 a
CROSS JOIN seq_10 b
CROSS JOIN seq_10 c
CROSS JOIN seq_10 d
CROSS JOIN seq_10 e;

-- === 데이터 적재 ===

-- 사용자 50,000건
INSERT INTO test_users (name, email, created_at)
SELECT
  CONCAT('User ', id)                        AS name,
  CONCAT('user', id, '@example.com')         AS email,
  NOW() - INTERVAL (id MOD 3650) DAY         AS created_at
FROM seq_1e5
WHERE id <= 50000;
COMMIT;

-- 상품 5,000건
INSERT INTO test_products (name, category, price, stock_quantity)
SELECT
  CONCAT('Product ', id)                                                   AS name,
  ELT(1 + (id MOD 5), 'electronics','books','fashion','home','sports')     AS category,
  1000 + (id MOD 9000)                                                     AS price,
  10 + (id MOD 90)                                                         AS stock_quantity
FROM seq_1e5
WHERE id <= 5000;
COMMIT;

-- 주문 500,000건 (1,000행 × 500회)
DELIMITER //

DROP PROCEDURE IF EXISTS load_orders_big //
CREATE PROCEDURE load_orders_big()
BEGIN
  DECLARE i INT DEFAULT 0;
  DECLARE chunk INT DEFAULT 1000;
  DECLARE base INT;

  WHILE i < 500 DO
    SET base = i * chunk;

    INSERT /*+ SET_VAR(max_execution_time=0) */ INTO test_orders
      (user_id, product_id, order_date, quantity, status)
    SELECT
      ((base + s.id - 1) MOD 50000) + 1       AS user_id,
      ((base + s.id - 1) MOD 5000)  + 1       AS product_id,
      CURRENT_DATE - INTERVAL ((base + s.id) MOD 180) DAY
        + INTERVAL ((base + s.id) MOD 86400) SECOND AS order_date,
      ((base + s.id - 1) MOD 5) + 1            AS quantity,
      CASE WHEN ((base + s.id) MOD 10) = 0
           THEN 'COMPLETED' ELSE 'SHIPPED' END  AS status
    FROM seq_1e5 s
    WHERE s.id BETWEEN 1 AND chunk;

    COMMIT;
    SET i = i + 1;
  END WHILE;
END //
DELIMITER ;

CALL load_orders_big();

-- === 설정 원복 ===
SET foreign_key_checks = 1;
SET unique_checks = 1;
SET autocommit = 1;

-- === 검증 ===
SELECT COUNT(*) FROM test_users;     -- 기대: 50000
SELECT COUNT(*) FROM test_products;  기대: 5000
SELECT COUNT(*) FROM test_orders;    -- 기대: 500000


```

</details>

<details>

<summary>LEFT vs RIGHT JOIN (인덱스 X, WHERE X)</summary>

* LEFT JOIN

```sql
EXPLAIN
SELECT SQL_NO_CACHE
       o.order_id, u.name, p.name AS product_name
FROM test_orders   AS o  IGNORE INDEX (idx_orders_status_date, idx_orders_user, idx_orders_product)
LEFT JOIN test_users    AS u  IGNORE INDEX (PRIMARY)  ON u.user_id    = o.user_id
LEFT JOIN test_products AS p  IGNORE INDEX (PRIMARY)  ON p.product_id = o.product_id;
```

<figure><img src="../.gitbook/assets/image (456).png" alt=""><figcaption></figcaption></figure>

```
-> Left hash join (p.product_id = o.product_id)  (cost=12.5e+12 rows=125e+12) (actual time=32.1..245 rows=500000 loops=1)
    -> Left hash join (u.user_id = o.user_id)  (cost=2.5e+9 rows=25e+9) (actual time=29.5..156 rows=500000 loops=1)
        -> Table scan on o  (cost=50340 rows=498348) (actual time=0.791..55.1 rows=500000 loops=1)
        -> Hash
            -> Table scan on u  (cost=0.0153 rows=50116) (actual time=0.0877..18 rows=50000 loops=1)
    -> Hash
        -> Table scan on p  (cost=0.0201 rows=5000) (actual time=0.101..1.29 rows=5000 loops=1)
```

* RIGHT JOIN

```sql
EXPLAIN 
SELECT SQL_NO_CACHE
       o.order_id, u.name, p.name AS product_name
FROM test_products AS p IGNORE INDEX (PRIMARY)
RIGHT JOIN test_orders AS o IGNORE INDEX (idx_orders_status_date, idx_orders_user, idx_orders_product)
    ON p.product_id = o.product_id
JOIN test_users     AS u IGNORE INDEX (PRIMARY)
    ON u.user_id = o.user_id;
```

<figure><img src="../.gitbook/assets/image (457).png" alt=""><figcaption></figcaption></figure>

```
-> Left hash join (p.product_id = o.product_id)  (cost=25.1e+6 rows=251e+6) (actual time=26.6..243 rows=500000 loops=1)
    -> Inner hash join (o.user_id = u.user_id)  (cost=2.5e+9 rows=50116) (actual time=22.6..154 rows=500000 loops=1)
        -> Table scan on o  (cost=1.99 rows=498348) (actual time=0.759..58.2 rows=500000 loops=1)
        -> Hash
            -> Table scan on u  (cost=5068 rows=50116) (actual time=0.0691..12.5 rows=50000 loops=1)
    -> Hash
        -> Table scan on p  (cost=0.0302 rows=5000) (actual time=0.0845..1.87 rows=5000 loops=1)
```

</details>

<details>

<summary>LEFT vs RIGHT JOIN (인덱스 O, WHERE X)</summary>

* LEFT JOIN

```sql
EXPLAIN
SELECT SQL_NO_CACHE 
       o.order_id, u.name, p.name AS product_name
FROM test_orders o
LEFT JOIN test_users u    ON u.user_id = o.user_id
LEFT JOIN test_products p ON p.product_id = o.product_id;
```

<figure><img src="../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>

```
-> Nested loop left join  (cost=399183 rows=498348) (actual time=2..446 rows=500000 loops=1)
    -> Nested loop left join  (cost=224762 rows=498348) (actual time=1.92..270 rows=500000 loops=1)
        -> Table scan on o  (cost=50340 rows=498348) (actual time=1.69..58 rows=500000 loops=1)
        -> Single-row index lookup on u using PRIMARY (user_id=o.user_id)  (cost=0.25 rows=1) (actual time=326e-6..343e-6 rows=1 loops=500000)
    -> Single-row index lookup on p using PRIMARY (product_id=o.product_id)  (cost=0.25 rows=1) (actual time=252e-6..269e-6 rows=1 loops=500000)
```

* RIGHT JOIN

```
EXPLAIN
SELECT SQL_NO_CACHE 
       o.order_id, u.name, p.name AS product_name
FROM test_users u
RIGHT JOIN test_orders o   ON u.user_id = o.user_id
JOIN test_products p       ON p.product_id = o.product_id;
```

<figure><img src="../.gitbook/assets/image (455).png" alt=""><figcaption></figcaption></figure>

```
-> Nested loop left join  (cost=4005 rows=5000) (actual time=0.489..432 rows=500000 loops=1)
    -> Nested loop inner join  (cost=2255 rows=5000) (actual time=0.477..230 rows=500000 loops=1)
        -> Table scan on p  (cost=505 rows=5000) (actual time=0.228..0.954 rows=5000 loops=1)
        -> Index lookup on o using idx_orders_product (product_id=p.product_id)  (cost=0.25 rows=1) (actual time=0.00428..0.0426 rows=100 loops=5000)
    -> Single-row index lookup on u using PRIMARY (user_id=o.user_id)  (cost=0.25 rows=1) (actual time=308e-6..325e-6 rows=1 loops=500000)
```

</details>

<details>

<summary>LEFT vs RIGHT JOIN (인덱스 O, WHERE O)</summary>

* LEFT JOIN (WHERE 조건)

```sql
EXPLAIN, ANALYZE
SELECT SQL_NO_CACHE 
       o.order_id, u.name, p.name AS product_name
FROM test_orders o
LEFT JOIN test_users u    ON u.user_id = o.user_id
LEFT JOIN test_products p ON p.product_id = o.product_id
WHERE o.status = 'COMPLETED'
  AND o.order_date >= CURRENT_DATE - INTERVAL 30 DAY;
```

<figure><img src="../.gitbook/assets/image (452).png" alt=""><figcaption></figcaption></figure>

```
-> Nested loop left join  (cost=24194 rows=21038) (actual time=7.02..63.9 rows=11111 loops=1)
    -> Nested loop left join  (cost=16831 rows=21038) (actual time=6.98..54.8 rows=11111 loops=1)
        -> Index range scan on o using idx_orders_status_date over (status = 'COMPLETED' AND '2025-07-15 00:00:00' <= order_date), with index condition: ((o.`status` = 'COMPLETED') and (o.order_date >= <cache>((curdate() - interval 30 day))))  (cost=9467 rows=21038) (actual time=6.94..33.3 rows=11111 loops=1)
        -> Single-row index lookup on u using PRIMARY (user_id=o.user_id)  (cost=0.25 rows=1) (actual time=0.00179..0.00182 rows=1 loops=11111)
    -> Single-row index lookup on p using PRIMARY (product_id=o.product_id)  (cost=0.25 rows=1) (actual time=668e-6..695e-6 rows=1 loops=11111)
```



* RIGHT JOIN (WHERE 조건)

```sql
EXPLAIN, ANALYZE
SELECT SQL_NO_CACHE 
       o.order_id, u.name, p.name AS product_name
FROM test_users u
RIGHT JOIN test_orders o   ON u.user_id = o.user_id
JOIN test_products p       ON p.product_id = o.product_id
WHERE o.status = 'COMPLETED'
  AND o.order_date >= CURRENT_DATE - INTERVAL 30 DAY;
```

<figure><img src="../.gitbook/assets/image (453).png" alt=""><figcaption></figcaption></figure>

```
-> Nested loop left join  (cost=2343 rows=250) (actual time=5.32..328 rows=11111 loops=1)
    -> Nested loop inner join  (cost=2255 rows=250) (actual time=5.31..321 rows=11111 loops=1)
        -> Table scan on p  (cost=505 rows=5000) (actual time=0.105..0.871 rows=5000 loops=1)
        -> Filter: ((o.`status` = 'COMPLETED') and (o.order_date >= <cache>((curdate() - interval 30 day))))  (cost=0.25 rows=0.05) (actual time=0.0571..0.0639 rows=2.22 loops=5000)
            -> Index lookup on o using idx_orders_product (product_id=p.product_id)  (cost=0.25 rows=1) (actual time=0.00591..0.06 rows=100 loops=5000)
    -> Single-row index lookup on u using PRIMARY (user_id=o.user_id)  (cost=0.25 rows=1) (actual time=524e-6..541e-6 rows=1 loops=11111)
```

</details>

정리:

상황 1: 인덱스 X, WHERE X

#### Observation

* 두 쿼리 모두 **해시 기반 조인(BNL/Hash Join)** 경로로 갔고(`Using join buffer (hash join)`),\
  `LEFT`(o→u→p)와 `RIGHT`(p→o→u)의 **실행 시간이 유사**.
* `o`/`u`/`p`가 모두 **풀 스캔**으로 해시 빌드/프로브를 수행.

#### Reason

* **필터가 없고** 인덱스를 막았기 때문에, 어느 쪽을 먼저 읽어도 **결과 행수(=50만)와 필요한 비교량**이 비슷함.
* 해시 조인은 “읽은 만큼 해시 테이블을 만들고 매칭”하므로, **조인 순서보다 입력 크기**가 지배.

#### Practice

* 인덱스가 전혀 없고 필터도 없으면 **조인 순서로 큰 차이를 만들기 어렵다.**\
  이 구간에서의 최적화는 **join\_buffer\_size**(세션) 튜닝이나 **배치/병렬화**가 더 효과적.
* 해시/BNL 경로를 확실히 관찰하고 싶으면 `IGNORE INDEX` + `EXPLAIN ANALYZE`로 재현 가능.

***

상황 2: 인덱스 O, WHERE X

#### Observation

* `LEFT`(orders 드라이빙): `orders` 풀스캔 + `users PK`/`products PK` 단건 점프 → **Nested Loop**로 50만 루프.
* `RIGHT`(products 드라이빙): `products` 풀스캔 + `orders(product_id)` 범위 조회(평균 \~100건) + `users PK` 점프 → **결국 총 점프량이 비슷**해서 시간도 비슷.

#### Reason

* 어느 방향이든 최종 결과는 **항상 50만 행**이고, 조인 키에 **적절한 인덱스가 모두 존재**하므로\
  **랜덤 접근 비용이 상쇄**되어 체감 차이가 작음.

#### Practice

* WHERE가 없고 조인 키 인덱스가 **양쪽에** 잘 있으면 **조인 순서의 체감 이득은 작다**.

***

상황 3: 인덱스 O, WHERE O (o.status + o.order\_date)

#### Observation

* `LEFT`(orders 먼저): `(status, order_date)` **인덱스 범위 스캔**으로 **초기에 11,111행**으로 급감 → 이후 PK 점프 → **\~64ms** 수준.
* `RIGHT`(products 먼저): `products`를 먼저 스캔하고 `orders(product_id)`를 찾아 **나중에** 상태/기간 필터 → **\~328ms**, **5배 이상 느림**.

#### Reason

* \*\*선택도 높은 필터가 걸리는 테이블(orders)\*\*을 **드라이빙**으로 잡으면\
  초기에 **행 수를 크게 줄여** 이후 조인 비용이 폭감.
* `RIGHT` 경로는 필터가 **사후 적용**되어 **불필요 접근**이 많이 발생.

#### Practice

* **규칙**: “**가장 잘 거르는 테이블**(선택도 高) → **드라이빙**”.
* 해당 테이블에 **복합 인덱스의 선두를 필터 컬럼 순서**로 둔다.\
  예) 이번 케이스: `(status, order_date)` → 범위 후 **조인 키로 점프**가 쉬움.
* 옵티마이저가 순서를 바꿔 비효율을 택할 때는\
  `STRAIGHT_JOIN`/힌트(`JOIN_ORDER`, `NO_BNL` 등, 버전 의존)로 **순서 고정** 실험.

***

요약

* **인덱스 X, WHERE X**: 조인 순서보다 **입력 크기**가 지배 → 해시/BNL 경로에서 **차이 미미**.
* **인덱스 ○, WHERE X**: **조인 키 인덱스가 양쪽에 있으면** 순서 차이는 **작다**.
* **인덱스 ○, WHERE ○**: **선택도 높은 조건이 걸리는 테이블을 드라이빙**으로 잡으면 **결정적 차이**가 난다(이번 실험에서 **\~5×**).





### Q. `LIKE '검색어%'`와 `LIKE '%검색어%'`의 성능 차이는 얼마나 날까?

A. `LIKE '검색어%'`는 인덱스 사용으로 빠르게 동작하지만, `LIKE '%검색어%'`는 인덱스 미사용으로 풀스캔이 발생해 큰 성능 차이가 납니다.

```sql
-- 접두사: 인덱스 범위 스캔 기대 (UNIQUE(email) 활용)
EXPLAIN ANALYZE
SELECT SQL_NO_CACHE COUNT(*)         -- 출력 최소화
FROM test_users
WHERE email LIKE 'user1%';

-> Aggregate: count(0)  (cost=7981 rows=1) (actual time=6.62..6.62 rows=1 loops=1)
    -> Filter: (test_users.email like 'user1%')  (cost=5572 rows=24094) (actual time=0.0791..6.1 rows=11111 loops=1)
        -> Covering index range scan on test_users using email over ('user1' <= email <= 'user1????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????')  (cost=5572 rows=24094) (actual time=0.074..4.71 rows=11111 loops=1)


-- 포함: 좌측 와일드카드로 인해 인덱스 사용 불가 → 풀스캔 예상
EXPLAIN ANALYZE
SELECT SQL_NO_CACHE COUNT(*)
FROM test_users
WHERE email LIKE '%example%';

-> Aggregate: count(0)  (cost=5625 rows=1) (actual time=25.2..25.2 rows=1 loops=1)
    -> Filter: (test_users.email like '%exam%')  (cost=5068 rows=5568) (actual time=0.0609..23 rows=50000 loops=1)
        -> Covering index scan on test_users using email  (cost=5068 rows=50116) (actual time=0.0574..11.1 rows=50000 loops=1)


```

