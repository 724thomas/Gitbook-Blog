---
description: 오버라이딩과 오버로딩
---

# Override, Overload

## 1. 오버라이딩의 개념과 동작 원리

\*\*오버라이딩(Overriding)\*\*은 자바의 객체지향 프로그래밍(OOP)에서 매우 중요한 개념으로, 상속 관계에 있는 부모 클래스의 메서드를 자식 클래스에서 재정의하는 기술입니다. 이는 자바 스프링을 포함한 다양한 프레임워크와 애플리케이션에서 코드의 유연성과 재사용성을 극대화하는 데 중요한 역할을 합니다. 오버라이딩은 다형성(Polymorphism), 동적 바인딩(Dynamic Binding), 그리고 OCP(Open-Closed Principle) 등 여러 객체지향 원칙과 밀접하게 연관되어 있습니다.

### 1.1 오버라이딩의 기본 정의

오버라이딩은 부모 클래스에서 이미 정의된 메서드를 자식 클래스에서 동일한 메서드 시그니처(메서드 이름, 매개변수의 타입 및 순서)를 사용하여 재정의하는 것을 의미합니다. 이를 통해 자식 클래스는 부모 클래스의 메서드를 상속받되, 자식 클래스에 적합한 동작을 구현할 수 있습니다. 이는 객체지향 프로그래밍의 다형성(Polymorphism)을 구현하는 중요한 방법 중 하나입니다.

<details>

<summary>class Parent<br>class Child extends Parent</summary>

```java
class Parent {
    void showMessage() {
        System.out.println("Message from Parent");
    }
}

class Child extends Parent {
    @Override
    void showMessage() {
        System.out.println("Message from Child");
    }
}
```

위의 예제에서 `Child` 클래스는 `Parent` 클래스의 `showMessage` 메서드를 오버라이딩하여, `Child` 객체가 이 메서드를 호출할 때 `Child` 클래스에서 정의한 메서드가 실행됩니다.

</details>



### 1.2 오버라이딩의 규칙

오버라이딩은 몇 가지 중요한 규칙을 따릅니다:

1. **메서드 시그니처 일치**: 자식 클래스의 오버라이딩된 메서드는 부모 클래스의 메서드와 동일한 메서드 이름과 매개변수 목록을 가져야 합니다.
2. **접근 제어자 제한**: 오버라이딩된 메서드는 부모 클래스의 메서드보다 더 제한적인 접근 제어자를 가질 수 없습니다. 예를 들어, 부모 클래스의 메서드가 `protected`라면, 자식 클래스에서는 `protected`나 `public`으로만 오버라이딩할 수 있고, `private`으로는 오버라이딩할 수 없습니다.
3. **예외 처리 제한**: 자식 클래스의 오버라이딩된 메서드는 부모 클래스 메서드에서 선언된 예외보다 더 상위의 예외를 던질 수 없습니다. 그러나 부모 클래스에서 던지지 않은 새로운 체크 예외를 던질 수는 없습니다.
4. **반환 타입의 일관성**: 오버라이딩된 메서드는 부모 클래스의 메서드와 동일한 반환 타입을 가지거나, 자바 5 이후로는 더 구체적인 반환 타입(공변 반환 타입, Covariant Return Type)을 가질 수 있습니다.



### 1.3 오버라이딩의 실제 동작: 동적 바인딩(Dynamic Binding)

오버라이딩은 동적 바인딩(Dynamic Binding) 또는 런타임 바인딩(Runtime Binding)과 관련이 깊습니다. 동적 바인딩은 메서드 호출이 런타임에 실제 객체의 타입에 따라 결정되는 방식입니다. 이는 자바에서 다형성을 실현하는 데 필수적인 요소로, 부모 클래스 타입의 참조 변수가 자식 클래스의 객체를 참조할 때 오버라이딩된 메서드가 실행됩니다.

```java
Parent obj = new Child();
obj.showMessage(); // "Message from Child"
```

위 코드에서 `obj`는 `Parent` 타입의 변수이지만, 실제로는 `Child` 객체를 참조하고 있습니다. 따라서 `showMessage` 메서드를 호출할 때 `Child` 클래스의 메서드가 실행됩니다. 이처럼 자바는 런타임에 객체의 실제 타입을 확인하고, 그에 맞는 메서드를 호출하는 동적 바인딩을 사용합니다.

***

## 2. 오버라이딩과 객체지향 원칙

### 2.1 다형성(Polymorphism)

오버라이딩은 다형성을 구현하는 핵심 기법입니다. 다형성은 같은 인터페이스나 부모 클래스를 공유하는 객체들이 동일한 메서드 호출에 대해 서로 다른 방식으로 응답할 수 있게 해줍니다. 자바에서는 오버라이딩된 메서드를 통해 다양한 형태의 동작을 구현할 수 있으며, 이는 코드의 유연성과 확장성을 크게 높여줍니다.

```java
Parent obj1 = new Parent();
Parent obj2 = new Child();

obj1.showMessage(); // "Message from Parent"
obj2.showMessage(); // "Message from Child"
```

이 예시에서 두 객체는 같은 타입(Parent)이지만, 각각의 메서드 호출에 대해 다른 결과를 반환합니다. 이는 다형성을 활용한 코드로, 코드의 재사용성과 유지보수성을 높입니다.

### 2.2 OCP(Open-Closed Principle)

OCP는 "확장에는 열려 있고, 수정에는 닫혀 있어야 한다"는 객체지향 설계 원칙으로, 기존 코드를 수정하지 않고 새로운 기능을 추가할 수 있어야 한다는 의미입니다. 오버라이딩은 OCP를 실현하는 데 중요한 역할을 합니다. 자식 클래스에서 부모 클래스의 메서드를 오버라이딩함으로써 기존 클래스 코드를 수정하지 않고도 새로운 기능을 구현할 수 있습니다.

<details>

<summary>class Shape<br>class Circle extends Shape<br>class Square extends Shape</summary>

```java
class Shape {
    void draw() {
        System.out.println("Drawing a shape");
    }
}

class Circle extends Shape {
    @Override
    void draw() {
        System.out.println("Drawing a circle");
    }
}

class Square extends Shape {
    @Override
    void draw() {
        System.out.println("Drawing a square");
    }
}
```

위 코드에서 `Shape` 클래스는 수정하지 않고, `Circle`과 `Square` 클래스에서 새로운 동작을 구현했습니다. 이는 OCP를 따르는 설계 방식입니다.

</details>



### 2.3 리스코프 치환원칙 LSP(Liskov Substitution Principle)

LSP는 "서브 타입은 언제나 자신의 기반 타입으로 대체할 수 있어야 한다"는 원칙입니다. 오버라이딩을 통해 자식 클래스는 부모 클래스의 인터페이스를 일관되게 유지하며, 부모 클래스의 역할을 대체할 수 있습니다. 이는 코드에서 자식 클래스가 부모 클래스로 대체되어도 프로그램의 정확성과 기능이 유지될 수 있음을 보장합니다.&#x20;

<details>

<summary>잘못된  사례<br>class Rectangle<br>class Square extends Rectangle</summary>

```java
class Rectangle {
    int width, height;

    void setWidth(int width) {
        this.width = width;
    }

    void setHeight(int height) {
        this.height = height;
    }

    int area() {
        return width * height;
    }
}

class Square extends Rectangle {
    @Override
    void setWidth(int width) {
        this.width = width;
        this.height = width;
    }

    @Override
    void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}
```

`Square` 클래스는 `Rectangle` 클래스를 상속받고 있지만, LSP(Liskov Substitution Principle)를 위반하고 있습니다. LSP는 자식 클래스가 부모 클래스의 기능을 유지하면서도 확장할 수 있어야 한다는 원칙입니다. 하지만 이 예제에서는 `Square`가 `Rectangle`의 메서드를 오버라이딩하여 `Rectangle`의 기본 기능을 왜곡하고 있습니다. 구체적으로는, `Square`가 사각형의 특성(너비와 높이가 서로 다를 수 있음)을 무시하고, 이를 고정시킴으로써 `Rectangle`의 본래 역할을 수행할 수 없게 만듭니다.



</details>

<details>

<summary>상속 대신 별도의 클래스 구조 사용<br>interface Shape<br>class Rectangle implements Shape<br>class Square implements Shape</summary>

```java
interface Shape {
    int area();
}

class Rectangle implements Shape {
    protected int width, height;

    void setWidth(int width) {
        this.width = width;
    }

    void setHeight(int height) {
        this.height = height;
    }

    @Override
    public int area() {
        return width * height;
    }
}

class Square implements Shape {
    private int side;

    void setSide(int side) {
        this.side = side;
    }

    @Override
    public int area() {
        return side * side;
    }
}
```

이렇게 하면 `Rectangle`과 `Square`는 각각 자신의 특성에 맞게 설계되고, 공통 기능인 `area()` 메서드는 `Shape` 인터페이스를 통해 구현됩니다. 이 접근법은 LSP를 준수하며, 클래스들이 각각의 역할을 명확히 할 수 있게 합니다.

</details>

### 2.4 ISP(Interface Segregation Principle)

ISP는 "클라이언트는 자신이 사용하지 않는 인터페이스에 의존하지 않아야 한다"는 원칙입니다. 오버라이딩을 통해 자식 클래스는 필요 없는 부모 클래스의 메서드를 오버라이딩하지 않거나, 특정 기능만 구현하도록 설계할 수 있습니다. 이는 인터페이스의 분리를 통해 코드의 응집도를 높이고, 불필요한 의존성을 줄일 수 있습니다.

<details>

<summary>잘못된  사례<br>interface Worker<br>class Developer implements Worker<br>lass Manager implements Worker</summary>

```java
interface Worker {
    void work();
    void eat();
}

class Developer implements Worker {
    @Override
    public void work() {
        System.out.println("Writing code");
    }

    @Override
    public void eat() {
        // Developer가 일과 관련된 코드만 필요하지만, eat 메서드를 구현해야 합니다.
        System.out.println("Eating lunch");
    }
}

class Manager implements Worker {
    @Override
    public void work() {
        System.out.println("Managing project");
    }

    @Override
    public void eat() {
        System.out.println("Eating lunch");
    }
}
```

이 예제에서 `Worker` 인터페이스는 `work()`와 `eat()` 메서드를 모두 포함하고 있습니다. `Developer`와 `Manager` 클래스는 실제로는 `work()` 메서드만 필요하지만, `eat()` 메서드도 구현해야 합니다. 이는 불필요한 의존성을 증가시키고, 인터페이스의 응집도를 낮춥니다.

</details>

<details>

<summary>작은 인터페이스로 분리<br>interface Workable<br>interface Eatable<br>class Developer implements Workable<br>class Manager implements Workable, Eatable</summary>

ISP를 적용하여, 클라이언트가 자신에게 필요한 기능만 구현하도록 인터페이스를 분리할 수 있습니다.

```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class Developer implements Workable {
    @Override
    public void work() {
        System.out.println("Writing code");
    }
}

class Manager implements Workable, Eatable {
    @Override
    public void work() {
        System.out.println("Managing project");
    }

    @Override
    public void eat() {
        System.out.println("Eating lunch");
    }
}
```

이 예제에서 `Workable`과 `Eatable` 인터페이스를 분리하여, `Developer` 클래스는 자신에게 필요한 `work()` 메서드만 구현하고, `Manager` 클래스는 `work()`와 `eat()` 메서드를 모두 구현하도록 설계했습니다. 이로써 불필요한 메서드를 구현하지 않아도 되며, 인터페이스가 클라이언트의 요구에 맞게 분리되어 코드의 응집도가 높아지고, 불필요한 의존성이 제거되었습니다.

</details>



***

## 3. 오버라이딩과 관련된 키워드

### 3.1 `@Override` 애노테이션

**`@Override`** 애노테이션은 자바에서 메서드가 부모 클래스의 메서드를 오버라이딩하고 있음을 명시적으로 선언하는 데 사용됩니다. 이 애노테이션을 사용하면 컴파일러가 메서드 시그니처의 일치를 검증하여, 실수로 잘못된 시그니처를 작성하는 것을 방지할 수 있습니다. 이는 코드의 가독성과 유지보수성을 높이는 데 중요한 역할을 합니다.

<details>

<summary>메서드 시그니처란?</summary>

**메서드 시그니처**는 메서드의 이름과 매개변수의 타입 및 개수를 합한 것

```java
class Parent {
    void display(String message) {
        System.out.println(message);
    }
}

class Child extends Parent {
    @Override
    void display(String message) {
        System.out.println("Child: " + message);
    }
}
```

여기서 `display(String message)`는 `display`라는 메서드 이름과 `String` 타입의 매개변수를 가지며, `Parent`와 `Child` 클래스 모두 동일한 메서드 시그니처를 가지고 있습니다.

</details>



### 3.2 `super` 키워드

**`super`** 키워드는 자식 클래스에서 부모 클래스의 메서드나 생성자를 호출할 때 사용됩니다. 오버라이딩된 메서드 내에서 부모 클래스의 원래 메서드를 호출해야 하는 경우, `super`를 사용하여 부모 클래스의 메서드에 접근할 수 있습니다.

<details>

<summary>class Parent<br>class Child extends Parent</summary>

```java
class Parent {
    void showMessage() {
        System.out.println("Message from Parent");
    }
}

class Child extends Parent {
    @Override
    void showMessage() {
        super.showMessage(); // 부모 클래스의 메서드 호출
        System.out.println("Message from Child");
        // 실행 결과:
        // Message from Parent
        // Message from Child
    }
}

```

위의 예제에서 `Child` 클래스는 부모 클래스의 `showMessage` 메서드를 호출한 후, 자신의 메서드 작업을 추가로 수행합니다.

</details>



### 3.3 추상 클래스(Abstract Class)와 오버라이딩

**추상 클래스**는 하나 이상의 추상 메서드를 포함할 수 있는 클래스입니다. 추상 메서드는 구현이 없는 메서드로, 자식 클래스에서 반드시 오버라이딩하여 구현해야 합니다. 이를 통해 부모 클래스에서 메서드의 기본 구조만 정의하고, 구체적인 구현은 자식 클래스에서 하도록 강제할 수 있습니다.

<details>

<summary>abstract class Animal<br>class Dog extends Animal<br>class Cat extends Animal</summary>

```java
abstract class Animal {
    abstract void sound();
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Meow");
    }
}
```

위의 예제에서 `Animal` 클래스는 `sound`라는 추상 메서드를 가지며, 각 자식 클래스는 이를 오버라이딩하여 자신만의 소리를 구현합니다.

</details>



### 3.4 인터페이스(Interface)와 오버라이딩

인터페이스는 클래스가 구현해야 할 메서드의 시그니처를 정의하는 일종의 계약(contract)입니다. 클래스가 인터페이스를 구현할 때, 인터페이스에 정의된 모든 메서드를 오버라이딩해야 합니다. 이는 다형성을 실현하는 또 다른 방법으로, 자바 스프링에서 의존성 주입(Dependency Injection)이나 스프링 빈(Spring Bean) 등 다양한 기능에 활용됩니다.

<details>

<summary>interface Printer<br>class ConsolePrinter implements Printer<br>class FilePrinter implements Printer</summary>

```java
interface Printer {
    void print();
}

class ConsolePrinter implements Printer {
    @Override
    public void print() {
        System.out.println("Printing to console");
    }
}

class FilePrinter implements Printer {
    @Override
    public void print() {
        System.out.println("Printing to file");
    }
}
```

이 예제에서 `ConsolePrinter`와 `FilePrinter` 클래스는 각각 `Printer` 인터페이스의 `print` 메서드를 오버라이딩하여 자신만의 출력을 구현합니다.

</details>

#### `abstract class`와 `interface`의 기본 차이

<table data-header-hidden data-full-width="true"><thead><tr><th width="98"></th><th></th><th></th></tr></thead><tbody><tr><td>특징</td><td><code>abstract class</code></td><td><code>interface</code></td></tr><tr><td><strong>목적</strong></td><td>공통된 동작과 상태를 정의하며, 일부 구현을 제공</td><td>행동의 계약을 정의하며, 구현은 전혀 포함하지 않음</td></tr><tr><td><strong>구현</strong></td><td>일반 메서드와 추상 메서드 모두 포함 가능</td><td>디폴트 메서드와 정적 메서드를 제외하고 모든 메서드가 추상 메서드</td></tr><tr><td><strong>상속</strong></td><td>단일 상속만 가능</td><td>다중 구현 가능</td></tr><tr><td><strong>필드</strong></td><td>인스턴스 변수와 상수 선언 가능</td><td>상수만 선언 가능 (자바 8부터 디폴트 메서드와 정적 메서드 포함 가능)</td></tr><tr><td><strong>생성자</strong></td><td>생성자 선언 가능</td><td>생성자 선언 불가</td></tr><tr><td><strong>사용 시기</strong></td><td>기본 동작이나 상태를 공유하는 클래스들이 있을 때 사용</td><td>클래스들이 공통적으로 가져야 할 행동을 정의할 때 사용</td></tr></tbody></table>



***

## 4. 오버라이딩과 디자인 패턴

### 4.1 템플릿 메서드 패턴(Template Method Pattern)

**템플릿 메서드 패턴**은 알고리즘의 구조를 부모 클래스에서 정의하고, 구체적인 구현을 자식 클래스에서 오버라이딩하여 수행하는 패턴입니다. 이 패턴은 공통적인 알고리즘 구조를 유지하면서도, 세부적인 동작을 자식 클래스에서 유연하게 변경할 수 있게 해줍니다.

<details>

<summary>abstract class Game<br>class Football extends Game<br>class Cricket extends Game</summary>

```java
abstract class Game {
    final void play() {
        initialize();
        startPlay();
        endPlay();
    }

    abstract void initialize();
    abstract void startPlay();
    abstract void endPlay();
}

class Football extends Game {
    @Override
    void initialize() {
        System.out.println("Football Game Initialized.");
    }

    @Override
    void startPlay() {
        System.out.println("Football Game Started.");
    }

    @Override
    void endPlay() {
        System.out.println("Football Game Finished.");
    }
}

class Cricket extends Game {
    @Override
    void initialize() {
        System.out.println("Cricket Game Initialized.");
    }

    @Override
    void startPlay() {
        System.out.println("Cricket Game Started.");
    }

    @Override
    void endPlay() {
        System.out.println("Cricket Game Finished.");
    }
}
```

위의 예제에서 `Game` 클래스는 템플릿 메서드를 정의하고, 세부적인 단계는 `Football`과 `Cricket` 클래스에서 오버라이딩하여 구현합니다.

</details>



### 4.2 전략 패턴(Strategy Pattern)

**전략** **패턴**은 다양한 알고리즘을 인터페이스로 캡슐화하여 상호 교체 가능하게 만드는 패턴입니다. 오버라이딩을 통해 각 전략 클래스는 동일한 인터페이스를 구현하면서도, 자신만의 독립적인 동작을 정의할 수 있습니다. 이 패턴은 런타임에 전략을 변경하거나 확장할 수 있는 유연성을 제공합니다.

<details>

<summary>interface PaymentStrategy<br>class CreditCardPayment implements PaymentStrategy<br>class PaypalPayment implements PaymentStrategy</summary>

```java
interface PaymentStrategy {
    void pay(int amount);
}

class CreditCardPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}

class PaypalPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}
```

위의 예제에서 `PaymentStrategy` 인터페이스를 구현한 `CreditCardPayment`와 `PaypalPayment` 클래스는 각각의 `pay` 메서드를 오버라이딩하여 다른 방식의 결제를 구현합니다.

</details>



### 4.3 팩토리 메서드 패턴(Factory Method Pattern)

**팩토리 메서드 패턴**은 객체 생성의 책임을 서브클래스에 위임하는 디자인 패턴입니다. 이 패턴에서는 부모 클래스가 객체 생성에 필요한 인터페이스만 제공하고, 실제 객체 생성은 자식 클래스에서 오버라이딩하여 수행합니다.

<details>

<summary>abstract class Creator<br>class ConcreteCreator extends Creator</summary>

```java
abstract class Creator {
    abstract Product createProduct();

    void doSomething() {
        Product product = createProduct();
        product.use();
    }
}

class ConcreteCreator extends Creator {
    @Override
    Product createProduct() {
        return new ConcreteProduct();
    }
}
```

이 예제에서 `ConcreteCreator` 클래스는 `createProduct` 메서드를 오버라이딩하여 `ConcreteProduct` 객체를 생성합니다. 팩토리 메서드 패턴은 객체 생성 로직을 캡슐화하고, 서브클래스를 통해 다양한 객체 생성 방식을 제공할 수 있도록 합니다.

</details>



***

## 결론

오버라이딩은 자바에서 객체지향 프로그래밍의 핵심 원칙을 구현하는 데 필수적인 기법으로, 특히 자바 스프링과 같은 프레임워크에서 다형성, 동적 바인딩, OCP, LSP 등의 원칙과 결합하여 코드의 유연성과 확장성을 높이는 데 중요한 역할을 합니다. 오버라이딩을 제대로 이해하고 활용하는 것은 견고하고 유지보수하기 쉬운 소프트웨어를 개발하는 데 필수적입니다. 이를 통해 코드는 재사용 가능하고, 변경에 유연하며, 다양한 디자인 패턴을 통해 구조화할 수 있습니다.



***





## 5. 오버로딩의 개념과 동작 원리

\*\*오버로딩(Overloading)\*\*은 자바의 객체지향 프로그래밍(OOP)에서 하나의 클래스 내에서 동일한 이름의 메서드를 여러 개 정의하되, 각 메서드가 서로 다른 매개변수 리스트를 가지도록 하는 기법입니다. 이는 메서드의 이름을 재사용하면서도 다양한 입력을 처리할 수 있게 해주며, 코드의 가독성을 높이고 유연성을 제공합니다. 오버로딩은 컴파일 타임에 결정되며, 자바에서 정적 다형성(Static Polymorphism)을 구현하는 방법 중 하나입니다.

### 5.1 오버로딩의 기본 정의

오버로딩은 동일한 메서드 이름을 사용하지만, 서로 다른 매개변수의 타입, 개수, 또는 순서를 사용하여 여러 개의 메서드를 정의하는 것입니다. 이렇게 하면 같은 이름의 메서드가 다양한 입력에 대해 다르게 동작하도록 할 수 있습니다. 이는 코드의 가독성을 높이고, 개발자가 메서드 이름을 일관되게 사용할 수 있게 해줍니다.

<details>

<summary>class MathOperations</summary>

```java
class MathOperations {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

위의 예제에서 `MathOperations` 클래스는 `add` 메서드를 세 번 오버로딩하고 있습니다. 각 `add` 메서드는 매개변수의 타입과 개수가 다르며, 이를 통해 다양한 형태의 덧셈을 지원합니다.

</details>



### 5.2 오버로딩의 규칙

오버로딩을 사용할 때는 몇 가지 중요한 규칙을 따라야 합니다:

1. **메서드 이름 동일**: 오버로딩된 메서드들은 모두 동일한 메서드 이름을 가져야 합니다.
2. **매개변수의 차이**: 오버로딩된 메서드들은 매개변수의 타입, 개수, 또는 순서가 달라야 합니다. 매개변수의 이름만 다른 경우 오버로딩이 성립되지 않습니다.
3. **반환 타입**: 반환 타입이 다른 것은 오버로딩에 영향을 미치지 않습니다. 오버로딩은 오로지 매개변수 리스트의 차이에 따라 결정됩니다.

<details>

<summary>class Display</summary>

```java
class Display {
    void show(int a) {
        System.out.println("Integer: " + a);
    }

    void show(String a) {
        System.out.println("String: " + a);
    }

    void show(int a, String b) {
        System.out.println("Integer: " + a + ", String: " + b);
    }
}
```

이 예제에서 `show` 메서드는 매개변수의 타입과 개수에 따라 여러 번 오버로딩되어 있습니다.

</details>

***

## 6. 오버로딩과 객체지향 원칙

### 6.1 정적 다형성(Static Polymorphism)

오버로딩은 자바에서 \*\*정적 다형성(Static Polymorphism)\*\*을 구현하는 방법입니다. 컴파일 타임에 호출할 메서드가 결정되기 때문에, 이를 **컴파일 타임 다형성**이라고도 합니다. 이는 런타임에 메서드가 결정되는 오버라이딩과 대비됩니다.

<details>

<summary>class MathOperations</summary>

```java
class MathOperations {
    int multiply(int a, int b) {
        return a * b;
    }

    double multiply(double a, double b) {
        return a * b;
    }
}

public class Main {
    public static void main(String[] args) {
        MathOperations math = new MathOperations();
        System.out.println(math.multiply(2, 3)); // multiply(int, int) 호출
        System.out.println(math.multiply(2.5, 3.5)); // multiply(double, double) 호출
    }
}
```

위의 예제에서 컴파일러는 각 `multiply` 메서드 호출 시점에 메서드의 시그니처를 분석하여 적절한 메서드를 호출합니다.

</details>



### 6.2 SRP(Single Responsibility Principle)

오버로딩은 SRP와도 관련이 있습니다. SRP는 "클래스는 하나의 책임만 가져야 한다"는 원칙으로, 오버로딩된 메서드들은 같은 이름을 가지지만, 입력 매개변수의 타입이나 개수에 따라 서로 다른 기능을 수행할 수 있습니다. 이를 통해 클래스가 하나의 책임을 중심으로 다양한 상황에 대처할 수 있도록 설계할 수 있습니다.

<details>

<summary>class Logger</summary>

```java
class Logger {
    void log(String message) {
        System.out.println("Log: " + message);
    }

    void log(int errorCode) {
        System.out.println("Error Code: " + errorCode);
    }

    void log(String message, int errorCode) {
        System.out.println("Log: " + message + " (Error Code: " + errorCode)");
    }
}
```

이 예제에서 `Logger` 클래스는 다양한 타입의 로그 메시지를 처리할 수 있도록 오버로딩된 `log` 메서드를 제공합니다. 이는 로그를 기록하는 하나의 책임을 중심으로, 다양한 입력을 처리하는 방법을 제시합니다.

</details>



### 6.3 DRY(Don't Repeat Yourself) 원칙

오버로딩은 DRY 원칙을 준수하는 데 도움이 됩니다. DRY 원칙은 "중복을 피하라"는 의미로, 코드의 중복을 줄임으로써 유지보수성을 높이는 원칙입니다. 오버로딩을 통해 동일한 목적을 가진 메서드들을 한 곳에 모아둠으로써 코드의 중복을 피하고, 일관된 메서드 이름을 유지할 수 있습니다.

<details>

<summary>DRY(Don't Repeat Yourself) 원칙</summary>

#### DRY 원칙의 중요성

1. **유지보수성 향상**: 코드가 중복되면, 동일한 로직에 변경이 필요할 때 모든 중복된 부분을 수정해야 합니다. 이는 실수를 초래할 수 있으며, 유지보수를 어렵게 만듭니다. DRY 원칙을 따르면, 수정이 필요한 부분이 한 곳에만 존재하므로 유지보수가 쉬워집니다.
2. **가독성 향상**: 코드가 중복되면 코드베이스가 불필요하게 길어져, 가독성이 떨어집니다. DRY 원칙을 따르면, 코드는 더 간결해지고, 이해하기 쉬워집니다.
3. **재사용성 증가**: 중복을 피하고, 공통된 기능을 하나의 모듈로 분리하면, 이 기능을 여러 곳에서 재사용할 수 있습니다. 이는 코드의 재사용성을 높여 개발 효율성을 향상시킵니다.

#### DRY 원칙을 구현하는 방법

1.  **함수와 메서드의 활용**: 반복되는 코드를 함수나 메서드로 추출하여, 재사용할 수 있습니다.

    ```java
    // DRY 원칙을 위반한 코드
    System.out.println("User ID: 123");
    System.out.println("User Name: Alice");
    System.out.println("User Email: alice@example.com");

    System.out.println("User ID: 456");
    System.out.println("User Name: Bob");
    System.out.println("User Email: bob@example.com");

    // DRY 원칙을 준수한 코드
    void printUserInfo(int id, String name, String email) {
        System.out.println("User ID: " + id);
        System.out.println("User Name: " + name);
        System.out.println("User Email: " + email);
    }

    // 함수 호출
    printUserInfo(123, "Alice", "alice@example.com");
    printUserInfo(456, "Bob", "bob@example.com");
    ```
2.  **상속과 인터페이스 사용**: 공통된 기능을 부모 클래스나 인터페이스로 추출하여, 이를 상속하거나 구현함으로써 중복을 피할 수 있습니다.

    ```java
    // DRY 원칙을 위반한 코드
    class Dog {
        void makeSound() {
            System.out.println("Bark");
        }
    }

    class Cat {
        void makeSound() {
            System.out.println("Meow");
        }
    }

    // DRY 원칙을 준수한 코드
    abstract class Animal {
        abstract void makeSound();
    }

    class Dog extends Animal {
        @Override
        void makeSound() {
            System.out.println("Bark");
        }
    }

    class Cat extends Animal {
        @Override
        void makeSound() {
            System.out.println("Meow");
        }
    }
    ```
3.  **상수와 변수의 활용**: 반복되는 값이나 표현식을 상수나 변수로 추출하여 중복을 피할 수 있습니다.

    ```java
    // DRY 원칙을 위반한 코드
    double radius1 = 7;
    double area1 = 3.14159 * radius1 * radius1;

    double radius2 = 14;
    double area2 = 3.14159 * radius2 * radius2;

    // DRY 원칙을 준수한 코드
    final double PI = 3.14159;

    double calculateArea(double radius) {
        return PI * radius * radius;
    }

    double area1 = calculateArea(7);
    double area2 = calculateArea(14);
    ```

</details>

***

## 7. 오버로딩과 관련된 키워드

### 7.1 컴파일 타임 결정

오버로딩된 메서드는 **컴파일 타임**에 어떤 메서드가 호출될지가 결정됩니다. 자바 컴파일러는 메서드 호출 시점에서 전달된 인수의 타입과 개수를 분석하여 적절한 메서드를 선택합니다. 이로 인해 오버로딩된 메서드는 성능 면에서 효율적이며, 컴파일러가 이를 최적화할 수 있습니다.

<details>

<summary>class Calculator</summary>

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}

public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println(calc.add(2, 3)); // 컴파일 타임에 add(int, int) 선택
        System.out.println(calc.add(2.5, 3.5)); // 컴파일 타임에 add(double, double) 선택
    }
}
```

이 예제에서 각 `add` 메서드는 컴파일 타임에 적절한 메서드가 선택되어 호출됩니다.

</details>



### 7.2 메서드 시그니처(Method Signature)

**메서드 시그니처**는 메서드 이름과 매개변수 리스트(타입과 순서)를 합한 것을 의미합니다. 오버로딩은 동일한 메서드 이름을 가지면서도 서로 다른 시그니처를 가진 메서드들을 정의할 수 있게 해줍니다.

<details>

<summary>class Printer</summary>

```java
class Printer {
    void print(String message) {
        System.out.println("String: " + message);
    }

    void print(int number) {
        System.out.println("Integer: " + number);
    }

    void print(String message, int number) {
        System.out.println("String: " + message + ", Integer: " + number);
    }
}
```

여기서 `print` 메서드는 서로 다른 시그니처를 가지므로 오버로딩된 메서드로 취급됩니다.

</details>



### 7.3 디폴트 매개변수와 오버로딩

자바에서는 C++ 같은 언어와 달리, 디폴트 매개변수를 직접적으로 지원하지 않습니다. 그러나 오버로딩을 통해 비슷한 효과를 구현할 수 있습니다. 여러 오버로딩된 메서드를 정의하여 매개변수의 개수를 다양하게 받아들이도록 하여, 디폴트 매개변수를 사용하는 것처럼 동작할 수 있습니다.

<details>

<summary>class Greeter</summary>

```java
class Greeter {
    void greet(String name) {
        System.out.println("Hello, " + name);
    }

    void greet() {
        greet("Guest"); // 디폴트 이름으로 오버로딩 사용
    }
}
```

이 예제에서 `greet()` 메서드는 `greet(String name)` 메서드를 호출하면서 기본 값을 전달하여 디폴트 매개변수처럼 작동합니다.

</details>



***

## 8. 오버로딩과 디자인 패턴

### 8.1 팩토리 메서드 패턴(Factory Method Pattern)

**팩토리 메서드 패턴**은 객체 생성의 책임을 서브클래스에 위임하는 디자인 패턴입니다. 오버로딩된 메서드를 통해 팩토리 메서드 패턴을 구현할 수 있으며, 다양한 매개변수에 따라 서로 다른 객체를 생성할 수 있습니다.

<details>

<summary>class ShapeFactory</summary>

```java
class ShapeFactory {
    Shape createShape(String type) {
        if (type.equals("circle")) {
            return new Circle();
        } else if (type.equals("rectangle")) {
            return new Rectangle();
        }
        return null;
    }

    Shape createShape(int sides) {
        if (sides == 3) {
            return new Triangle();
        } else if (sides == 4) {
            return new Square();
        }
        return null;
    }
}
```

이 예제에서 `ShapeFactory` 클래스는 `createShape` 메서드를 오버로딩하여, 입력된 매개변수에 따라 서로 다른 `Shape` 객체를 생성합니다.

</details>



### 8.2 빌더 패턴(Builder Pattern)

**빌더 패턴**은 복잡한 객체를 단계별로 생성할 수 있도록 하는 디자인 패턴입니다. 빌더 패턴에서 오버로딩을 사용하면 객체 생성 시 다양한 조합의 매개변수를 허용하여 객체를 유연하게 구성할 수 있습니다.

<details>

<summary>class Computer</summary>

```java
class Computer {
    private String CPU;
    private int RAM;
    private int storage;

    // 필수 매개변수만으로 생성
    Computer(String CPU) {
        this.CPU = CPU;
    }

    // 필수 매개변수 + 선택적 매개변수로 생성
    Computer(String CPU, int RAM) {
        this(CPU);
        this.RAM = RAM;
    }

    // 모든 매개변수로 생성
    Computer(String CPU, int RAM, int storage) {
        this(CPU, RAM);
        this.storage = storage;
    }
}
```

이 예제에서 `Computer` 클래스는 여러 개의 생성자를 오버로딩하여, 매개변수에 따라 객체를 다양한 방식으로 생성할 수 있습니다.

</details>



***

## 결론

오버로딩은 자바에서 메서드 이름의 일관성을 유지하면서도 다양한 입력을 처리할 수 있도록 하는 강력한 기법입니다. 이를 통해 코드의 가독성과 유연성이 향상되며, 정적 다형성을 구현할 수 있습니다. 오버로딩을 적절히 활용하면 SRP와 DRY 같은 객체지향 원칙을 준수하면서도 다양한 설계 패턴을 효과적으로 구현할 수 있습니다. 이러한 기법을 이해하고 적용함으로써 자바 코드를 더욱 견고하고 유지보수하기 쉽게 만들 수 있습니다.
