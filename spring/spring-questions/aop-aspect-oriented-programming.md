---
description: AOP
---

# AOP (Aspect-Oriented Programming)

**AOP의 개념과 적용**

## AOP(Aspect-Oriented Programming)란 무엇이며, 어떤 문제를 해결하기 위한 것인가요?

* **관심사의 분리(Separation of Concerns)**를 통해 공통적인 기능을 모듈화하여 코드의 중복을 줄이고 가독성을 높이는 프로그래밍 패러다임
* 로깅, 트랜잭션 관리, 보안, 예외처리 등을 비즈니스 로직과 분리하여 중복을 제거하는 것이 목표.



## 스프링에서 AOP를 어떻게 지원하나요?

* AOP는 프록시 패턴을 통해 구현됩니다.
* @Aspect 어노테이션과 부가 기능으로 @Before @After @Around 등의 에너테이션이 같이 사용됩니다.
* Aspect라는 개념을 사용하여 공통 기능을 정의하고, PointCut을 통해 어디에 적용할지 지정.
* AOP는 메서드 단위로 동작하면서 주로 서비스의 부가적인 기능을 처리합니다.



## 프록시 패턴이란 무엇이며, AOP에서 어떻게 활용되나요?

* **구조적 디자인 패턴**으로, 실제 객체에 대한 대리자 또는 중간 매개체 역할을 하는 객체를 생성하여, 그 객체를 통해 실제 객체에 접근하는 방식
* 실제 객체에 대한 **접근을 제어**하거나, 메서드 호출 전에 부가적인 작업을 처리하기 위한 **대리 객체**를 만드는 것이 목적
* 캐싱, 권한 확인, 로깅 등의 작업을 합니다.



## Aspect(측면)란 무엇이며, 어떤 구성 요소들이 있나요?

* AOP의 핵심 개념 중 하나(Advice, Pointcut, JoinPoint, Aspect)로, 프로그램의 여러 부분에서 공통적으로 사용되는 기능을 모듈화한 것
* Aspect는 주로 비즈니스 로직과는 별도로 존재하는 보조적인 기능을 처리합니다.



## 조인 포인트와 포인트컷에 대해 설명해주세요.

조인 포인트

* **Aspect**가 적용될 수 있는 프로그램 내의 **특정 실행 시점**을 의미. AOP에서는 주로 **메서드 실행 시점**을 조인 포인트로 다룹니다.
* 메서드 실행전, 실행 후, 예외가 발생했을때, 생성자가 호출될 떄 등이 조인 포인트

<details>

<summary>Example</summary>

조인 포인트의 핵심은 이러한 실행 지점이 Aspect가 개입할 수 있는 위치라는 점입니다. 조인 포인트는 프로그램의 흐름에서 애드바이스(Advice)를 삽입할 수 있는 지점을 말하며, 이를 통해 비즈니스 로직과 횡단 관심사를 분리할 수 있게 해줍니다.

#### 조인 포인트 예시

조인 포인트가 어떻게 AOP의 개념에서 활용되는지를 명확히 설명하기 위한 예시입니다:

```java
public class OrderService {

    // 이 메서드 실행 자체가 조인 포인트입니다.
    public void placeOrder() {
        System.out.println("Order placed successfully.");
    }

    // 메서드 실행 후에도 조인 포인트로 간주됩니다.
    public void cancelOrder() {
        System.out.println("Order canceled successfully.");
    }

    // 예외가 발생하면 해당 지점도 조인 포인트가 됩니다.
    public void refundOrder() throws Exception {
        System.out.println("Attempting to refund...");
        throw new Exception("Refund failed!");
    }
}
```

#### 조인 포인트 설명

* **메서드 실행 전**: `placeOrder()` 메서드가 호출되기 직전의 지점이 조인 포인트입니다.
* **메서드 실행 후**: `cancelOrder()` 메서드가 실행된 직후의 지점이 조인 포인트입니다.
* **예외 발생 시**: `refundOrder()` 메서드에서 예외가 발생하는 지점이 조인 포인트입니다.
* **생성자 호출**: 만약 `OrderService` 객체가 생성되는 시점도 조인 포인트가 될 수 있습니다.

이러한 다양한 실행 지점이 조인 포인트로 사용될 수 있으며, AOP에서 이 지점들을 선택하여 애드바이스를 적용하는 것이 가능합니다. 조인 포인트는 특정 지점에서 애드바이스가 실행될 수 있도록 하는 모든 잠재적 지점을 나타내며, 실제 어떤 지점에서 애드바이스가 실행될지는 포인트컷(Pointcut)에 의해 결정됩니다.

</details>

포인트 컷

* 어떤 조인 포인트에 Aspect를 적용할지 결정하는 필터.
* 여러 조인 포인트 중에서 특정 조건을 만족하는 조인 포인트만 선택하여 부가 기능 적용.

<details>

<summary>Example</summary>

포인트컷은 조인 포인트를 선택하는 필터 역할을 합니다. 예를 들어, 특정 클래스의 메서드 실행 시 어드바이스를 적용하려면 다음과 같이 포인트컷을 설정할 수 있습니다.

**예시 코드: 포인트컷 설정**

```java
@Aspect
public class LoggingAspect {

    // 포인트컷 정의: UserService 클래스의 모든 메서드를 대상으로 함
    @Pointcut("execution(* com.example.service.UserService.*(..))")
    private void userServiceMethods() {}

    // 어드바이스 적용: 위에서 정의한 포인트컷에 로깅 어드바이스를 적용
    @Before("userServiceMethods()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Before method: " + joinPoint.getSignature().getName());
    }
}
```

**설명**

* **포인트컷**: `@Pointcut("execution(* com.example.service.UserService.*(..))")`은 `UserService` 클래스의 모든 메서드에 적용되는 조인 포인트를 선택하는 포인트컷입니다.
* **어드바이스**: `@Before("userServiceMethods()")`는 포인트컷에서 선택한 조인 포인트에 메서드 실행 전에 로깅을 수행하는 어드바이스를 적용합니다.

</details>
