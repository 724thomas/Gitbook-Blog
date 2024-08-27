---
description: 추상 클래스보다는 인터페이스를 우선하라
---

# Item20. Prefer Interfaces to Abstract Classes

자바에서 객체지향 프로그래밍을 할 때, 추상 클래스와 인터페이스는 매우 중요한 개념입니다. 두 개념은 모두 공통된 동작을 정의하고, 클래스 간의 다형성을 제공하는 데 사용됩니다. 그러나 이펙티브 자바에서는 가능하다면 인터페이스를 추상 클래스보다 우선해서 사용하라고 권장합니다. 이는 코드의 유연성과 재사용성을 높이기 위함입니다.



## 1. 추상 클래스와 인터페이스의 기본 개념

### **1.1 추상 클래스란 무엇인가?**

추상 클래스는 인스턴스를 생성할 수 없는 클래스입니다. 주로 여러 클래스에 공통된 코드를 제공하기 위해 사용됩니다. 추상 클래스는 다른 클래스가 상속받아 사용할 수 있는 공통 메서드를 정의하거나, 반드시 구현해야 할 추상 메서드를 포함할 수 있습니다.

예를 들어, 동물(Animal)이라는 추상 클래스를 정의하고, 이를 상속받아 구체적인 동물(예: 개, 고양이)을 구현할 수 있습니다. 이때 동물 클래스에는 모든 동물이 공유하는 메서드(예: 숨쉬기, 이동하기 등)를 정의하고, 개별 동물에 따라 달라지는 메서드(예: 소리내기 등)를 추상 메서드로 선언할 수 있습니다.

<details>

<summary>Abstract Class</summary>

```java
abstract class Animal {
    abstract void makeSound();
    
    void breathe() {
        System.out.println("Breathing...");
    }
}
```

위 코드에서 `breathe()` 메서드는 공통적으로 구현된 메서드이며, `makeSound()` 메서드는 하위 클래스에서 반드시 구현해야 하는 추상 메서드입니다.

</details>

### **1.2 인터페이스란 무엇인가?**

인터페이스는 클래스가 구현해야 할 메서드의 집합을 정의합니다. 인터페이스 자체는 동작을 구현하지 않으며, 오로지 메서드 시그니처만 정의합니다. 자바 8부터는 디폴트 메서드와 정적 메서드를 가질 수 있게 되었지만, 기본적으로는 메서드 정의만을 포함하는 것이 원칙입니다.

인터페이스는 다중 상속을 지원합니다. 즉, 한 클래스가 여러 인터페이스를 구현할 수 있습니다. 이를 통해 클래스는 여러 역할을 수행할 수 있으며, 코드의 유연성을 높일 수 있습니다.

<details>

<summary>Interface</summary>

```java
interface Animal {
    void makeSound();
}
```

</details>



## 2. 추상 클래스의 제한사항

### **2.1 단일 상속의 제약**

추상 클래스의 가장 큰 단점은 단일 상속만을 허용한다는 점입니다. 자바에서 한 클래스는 오직 하나의 클래스만을 상속받을 수 있습니다. 이는 다중 상속이 허용되지 않기 때문에 발생하는 제약입니다. 만약 두 개 이상의 클래스에서 공통된 기능을 상속받아야 하는 경우, 추상 클래스는 적합하지 않을 수 있습니다.

<details>

<summary>Example</summary>

```java
abstract class Animal {
    abstract void makeSound();
}

abstract class Machine {
    abstract void operate();
}

class RobotDog extends Animal, Machine { // Error: 다중 상속은 불가능합니다.
    void makeSound() {
        System.out.println("Beep Boop");
    }

    void operate() {
        System.out.println("Operating...");
    }
}
```

위 예시처럼, 자바에서는 다중 상속이 허용되지 않으므로 하나의 클래스가 두 개 이상의 부모 클래스를 상속받을 수 없습니다. 이는 코드의 재사용성과 유연성을 제한하는 요소가 될 수 있습니다.

</details>



### **2.2 계층 구조의 복잡성**

추상 클래스는 복잡한 계층 구조를 야기할 수 있습니다. 상속이 깊어질수록 클래스 간의 의존성이 강해지며, 이는 코드의 유지보수성을 떨어뜨릴 수 있습니다. 또한, 상위 클래스에서 변경이 발생하면 모든 하위 클래스에 영향을 미치게 되어 예기치 않은 부작용을 초래할 수 있습니다.

예를 들어, 상위 클래스에 새로운 메서드를 추가하거나 기존 메서드를 수정할 경우, 이를 상속받는 하위 클래스들에서 해당 메서드를 재정의해야 할 수 있습니다. 이로 인해 코드베이스가 커질수록 관리가 어려워질 수 있습니다.

<details>

<summary>Example</summary>

```java
abstract class Vehicle {
    abstract void startEngine();
}

class Car extends Vehicle {
    void startEngine() {
        System.out.println("Car engine started");
    }
}

class Motorcycle extends Vehicle {
    void startEngine() {
        System.out.println("Motorcycle engine started");
    }
}
```

이 예제에서, 만약 `Vehicle` 클래스에 새로운 메서드를 추가하면 `Car`와 `Motorcycle` 클래스도 이를 구현해야 합니다. 이러한 방식은 작은 프로젝트에서는 문제가 되지 않을 수 있지만, 큰 프로젝트에서는 관리가 어렵고 실수의 여지가 많아질 수 있습니다.

</details>



## 3. 인터페이스의 장점

### **3.1 다중 상속이 가능**

인터페이스는 다중 상속을 지원합니다. 이는 하나의 클래스가 여러 인터페이스를 구현함으로써 다양한 동작을 수행할 수 있음을 의미합니다. 이를 통해 코드의 재사용성을 극대화하고, 유연한 설계를 가능하게 합니다.

<details>

<summary>Example</summary>

예를 들어, `Comparable`과 `Serializable` 인터페이스를 동시에 구현함으로써 객체가 정렬 가능하고, 직렬화될 수 있도록 만들 수 있습니다.

```java
class Dog implements Animal, Comparable<Dog>, Serializable {
    public void makeSound() {
        System.out.println("Bark!");
    }

    public int compareTo(Dog other) {
        return this.age - other.age;
    }
}
```

이 예제에서 `Dog` 클래스는 `Animal`, `Comparable`, `Serializable` 인터페이스를 동시에 구현함으로써 다양한 역할을 수행할 수 있습니다. 이는 추상 클래스가 제공하지 못하는 유연성을 제공합니다.

</details>



### **3.2 유연성과 확장성**

인터페이스는 추상 클래스보다 훨씬 유연하고 확장성이 높습니다. 인터페이스를 사용하면 기존의 클래스 계층 구조를 변경할 필요 없이 새로운 기능을 추가할 수 있습니다. 인터페이스에 새로운 메서드를 추가하고 이를 구현하는 클래스만 수정하면 됩니다.

또한, 인터페이스는 특정 기능을 여러 클래스에 쉽게 적용할 수 있도록 도와줍니다. 이는 코드의 재사용성과 유지보수성을 높이는 데 매우 유리합니다.

<details>

<summary>Example</summary>

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Animal, Flyable, Swimmable {
    public void makeSound() {
        System.out.println("Quack!");
    }

    public void fly() {
        System.out.println("Flying...");
    }

    public void swim() {
        System.out.println("Swimming...");
    }
}
```

이 예제에서 `Duck` 클래스는 `Animal`, `Flyable`, `Swimmable` 인터페이스를 모두 구현합니다. 이를 통해 오리 객체는 다양한 행동을 수행할 수 있으며, 이러한 설계는 코드의 유연성을 높입니다.

</details>



### **3.3 믹스인(Mixin) 인터페이스**

인터페이스는 특정 동작을 여러 클래스에 추가할 수 있는 믹스인(Mixin) 형태로도 사용할 수 있습니다. 믹스인 인터페이스는 클래스가 특정 행동을 쉽게 공유하도록 돕습니다. 예를 들어, `Logging` 인터페이스를 만들어 여러 클래스에서 로그 기능을 쉽게 사용할 수 있습니다.

<details>

<summary>Example</summary>

```java
interface Logging {
    default void log(String message) {
        System.out.println("Log: " + message);
    }
}

class ServiceA implements Logging {
    void performAction() {
        log("ServiceA action performed");
    }
}

class ServiceB implements Logging {
    void performAction() {
        log("ServiceB action performed");
    }
}
```

위 코드에서 `ServiceA`와 `ServiceB` 클래스는 `Logging` 인터페이스를 통해 공통된 로깅 기능을 공유합니다. 이를 통해 코드의 중복을 줄이고, 기능을 쉽게 확장할 수 있습니다.

</details>



## 4. 디폴트 메서드와 자바 8 이후의 변화

자바 8에서는 인터페이스에 디폴트 메서드와 정적 메서드를 추가할 수 있는 기능이 도입되었습니다. 이는 기존의 인터페이스에 새로운 메서드를 추가할 때, 모든 구현체에 해당 메서드를 강제하지 않고도 새로운 기능을 제공할 수 있다는 장점을 제공합니다.

### **4.1 디폴트 메서드의 장점**

디폴트 메서드는 인터페이스에 새로운 메서드를 추가하면서도, 기존의 구현체를 변경하지 않고 그대로 사용할 수 있게 합니다. 이를 통해 코드의 변경 없이도 기능을 확장할 수 있습니다.

<details>

<summary>Example</summary>

```java
interface Animal {
    void makeSound();
    
    default void breathe() {
        System.out.println("Breathing...");
    }
}

class Dog implements Animal {
    public void makeSound() {
        System.out.println("Bark!");
    }
}
```

위 코드에서 `Dog` 클래스는 `Animal` 인터페이스를 구현합니다. `Animal` 인터페이스에 추가된 `breathe()` 메서드는 디폴트 메서드로 정의되어 있어, `Dog` 클래스는 이를 별도로 구현할 필요가 없습니다.

</details>



### **4.2 정적 메서드의 활용**

인터페이스는 정적 메서드도 가질 수 있습니다. 이를 통해 공통적으로 사용되는 유틸리티 메서드를 인터페이스 내에서 정의할 수 있습니다. 정적 메서드는 클래스 메서드처럼 사용되며, 인터페이스 이름으로 직접 호출할 수 있습니다.

<details>

<summary>Example</summary>

```java
interface MathOperations {
    static int add(int a, int b) {
        return a + b;
    }
}

class Calculator {
    int result = MathOperations.add(3, 5);
}
```

위 예제에서 `MathOperations` 인터페이스는 `add()`라는 정적 메서드를 정의하고, `Calculator` 클래스는 이를 활용합니다. 이를 통해 공통된 기능을 인터페이스 내부에 정의하고, 클래스에서 이를 재사용할 수 있습니다.

</details>



### **4.3 디폴트 메서드의 주의사항**

디폴트 메서드는 매우 유용하지만, 남용할 경우 인터페이스의 본래 목적을 해칠 수 있습니다. 인터페이스는 본래 동작을 정의하는 것이 아닌, 동작을 약속하는 역할을 하기 때문에, 디폴트 메서드를 너무 많이 추가하면 추상 클래스와의 경계가 모호해질 수 있습니다.

따라서, 디폴트 메서드는 인터페이스를 구현하는 클래스의 변경 없이 기능을 확장할 수 있는 유연한 방법으로 사용하되, 필요 이상으로 사용하지 않도록 주의해야 합니다.



## 7. 추상 클래스와 인터페이스의 혼합 사용

추상 클래스와 인터페이스는 각기 다른 용도로 설계되었지만, 실제 개발에서는 이 둘을 혼합하여 사용하는 경우가 많습니다. 특히 대규모 시스템에서 특정 기능을 여러 클래스에 적용하면서도, 공통된 로직을 공유하고자 할 때 이러한 접근법이 유용합니다.

### **7.1 혼합 사용의 필요성**

복잡한 시스템에서는 여러 클래스가 동일한 인터페이스를 구현해야 하지만, 이들 클래스 간에 공통된 로직이 존재할 수 있습니다. 이때 추상 클래스를 사용하여 공통된 로직을 정의하고, 인터페이스를 통해 필요한 메서드들을 강제하는 방식이 효과적일 수 있습니다.

예를 들어, 파일을 처리하는 시스템에서 여러 파일 형식을 지원한다고 가정해 봅시다. 각 파일 형식에 대해 읽기, 쓰기, 삭제 등의 공통된 기능이 필요하지만, 파일 형식에 따라 구현 방식이 달라질 수 있습니다.

<details>

<summary>Example</summary>

```java
interface FileProcessor {
    void readFile(String fileName);
    void writeFile(String fileName, String data);
}

abstract class AbstractFileProcessor implements FileProcessor {
    @Override
    public void readFile(String fileName) {
        System.out.println("Reading file: " + fileName);
    }
    
    @Override
    public void writeFile(String fileName, String data) {
        System.out.println("Writing data to file: " + fileName);
    }

    abstract void processFile(String fileName);
}
```

이 코드에서 `AbstractFileProcessor`는 `FileProcessor` 인터페이스를 구현하면서, 공통된 `readFile`과 `writeFile` 메서드를 제공합니다. 그러나 `processFile` 메서드는 파일 형식에 따라 달라질 수 있기 때문에 추상 메서드로 남겨 두었습니다.

</details>



### **7.2 구체적인 구현**

`AbstractFileProcessor`를 상속받아 특정 파일 형식에 대한 처리를 구현하는 클래스를 만들 수 있습니다.

<details>

<summary>Example</summary>

```java
class TextFileProcessor extends AbstractFileProcessor {
    @Override
    void processFile(String fileName) {
        System.out.println("Processing text file: " + fileName);
    }
}

class BinaryFileProcessor extends AbstractFileProcessor {
    @Override
    void processFile(String fileName) {
        System.out.println("Processing binary file: " + fileName);
    }
}
```

위 코드에서 `TextFileProcessor`와 `BinaryFileProcessor`는 각각 텍스트 파일과 바이너리 파일을 처리하는 방법을 정의합니다. 이 방식으로 인터페이스를 사용해 공통된 동작을 강제하고, 추상 클래스를 통해 코드의 재사용성을 높일 수 있습니다.

</details>



### **7.3 혼합 사용의 장점**

이러한 혼합 사용은 코드의 중복을 줄이고, 유지보수를 쉽게 하며, 확장성을 높이는 데 매우 유용합니다. 공통된 기능을 추상 클래스에서 제공하므로, 이를 상속받는 클래스들은 오직 자신만의 고유한 기능에 집중할 수 있습니다.



## 9. 인터페이스의 고급 활용 기법

인터페이스는 자바에서 매우 강력한 도구로, 다양한 패턴과 기법에 활용될 수 있습니다. 이 섹션에서는 인터페이스를 활용한 몇 가지 고급 기법을 다루겠습니다.

### **9.1 전략 패턴 (Strategy Pattern)**

전략 패턴은 런타임 시 동작을 변경할 수 있는 유연한 설계를 제공합니다. 인터페이스를 사용해 다양한 전략을 정의하고, 이들을 필요에 따라 교체할 수 있습니다.

<details>

<summary>Example</summary>

```java
interface PaymentStrategy {
    void pay(int amount);
}

class CreditCardPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using credit card");
    }
}

class PaypalPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}

class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}
```

위 코드에서 `PaymentStrategy` 인터페이스는 결제 방식을 정의합니다. `CreditCardPayment`와 `PaypalPayment` 클래스는 각각 다른 결제 방식을 구현하며, `ShoppingCart` 클래스는 사용자가 선택한 결제 전략을 사용하여 결제를 처리합니다.

</details>



### **9.2 템플릿 메서드 패턴 (Template Method Pattern)**

템플릿 메서드 패턴은 알고리즘의 구조를 정의하면서, 일부 단계의 구현을 서브클래스에 맡기는 디자인 패턴입니다. 추상 클래스와 인터페이스를 함께 사용하여 구현할 수 있습니다.

<details>

<summary>Example</summary>

```java
abstract class Game {
    abstract void initialize();
    abstract void startPlay();
    abstract void endPlay();

    // template method
    public final void play() {
        initialize();
        startPlay();
        endPlay();
    }
}

class Football extends Game {
    @Override
    void initialize() {
        System.out.println("Football Game Initialized!");
    }

    @Override
    void startPlay() {
        System.out.println("Football Game Started!");
    }

    @Override
    void endPlay() {
        System.out.println("Football Game Finished!");
    }
}
```

위 코드에서 `Game` 클래스는 템플릿 메서드 `play()`를 정의합니다. 이 메서드는 게임의 기본 흐름을 정의하며, 구체적인 초기화, 시작, 종료 단계는 서브클래스에서 구현됩니다.

</details>

### **9.3 디폴트 메서드를 사용한 인터페이스 확장**

앞서 설명한 바와 같이, 자바 8에서는 인터페이스에 디폴트 메서드를 추가할 수 있습니다. 이를 통해 기존의 인터페이스에 새로운 기능을 추가하면서도, 하위 호환성을 유지할 수 있습니다.

예를 들어, 기존 인터페이스에 새로운 로깅 기능을 추가하고자 할 때, 디폴트 메서드를 활용할 수 있습니다.

<details>

<summary>Example</summary>

```java
interface Service {
    void execute();

    default void log(String message) {
        System.out.println("Log: " + message);
    }
}

class MyService implements Service {
    public void execute() {
        log("Executing service logic");
    }
}
```

이 코드에서 `MyService` 클래스는 `Service` 인터페이스를 구현하며, 디폴트 메서드 `log()`를 사용하여 로그를 기록합니다. 이렇게 디폴트 메서드를 사용하면, 기존의 구현체에 영향을 주지 않고도 인터페이스의 기능을 확장할 수 있습니다.

</details>

