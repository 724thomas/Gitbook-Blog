---
description: 오리 시뮬레이션 게임
---

# Ch1. Introduction to SimUDuck

## **SimUDuck의 초기 설계와 문제점**

기본적으로, 모든 오리(클래스)는 `quack()`, `swim()`, `display()` 메서드를 가져야 합니다. 하지만 이 메서드들이 모든 오리에게 공통적인 것이 아니므로, 자식 클래스에서 각기 다르게 구현됩니다. 그러나 새로운 오리 종류가 추가되거나, 행동이 변경될 경우 각 오리 클래스마다 수정이 필요해집니다. 이는 유지보수와 코드 확장에 문제를 야기합니다.

<details>

<summary>오리 클래스</summary>

```java
class Duck {
    void quack() {
        System.out.println("Quack");
    }

    void swim() {
        System.out.println("Swimming");
    }

    void display() {
        System.out.println("I'm a duck!");
    }
}
```

위의 예제에서는 오리의 다양한 행동을 구현하기 위해 다형성을 사용했지만, 새로운 행동을 추가하거나 변경할 때마다 코드가 수정되어야 하는 비효율이 발생합니다.

</details>

## 경고! 심각한 문제 발생

SimUDuck 초기 설계에서는 상속을 사용하여 코드 재사용을 시도했으나, 이는 결국 유연성 부족으로 이어졌습니다. 모든 오리 종류가 `Duck` 클래스를 상속받으면서, 공통적으로 필요한 메서드를 포함하게 되었습니다. 그러나 몇몇 오리들에게는 불필요한 메서드나 잘못된 메서드 구현이 추가되면서 문제가 발생합니다.

<details>

<summary>RubberDuck 클래스</summary>

```java
class RubberDuck extends Duck {
    @Override
    void quack() {
        System.out.println("Squeak");
    }

    @Override
    void display() {
        System.out.println("I'm a rubber duck!");
    }
}
```

RubberDuck의 경우, 소리를 내는 방식이 `quack()`이 아니라 `squeak()`이지만, 부모 클래스에 정의된 메서드를 오버라이드하는 방식으로 처리해야 하므로, 유지보수가 어려워집니다. 이처럼 상속의 남용은 코드의 중복과 비효율성을 초래합니다.

</details>



## 상속을 생각하기

### **전략 패턴 도입**

상속의 단점을 극복하기 위해 **전략 패턴**을 도입합니다. 전략 패턴에서는 오리의 행동을 별도의 클래스로 캡슐화하여, 런타임 시 오브젝트의 행동을 동적으로 변경할 수 있게 합니다. 이렇게 하면 코드의 재사용성과 유연성을 크게 높일 수 있습니다.

<details>

<summary><code>QuackBehavior</code> 인터페이스</summary>

```java
interface QuackBehavior {
    void quack();
}

class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Quack");
    }
}

class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Squeak");
    }
}
```

위와 같이 `QuackBehavior` 인터페이스를 정의하고, 다양한 구현 클래스를 통해 오리의 행동을 분리함으로써 전략 패턴을 적용할 수 있습니다. 이를 통해 각 오리 클래스는 행동을 직접 구현하지 않고, 필요한 행동을 주입받아 사용할 수 있게 됩니다.

</details>



## **인터페이스 설계하기**

전략 패턴의 핵심은 행동을 오리 클래스에서 분리하는 것입니다. 이를 위해 행동 인터페이스를 설계하고, 오리 클래스가 이를 이용하도록 구조를 변경합니다.

<details>

<summary><code>Duck</code> 클래스</summary>

```java
abstract class Duck {
    QuackBehavior quackBehavior;

    public Duck(QuackBehavior quackBehavior) {
        this.quackBehavior = quackBehavior;
    }

    void performQuack() {
        quackBehavior.quack();
    }

    abstract void display();
}
```

이제 `Duck` 클래스는 `QuackBehavior` 인터페이스를 통해 행동을 위임받습니다. 이렇게 하면 새로운 행동을 추가하거나 변경할 때마다 오리 클래스를 수정할 필요 없이, 새로운 `QuackBehavior` 구현체를 만들기만 하면 됩니다.

</details>



## **해결 방법 고민하기**

### **결합도를 낮추는 방법**

이제 결합도를 낮추고, 유연성을 극대화한 구조를 만들어야 합니다. 이를 위해 오리 클래스는 더 이상 특정 행동에 의존하지 않으며, 필요한 행동을 주입받아 사용하게 됩니다.

<details>

<summary><code>MallardDuck</code> 클래스</summary>

```java
class MallardDuck extends Duck {
    public MallardDuck() {
        super(new Quack());
    }

    @Override
    void display() {
        System.out.println("I'm a Mallard Duck");
    }
}
```

`MallardDuck` 클래스는 이제 `QuackBehavior`를 주입받아 사용합니다. 필요에 따라 다른 `QuackBehavior` 구현체를 주입받을 수 있으며, 런타임 시에도 행동을 변경할 수 있습니다.

</details>



## **소프트웨어 개발 불변의 진리**

### **변화와 확장**

소프트웨어 개발에서 유일하게 변하지 않는 진리는 **변화**입니다. 소프트웨어 요구사항은 시간이 지남에 따라 변하기 마련이며, 이 변화에 유연하게 대응할 수 있는 시스템을 구축하는 것이 매우 중요합니다. 이는 SimUDuck 예제에서도 마찬가지로 적용됩니다. 초기 설계에서는 오리의 행동이 각 클래스에 하드코딩되어 있어 변경이 매우 어렵고, 새로운 요구사항에 맞추어 확장하는 것도 제한적이었습니다.



## 문제를 명확하게 파악하기

### **행동의 유연성 부족**

SimUDuck의 초기 설계에서는 각 오리 클래스가 자신의 행동을 직접 정의하고 있었습니다. 이는 행동이 추가되거나 변경될 때마다 각 오리 클래스를 수정해야 하는 문제를 일으켰습니다. 따라서 행동을 클래스 외부로 분리하여 유연성을 확보하는 것이 필요했습니다. 이를 통해 오리 클래스는 자신만의 고유한 속성(예: 외형)에 집중할 수 있게 되고, 행동의 변경이나 확장은 별도의 행동 클래스를 통해 처리할 수 있습니다.



## **바뀌는 부분과 그렇지 않은 부분 분리하기**

소프트웨어 설계의 중요한 원칙 중 하나는 **변하는 것과 변하지 않는 것을 분리**하는 것입니다. 오리 시뮬레이션에서 오리의 외형(예: `display()` 메서드)이나 수영(`swim()` 메서드) 같은 부분은 잘 변하지 않지만, 날거나 소리를 내는 행동은 자주 변경될 수 있습니다. 따라서 이러한 변경 가능성이 높은 부분을 별도의 클래스로 분리하여 관리하면, 코드의 유연성과 재사용성을 크게 높일 수 있습니다.

<details>

<summary><code>Duck</code> 클래스</summary>

```java
abstract class Duck {
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;

    void performFly() {
        flyBehavior.fly();
    }

    void performQuack() {
        quackBehavior.quack();
    }

    void swim() {
        System.out.println("All ducks float, even decoys!");
    }

    abstract void display();

    void setFlyBehavior(FlyBehavior fb) {
        flyBehavior = fb;
    }

    void setQuackBehavior(QuackBehavior qb) {
        quackBehavior = qb;
    }
}
```

위의 코드는 `Duck` 클래스에서 변하지 않는 부분인 `swim()` 메서드를 포함하면서, `flyBehavior`와 `quackBehavior`와 같은 변화할 수 있는 부분을 별도의 행동 인터페이스로 분리한 것입니다. 이렇게 하면 새로운 행동을 추가하거나 기존 행동을 변경할 때 오리 클래스 자체를 수정할 필요가 없어집니다.

</details>



## **오리의 행동을 디자인하는 방법**

### **행동 인터페이스 설계**

오리의 행동을 분리하여 디자인할 때는 각각의 행동을 인터페이스로 정의하고, 다양한 구현 클래스를 통해 구체적인 행동을 표현합니다. 예를 들어, `QuackBehavior`와 `FlyBehavior` 인터페이스를 정의하여 다양한 오리의 행동을 캡슐화할 수 있습니다.

<details>

<summary><code>QuackBehavior</code>와 <code>FlyBehavior</code> 인터페이스</summary>

```java
interface FlyBehavior {
    void fly();
}

interface QuackBehavior {
    void quack();
}

class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I'm flying with wings!");
    }
}

class FlyNoWay implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I can't fly");
    }
}

class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Quack");
    }
}

class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Squeak");
    }
}
```

이러한 인터페이스 설계를 통해 오리의 행동을 유연하게 정의할 수 있습니다. 각각의 오리 클래스는 자신이 필요로 하는 행동을 선택적으로 사용하고, 런타임에 동적으로 변경할 수 있습니다.

</details>



## **오리의 행동을 구현하는 방법**

### **구체적 행동 클래스**

위에서 정의한 인터페이스를 기반으로 오리의 구체적인 행동을 구현할 수 있습니다. 이와 같이 행동을 별도의 클래스로 구현하면, 새로운 행동이 필요할 때 기존 코드를 수정할 필요 없이 새로운 클래스를 추가하는 방식으로 손쉽게 확장할 수 있습니다.

<details>

<summary><code>MallardDuck</code>과 <code>RubberDuck</code> 클래스</summary>

```java
class MallardDuck extends Duck {
    public MallardDuck() {
        flyBehavior = new FlyWithWings();
        quackBehavior = new Quack();
    }

    @Override
    void display() {
        System.out.println("I'm a Mallard Duck");
    }
}

class RubberDuck extends Duck {
    public RubberDuck() {
        flyBehavior = new FlyNoWay();
        quackBehavior = new Squeak();
    }

    @Override
    void display() {
        System.out.println("I'm a Rubber Duck");
    }
}
```

`MallardDuck`과 `RubberDuck` 클래스는 각각 고유한 행동을 가지며, 필요에 따라 행동을 변경할 수 있는 구조로 설계되었습니다. 예를 들어 `RubberDuck`은 날 수 없고(`FlyNoWay`), `MallardDuck`은 날 수 있습니다(`FlyWithWings`). 또한 소리 내는 방식도 각기 다릅니다(`Quack` vs `Squeak`).

</details>





***

## **소프트웨어 개발 불변의 진리**

### **변화와 확장**

소프트웨어 개발에서 유일하게 변하지 않는 진리는 **변화**입니다. 소프트웨어 요구사항은 시간이 지남에 따라 변하기 마련이며, 이 변화에 유연하게 대응할 수 있는 시스템을 구축하는 것이 매우 중요합니다. 이는 SimUDuck 예제에서도 마찬가지로 적용됩니다. 초기 설계에서는 오리의 행동이 각 클래스에 하드코딩되어 있어 변경이 매우 어렵고, 새로운 요구사항에 맞추어 확장하는 것도 제한적이었습니다.

***

### **문제를 명확하게 파악하기**

### **행동의 유연성 부족**

SimUDuck의 초기 설계에서는 각 오리 클래스가 자신의 행동을 직접 정의하고 있었습니다. 이는 행동이 추가되거나 변경될 때마다 각 오리 클래스를 수정해야 하는 문제를 일으켰습니다. 따라서 행동을 클래스 외부로 분리하여 유연성을 확보하는 것이 필요했습니다. 이를 통해 오리 클래스는 자신만의 고유한 속성(예: 외형)에 집중할 수 있게 되고, 행동의 변경이나 확장은 별도의 행동 클래스를 통해 처리할 수 있습니다.

***

### **바뀌는 부분과 그렇지 않은 부분 분리하기**

소프트웨어 설계의 중요한 원칙 중 하나는 **변하는 것과 변하지 않는 것을 분리**하는 것입니다. 오리 시뮬레이션에서 오리의 외형(예: `display()` 메서드)이나 수영(`swim()` 메서드) 같은 부분은 잘 변하지 않지만, 날거나 소리를 내는 행동은 자주 변경될 수 있습니다. 따라서 이러한 변경 가능성이 높은 부분을 별도의 클래스로 분리하여 관리하면, 코드의 유연성과 재사용성을 크게 높일 수 있습니다.

<details>

<summary><code>Duck</code> 클래스</summary>

```java
abstract class Duck {
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;

    void performFly() {
        flyBehavior.fly();
    }

    void performQuack() {
        quackBehavior.quack();
    }

    void swim() {
        System.out.println("All ducks float, even decoys!");
    }

    abstract void display();

    void setFlyBehavior(FlyBehavior fb) {
        flyBehavior = fb;
    }

    void setQuackBehavior(QuackBehavior qb) {
        quackBehavior = qb;
    }
}
```

위의 코드는 `Duck` 클래스에서 변하지 않는 부분인 `swim()` 메서드를 포함하면서, `flyBehavior`와 `quackBehavior`와 같은 변화할 수 있는 부분을 별도의 행동 인터페이스로 분리한 것입니다. 이렇게 하면 새로운 행동을 추가하거나 기존 행동을 변경할 때 오리 클래스 자체를 수정할 필요가 없어집니다.

</details>

***

### **오리의 행동을 디자인하는 방법**

### **행동 인터페이스 설계**

오리의 행동을 분리하여 디자인할 때는 각각의 행동을 인터페이스로 정의하고, 다양한 구현 클래스를 통해 구체적인 행동을 표현합니다. 예를 들어, `QuackBehavior`와 `FlyBehavior` 인터페이스를 정의하여 다양한 오리의 행동을 캡슐화할 수 있습니다.

<details>

<summary><code>QuackBehavior</code>와 <code>FlyBehavior</code> 인터페이스</summary>

```java
interface FlyBehavior {
    void fly();
}

interface QuackBehavior {
    void quack();
}

class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I'm flying with wings!");
    }
}

class FlyNoWay implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I can't fly");
    }
}

class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Quack");
    }
}

class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Squeak");
    }
}
```

이러한 인터페이스 설계를 통해 오리의 행동을 유연하게 정의할 수 있습니다. 각각의 오리 클래스는 자신이 필요로 하는 행동을 선택적으로 사용하고, 런타임에 동적으로 변경할 수 있습니다.

</details>

***

### **오리의 행동을 구현하는 방법**

### **구체적 행동 클래스**

위에서 정의한 인터페이스를 기반으로 오리의 구체적인 행동을 구현할 수 있습니다. 이와 같이 행동을 별도의 클래스로 구현하면, 새로운 행동이 필요할 때 기존 코드를 수정할 필요 없이 새로운 클래스를 추가하는 방식으로 손쉽게 확장할 수 있습니다.

<details>

<summary><code>MallardDuck</code>과 <code>RubberDuck</code> 클래스</summary>

```java
class MallardDuck extends Duck {
    public MallardDuck() {
        flyBehavior = new FlyWithWings();
        quackBehavior = new Quack();
    }

    @Override
    void display() {
        System.out.println("I'm a Mallard Duck");
    }
}

class RubberDuck extends Duck {
    public RubberDuck() {
        flyBehavior = new FlyNoWay();
        quackBehavior = new Squeak();
    }

    @Override
    void display() {
        System.out.println("I'm a Rubber Duck");
    }
}
```

`MallardDuck`과 `RubberDuck` 클래스는 각각 고유한 행동을 가지며, 필요에 따라 행동을 변경할 수 있는 구조로 설계되었습니다. 예를 들어 `RubberDuck`은 날 수 없고(`FlyNoWay`), `MallardDuck`은 날 수 있습니다(`FlyWithWings`). 또한 소리 내는 방식도 각기 다릅니다(`Quack` vs `Squeak`).

</details>

***

## **오리 행동 통합하기**

### **다양한 행동의 조합**

전략 패턴을 통해 오리의 행동을 다양한 방식으로 통합할 수 있습니다. 이 패턴의 강점은 동일한 오리 클래스가 상황에 따라 서로 다른 행동을 보일 수 있다는 것입니다. 이를 통해 코드의 유연성을 극대화할 수 있습니다.

<details>

<summary>최종 코드</summary>

```java
import lombok.AllArgsConstructor;

@AllArgsConstructor
abstract class Duck {
    QuackBehavior quackBehavior;
    FlyBehavior flyBehavior;

    void performQuack() {
        quackBehavior.quack();
    }

    void performFly() {
        flyBehavior.fly();
    }

    abstract void display();

    void setQuackBehavior(QuackBehavior qb) {
        this.quackBehavior = qb;
    }

    void setFlyBehavior(FlyBehavior fb) {
        this.flyBehavior = fb;
    }
}

interface QuackBehavior {
    void quack();
}

class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Quack");
    }
}

class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Squeak");
    }
}

interface FlyBehavior {
    void fly();
}

class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I'm flying with wings!");
    }
}

class FlyNoWay implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I can't fly");
    }
}

class MallardDuck extends Duck {
    public MallardDuck() {
        super(new Quack(), new FlyWithWings());
    }

    @Override
    void display() {
        System.out.println("I'm a Mallard Duck");
    }
}

class RubberDuck extends Duck {
    public RubberDuck() {
        super(new Squeak(), new FlyNoWay());
    }

    @Override
    void display() {
        System.out.println("I'm a Rubber Duck");
    }
}
```

위의 코드에서는 `QuackBehavior`와 `FlyBehavior` 두 가지 행동을 통합하여 오리의 행동을 정의하고 있습니다. `MallardDuck`과 `RubberDuck`은 각각의 행동을 서로 다르게 설정하여, 각 오리의 특성에 맞는 행동을 하도록 설계되었습니다.

이러한 통합 방식을 통해 특정 오리의 행동을 쉽게 변경할 수 있으며, 다른 오리 클래스에서 동일한 행동을 재사용할 수 있어 코드의 중복을 줄이고 유지보수를 간편하게 할 수 있습니다.

</details>



## **오리 코드 테스트**

### **단위 테스트 작성**

전략 패턴을 사용하여 설계된 오리 행동은 테스트하기가 훨씬 용이합니다. 각 행동을 독립적으로 테스트할 수 있으며, 오리 클래스에 행동을 주입하여 올바르게 작동하는지 검증할 수 있습니다.

<details>

<summary>DuckTest 클래스</summary>

```java
public class DuckTest {
    public static void main(String[] args) {
        // MallardDuck 테스트
        Duck mallard = new MallardDuck();
        mallard.performQuack(); // Quack 출력
        mallard.performFly();   // I'm flying with wings! 출력

        // 행동 변경
        mallard.setFlyBehavior(new FlyNoWay());
        mallard.performFly();   // I can't fly 출력

        // RubberDuck 테스트
        Duck rubberDuck = new RubberDuck();
        rubberDuck.performQuack(); // Squeak 출력
        rubberDuck.performFly();   // I can't fly 출력
    }
}
```

위의 테스트 코드는 오리의 다양한 행동을 확인하는 데 사용됩니다. `MallardDuck`은 초기 설정에서 날 수 있지만, 런타임에 `FlyNoWay`로 변경하여 더 이상 날지 못하게 만들 수 있습니다. `RubberDuck`의 경우 처음부터 날 수 없는 상태로 설정되어 있으며, 고무 오리가 삑삑 소리를 내는지 확인할 수 있습니다.

전략 패턴을 사용하면 이렇게 런타임에 행동을 쉽게 변경할 수 있어 유연한 코드 작성이 가능합니다.

</details>



## **동적으로 행동 지정하기**

### **런타임에서의 행동 변경**

전략 패턴의 또 다른 강점은 런타임에서 행동을 동적으로 변경할 수 있다는 점입니다. 이는 프로그램 실행 중에도 오리의 행동을 자유롭게 바꿀 수 있어 유연한 설계가 가능합니다.

<details>

<summary>예시 코드</summary>

```java
Duck mallard = new MallardDuck();
mallard.performFly(); // I'm flying with wings! 출력

// 런타임에서 행동 변경
mallard.setFlyBehavior(new FlyNoWay());
mallard.performFly(); // I can't fly 출력
```

위 코드에서 보듯이, `MallardDuck`은 초기 설정에서 날 수 있지만, 런타임에 행동을 변경하여 날지 못하게 할 수 있습니다. 이와 같은 기능은 상황에 따라 프로그램의 동작을 유연하게 조정할 수 있게 해줍니다. 이러한 동적 행동 변경은 다양한 시나리오에서 재사용 가능한 코드를 작성하는 데 매우 유용합니다.

</details>



## **캡슐화된 행동 살펴보기**

### **내부 구현의 변화**

캡슐화된 행동을 사용하면, 행동의 내부 구현을 변경해도 외부 클래스에는 영향을 미치지 않습니다. 예를 들어, `FlyBehavior`의 `FlyWithWings` 구현이 변경되어도 `Duck` 클래스나 그 하위 클래스는 전혀 수정할 필요가 없습니다.

<details>

<summary><code>FlyWithWings</code>의 구현</summary>

```java
class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I'm soaring with wings!"); // 변경된 출력 메시지
    }
}
```

위와 같이 `FlyWithWings`의 구현이 변경되었지만, 오리 클래스는 이 변화를 전혀 알 필요가 없습니다. 이는 코드의 유지보수를 간편하게 하고, 다양한 요구사항에 따라 행동을 변경할 수 있는 유연성을 제공합니다.

캡슐화는 객체 지향 설계의 핵심 개념으로, 시스템의 각 부분을 독립적으로 개발, 수정할 수 있게 해줍니다. 이는 코드의 안정성을 높이고, 오류 발생 가능성을 줄이는 데 큰 도움이 됩니다.

</details>



## **두 클래스를 합치는 방법**

### **Combining Two Classes: 합성 대 상속의 선택**

전략 패턴에서는 \*\*합성(composition)\*\*을 사용하여 오리의 행동을 결합합니다. 이는 \*\*상속(inheritance)\*\*보다 유연성과 재사용성을 높이는 방법입니다. 상속은 클래스 간 강한 결합도를 초래하지만, 합성은 독립적인 행동 클래스를 구성 요소로 사용하여 결합도를 낮출 수 있습니다.

<details>

<summary>예시 코드</summary>

```java
// 상속 대신 합성을 사용하여 유연성 확보
Duck customDuck = new MallardDuck();
customDuck.setFlyBehavior(new FlyNoWay());
customDuck.setQuackBehavior(new Squeak());
customDuck.performFly(); // I can't fly 출력
customDuck.performQuack(); // Squeak 출력
```

위의 예제에서 `MallardDuck`의 행동을 `Squeak`과 `FlyNoWay`로 변경하였습니다. 이를 통해 필요에 따라 오리의 행동을 동적으로 변경할 수 있는 유연성을 제공하며, 재사용성과 확장성을 극대화할 수 있습니다.

합성은 시스템의 모듈화를 촉진하고, 각 모듈이 독립적으로 동작할 수 있게 하여, 변화에 유연하게 대응할 수 있는 설계를 가능하게 합니다.

</details>



## **첫 번째 디자인 패턴: 전략 패턴**

### **전략 패턴의 강점**

전략 패턴은 행동을 클래스 외부로 분리하여 코드의 유연성을 높이고, 중복을 최소화하는 데 매우 효과적입니다. 이 패턴을 통해 새로운 행동을 추가하거나 변경하는 작업이 쉬워지며, 런타임 시 행동을 동적으로 변경할 수 있어 다양한 시나리오에 대응할 수 있습니다.

<details>

<summary>MuteQuack 클래스</summary>

```java
// 전략 패턴을 사용한 새로운 행동 추가
class MuteQuack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("<< Silence >>");
    }
}

Duck silentDuck = new MallardDuck();
silentDuck.setQuackBehavior(new MuteQuack());
silentDuck.performQuack(); // << Silence >> 출력
```

위 코드에서는 `MuteQuack`이라는 새로운 행동을 추가하여, 오리가 소리를 내지 않도록 설정할 수 있습니다. 기존 오리 클래스나 행동 클래스를 전혀 수정하지 않고도, 새로운 기능을 쉽게 추가할 수 있는 것이 전략 패턴의 큰 장점입니다.

</details>

전략 패턴은 소프트웨어 디자인에서 매우 중요한 패턴 중 하나로, 다양한 상황에서 사용될 수 있습니다. 이는 특히 동적 행동 변화가 필요한 경우에 매우 유용하며, 코드의 재사용성과 유지보수성을 높이는 데 큰 역할을 합니다.



## **디자인 패턴 안내서**

### **전략 패턴 외의 다른 패턴들**

소프트웨어 개발에서 디자인 패턴은 자주 발생하는 설계 문제를 해결하기 위한 표준화된 방법입니다. 전략 패턴 외에도 다양한 디자인 패턴이 있으며, 각각의 패턴은 특정 문제를 해결하는 데 특화되어 있습니다.

예를 들어, \*\*옵저버 패턴(Observer Pattern)\*\*은 객체 간의 일대다 관계를 정의하여 한 객체의 상태가 변경될 때 그와 연관된 다른 객체들이 자동으로 알림을 받고, 갱신될 수 있도록 하는 패턴입니다. 이 패턴은 이벤트 중심의 프로그램에서 자주 사용되며, 한 객체의 상태 변화에 따라 여러 객체들이 동기화될 수 있는 유연한 설계를 제공합니다.

다음 장에서는 다양한 디자인 패턴을 소개하고, 실습을 통해 이해를 돕겠습니다. 각 패턴이 어떤 상황에서 유용한지, 그리고 어떻게 구현할 수 있는지를 살펴봅니다.

***

## **패턴 전문 용어**

### **주요 개념 정리**

디자인 패턴을 이해하기 위해서는 몇 가지 중요한 용어를 숙지해야 합니다. 여기서는 전략 패턴과 관련된 주요 용어를 정리해 보겠습니다.

1. **전략(Strategy):** 특정 행동을 캡슐화하여 동적으로 교체할 수 있는 알고리즘군을 정의합니다. 전략은 알고리즘의 변형을 사용자가 직접 선택할 수 있게 하는 패턴입니다.
2. **컨텍스트(Context):** 전략 객체를 사용하는 환경을 의미합니다. 오리 예제에서 `Duck` 클래스가 바로 컨텍스트에 해당합니다.
3. **구체적인 전략(Concrete Strategy):** 전략 인터페이스를 구현한 클래스들입니다. 예를 들어, `Quack`, `Squeak`, `MuteQuack` 클래스들이 여기에 속합니다.
4. **캡슐화(Encapsulation):** 객체의 데이터를 외부에서 직접 접근하지 못하도록 하고, 메서드를 통해서만 접근할 수 있도록 하는 원칙입니다. 디자인 패턴에서 자주 사용되는 개념입니다.

이러한 용어들을 이해하면, 디자인 패턴의 적용이 훨씬 수월해집니다. 각각의 용어가 실제 코드에서 어떻게 사용되는지 예시를 통해 학습하면 패턴을 적용할 때 더 큰 도움이 될 것입니다.



## **디자인 패턴 사용법**

### **실습을 통한 이해**

디자인 패턴은 소프트웨어 개발에서 매우 중요한 도구입니다. 이를 효과적으로 사용하려면 실제 프로젝트에 적용해보는 것이 가장 좋습니다.&#x20;

예를 들어, 웹 애플리케이션에서 사용자가 선택한 결제 방식에 따라 다른 결제 알고리즘을 실행해야 하는 상황을 가정해봅시다. 이때 전략 패턴을 적용하면 결제 방식이 추가되거나 변경될 때 코드를 간단히 확장할 수 있습니다.

<details>

<summary>예시 코드</summary>

<pre class="language-java"><code class="lang-java"><strong>public interface PaymentStrategy {
</strong>    void pay(int amount);
}

public class CreditCardPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}

public class PaypalPayment implements PaymentStrategy {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}

public class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}
</code></pre>

이 코드는 사용자가 결제 방식을 선택할 수 있게 하며, 새로운 결제 방식이 추가될 때마다 `PaymentStrategy`를 구현하는 새로운 클래스를 추가하기만 하면 됩니다. 실제로 `ShoppingCart`는 결제 방식의 세부사항을 알 필요가 없으며, 오직 `PaymentStrategy` 인터페이스를 통해 결제 메서드를 호출합니다. 이렇게 하면 코드의 유연성과 확장성을 높일 수 있습니다.

</details>

## **패턴 전문 용어**

### **심화된 개념 학습**

디자인 패턴을 이해하기 위해서는 기본적인 용어 외에도 심화된 개념을 이해하는 것이 중요합니다. 여기서는 패턴 사용 시 자주 등장하는 몇 가지 심화 용어를 설명하겠습니다.

1. **의존성 주입(Dependency Injection, DI):** 객체가 스스로 필요한 객체를 생성하는 것이 아니라 외부에서 필요한 객체를 주입받아 사용하는 방식입니다. 이는 객체 간의 결합도를 낮추고, 유연한 설계를 가능하게 합니다.
2. **싱글톤(Singleton):** 클래스의 인스턴스를 하나만 생성하여 전역적으로 접근할 수 있게 하는 디자인 패턴입니다. 특정 자원이 하나만 존재해야 하는 경우에 유용합니다.
3. **팩토리 패턴(Factory Pattern):** 객체 생성 코드를 캡슐화하여, 객체 생성 로직을 변경하지 않고도 다양한 타입의 객체를 생성할 수 있도록 하는 패턴입니다. 클라이언트 코드의 변경 없이 객체 생성 방식을 변경할 수 있습니다.

이러한 심화 개념은 대규모 소프트웨어 설계에서 매우 유용합니다. 실제 개발 환경에서 디자인 패턴을 활용할 때 이러한 개념을 염두에 두고 설계하면 더욱 효율적인 코드를 작성할 수 있습니다.

***
