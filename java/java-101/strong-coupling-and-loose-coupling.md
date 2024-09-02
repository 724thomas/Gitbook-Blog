---
description: 강한 결합과 느슨한 결합
---

# Strong Coupling and Loose Coupling

결합이라는 개념은 클래스나 모듈 간의 의존성을 나타내는 개념입니다. 결합도에 따라 유지보수성, 확장성, 코드의 재사용성이 달라집니다.



## 결합

결합은 하나의 클래스나 모듈이 닫른 클래스나 모듈에 얼마나 의존하는지를 의미합니다. 두 개의 클래스가 서로 직접적으로 많은 정보를 주고 받거나, 한 클래스가 다른 클래스의 내부 구조를 많이 알고 있을수록 결합이 강하다고 할 수 있습니다. 반면, 서로의 내부 구현에 대한 정보가 적고, 인터페이스나 추상화를 통해 상호 작용하면 결합이 느슨하다고 할 수 있습니다.



<details>

<summary> 강한 결합</summary>

```java
class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private Engine engine = new Engine();

    public void startCar() {
        engine.start();
    }
}
```

위의 코드에서 `Car` 클래스는 `Engine` 클래스와 강하게 결합되어 있습니다. `Car` 클래스는 `Engine` 클래스의 인스턴스를 직접 생성하고, `start` 메서드를 호출합니다. 만약 `Engine` 클래스의 내부 구조나 메서드가 변경되면, `Car` 클래스도 이에 맞춰 수정해야 합니다. 이는 강한 결합의 대표적인 예시입니다.

</details>

<details>

<summary> 느슨한 결합</summary>

```java
interface Engine {
    void start();
}

class DieselEngine implements Engine {
    public void start() {
        System.out.println("Diesel Engine started");
    }
}

class Car {
    private Engine engine;

    public Car(Engine engine) {
        this.engine = engine;
    }

    public void startCar() {
        engine.start();
    }
}
```

위의 코드에서는 `Car` 클래스가 `Engine` 인터페이스에 의존하고 있으며, 실제 구현체(`DieselEngine`)는 `Car` 클래스의 생성자를 통해 주입됩니다. 이 방식은 느슨한 결합을 구현한 예시입니다. `Engine` 인터페이스의 구현체를 변경해도 `Car` 클래스는 영향을 받지 않으며, 다양한 엔진 타입을 쉽게 교체할 수 있습니다.

</details>

## 강한 결합

두 클래스가 서로 깊이 연관되어 있음을 의미합니다. 한 클래스가 다른 클래스의 인스턴스를 직접 생성하고, 해당 인스턴스의 메서드를 호출하거나 내부 구조에 강하게 의존할 때 발생합니다. 이러한 상황에서는 하나의 클래스에 변경이 생기면 해당 클래스와 강하게 결합된 다른 클래스도 영향을 받아 수정이 필요하게 됩니다.

### 장점:

* 유연성 부족
* 테스트 어려움: 특정 클래스가 다른 클래스와 강하게 결합돼 있으면, 단위 테스트를 수행하기 어렵습니다. (예: Car 클래스를 테스트하려면 Engine 클래스의 동작도 함께 검증해야 합니다.)
* 재사용성 저하



## 느슨한 결합

느슨한 결합은 클래스 간의 의존성을 최소화하는 것을 목표로 합니다. 이때 클래스는 직접적으로 서로의 인스턴스를 생성하거나 내부 구조에 의존하는 대신, 인터페이스나 추상 클래스를 통해 상호작용합니다. 느슨한 결합을 통해 시스템은 더 유연해지고, 클래스 간의 변경이 다른 클래스에 미치는 영향을 최소화할 수 있습니다.

### 장점:

* 유연성 향상
* 테스트 용이
* 재사용성 증가



## 느슨한 결합 구현

### 인터페이스 사용

<details>

<summary> 인터페이스 사용</summary>

```java
interface PaymentProcessor {
    void processPayment(double amount);
}

class CreditCardProcessor implements PaymentProcessor {
    public void processPayment(double amount) {
        System.out.println("Processing credit card payment: " + amount);
    }
}

class PaypalProcessor implements PaymentProcessor {
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment: " + amount);
    }
}

class PaymentService {
    private PaymentProcessor processor;

    public PaymentService(PaymentProcessor processor) {
        this.processor = processor;
    }

    public void pay(double amount) {
        processor.processPayment(amount);
    }
}
```

위의 예시에서 `PaymentProcessor` 인터페이스는 결제 처리에 필요한 공통된 메서드를 정의합니다. `PaymentService` 클래스는 `PaymentProcessor` 인터페이스에 의존하므로, 결제 처리 방식이 바뀌어도 `PaymentService` 클래스는 변경될 필요가 없습니다. 이렇게 인터페이스를 사용하면 클래스 간의 결합도를 낮출 수 있습니다.

</details>

### 팩토리 패턴

팩토리 패턴은 객체 생성 로직을 캡슐화하여 클라이언트 코드와 객체 생성 방식을 분리하는 디자인 패턴입니다. 이를 통해 클래스 간의 결합을 줄이고, 객체 생성 방법을 유연하게 변경할 수 있습니다.

<details>

<summary>팩토리 패턴</summary>

```java
interface Engine {
    void start();
}

class DieselEngine implements Engine {
    public void start() {
        System.out.println("Diesel Engine started");
    }
}

class ElectricEngine implements Engine {
    public void start() {
        System.out.println("Electric Engine started");
    }
}

class EngineFactory {
    public static Engine createEngine(String type) {
        if (type.equals("diesel")) {
            return new DieselEngine();
        } else if (type.equals("electric")) {
            return new ElectricEngine();
        }
        throw new IllegalArgumentException("Unknown engine type");
    }
}

class Car {
    private Engine engine;

    public Car(String engineType) {
        this.engine = EngineFactory.createEngine(engineType);
    }

    public void startCar() {
        engine.start();
    }
}
```

위의 예시에서 `EngineFactory` 클래스는 `Engine` 객체의 생성 로직을 캡슐화하여 `Car` 클래스와 엔진의 구체적인 구현을 분리합니다. `Car` 클래스는 더 이상 `Engine` 클래스의 구체적인 타입을 알 필요가 없으며, 엔진 타입이 변경되어도 `Car` 클래스는 수정되지 않습니다.

</details>

### 의존성 주입

의존성 주입(Dependency Injection, DI)은 느슨한 결합을 구현하는 가장 강력한 방법 중 하나입니다. DI는 객체의 의존성을 외부에서 주입받아, 클래스 간의 결합도를 낮춥니다.

* 생성자 주입
* 세터 주입
* 필드 주입
