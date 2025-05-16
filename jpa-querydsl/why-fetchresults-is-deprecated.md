# why fetchResults() is deprecated

Spring + QueryDSL 환경에서 페이징 처리 시 자주 사용되던 fetchResults() 메서드가 QueryDSL 5.0부터 deprecated 되었습니다. 왜 fetchResult()를 더 이상 사용하지 않는지, 실제 쿼리 동작과 로그, 대체 방법 등을 소개합니다.



## 페이징 처리

클라이언트에 데이터 리스트를 응답으로 제공할 때, 페이지 단위로 쪼개서 보내는 방식입니다. 여기에 포함되는 정보는 현재 페이지에 해당하는 데이터 리스트, 전체 아이템 개수 등이 있습니다.



현재 QueryDSL 5.0에서 사용하는 방식은 fetch()와 count()입니다. (개발중인 코드에서 가져왔습니다)

```java
public List<ProductSearchDto> findProductsByCondition(ProductSearchConditionDto conditionDto) {
    List<ProductSearchDto> content = queryFactory
        .select(Projections.constructor(
            ProductSearchDto.class,
            product.id,
            product.productName,
            product.price,
            product.stockQuantity,
            product.category.name
        ))
        .from(product)
        .where(buildWhere(conditionDto))
        .offset(conditionDto.getPage() * conditionDto.getSize())
        .limit(conditionDto.getSize())
        .fetch();

    long total = queryFactory
        .select(product.count())
        .from(product)
        .where(buildWhere(conditionDto))
        .fetchOne();

    return content; // total은 PageImpl로 감싸 반환할 때 사용
}
```

그리고, 5.0 이전 방식인 fetchResult()를 사용했을때 입니다.

```java
public void testFetchResults(ProductSearchConditionDto conditionDto) {
    QueryResults<ProductSearchDto> result = queryFactory
        .select(Projections.constructor(
            ProductSearchDto.class,
            product.id,
            product.productName,
            product.price,
            product.stockQuantity,
            product.category.name
        ))
        .from(product)
        .where(product.price.gt(1000))
        .offset(0)
        .limit(10)
        .fetchResults();

    List<ProductSearchDto> content = result.getResults();
    long total = result.getTotal();
}
```



## 왜 Deprecated 됐을까?

바로 위 코드를 보면, select구문을 통해 결과값을 가져오고, 가져온 결과값에 total값이 포함되어 있는걸로 보입니다. 한번의 select를 통해 데이터와 count값을 가져올 수 있는데, 왜 deprecated가 되었을까 의문이 들었습니다. 왜냐하면 변경되고 난 후에는, 데이터를 가져오는 select와, count를 가져오는 select를 두번 호출 하는 것처럼 보입니다.



### Query의 개수

사실, fetchResult()는 보기에는 한번의 호출처럼 보이지만, 실제로는 fetch()와 count()가 내부에서 모두 실행됩니다.

<figure><img src="../.gitbook/assets/image (426).png" alt=""><figcaption></figcaption></figure>

fetchResult()를 실행했을때 확인해보면, 실제로는 두개의 쿼리가 나가는것을 확인했습니다. 쿼리를 날리는 횟수는 동일한 것으로 확인했습니다. 그러면 쿼리를 날리는 수가 동일한데, 왜 변경을 했을까? 라는 의문이 들었습니다.



### 유연성

복잡한 쿼리의 경우(join이나 필터 조건), 조회 쿼리에서는 join이 필요하지만, count 쿼리에서는 사실 join이 필요가 없습니다. 오히려 count에서 join을 하게 되면 성능저하가 발생합니다.

현재 예시에서는 내부적으로 join을 하지 않지만, join을 하게 되는 경우가 있습니다.

* select()에 join된 테이블의 필드를 사용했을 경우

```
queryFactory
  .select(Projections.constructor(ProductDto.class,
      product.id,
      product.productName,
      product.price,
      category.name // 조인 대상 필드 사용
  ))
  .from(product)
  .join(product.category, category)
  .where(product.price.gt(1000))
  .offset(0)
  .limit(10)
  .fetchResults();

--결과--
Hibernate:
    select count(p1_0.id)
    from product p1_0
    join category c1_0 on c1_0.id = p1_0.category_id
    where p1_0.price > ?

Hibernate:
    select p1_0.id, p1_0.product_name, p1_0.price, c1_0.name
    from product p1_0
    join category c1_0 on c1_0.id = p1_0.category_id
    where p1_0.price > ?
    limit ?, ?

```

* join fetch를 사용했을 경우

```
queryFactory
  .select(Projections.constructor(ProductDto.class,
      product.id,
      product.productName,
      product.price
  ))
  .from(product)
  .join(product.category, category)
  .where(
      product.price.gt(1000),
      category.type.eq("ELECTRONICS") // 조인된 필드 사용
  )
  .offset(0)
  .limit(10)
  .fetchResults();


--결과-- 
select count(p1_0.id)
from product p1_0
join category c1_0 on c1_0.id = p1_0.category_id
where p1_0.price > ? and c1_0.type = ?

select p1_0.id, p1_0.product_name, p1_0.price
from product p1_0
join category c1_0 on c1_0.id = p1_0.category_id
where p1_0.price > ? and c1_0.type = ?
limit ?, ?
```

* 특정 where절이나, groupBy 등에 join 대상 테이블의 필드를 사용했을 경우

```
queryFactory
  .selectFrom(product)
  .join(product.category, category).fetchJoin() // 강제 fetchJoin
  .where(product.price.gt(1000))
  .offset(0)
  .limit(10)
  .fetchResults();

--결과--
select count(p1_0.id)
from product p1_0
join category c1_0 on c1_0.id = p1_0.category_id
where p1_0.price > ?

select p1_0.id, ..., c1_0.id, c1_0.name, ...
from product p1_0
join category c1_0 on c1_0.id = p1_0.category_id
where p1_0.price > ?
limit ?, ?

```

위와 같은 상황을 봤을때, fetchResult()을 사용하면 쿼리 구조를 제어할 수 없습니다. 그렇기 때문에 실제로는 fetchJoin()이 사용되지 않고, 아래와 같이 분리해서 사용하는게 바람직합니다.

```java
List<ProductDto> content = queryFactory
    .select(...)
    .from(...)
    .join(...) // 조회에 필요한 join
    .where(...)
    .offset(...)
    .limit(...)
    .fetch();

Long total = queryFactory
    .select(product.count()) // 조인 생략 가능
    .from(product)
    .where(...) // 필요한 조건만
    .fetchOne();

// 동일 메서드에서 조회 또는 메서드 분리
```

## 결론

fetchResult()는 간편하지만, 쿼리 성능을 제어하기 어렵고 불필요한 조인으로 인해 성능 저하가 발생할 수 있습니다. fetch()와 count()를 명시적으로 분리하는 것이 더 좋은 판단입니다.
