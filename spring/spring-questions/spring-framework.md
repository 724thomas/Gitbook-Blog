---
description: 스프링 프레임워크의 기본 개념
---

# Spring Framework

**Spring Framework와 IoC/DI의 개념과 동작 원리**

## Spring Framework가 무엇인지 설명해주세요.

"자바의 오픈소스 애플리케이션 프레임워크 중 하나로, 스프링의 기본 철학은 특정 기술에 종속되지 않고 객체를 관리할 수 있는 프레임워크를 제공하는 것입니다. 컨테이너로 자바 객체를 관리하면서 의존성 주입과 제어의 역전을 통해 결합도를 낮추게 됩니다."

* **Spring Framework**는 **자바 기반의 경량화된 엔터프라이즈 애플리케이션 프레임워크**로, 복잡한 자바 애플리케이션 개발을 단순화하기 위해 설계
* **의존성 주입(DI, Dependency Injection)**&#xACFC; **제어의 역전(IoC, Inversion of Control)**&#xC744; 기반으로 하여 객체 간의 의존 관계를 효율적으로 관리하고 결합도를 낮춥니다.
* **트랜잭션 관리**, **AOP(Aspect-Oriented Programming)**, **MVC 아키텍처** 지원, **데이터 접근 통합**과 같은 다양한 모듈을 제공
* **POJO 기반 개발**을 지원하여 단순한 자바 객체로 비즈니스 로직을 구현할 수 있게 도와줍니다.
* 추가로, 테스트 용이성, 경량 컨테이너 역할이 있습니다.

## IoC(Inversion of Control, 제어의 역전)란 무엇이며, 어떻게 동작하나요?

"제어의 역전은 제어권이 사용자에게 있지 않고, 프레임워크에 있어서 필요에 따라 사용자의 코드를 호출하게 됩니다. 스프링에서는 인스턴스의 생성부터 소멸까지 개발자가 아닌 컨테이너에서 대신 관리하게 됩니다."

* 애플리케이션의 흐름을 개발자가 직접 제어하는 것이 아니라, **프레임워크나 컨테이너가 그 흐름을 제어하는 디자인 패턴**
* 애플리케이션 코드가 필요한 객체가 직접 생성하고 의존성을 관리하는 작업을 IOC에서는 스프링 IOC 컨테이너가 대신 수행합니다.
* IoC는 DI를 통해 구현되며, DI는 객체가 필요한 의존성을 외부에서 주입받는 방식으로 객체의 생성과 의존성 설정을 컨테이너가 담당합니다.&#x20;
* 의존성에 따라 객체의 생성 순서를 IoC 컨테이너가 자동으로 관리

## DI(Dependency Injection, 의존관계 주입)의 개념과 동작 방식을 설명해주세요.

"의존성 주입은 객체간의 의존 관계를 미리 설정해두면 스프링 컨테이너가 의존 관계를 자동으로 연결해줍니다. 이렇게 되면 직접 의존하는 객체를 생성하여 가져올 필요가 없어서 결합도가 낮아지는 장점이 있습니다"

* **DI(Dependency Injection, 의존성 주입)**&#xB294; 객체가 자신의 의존성을 스스로 생성하지 않고, 외부에서 주입받는 설계 패턴
* 생성자, Setter, 필드 주입(@Autowired) 방식이 사용됩니다.



## Spring에서 IoC와 DI를 어떻게 지원하나요?

IoC 지원 방식

* IoC 컨테이너는 애플리케이션이 시작 될때 모든 객치(빈)을 미리 생성하고 각 객체가 필요한 의존성을 알아서 주입해 줍니다.
* 개발자는 빈을 정의하고, 그 빈이 의존하는 다른 빈들을 설정해 놓으면 컨테이너가 객체를 생성하고, 그 객체들이 서로 의존하는 다른 객체를 주입해 주면서 전체 애플리케이션의 흐름을 제어하게 됩니다.

DI 지원 방식

* DI는 IoC의 한가지 구현 방법입니다. 필요한 의존성을 외부에서 자동으로 주입하는 방식입니다.
* 생성자 주입, Setter 주입, 필드 주입(@Autowired) 방식이 있습니다.



## DI의 종류와 그 차이점에 대해 설명해주세요.

#### 생성자 주입: **생성자를 통해 의존성**을 주입하는 방식

생성자를 통해 의존성을 주입하는 방식입니다. 객체가 생성될 때 필수 의존성을 함께 주입받습니다.

장점

* 불변성: final
* 필수 의존성을 강제 (누락 없이 주입됨)
* 테스트 용이성

단점

* 설계를 잘못하면 생성자가 복잡해집니다.

#### 필드 주입: @Autowired 어노테이션 사용

객체 생성 시 의존성이주입되지만, 불변성을 보장하지 않습니다.

단점

* 테스트 용이성이 떨어짐.
* 불변성 보장 X
* 리플랙션을 사용하여 객체 상태 예측 불가

#### 세터 주입: **Setter 메서드**를 사용하여 의존성을 주입하는 방식

객체  생성시 의존성이 필수가 아닌 경우 주입할 수 있으며 객체 생성 후에도 주입할 수 있습니다.

장점

* 선택적 의존성을 설정할 수 있습니다. 상황에 따라 주입할 수도 안할 수도.

단점

* 불변성 보장 X
* 런타임에 의존성 누락 가능성





**Spring Bean과 IoC 컨테이너**

## Spring Bean이란 무엇인가요?

* 스프링 IoC 컨테이너에 의해 관리되는 객체
* @Component, @Service, @Repository, @Controller 등의 어노테이션 또는 @Configuration 클래스 내에 @Bean 메서드로 정의됩니다.
* **기본적으로 싱글톤(Singleton) 스코프**로 생성됩니다.
* 생성 -> 의존성 주입 -> 초기화(@PostConstruct) -> 사용 -> 소멸(@PreDestroy)



## Bean의 생명주기와 생성 과정을 설명해주세요.

* 생성: 빈의 인스턴스화\
  **싱글톤(Singleton) 스코프**의 빈은 **IoC 컨테이너가 초기화될 때** 생성\
  **프로토타입(Prototype) 스코프**의 빈은 빈이 필요할 때 요청으로 인해 생성
* 의존성 주입\
  스프링 컨테이너가 자동으로 의존성을 주입하며, 생성자/Setter 로 주입됩니다.
* 초기화\
  @PostConstruct를 통해 초기화 작업을 정의할 수 있습니다.\
  예를 들어,의존성 주입 후 초기화해야 할 로직이 있을 때, 또는 초기 상태를 설정해야 할 때 유용
* 사용
* 소멸\
  싱글톤 빈의 경우 애플리케이션 컨텍스트가 종료될 때 소멸\
  프로토타입 빈은 컨테이너에서 직접 관리하지 않기 때문에 별도의 소멸 과정이 없음\
  @PreDestroy 어노테이션을 사용하여 종료 전 작업을 정의할 수 있습니다.\
  예를 들어, 파일 스트림을 닫거나, 데이터베이스 연결을 종료하거나, 캐시를 비우는 등의 작업에 사용됩니다.



## Bean Scope에 대해 설명해주세요.

**스프링 IoC 컨테이너**가 빈의 **생명주기**와 **범위**를 정의하는 방법

빈이 언제 생성되고, 얼마나 오래 살아남으며, 몇 개의 인스턴스가 만들어지는지를 결정하는 설정

* 싱글톤: 하나의 인스턴스 공유 (기본 스코프, 스프링 컨테이너 기반, 상태 저장X)
* 프로토타입: 요청시마다 새로운 인스턴스 생성
* 요청: HTTP 요청마다 새로운 인스턴스 생성 (HTTP 요청이 처리되는 동안에만 유효: 비상태성)
* 세션: HTTP 세션마다 새로운 인스턴스 생성 (로그인 정보: 상태성)
* 애플리케이션: 애플리케이션 전체에서 하나의 인스턴스를 공유 (서블릿 컨텍스트 기반, 상태 저장)



## IoC 컨테이너의 역할은 무엇인가요?

* 애플리케이션에서 사용되는 **객체(빈, Bean)**&#xB4E4;을 생성하고 관리하며, 빈의 **생명주기**를 제어하는 역할
* 의존성 주입을 통해 객체가 필요한 의존성을 외부에서 주입해줍니다.
* 다양한 스코프로 빈을 관리. (싱글톤 프로토타입 등)
* ApplicationContext와 BeanFactory 인터페이스를 통해 구현됩니다.



## @Component와 @Bean의 차이는 무엇인가요?

@Component

* **클래스 레벨**에서 사용되며, **자동으로 빈을 등록**하는 어노테이션
* Service, Controller, Repository가 Component의 특수화된 형태

@Bean

* **메서드 레벨**에서 사용되며, 개발자가 명시적으로 **메서드를 통해 빈을 정의**할 때 사용
* @Configuration 에서 사용되며 직접 메서드를 통해 빈의 생성 로직을 명시적으로 작성



## Autowiring 과정에 대해 설명해주세요.

* 객체(빈) 간의 의존성을 자동으로 연결해 주는 기능 (주로 @Autowired 사용)
* IoC 컨테이너는 애플리케이션이 시작될 때 모든 **빈(Bean)**&#xC744; 생성
* **@Autowired**가 적용된 필드, 생성자, 또는 Setter 메서드에서 해당 빈의 **타입**을 기준으로 적절한 의존성을 자동으로 찾습니다.
* 찾은 빈을 **자동으로 주입**합니다. 만약 여러 개의 빈이 존재할 경우, **빈 이름** 또는 **@Qualifier** 어노테이션을 통해 어느 빈을 주입할지 지정할 수 있습니다.





**의존성 주입과 생성자 주입**

## 생성자 주입을 사용하는 이유와 그 장점은 무엇인가요?

* 불변성(final) - 동시성 문제를 피함
* 필수 의존성을 강제로 주입 - 의존성 누락X. 누락시 애플리케이션 실행 초기에 잡을 수 있습니다.
* 테스트 용이성 - 목 객체를 쉽게 주입.
* 순환 의존성을 애플리케이션 실행 시점에 감지

## 의존성과 설정값을 생성자 인자로 주입해야 하는 이유는 무엇인가요?

* 필수 의존성을 강제
* 불변성(Immutable)을 보장
* 객체의 일관성을 보장 (완전히 초기화된 상태. Setter는 불완전함)
* 테스트 용이성을 제공 (Mock 객체 주입이 쉽습니다.)





**POJO와 Annotation**

## POJO란 무엇이며, Spring Framework에서 어떻게 활용되나요?

Java가 처음 나왔을때 객체 지향적인 코드를 작성하고자 했는데, EJB로 인해 객체 지향적이지 않는 경우가 있었고, 그 후에 다시 객체 지향적인걸 따르자고 해서 POJO가 나왔다.

**Plain Old Java Object**의 약자로, **특정 프레임워크, 라이브러리, 기술에 의존하지 않는 객체**를 의미

* 예시) JPA의 @Entity @ID @GeneratedValue와 같은 JPA규약을 따르는 것. JPA는 POJO 객체를 데이터베이스의 테이블과 매핑해서 객체 지향적인 방식으로 DB와 상호작용할 수 있도록 해주는 기술.
* Java나 Java의 스펙에 정의된 것 이외에는 다른 기술이나 규약에 얽매이지 않아야 한다.
* 특정 환경이나 기술에 종속적이지 않으면 재사용이 가능하고, 확장 가능한 유연한 코드를 작성할 수 있다.
* 저수준 레벨의 기술과 환경에 종속적인 코드를 제거하여 코드를 간결해지며 디버깅하기에도 상대적으로 쉬워진다.
* 특정 기술이나 환경에 종속적이지 않기 때문에 테스트가 단순해진다.
* 객체지향적인 설계를 제한 없이 적용할 수 있다. (가장 중요한 이유)



## Annotation이란 무엇이고, 어떤 역할을 하나요?

**Annotation**은 **자바 코드에 메타데이터**를 추가하는 일종의 **표기법**

* 컴파일러에 힌트 제공 (@Deprecated)
* 런타임에 동작 정의(@Autowired)



## Spring에서 제공하는 대표적인 Annotation과 그 역할을 설명해주세요.

* Component: 스프링 빈 등록
* Autowired: 의존성 주입 자동으로 처리
* Configuration & Bean: 스프링 설정 클래스를 정의. Bean은 메서드에서 반환되는 객체를 스프링빈으로 등록
* Transactional: 트랜잭션 관리



## Java에서 제공하는 대표적인 Annotation과 그 역할을 설명해주세요

* Override
* Deprecated
