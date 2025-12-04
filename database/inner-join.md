# Inner Join

## 요약

* 내부 조인은 두 테이블에 공통으로 존재하는 데이터만 결과로 보여줌
* ON 절에 명시된 연결 조건이 참이 되는 행들만 결과에 포함
* 조인의 논리적 처리 순서는 FROM/JOIN으로 테이블을 결합하고 WHERE로 조건을 필터링한 후 SELECT로 원하는 컬럼을 선택하는 순서로 진행
* 내부 조인에서 INNER 키워든느 생략하고 JOIN만 사용해도 된다.
* 두 테이블 양쪽에 모두 연결고리가 있는 데이터만 결과에 포함된다
* 한쪽 테이블만 존재하는 데이터는 조인 결과에서 제외된다.
* 내부 조인은 테이블의 순서를 바꿔도 같은 결과를 반환한다.



## 1. 논리적 개념

* INNER JOIN은 ON 조건을 만족하는 두 테이블의 행만 반환.
* 교집합 처럼 동작하지만, ON 절의 조건에 따라 달라짐.

```sql
select u.name AS user_name, p.name AS product_name, o.order_date
from orders o
inner join users u on o.user_id = u.user_id
inner join products p on o.product_id = p.product_id
where o.status = 'COMPLETED';
```

* 논리 흐름:
  * orders와 users를 o.user\_id = u.user\_id 조건으로 매칭
  * 그 결과에 products를 p.product\_id = p.product\_id 조건으로 매칭
  * 최종 결과에서 status = 'COMPLETED' 조건을 만족하는 행만 출력



## 2. 물리적 실행 방식 (조인 알고리즘)

1. Nested Loop Join (중첩 루프 조인)
   1. 작동 방식
      1. 외부 테이블의 한 행을 선택
      2. 내부 테이블에서 ON 조건에 맞는 행을 탐색
      3. 일치하면 결과로 출력
      4. 외부 테이블의 다음 행으로 반복
   2. 특징
      1. 작은 테이블 <-> 큰 테이블 조인에 적합
      2. 내부 테이블에 인덱스가 있으면 빠름
   3. 복잡도
      1. 인덱스 없으면 O(N\*M)
      2. 인덱스 있으면 내부 탐색이 O(log M)
2. Hash Join (해시 조인)
   1. 작동 방식
      1. 작은 테이블을 메모리에 올려 해시 테이블 생성 (Build 단계)
      2. 큰 테이블의 각 행을 읽으며 해시 테이블에서 매칭 검색 (Probe 단계)
   2. 특징
      1. 인덱스 없어도 효율적
      2. 대용량 데이터에 강함
      3. 메모리 사용량 큼
3. Merge Join (Sort-Merge Join)
   1. 작동 방식
      1. 두 테이블을 조인 키 기준으로 정렬
      2. 두 정렬된 리스트를 병합하며 조건 만족하는 행만 반환
   2. 특징
      1. 정렬된 상태라면 빠름
      2. 범위 조건 처리에도 적합
      3. 정렬 비용이 크면 비효율적



## 3. 실행 흐름

1. orders 테이블 풀스캔: 7행 전체에서 각행에 대해 'COMPLETED' 조건 확인 후, 통과된 행만 다음 단계로 진행
2. users 조인: o.user\_id 값으로 users 테이블을 PK 검색 (eq\_ref)
3. products 조인: o.product\_id 값으로 products 테이블을 PK 검색 (eq\_ref)



## 4. 쿼리 옵티마이저 단계

1. 파싱 (Parsing)

SQL 문을 구문 분석해 추상 구문 트리(AST)를 만듭니다.

<details>

<summary>AST(Abstract Syntax Tree. 추상 구문 트리)</summary>

코드 -> 파서가 읽어서 구문적 요소를 계층 구조로 표현한 것.&#x20;

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

</details>

2. 논리적 최적화
   1. 조인 순서 변경
   2. 푸시다운: WHERE 조건이나 조인 조건을 가능한 한 빨리 적용해 데이터 양을 줄임
3. 물리적 최적화
   1. 어떤 조인 알고리즘을 쓸지 결정 (Nested Loop, Hash Join, Merge Join 등)
   2. 어떤 인덱스를 사용할지 결정
4. 실행 계획 생성 (EXPLAIN 키워드)

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* id: 서브쿼리 없이 하나의 SELECT 블록에서 실행
* select\_type: 단일 SELECT 실행 계획(서브쿼리나 UNION 없이 단순 SELECT)
* type:
  * ALL: 풀 테이블 스캔
  * eq\_ref: PK나 UNIQUE 인덱스를 사용해서 정확히 1건 매칭.
* possible keys: 인덱스 후보, fk\_orders\_users, fk\_orders\_products 라는 외래키 인덱스 후보는 있지만, WHERE 조건에는 맞지 않아 사용 안됨
* key: 현재 사용된 인덱스
* rows:&#x20;
  * order는 전체 스캔.
  * users, products는 PK 검색이라 1건
*  filtered: 필터링 후 남는 행의 비율
* extra:
  * Using where: where 조건 필터링 수행
  * NULL: 추가 작업 없음



## 5. 최적화

* orders.status가 풀테이블 스캔이 발생.
* status에 인덱스를 추가

```sql
CREATE INDEX idx_orders_status ON orders(status);
```

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

* **possible\_keys**: 사용할 수 있는 후보 인덱스 목록에 `idx_orders_status`가 포함됨.
* **key\_len=203**: 무의미. (사용된 인덱스 키 길이를 바이트로 보여줌)
* **rows=4**: 스캔 범위가 줄어듬. (7 -> 4)



## 6. 비교

{% code fullWidth="true" %}
```
인덱스X
-> Nested loop inner join  (cost=1.65 rows=1) (actual time=0.0772..0.0954 rows=4 loops=1)
    -> Nested loop inner join  (cost=1.3 rows=1) (actual time=0.0693..0.0827 rows=4 loops=1)
        -> Filter: (o.`status` = 'COMPLETED')  (cost=0.95 rows=1) (actual time=0.0505..0.0584 rows=4 loops=1)
            -> Table scan on o  (cost=0.95 rows=7) (actual time=0.0455..0.0512 rows=7 loops=1)
        -> Single-row index lookup on u using PRIMARY (user_id=o.user_id)  (cost=0.35 rows=1) (actual time=0.00525..0.00529 rows=1 loops=4)
    -> Single-row index lookup on p using PRIMARY (product_id=o.product_id)  (cost=0.35 rows=1) (actual time=0.00278..0.00283 rows=1 loops=4)
    
    
인덱스O
-> Nested loop inner join  (cost=3.7 rows=4) (actual time=0.047..0.0548 rows=4 loops=1)
    -> Nested loop inner join  (cost=2.3 rows=4) (actual time=0.0415..0.0465 rows=4 loops=1)
        -> Index lookup on o using idx_orders_status (status='COMPLETED')  (cost=0.9 rows=4) (actual time=0.032..0.0341 rows=4 loops=1)
        -> Single-row index lookup on u using PRIMARY (user_id=o.user_id)  (cost=0.275 rows=1) (actual time=0.00241..0.00243 rows=1 loops=4)
    -> Single-row index lookup on p using PRIMARY (product_id=o.product_id)  (cost=0.275 rows=1) (actual time=0.00185..0.00187 rows=1 loops=4)
```
{% endcode %}

* orders(o) 테이블 풀 스캔 (ALL) vs idx\_orders\_status 인덱스 범위 스캔
* **Before**: `orders` **Table scan + Filter** → 남은 4행에 대해 `users/products` **PK lookup**
* **After**: `orders` **Index lookup(status)** → 바로 4행만 읽고 **PK lookup**\
  실제 시간 **0.05ms → 0.03ms대**로 줄었음



## 7. Problems

```sql
-- problem 1
select o.order_id, u.name as user_name, p.name as product_name, o.order_date
from orders o
inner join products p on o.product_id = p.product_id
inner join users u on o.user_id = u.user_id
where o.status = 'SHIPPED';

-- problem2
explain analyze select u.name AS user_name, p.name AS product_name, o.order_date
from orders o
inner join users u on o.user_id = u.user_id
inner join products p on o.product_id = p.product_id
where o.status = 'COMPLETED';

-- problem3
select u.name as user_name, sum(p.price * o.quantity) as total_purchase_amount
from orders o
inner join products p on o.product_id = p.product_id
inner join users u on o.user_id = u.user_id
group by u.name
order by total_purchase_amount desc;
```
