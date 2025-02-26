# CreatedAt, ModifiedAt

## 1. 개요

CreatedAt과 ModifiedAt(또는 created\_date, updated\_date, created\_time, updated\_time 등)은 이력이 언제 생성/수정되었는지 확인하기 위해서입니다. **CreatedAt, ModifiedAt** 필드는 거의 모든 프로젝트에서 기본적으로 필요한 **타임스탬프**입니다. 제대로 설계해두면:

* **데이터 이력 관리**와 **운영 분석**이 쉬워지고,
* **정렬, 검색, 통계** 기능을 간단히 구현할 수 있으며,
* 수정 충돌이나 변경 추적 시에도 큰 도움을 줍니다.



## 2. DB 설계 관점

### 2.1. 테이블 컬럼 타입

일반적으로 다음 중 하나를 택합니다:

1. **DATETIME** (또는 TIMESTAMP)
2. **BIGINT**(epoch time) 형태로 초 단위 또는 밀리초 단위 저장
3. **DATE**(일자만 필요할 경우)

MySQL, PostgreSQL, Oracle, MSSQL 등 관계형 DB에서 가장 흔한 방식은 **DATETIME** 또는 **TIMESTAMP** 컬럼을 두는 것입니다. 예:

```sql
created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP
```

이렇게 하면, Insert 시 `created_at`이 자동으로 현재 시간으로 채워지고, Update 시 `updated_at`이 자동으로 갱신되도록 설정할 수도 있습니다(각 DB별로 약간의 차이가 있음).

### 2.2. MySQL 예시

MySQL 5.6 이상에서는 테이블에 하나의 TIMESTAMP 컬럼에 `DEFAULT CURRENT_TIMESTAMP`와 `ON UPDATE CURRENT_TIMESTAMP`를 적용할 수 있습니다. 하지만 **둘 이상의 컬럼**(`created_at`, `updated_at`) 모두를 자동으로 다루려면 조금 더 세심한 설정이 필요하거나, 애플리케이션 레벨에서 업데이트해줄 수도 있습니다.

```sql
CREATE TABLE products (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

이 예시는 `created_at`와 `updated_at`에 동일한 디폴트와 업데이트 설정을 걸었지만, 실제로는 `created_at`만 Insert 시 자동 생성하고, `updated_at`은 Application 레벨에서만 업데이트하게 하기도 합니다.

### 2.3. PostgreSQL 예시

PostgreSQL에서는 `TIMESTAMP`나 `TIMESTAMPTZ`(타임존 포함)를 사용하고, `DEFAULT now()`로 기본값을 세팅할 수 있습니다.

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NULL
```

`updated_at`을 자동 업데이트하려면 트리거를 쓰거나, 애플리케이션에서 UPDATE 구문 실행 시 현재 시간을 세팅해야 합니다.



## 3. 애플리케이션 레벨 (Java Spring) 관점

### 3.1. JPA Auditing

스프링 부트 + JPA에서 **Auditing** 기능을 사용하면, 엔티티가 Insert될 때와 Update될 때 자동으로 특정 필드를 채워줄 수 있습니다. 예:

```java
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public abstract class BaseEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

이후 `@EnableJpaAuditing` 등을 설정하면, JPA가 **엔티티 생성 시점**에 `createdAt`, **엔티티 변경 시점**에 `updatedAt`을 자동으로 채워줍니다.

#### 3.1.1. BaseEntity 상속

실무에서 공통 필드는 **추상 클래스(BaseEntity)** 에 모아두고, 각 엔티티가 상속받도록 많이 사용합니다.

```java
@Entity
public class Product extends BaseEntity {
    @Id
    private Long id;
    private String name;
    // ...
}
```

이렇게 하면 모든 엔티티가 `createdAt`, `updatedAt`를 자동으로 가집니다.

### 3.2. Lombok과 LocalDateTime

스프링 부트 2.0 이상에서는 `LocalDateTime`, `LocalDate`, `Instant` 등을 쓰는 경우가 많습니다. 예:

```java
private LocalDateTime createdAt;
private LocalDateTime updatedAt;
```

또는 타임존을 고려해 `ZonedDateTime`을 쓸 수도 있지만, 일반적으로 DB에는 UTC로 저장하고, 애플리케이션에서는 LocalDateTime으로 관리하는 식이 무난합니다.

### 3.3. Insert/Update 시점 제어

스프링 데이터 JPA Auditing을 쓰면 편리하지만, 경우에 따라 “엔티티 수정은 자주 일어나지만, `updatedAt`을 꼭 매번 갱신할 필요가 없는 속성 변경” 같은 예외 상황이 있을 수 있습니다. 그런 경우, 특정 필드는 수동으로 업데이트해야 하거나, Auditing을 부분적으로만 적용해야 할 수도 있습니다.



## 4. 타임존 이슈

### 4.1. 서버 vs DB vs 클라이언트

* **서버 시간**은 UTC로 설정하는 것이 일반적
* **DB**도 UTC 기준으로 `created_at`, `updated_at`을 저장하면, 다른 타임존 지역에서 접속해도 혼동이 적다
* **클라이언트**(웹, 앱)에서는 `Asia/Seoul` 등 사용자 타임존에 맞게 표시

### 4.2. 날짜/시간 변환

Java에서는 `ZonedDateTime`, `OffsetDateTime`, `LocalDateTime` 등을 적절히 써서 변환해야 합니다.

* DB에 UTC로 저장된 `TIMESTAMP`를 Java로 가져오면, `LocalDateTime` UTC가 됩니다.
* 클라이언트에 보여줄 때는 원하는 타임존으로 변환.
* REST API 응답 시 ISO 8601 형식(`yyyy-MM-dd'T'HH:mm:ss.SSSZ`)으로 통일하면 좋습니다.

### 4.3. MySQL TIMESTAMP vs DATETIME 차이

* **TIMESTAMP**는 MySQL이 내부적으로 UTC 변환 후 저장, DB 서버 타임존에 따라 반환값이 달라질 수 있음.
* **DATETIME**은 타임존 정보 없이 “그냥 YYYY-MM-DD HH:MM:SS”를 저장.
* 글로벌 환경이라면 `TIMESTAMP + UTC` 방식을 권장, 혹은 PostgreSQL의 `TIMESTAMPTZ`도 권장됩니다.



## 5. 자동 vs 수동 업데이트

1. **자동 업데이트**: DB의 `ON UPDATE CURRENT_TIMESTAMP`나 JPA Auditing을 통해, 레코드가 변경될 때마다 `updatedAt`을 자동 설정.
2. **수동 업데이트**: 애플리케이션에서 “진짜로 우리가 변경 시점을 기록해야 하는 변경만” 발생 시에 `updatedAt = now()`를 하도록 제어.

예를 들어, 일부 시스템에서 “조회수 증가”와 같은 사소한 변경은 `updatedAt`을 갱신하지 않을 수도 있습니다. 반면, **실질적인 비즈니스 로직**(상품명, 가격, 재고 등) 변경이 있을 때만 `updatedAt`을 갱신하도록 하고 싶다면 **수동 방식**이 더 적절합니다.



## 6. CreatedAt, ModifiedAt와 삭제(DeletedAt)

### 6.1. Soft Delete (DeletedAt)

많은 시스템에서 “레코드를 실제로 지우지 않고, **삭제 일시(DeletedAt)** 를 기록”하는 소프트 삭제(Soft delete) 방식을 씁니다. 이 경우, “복원”도 가능하고, “언제 삭제됐는지” 추적도 가능합니다.

* **deleted\_at**: null이면 존재, not null이면 삭제된 상태
* 쿼리 시 `WHERE deleted_at IS NULL`을 기본 조건에 넣어서 실제 삭제된 레코드를 숨김

### 6.2. 장점과 단점

* **장점**: 실수로 삭제한 데이터를 복원할 수 있음, 역사 기록 보존
* **단점**: 테이블이 계속 커지므로 성능/관리 비용 증가, WHERE 절에 조건 추가해야 해서 쿼리가 복잡해짐



## 7. API 설계 관점

REST API 설계 시, 리소스의 JSON 응답에 `createdAt`, `updatedAt` 필드를 포함해주는 것이 일반적입니다. 예:

```json
json복사편집{
  "id": 101,
  "name": "New Product",
  "createdAt": "2025-02-21T12:34:56Z",
  "updatedAt": "2025-02-22T09:10:11Z"
}
```

* **포맷**: ISO 8601 (`YYYY-MM-DDTHH:mm:ssZ`)를 권장
* **시간대**: 보통 `Z`(UTC) 표기로 넘기고, 클라이언트가 지역 시간대로 변환



## 8. 예시 코드: Spring Boot + MySQL

#### 8.1. DB 스키마

```sql
CREATE TABLE review (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    product_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    rating INT NOT NULL,
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### 8.2. Entity

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Review {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long productId;
    private Long userId;
    private int rating;
    private String comment;

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
    // ...
}
```

#### 8.3. 설정

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```

이렇게 하면, Spring Data JPA가 `createdAt`, `updatedAt`을 Insert/Update 시 자동으로 채워줍니다.





