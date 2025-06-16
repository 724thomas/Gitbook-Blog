# Domain Driven Design

## ㄷㄷㄷ: 레거시 서버에 DDD를 적용하여 MSA로 전환한 경험 공유

## 1. Domain Driven Design?

### 배경

* 파트너 사이트는 카카오페이지 앱에서 제공되는 모든 컨텐츠에 대한 관리 및 그에 따른 기능 등을 제공
* 출판사/제작사 등에서 직접 파트너 사이트를 사용하여 컨텐츠의 생성, 업로드 등을 진행
* 작품에 대한 메타 정보와 판매 상태 등을 관리
* 정산이나 통계 정보 조회
* Legacy Server
  * Monolithic 구성
  * 기술 부채
  * 유지보수의 어려움
  * 기능의 고착화

### 특징

* 도메인의 모델과 로직에 집중
* Ubiquitous Language, 보편적 언어 사용 (단일화 된 의사소통 방식)
* Software Entity와 Domain간 개념의 일치

## 2. Why DDD?

![Image](https://github.com/user-attachments/assets/667f5024-41d5-4d78-a0ea-4d23b1d972e4)

* 필요한 기능들을 구별하고, 순차적으로 수정하며 개선하는 방식
* TDD/BBD 대신 사용.
* 레거시 서버의 큰 틀은 이미 존재했으며, 전체를 갈아엎는 것이 아니라 점진적 전환이 필요.
* 리소스 부족(사람/시간)에 대응하기 위해 레거시와 신규를 동시 운영

## 3. DDD 적용에 필요했던 것들

* 중요 3가지 개념들

#### Bounded Context | Context Map | Aggregate

### Bounded Context - 분리된 영역/경계

* 각 영역이 연관된 부분끼리 하나의 서비스로 묶임

![Image](https://github.com/user-attachments/assets/3879102d-9efb-481d-b75d-ad020548bd3a)

* 범위를 구분해 놓은 하위 도메인 개념인 Bounded Context들은 MSA에서 각각의 서비스로 나뉘게 됨
* 서로 데이터 조회 등을 위해서 API를 사용하여 통신하게 하면서 각 도메인을 철저하게 분리

### Context Map - Bounded Context 간의 관계를 보여줌

* 여러 bounded contextrㅏ 존재할때, 각각 서로 어떻게 연결되는지 나타내는 지도.

![Image](https://github.com/user-attachments/assets/f6e2304d-9e70-437a-b7da-b323511c2f4a)

* 업스트림(정보를 제공하는 쪽)과 다운스트림(정보를 소비하는 쪽)을 시각적으로 표현하여 한눈에 볼 수 있음
* 업/다운스트림 구조가 명확해짐으로 팀간 협업과 책임 분리 수월

### Aggregate - 데이터 변경 단위

![Image](https://github.com/user-attachments/assets/4f6e80e3-89df-4d7b-98cf-ad2cfdf4dfc8)

* 예시) Video product라는 Aggregate
* 일종의 라이프 사이클이 같은 도메인들을 한데 모아놓은 집합 개념
* 각 도메인 영역을 대표하는 도메인 객체들의 집합을 설계한 것
  * 컨텍스트 맵을 가지고 서비스 각각을 구현하게 될 때 서비스들이 가지고 있는 많은 객체들을 그대로 사용하면 의미가 없음
* Root Entity를 통해서만 하위 Entity에 접근(CRUD) 가능
* VideoProduct에서 Encoding 정보 등은 **Root Entity를 통해서만 접근 가능**
* 장점
  * 개별 객체들의 상호작용보다는 갳체들 사이의 관계를 조금 더 넓은 시야로 바라볼 수 있음
  * 제약 사항들을 하나의 맥락으로 관리할 수 잇음
  * Aggregate 들의 관계를 봄으로서 좀더 비즈니스 로직에 집중할 수 있음

![Image](https://github.com/user-attachments/assets/83e07bb6-12e6-4e06-84a1-643226aa296e)

```
- 상품 관리 서비스라는 bounded context 내부에 다양한 Aggregate존재
- 작품 Aggregate와 각 Aggregate를 연결해서 상품 관리시에 사용하게 됨
```

## 4. Architecture

DDD의 대표적인 3가지 아키텍처

#### Layered | Clean | Hexagonal

### Layered

![Image](https://github.com/user-attachments/assets/bf41d678-9e2c-4953-96e3-bd3f49c1ee3c)

* 4개의 계층으로 구성.
  * UI: input, output을 담당
  * Application: 유스케이스(도메인 객체의 사용법을 정의)를 구현하는 계층
  * Domain: 도메인과 관련된 직접적인 기능과 객체들(비즈니스 룰 등)
  * Infrastructure: 기술적인 기능을 제공해주는 계층(Repo 또는 메시지 전송 같은)

### Clean

![Image](https://github.com/user-attachments/assets/b8cea896-3b0a-4c47-b1e6-9d3b9b42c190)

* External Interface: DB 또는, Front Framework를 기반으로 한 UI
* Interface Adapter: DB, 파일 시스템같은 외부 소스에 저장하고 내부의 usecase를 위해 양방향으로 데이터 변화를 수행하는 역할
* Use Case: 비즈니스나 애플리케이션 로직으로 구성된 계층
* Entity: 엔티티 또는 도메인으로 구성된 도메인 계층

### Hexagonal

![Image](https://github.com/user-attachments/assets/c2b34dac-7a9a-4e38-8a15-b7e63883de3d)

* 알맞은 포트에 어댑터(구현체)를 끼워가며 사용
* 비즈니스 로직이 표현 로직이나 데이터 접근 로직에 의존적이지 않는 것이 목표

아키텍처 선택 (헥사고날)

시각적으로도 헥사고날 구조는 각 포트와 어댑터가 잘 구분되어 **구조적 명확성**과 **유지보수 용이함**

* 레이어드:
  * 비즈니스 로직이 거대해 지면서 애플리케이션 레이어가 오염되는 경우가 염려되어 제외
* 클린:
  * ‘포트 앤 어댑터’라는 명확한 구현 개념이 포함된 헥사고날을 선택하며서 클린 아키텍처 제외

### 코드 예시 (헥사고날을 적용한 파트너 사이트 예시)

![Image](https://github.com/user-attachments/assets/f2f175b0-054b-4317-8c45-b734dbcab7b8)

1. 유저 판매 요청 (컨트롤러 → 서비스)
2. 애플리케이션 및 도메인에서 필요한 처리
3. Repository에서 영속 어댑터로 데이터 변경에 대한 처리
4. 처리 결과는 notification system 쪽으로 알림을 위한 메시지를 발행

#### 도메인

![Image](https://github.com/user-attachments/assets/0101fc22-3290-4e96-8573-5827dcfcb1b7)

* 알맞은 상품인지에 대한 검증 로직과 상품 판매 시작 등의 “도메인 로직” 이 들어감

#### 컨트롤러와 유즈케이스

![Image](https://github.com/user-attachments/assets/15fb5450-b553-456b-8aca-bc8314478613)

* 상품을 판매하는 유즈케이스를 정의하여 서비스에서 해당 로직 구현

#### 유즈케이스 구현체(서비스)

![Image](https://github.com/user-attachments/assets/438f8681-ffdb-4bfb-be88-32b06ac3bac2)

* 도메인 로직을 실행하도록 하고 포트로 통해 알림을 발송하거나 DB 접근 등을 수행
* Load와 Save 포트는 데이터 접근에 대한 로직을 갖고 있음
  * crud 중 read를 제외한 나머지는 save로 나누어 CQRS 패턴. 커맨드와 쿼리 책임을 분리하기 위함
  * 이후 조회 성능 개선을 위해 조회를 Event-driven으로 바꿀 수 있음

#### DB 포트 구현체 (어댑터)

![Image](https://github.com/user-attachments/assets/d02f088b-afe0-41f4-b579-fe12e000bf67)

* Load, Save 포트는 퍼시스턴트 어뎁터에서 구현
* Mapper: Jpa Entity ↔ Product Entity 양방향 변경을 해주어야함
* 포트에 정의된 대로 DB 접근 로직을 구현

#### Mapper

![Image](https://github.com/user-attachments/assets/0eb6959e-bc62-4e1d-a466-d8aadd0da6ca)

* JpaEntity ↔ DomainEntity 간의 변환을 돕는 로직을 가짐
* 도메인 캡슐화를 유지하면서 ORM 프레임워크(JPA 등)와의 결합을 피함

## 5. There is No Silver Bullet

#### Cons

1. MSA에서 오는 단점은 별개로…
   * 복잡해진 개발 난이도(숙련도), 트랜잭션 관리 및 통합 테스트 어려움, 배포 복잡도
2. Architecture 구현에서 생성되는 생각보다 많은 코드
   * 도메인을 사용하기 위한 별도의 코드 필요(mapper 등)
3. 각 도메인에 대한 높은 이해도가 필요
   * 도메인 이해를 위한 시간

#### Pros (시스템 관점)

1. 업무용어 통일에 따른 빠른 커뮤니케이션
2. Aggregate를 사용하여 도메인간 관계 정리
3. 도메인의 분리에 따른 유지보수에 대한 편의성
4. 새로운 기능 및 요구 사항에 대한 유연성

#### Pros (개발자 관점)

1. Aggregate 사용으로 인한 도메인 캡슐화가 자동으로 진행됨
   1. 도메인 내부 VO, Entity, Domain 등에 대한 접근 캡슐화
2. Loose Coupling, High cohesion
   1. Usecase와 Port로 인한 의존도 낮추고 도메인 로직으로 응집도 높임
3. Domain Logic의 분리로 Business Logic에 집중
4. 코드 가독성
