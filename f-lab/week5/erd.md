---
description: admin, api, batch, common
---

# ERD - 1

## Tables & Relations

(j=i+1; j\<n; j++)

* User - 유저정보
  * Role - User가 어떤 권한을 가질지 Role 테이블로 관리 (N:1)
  * ShippingAddress - User는 여러 배송지를 가질 수 있음 (1:N)
  * Cart - 한 사용자는 하나의 장바구니를 가짐 (1:N)
  * Order - 한 사용자는 여러 주문을 생성 (1:N)
  * Product - 판매자일 경우, 여러 상품을 등록할 수 있음 (1:N)
  * Review - 유저는 여러개의 리뷰를 작성할 수 있음 (1:N)
  * EventLog - 유저의 비즈니스 로그(서버 로그X) 저장 (1:N)
  * WishList - 유저는 여러개의 제품을 찜 목록에 둘 수 있음 (1:N)
  * Notification - 특정 사용자에게 여러 알림이 발송될 수 있음 (1:N)
  * Coupon - 유저가 해당 제품 또는 어떠한 제품에 대해 사용할 수 있는 쿠폰 (1:N)
* Role - 권한
* ShippingAddress - 배송지
  * Order - 주문시 어떤 배송지를 사용할지 저장 (1:1)
* Cart - 장바구니
  * CartItem - 장바구니 안에 여러 상품이 포함됨 (1:N)
* CartItem - 장바구니에 담긴 제품
  * Product - 어떤 상품은 여러 장바구니에 담길 수 있음 (N:1)
* Order - 주문
  * OrderItem - 한 주문에는 여러 제품이 포함 될 수 있음 (1:N)
  * Payment - 한 주문에 대한 하나의 결제 정보가 연결되어야 함(1:1)
  * Shipping - 한 주문당 하나의 배송 정보가 연결되어야 함(1:1)
* OrderItem - 주문과 제품 N:N&#x20;
* Product - 제품
  * Category - 한 제품은 하나의 카테고리를 가질 수 있음(1:1)
  * Promotion - 한 제품은 여러개의 프로모션에 해당 될 수 있음(1:N)
  * Review - 한 제품은 여러개의 리뷰를 가질 수 있음 (1:N)
  * WishList - 여러 제품은 여러 유저의 찜목록에 포함될 수 있음 (N:N)
  * Coupon - 한 제품은 여러개의 쿠폰을 가질 수 있음 (1:N)
* Category - 제품 카테고리
* Payment - 주문 결제 정보를 저장
* Shipping - 배송 상태 및 송장 번호를 관리하는 테이블
* Promotion - 할인 쿠폰 및 이벤트 적용 정보를 저장하는 테이블
* Review - 리뷰
* Admin - 관리자 계정
* Statistics - 통계
* EventLog - 로그
* WishList - 찜 목록
* Notification - 알림
* Coupon - 사용자 쿠폰



## ERD (Draft)

<figure><img src="../../.gitbook/assets/image (328).png" alt=""><figcaption></figcaption></figure>

```sql
CREATE DATABASE Ecommerce;
USE Ecommerce;

-- 1. Role (사용자 역할)
CREATE TABLE Role (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name ENUM('CUSTOMER', 'SELLER', 'ADMIN') NOT NULL UNIQUE
);

-- 2. User (사용자)
CREATE TABLE User (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    role_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (role_id) REFERENCES Role(id)
);

-- 3. Category (카테고리)
CREATE TABLE Category (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL UNIQUE
);

-- 4. Product (상품)
CREATE TABLE Product (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT NOT NULL,
    user_id BIGINT NOT NULL, -- 판매자
    category_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(id),
    FOREIGN KEY (category_id) REFERENCES Category(id)
);

-- 5. ShippingAddress (배송지)
CREATE TABLE ShippingAddress (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    address VARCHAR(500) NOT NULL,
    city VARCHAR(100) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,
    country VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(id)
);

-- 6. Cart (장바구니)
CREATE TABLE Cart (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(id)
);

-- 7. Order (주문)
CREATE TABLE `Order` (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    shipping_address_id BIGINT NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    status ENUM('PENDING', 'PAID', 'SHIPPED', 'DELIVERED', 'CANCELLED') NOT NULL DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(id),
    FOREIGN KEY (shipping_address_id) REFERENCES ShippingAddress(id)
);

-- 8. Payment (결제 정보)
CREATE TABLE Payment (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT NOT NULL UNIQUE,
    payment_method ENUM('CREDIT_CARD', 'PAYPAL', 'BANK_TRANSFER') NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    status ENUM('PENDING', 'COMPLETED', 'FAILED') NOT NULL DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES `Order`(id)
);

-- 9. Shipping (배송 정보)
CREATE TABLE Shipping (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT NOT NULL UNIQUE,
    tracking_number VARCHAR(255) NOT NULL,
    status ENUM('PENDING', 'IN_TRANSIT', 'DELIVERED', 'RETURNED') NOT NULL DEFAULT 'PENDING',
    estimated_delivery DATE,
    FOREIGN KEY (order_id) REFERENCES `Order`(id)
);

-- 10. CartItem (장바구니 아이템)
CREATE TABLE CartItem (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    cart_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL DEFAULT 1,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (cart_id) REFERENCES Cart(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES Product(id) ON DELETE CASCADE
);

-- 11. OrderItem (주문 아이템)
CREATE TABLE OrderItem (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES `Order`(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES Product(id)
);

-- 12. Promotion (할인 정보)
CREATE TABLE Promotion (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    discount_percentage DECIMAL(5,2) NOT NULL,
    valid_from DATE NOT NULL,
    valid_until DATE NOT NULL
);

CREATE TABLE Order_Promotion (
    order_id BIGINT NOT NULL,
    promotion_id BIGINT NOT NULL,
    PRIMARY KEY (order_id, promotion_id),
    FOREIGN KEY (order_id) REFERENCES `Order`(id) ON DELETE CASCADE,
    FOREIGN KEY (promotion_id) REFERENCES Promotion(id) ON DELETE CASCADE
);

-- 13. Coupon (쿠폰 정보)
CREATE TABLE Coupon (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    code VARCHAR(50) NOT NULL UNIQUE,
    discount_amount DECIMAL(10,2) NOT NULL,
    expiration_date DATE NOT NULL
);

CREATE TABLE Order_Coupon (
    order_id BIGINT NOT NULL,
    coupon_id BIGINT NOT NULL,
    PRIMARY KEY (order_id, coupon_id),
    FOREIGN KEY (order_id) REFERENCES `Order`(id) ON DELETE CASCADE,
    FOREIGN KEY (coupon_id) REFERENCES Coupon(id) ON DELETE CASCADE
);

-- 14. Review (리뷰)
CREATE TABLE Review (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    order_item_id BIGINT NOT NULL UNIQUE,
    rating INT CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(id),
    FOREIGN KEY (product_id) REFERENCES Product(id),
    FOREIGN KEY (order_item_id) REFERENCES OrderItem(id)
);

-- 15. Admin (운영자 계정)
CREATE TABLE Admin (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL
);

-- 16. Statistics (상품 통계)
CREATE TABLE Statistics (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    product_id BIGINT NOT NULL,
    total_sales INT DEFAULT 0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES Product(id)
);

-- 17. EventLog (사용자 활동 기록)
CREATE TABLE EventLog (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    event_type VARCHAR(255) NOT NULL,
    event_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(id)
);

-- 18. WishList (찜 목록)
CREATE TABLE WishList (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(id),
    FOREIGN KEY (product_id) REFERENCES Product(id)
);

-- 19. Notification (알림)
CREATE TABLE Notification (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    message TEXT NOT NULL,
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(id)
);

```



##



## 1. User

사용자는 이커머스 플랫폼에서 상품을 구매하거나 판매하는 역할

<figure><img src="../../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

* **User : Role = N:1** (각 사용자는 하나의 역할을 가짐)
* **User : Order = 1:N** (각 사용자는 여러 개의 주문을 가질 수 있음)
* **User : Cart = 1:1** (각 사용자는 하나의 장바구니만 보유 가능)
* **User : ShippingAddress = 1:N** (각 사용자는 여러 개의 배송지를 가질 수 있음)
* User: WishList = 1 : N (각 사용자는 하나의 찜 목록을 가질 수 있음)

## 2. Role

일반 사용자, 판매자, 관리자로 구분

<figure><img src="../../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

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







피드백 :&#x20;

1. Role은 있지만 권한이 없음. 권한 관리 체계 공부. (Enum을 HardCoding하지 않도록)
2. User : Cart는 1:N로 여러 level 별로 만들 수 있지만, MVP에서는 1:1로만.
3. 유저 테이블 Password 직접 관리? Password 관리 방식 Salt 공부.
4. CreatedAt, ModifiedAt
5. EventLog - EventType:종류(Label), Payload
6. Admin, UserTable 분리가  맞다. Salt 추가. Admin을 따로 관리하게 되면 인증을 분리하고 다르게. 실제 기획에서는 admin 인증은 다르게 되어 있다.
7. 3P를 하려면, User - Product 사이 Merchant-Vendor-Product(계층) 등의 계층 추가.
8. Category: 실제로는 세금, 면세 품목 같은걸로 분류된다. 하지만 여기서는 Product기준으로
9. Product, Review에 이미지 Url이 들어가야한다.
10. User-Coupon연결이 안되어있다. Order랑 연결되어야한다. 만료, 중복할인 규칙 정해야한다.
11. Promotion 제거하는게 좋을거 같음.
12. Order - Shipping 좀 이상함. OrderItem마다 운송장이 있어야하지 않나?
13. Seller를 없앤다. 3PL로 하지말고, Retail로 한다.
14. 로그인 테이블 JWT
15. JWT Payload - LifeCycle. JWT공부
