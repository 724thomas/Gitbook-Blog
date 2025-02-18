# ERD

## 1. User

사용자는 이커머스 플랫폼에서 상품을 구매하거나 판매하는 역할

<figure><img src="../../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

* **User : Role = N:1** (각 사용자는 하나의 역할을 가짐)
* **User : Order = 1:N** (각 사용자는 여러 개의 주문을 가질 수 있음)
* **User : Cart = 1:1** (각 사용자는 하나의 장바구니만 보유 가능)
* **User : ShippingAddress = 1:N** (각 사용자는 여러 개의 배송지를 가질 수 있음)

## 2. Role

일반 사용자, 판매자, 관리자로 구분

<figure><img src="../../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

## 2. Product

* **Product : OrderItem** = **1:M** (하나의 상품은 여러 개의 주문에 포함될 수 있다.)
* **Product : Review** = **1:M** (하나의 상품은 여러 개의 리뷰를 받을 수 있다.)

## 3. ShippingAddress

사용자는 여러 개의 배송지를 저장할 수 있으며, 주문 시 특정 배송지를 선택

<figure><img src="../../.gitbook/assets/image (311).png" alt=""><figcaption></figcaption></figure>

* **User : ShippingAddress = 1:N** (각 사용자는 여러 개의 배송지를 가질 수 있음)
* **ShippingAddress : Order = 1:N** (각 배송지는 여러 개의 주문과 연결될 수 있음)

## 4. Cart

사용자는 하나의 장바구니를 가지며, 장바구니에는 여러 개의 상품이 포함

<figure><img src="../../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>

* **User : Cart = 1:1** (각 사용자는 하나의 장바구니만 가질 수 있음)
* **Cart : CartItem = 1:N** (각 장바구니는 여러 개의 상품을 포함 가능)

## 5. CartItem (Cart : Product. M:M)

장바구니에 담긴 개별 상품 정보를 저장

<figure><img src="../../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure>

* **Cart : CartItem = 1:N** (각 장바구니는 여러 개의 상품을 포함 가능)
* **Product : CartItem = 1:N** (각 상품은 여러 개의 장바구니에 포함될 수 있음)

## 6. Order

사용자가 결제를 완료하면 생성되는 주문 정보를 저장하는 테이블

<figure><img src="../../.gitbook/assets/image (314).png" alt=""><figcaption></figcaption></figure>

* **User : Order = 1:N** (각 사용자는 여러 개의 주문을 할 수 있음)
* **ShippingAddress : Order = 1:N** (각 배송지는 여러 개의 주문에 사용될 수 있음)
* **Order : OrderItem = 1:N** (각 주문에는 여러 개의 상품이 포함될 수 있음)
* **Order : Payment = 1:1** (각 주문에는 하나의 결제 정보가 연결됨)
* **Order : Shipping = 1:1** (각 주문에는 하나의 배송 정보가 연결됨)

## 7. OrderItem (Order : Product. M:M)

주문 내 개별 상품 정보를 저장하는 테이블

<figure><img src="../../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

* **Order : OrderItem = 1:N** (각 주문에는 여러 개의 상품이 포함될 수 있음)
* **Product : OrderItem = 1:N** (각 상품은 여러 개의 주문에서 참조될 수 있음)

## 8. Product

판매자가 등록한 상품 정보를 저장

<figure><img src="../../.gitbook/assets/image (316).png" alt=""><figcaption></figcaption></figure>

* **User : Product = 1:N** (각 판매자는 여러 개의 상품을 등록할 수 있음)
* **Category : Product = 1:N** (각 카테고리에는 여러 개의 상품이 포함될 수 있음)
* **Product : OrderItem = 1:N** (각 상품은 여러 개의 주문에서 참조될 수 있음)
* **Product : CartItem = 1:N** (각 상품은 여러 개의 장바구니에 포함될 수 있음)
* **Product : Review = 1:N** (각 상품은 여러 개의 리뷰를 가질 수 있음)

## 9. Category

상품을 분류하는 카테고리 테이블

<figure><img src="../../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

**Category : Product = 1:N** (각 카테고리에는 여러 개의 상품이 포함될 수 있음)

## 10. Payment

주문 결제 정보를 저장하는 테이블

<figure><img src="../../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

* **Order : Payment = 1:1** (각 주문은 하나의 결제와 연결됨)

## 11. Shipping

주문별 배송 상태 및 송장 번호를 관리하는 테이블

<figure><img src="../../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

* **Order : Shipping = 1:1** (각 주문은 하나의 배송과 연결됨)

## 12. Promotion

할인 쿠폰 및 이벤트 적용 정보를 저장하는 테이블

<figure><img src="../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

* **Order : Promotion = N:M** (하나의 주문에는 여러 개의 프로모션이 적용될 수 있음)

## 13. Review

사용자가 구매한 상품에 대한 리뷰 및 평점을 기록하는 테이블\
리뷰는 **배송 완료 후** 작성할 수 있으며, **상품별 평균 평점**을 계산

<figure><img src="../../.gitbook/assets/image (321).png" alt=""><figcaption></figcaption></figure>

* **User : Review = 1:N** (각 사용자는 여러 개의 리뷰를 작성 가능)
* **Product : Review = 1:N** (각 상품은 여러 개의 리뷰를 가질 수 있음)
* **OrderItem : Review = 1:1** (각 주문된 상품은 하나의 리뷰와 연결 가능)

## 14. Admin

관리자(운영자) 계정 정보를 저장하는 테이블 (일반 사용자와 분리)

<figure><img src="../../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure>

* **Admin 계정은 독립적으로 관리되며 User 테이블과 연결되지 않음**
* **관리자 역할(role)에 따라 권한을 부여할 수 있도록 구현 가능**

## 15. Statistics

<figure><img src="../../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure>

* **Product : Statistics = 1:N** (가장 많이 판매된 상품 ID와 연결 가능)
* **자동화 배치 작업을 통해 주기적으로 데이터가 업데이트됨**

## 16. EventLog

<figure><img src="../../.gitbook/assets/image (324).png" alt=""><figcaption></figcaption></figure>

* **User : EventLog = 1:N** (사용자와 연결된 이벤트 기록 가능.

## 17. WishList

사용자가 관심 있는 상품을 저장 (장바구니와 별도)

<figure><img src="../../.gitbook/assets/image (325).png" alt=""><figcaption></figcaption></figure>

* **User : Wishlist = 1:N** (사용자는 여러 개의 상품을 찜할 수 있음)
* **Product : Wishlist = 1:N** (하나의 상품이 여러 사용자의 찜 목록에 포함될 수 있음)

## 18. Notification

사용자에게 주문 상태 변경, 배송 정보 업데이트, 프로모션 등을 알리는 시스템

<figure><img src="../../.gitbook/assets/image (326).png" alt=""><figcaption></figcaption></figure>

* **User : Notification = 1:N** (각 사용자는 여러 개의 알림을 받을 수 있음)

## 19. Coupon

사용자가 사용할 수 있는 할인 쿠폰 정보 테이블

<figure><img src="../../.gitbook/assets/image (327).png" alt=""><figcaption></figcaption></figure>

* **Order : Coupon = N:M** (하나의 주문에는 여러 개의 쿠폰이 적용될 수 있음)
