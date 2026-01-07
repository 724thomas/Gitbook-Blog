---
description: Redisson과 JPA 비관적 락을 활용한 이중 락 기반 동시성 제어 시스템 설계 및 구현
---

# Design and Implementation of a Dual-Lock Concurrency Control System Using Redisson and JPA Pessimist

## 분산환경 동시성 제어 구현

Redisson 분산락 & JPA Pessimistic Lock 조합으로 이중 락 메커니즘 구현



### 프로젝트 개요

E-Commerce 플랫폼에서 동시 주문 시 재고 중복 차감 문제를 해결하기 위해 Redisson 분산락과 JPA Pessimistic Lock을 조합한 이중 락 메커니즘을 설계하고 구현했습니다.



**핵심 성과**

* **데이터 일관성 보장**: 동시 주문 시 재고 중복 차감 문제 해결
* **시스템 안정성 향상**: Redis 락 실패 시 명확한 에러 처리로 시스템 안정성 확보
* **코드 품질 향상**: 포괄적인 단위/통합 테스트 작성



**문제 상황**

“재고 100개인 상품에 동시 주문 200개가 들어왔을 때, 정확히 100개만 주문이 성공해야 한다.”

* **Race Condition**: 동시 주문 시 재고 검증과 차감 사이의 경쟁 상태
* **데이터 불일치**: 실제 재고보다 많은 주문이 성공하는 오버셀링 현상
* **시스템 신뢰성**: 고객 불만과 비즈니스 손실 발생

**락 계층 구조 - 최종 아키텍처**

<figure><img src="/broken/files/v0yNLr0SEeXPapykY8aP" alt=""><figcaption></figcaption></figure>



* 요청 → Redis 분산락 → JPA 비관적 락 → 안전한 재고 차감
* 1차 보호 (Redis 분산락): 여러 애플리케이션 인스턴스 간 동시성 제어
* 2차 보호 (DB 비관적 락): 데이터베이스 레벨에서의 추가 보호



#### **이중 락 메커니즘 선택 이유**

**1. Redis 분산락**

* 분산 환경에서 빠른 동시성 제어
* TTL 기반 자동 해제로 데드락 방지
* 높은 처리 성능 (평균 2-5ms 레이턴시)

**2. JPA 비관적 락**

* 데이터베이스 레벨 무결성 보장
* 다중 애플리케이션 환경에서도 안전
* Spring의 @Transactional의 트랜잭션 경계와 락의 생명주기가 일치



### 💻 핵심 구현 내용

#### 1. 예약-확정 패턴 설계

**Phase 1: 예약 단계 (Redis 분산락)**

```java
@Service
@RequiredArgsConstructor
public class StockService {
    
    private final RedisTemplate<String, String> redisTemplate;
    private final RedissonClient redissonClient;
    private final ProductApiRepository productApiRepository;
    
    public boolean tryReserve(Long orderId, Long productId, Long quantity) {
        String lockKey = "lock:product:" + productId;
        RLock lock = redissonClient.getLock(lockKey);
        
        try {
            // 5초 대기, 10초 유지 (주석과 실제 코드 불일치 발견)
            if (!lock.tryLock(5, 10, TimeUnit.SECONDS)) {
                log.warn("재고 락 획득 실패. lockKey: {}", lockKey);
                return false;
            }
            
            // Source of Truth: DB에서 실제 재고 조회
            Long dbAvailable = productApiRepository.findById(productId)
                    .map(Product::getStockQuantity)
                    .orElse(0L);
            
            // Redis Hash에서 현재 예약된 수량 합계 계산
            String reservationKey = "product:reservations:" + productId;
            Map<Object, Object> reservations = redisTemplate.opsForHash().entries(reservationKey);
            long totalReserved = reservations.values().stream()
                    .mapToLong(val -> Long.parseLong(val.toString()))
                    .sum();
            
            // 핵심 로직: 예약 가능 재고 = DB 재고 - 현재 예약량
            if ((dbAvailable - totalReserved) >= quantity) {
                redisTemplate.opsForHash().put(reservationKey, String.valueOf(orderId), String.valueOf(quantity));
                redisTemplate.expire(reservationKey, 24, TimeUnit.HOURS);
                return true;
            }
            return false;
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return false;
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

* 소스코드: [StockService.java,](https://github.com/f-lab-edu/ECom/blob/main/api/src/main/java/com/example/api/module/stock/service/StockService.java) [StockServiceTest.java](https://github.com/f-lab-edu/ECom/blob/main/api/src/test/java/com/example/api/module/stock/service/StockServiceTest.java)

**Phase 2: 확정 단계 (이중 락)**

```java
@Transactional
public void confirmStock(List<OrderProductRequestDto> orderProductRequestDtos) {
    // MultiLock으로 여러 상품 원자적 처리 (데드락 방지)
    List<RLock> locks = orderProductRequestDtos.stream()
            .map(dto -> redissonClient.getLock("lock:product:" + dto.getProductId()))
            .toList();
    
    RLock multiLock = redissonClient.getMultiLock(locks.toArray(new RLock[0]));
    
    try {
        multiLock.lock();
        
        for (OrderProductRequestDto dto : orderProductRequestDtos) {
            long productId = dto.getProductId(), quantity = dto.getQuantity();
            
            // JPA 비관적 락으로 DB 레벨 보호
            Product product = productApiRepository.findByIdWithPessimisticLock(productId)
                    .orElseThrow(() -> new IllegalStateException("Product not found: " + productId));
            
            // 최종 재고 검증 후 차감
            if (product.getStockQuantity() < quantity) {
                throw new IllegalStateException("Not enough stock for product: " + productId);
            }
            product.decreaseStock(quantity);
        }
    } finally {
        multiLock.unlock();
    }
}
```

#### 2. JPA 비관적 락 구현

```java
public interface ProductApiRepository extends JpaRepository<Product, Long> {
    
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select p from Product p where p.id = :id")
    Optional<Product> findByIdWithPessimisticLock(Long id);
}
```

* **소스코드:** [ProductApiRepository](https://github.com/f-lab-edu/ECom/blob/main/core/src/main/java/com/example/core/domain/product/api/ProductApiRepository.java)

#### 3. 쿠폰 서비스 이중 락 적용

```java
@Transactional
public void confirmCoupon(Long couponId) {
    RLock lock = redissonClient.getLock("lock:coupon:" + couponId);
    
    try {
        lock.lock();
        
        Coupon coupon = couponApiRepository.findByIdWithPessimisticLock(couponId)
                .orElseThrow(() -> new IllegalStateException("cannot find coupon: " + couponId));
        
        if (coupon.getStatus() != CouponStatus.AVAILABLE) {
            throw new IllegalStateException("already used or not available coupon: " + couponId);
        }
        coupon.use();
        
    } finally {
        if (lock.isLocked() && lock.isHeldByCurrentThread()) {
            lock.unlock();
        }
    }
}
```

* 소스코드: [CouponService.java](https://github.com/f-lab-edu/ECom/blob/main/api/src/main/java/com/example/api/module/coupon/service/CouponService.java), [CouponServiceTest.java](https://github.com/f-lab-edu/ECom/blob/main/api/src/test/java/com/example/api/module/coupon/service/CouponServiceTest.java)

### 🛠️ 기술적 설계 결정

#### 이중 락 메커니즘 선택 이유

**1. Redis 분산락의 장점**

* 분산 환경에서 빠른 동시성 제어
* TTL 기반 자동 해제로 데드락 방지
* 높은 처리 성능 (평균 2-5ms 레이턴시)

**2. JPA 비관적 락의 장점**

* 데이터베이스 레벨 무결성 보장
* 트랜잭션과의 완벽한 통합
* 복구 가능한 안정성

**3. 조합의 시너지**

* Redis 장애 시에도 DB 락으로 기본 보호
* 각각의 단점을 상호 보완
* 성능과 안정성의 균형

#### 구현 상세 기술

**1. Redisson MultiLock 활용**

```java
// 여러 상품 주문 시 데드락 방지
RLock multiLock = redissonClient.getMultiLock(locks.toArray(new RLock[0]));
multiLock.lock(); // 모든 락을 원자적으로 획득
```

**2. 예외 처리 전략**

* **타임아웃**: 재고(5초), 쿠폰(3초) → `false` 반환
* **스레드 중단**: `Thread.currentThread().interrupt()` 호출
* **락 해제 보장**: `finally` 블록으로 항상 보장

**3. TTL 관리**

* 예약 정보 24시간 자동 만료
* 메모리 효율성과 데이터 정합성 확보

### 📊 테스트 및 검증

#### 테스트 전략

**1. 단위 테스트**

* Mockito를 활용한 락 동작 검증
* 예외 상황별 처리 로직 테스트
* JUnit 5 기반 포괄적 테스트 작성

**2. 통합 테스트**

* Redis + MySQL 실제 연동 테스트
* H2 Database를 활용한 빠른 테스트 실행
* Spring Boot Test 활용

**3. 수동 검증**

* 동시 요청 시나리오 직접 테스트
* 재고 정확성 수동 확인
* 에러 로그 분석

#### 검증 결과

* **데이터 일관성**: 재고 중복 차감 0건
* **락 안전성**: 예외 상황에서도 100% 락 해제
* **성능**: 락 경합 상황에서도 안정적 응답 시간 유지

### 개발 환경 및 도구

#### 모니터링

* **로깅**: SLF4J + Logback
* **메트릭**: Spring Boot Actuator
* **알림**: 락 획득 실패 시 경고 로그

### 학습

**1. 분산 시스템 개념**

* CAP 정리: 일관성 vs 가용성 트레이드오프
* 분산 락 패턴: Redisson 내부 동작 원리

**2. 동시성 제어**

* 비관적 락 vs 낙관적 락 적용 시나리오
* 데드락 방지: 락 순서와 타임아웃 설정

**3. Spring 생태계**

* @Transactional: 트랜잭션 전파와 격리 수준
* JPA 락 모드: LockModeType 활용
* Spring Boot 테스트 전략

#### 문제 해결 과정

**복잡한 동시성 문제 해결**

* 체계적인 분석과 단계별 접근
* 테스트 주도 개발의 중요성

```
```
