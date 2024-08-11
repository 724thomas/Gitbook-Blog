---
description: 전략패턴
---

# Ch1. Strategy Pattern

## **첫 번째 디자인 패턴: 전략 패턴**

전략 패턴은 소프트웨어 디자인에서 매우 중요한 패턴 중 하나로, 다양한 상황에서 사용될 수 있습니다. 이는 특히 동적 행동 변화가 필요한 경우에 매우 유용하며, 코드의 재사용성과 유지보수성을 높이는 데 큰 역할을 합니다.

전략 패턴(Strategy Pattern)은 오리의 행동을 클래스 외부로 분리하여, 코드의 유연성을 높이고 중복을 최소화하는 데 매우 효과적입니다. 이 패턴을 통해 새로운 행동을 추가하거나 변경하는 작업이 쉬워지며, 런타임 시 행동을 동적으로 변경할 수 있어 다양한 시나리오에 대응할 수 있습니다.



## **오리 시뮬레이션 게임, SimUDuck**

**SimUDuck**은 다양한 오리 캐릭터를 가지고 여러 가지 시뮬레이션을 진행하는 게임입니다. 이 게임의 기본적인 목표는 다양한 종류의 오리가 각기 다른 행동을 수행하게 하는 것입니다. 예를 들어, 어떤 오리는 날 수 있고, 어떤 오리는 삑삑 소리를 내며, 또 어떤 오리는 단순히 수영만 할 수 있습니다.

초기 설계에서 SimUDuck의 오리들은 `Duck`이라는 기본 클래스를 상속받아 여러 가지 행동을 구현합니다. 모든 오리 클래스는 `quack()`, `swim()`, `display()` 메서드를 구현해야 합니다. 그러나 이러한 설계에는 몇 가지 문제가 발생할 수 있습니다.



## **오리 시뮬레이션 게임 차별화하기**

기본적으로, 모든 오리 클래스는 `quack()`, `swim()`, `display()`와 같은 공통 메서드를 가져야 합니다. 각 오리 클래스는 `display()` 메서드를 통해 자신의 외형을 나타내고, `quack()` 메서드를 통해 소리를 냅니다. 또한, `swim()` 메서드를 통해 물 위에서 헤엄치는 동작을 수행합니다.

<details>

<summary><code>Duck</code> 클래스</summary>

```java
public class Duck {
    public void quack() {
        System.out.println("Quack");
    }

    public void swim() {
        System.out.println("I'm swimming!");
    }

    public void display() {
        System.out.println("I'm a duck!");
    }
}
```

이 초기 설계에서 모든 오리들은 동일한 `quack()`, `swim()` 메서드를 상속받아 사용하게 됩니다. 오리의 외형은 `display()` 메서드에서 정의됩니다. 하지만, 오리의 행동이 모두 동일하지 않다는 점에서 문제가 발생할 수 있습니다.

</details>



## **경고! 심각한 문제 발생**

초기 설계에서는 상속을 사용하여 코드의 재사용성을 높이려 했으나, 이는 유연성 부족으로 이어졌습니다. 모든 오리 클래스는 `Duck` 클래스를 상속받으며, 공통적으로 필요한 메서드를 포함하게 됩니다. 그러나 몇몇 오리에게는 불필요한 메서드가 존재하거나, 잘못된 메서드 구현이 강제될 수 있습니다.

예를 들어, 고무 오리(`RubberDuck`)는 날 수 없고, 삑삑 소리를 내야 합니다. 하지만 `Duck` 클래스를 상속받아 `quack()`과 `fly()` 메서드를 오버라이드해야 합니다. 이는 코드 중복과 비효율적인 설계로 이어지며, 새로운 오리 종류가 추가되거나 행동이 변경될 경우 모든 관련된 오리 클래스들을 수정해야 하는 문제를 초래합니다.

이러한 문제로 인해 코드의 유지보수성과 확장성이 저하되며, 오리의 행동이 변경될 때마다 수많은 클래스를 수정해야 하는 비효율적인 상황이 발생합니다.



## **상속을 생각하기**

SimUDuck의 초기 설계에서는 상속을 사용하여 오리들의 공통적인 행동을 재사용하려고 했습니다. `Duck` 클래스를 만들어 모든 오리 클래스들이 이 클래스를 상속받도록 했습니다. 이를 통해 `quack()`, `swim()`, `fly()`와 같은 공통적인 행동을 각 오리 클래스에서 재사용할 수 있었습니다. 그러나, 이 방식은 오리의 종류가 늘어나고 행동이 다양해지면서 여러 문제를 야기했습니다.

<details>

<summary><code>Duck</code> 클래스</summary>

```java
public class Duck {
    public void quack() {
        System.out.println("Quack");
    }

    public void swim() {
        System.out.println("I'm swimming!");
    }

    public void fly() {
        System.out.println("I'm flying!");
    }

    public void display() {
        System.out.println("I'm a duck!");
    }
}
```

모든 오리들은 `Duck` 클래스를 상속받아 이 메서드들을 사용하게 되지만, 새로운 오리 종류가 추가되거나, 특정 오리가 독특한 행동을 하게 되는 경우에 문제가 발생합니다. 예를 들어, 고무 오리(`RubberDuck`)는 날 수 없고, 삑삑 소리를 내야 합니다. 이 경우 `fly()`와 `quack()` 메서드를 적절히 오버라이드해야 하지만, 상속 구조로 인해 불필요한 메서드들이 그대로 남아있게 됩니다.

</details>



## **인터페이스 설계하기**

상속의 단점을 극복하기 위해, 각 오리의 행동을 인터페이스로 분리하여 설계할 수 있습니다. 예를 들어, `FlyBehavior`와 `QuackBehavior`라는 두 개의 인터페이스를 정의하고, 오리 클래스는 이 인터페이스들을 구현한 클래스를 사용해 자신만의 행동을 설정하게 됩니다. 이를 통해 오리 클래스의 행동을 유연하게 관리할 수 있습니다.

<details>

<summary><code>FlyBehavior</code>와 <code>QuackBehavior 인터페이스</code></summary>

```java
// FlyBehavior 인터페이스
public interface FlyBehavior {
    void fly();
}

// QuackBehavior 인터페이스
public interface QuackBehavior {
    void quack();
}

// 날 수 있는 행동을 정의한 FlyWithWings 클래스
public class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I'm flying with wings!");
    }
}

// 날 수 없는 행동을 정의한 FlyNoWay 클래스
public class FlyNoWay implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I can't fly.");
    }
}

// 일반적인 울음소리를 정의한 Quack 클래스
public class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Quack");
    }
}

// 삑삑 소리를 정의한 Squeak 클래스
public class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Squeak");
    }
}
```

이제 오리 클래스는 이러한 행동 인터페이스들을 조합하여 원하는 행동을 설정할 수 있습니다. 예를 들어, `MallardDuck`은 `FlyWithWings`와 `Quack`을 사용할 수 있고, `RubberDuck`은 `FlyNoWay`와 `Squeak`을 사용할 수 있습니다.

</details>



## **해결 방법 고민하기**

인터페이스 기반의 전략 패턴을 도입함으로써, 코드의 결합도를 낮추고 유연성을 극대화할 수 있습니다. 오리 클래스는 더 이상 특정 행동에 의존하지 않으며, 필요한 행동을 인터페이스를 통해 주입받아 사용하게 됩니다.

<details>

<summary>Duck, MallardDuck, RubberDuck  클래스</summary>

```java
public abstract class Duck {
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;

    public Duck() {}

    public void performFly() {
        flyBehavior.fly();
    }

    public void performQuack() {
        quackBehavior.quack();
    }

    public void swim() {
        System.out.println("All ducks float, even decoys!");
    }

    public void setFlyBehavior(FlyBehavior fb) {
        flyBehavior = fb;
    }

    public void setQuackBehavior(QuackBehavior qb) {
        quackBehavior = qb;
    }

    abstract void display();
}

public class MallardDuck extends Duck {
    public MallardDuck() {
        flyBehavior = new FlyWithWings();
        quackBehavior = new Quack();
    }

    @Override
    public void display() {
        System.out.println("I'm a real Mallard Duck");
    }
}

public class RubberDuck extends Duck {
    public RubberDuck() {
        flyBehavior = new FlyNoWay();
        quackBehavior = new Squeak();
    }

    @Override
    public void display() {
        System.out.println("I'm a rubber duckie");
    }
}
```

이제 `MallardDuck`과 `RubberDuck`은 각각 다른 `FlyBehavior`와 `QuackBehavior`를 사용할 수 있습니다. 새로운 행동을 추가하고 싶다면, 단지 새로운 `FlyBehavior` 또는 `QuackBehavior`를 구현한 클래스를 추가하기만 하면 됩니다. 기존의 오리 클래스는 전혀 수정할 필요가 없습니다.

</details>



## **소프트웨어 개발 불변의 진리**

소프트웨어 개발에서 변하지 않는 진리는 **변화**입니다. SimUDuck의 초기 설계에서도 이러한 변화에 유연하게 대응하는 것이 매우 중요합니다. 처음에는 간단해 보였던 요구사항이 시간이 지남에 따라 복잡해지고, 새로운 행동이나 오리 종류가 추가되면서 코드의 유연성과 확장성이 중요한 이슈로 부각됩니다.

전략 패턴을 도입하여 각 행동을 인터페이스로 분리함으로써, 오리 클래스의 변경 없이도 새로운 행동을 추가하거나 기존 행동을 변경할 수 있습니다. 이로 인해 코드의 유지보수성이 크게 향상되고, 시스템의 확장도 쉬워집니다.

## **문제를 명확하게 파악하기**

SimUDuck의 초기 설계에서는 각 오리 클래스가 자신의 행동을 직접 정의하고 있었습니다. 이는 행동이 추가되거나 변경될 때마다 모든 오리 클래스를 수정해야 하는 문제를 일으켰습니다. 이 문제는 코드의 중복을 낳고, 유지보수를 어렵게 만듭니다.

이 문제를 해결하기 위해 행동을 클래스 외부로 분리하고, 각 행동을 인터페이스와 구현 클래스로 캡슐화하는 방식을 채택할 수 있습니다. 이를 통해 오리 클래스는 자신만의 고유한 속성(예: 외형)에 집중할 수 있게 되고, 행동의 변경이나 확장은 별도의 행동 클래스를 통해 처리할 수 있습니다.

<details>

<summary>Duck 클래스</summary>

```java
// 문제점 예시: 초기 설계에서는 오리 클래스에서 행동이 하드코딩됨
public class RedheadDuck extends Duck {
    @Override
    public void display() {
        System.out.println("I'm a Redhead Duck");
    }

    @Override
    public void fly() {
        System.out.println("I'm flying");
    }

    @Override
    public void quack() {
        System.out.println("Quack");
    }
}
```

위와 같은 초기 설계는 특정 오리의 행동이 하드코딩되어 있어, 변경이 필요할 때마다 오리 클래스 자체를 수정해야 하는 문제가 발생합니다. 이는 유지보수 비용을 높이고, 코드의 유연성을 떨어뜨립니다.

</details>



## **바뀌는 부분과 그렇지 않은 부분 분리하기**

소프트웨어 설계의 중요한 원칙 중 하나는 **변하는 것과 변하지 않는 것을 분리**하는 것입니다. 오리 시뮬레이션에서 오리의 외형(예: `display()` 메서드)이나 수영(`swim()` 메서드) 같은 부분은 잘 변하지 않지만, 날거나 소리를 내는 행동은 자주 변경될 수 있습니다.

따라서 이러한 변경 가능성이 높은 부분을 별도의 클래스로 분리하여 관리하면, 코드의 유연성과 재사용성을 크게 높일 수 있습니다. 전략 패턴을 통해 이러한 변경 가능성이 높은 부분을 독립적으로 관리할 수 있게 됩니다.

<details>

<summary>Duck 클래스</summary>

```java
// 변하지 않는 부분: 오리의 외형과 수영 동작
public abstract class Duck {
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;

    public void swim() {
        System.out.println("All ducks float, even decoys!");
    }

    public abstract void display();
}

// 변하는 부분: 날기와 소리 내기 동작은 전략 패턴으로 분리
public class ModelDuck extends Duck {
    public ModelDuck() {
        flyBehavior = new FlyNoWay(); // 초기 설정: 날 수 없음
        quackBehavior = new Quack();  // 초기 설정: 일반적인 울음소리
    }

    @Override
    public void display() {
        System.out.println("I'm a model duck");
    }
}
```

이제 오리 클래스는 변하지 않는 부분(외형과 수영)에만 집중하고, 변하는 부분(날기와 울음소리)은 전략 패턴을 통해 관리하게 됩니다. 이로써 코드의 유지보수성과 확장성을 극대화할 수 있습니다.

</details>



## **오리의 행동을 디자인하는 방법 - 행동 인터페이스 설계**

오리의 행동을 분리하여 디자인할 때는 각각의 행동을 인터페이스로 정의하고, 다양한 구현 클래스를 통해 구체적인 행동을 표현합니다. 예를 들어, `FlyBehavior`와 `QuackBehavior`라는 두 가지 인터페이스를 정의하여 오리의 날기와 소리 내기 행동을 캡슐화할 수 있습니다.

<details>

<summary><code>FlyBehavior</code>와 <code>QuackBehavior</code> 인터페이스</summary>

```java
// FlyBehavior 인터페이스
public interface FlyBehavior {
    void fly();
}

// QuackBehavior 인터페이스
public interface QuackBehavior {
    void quack();
}
```

이 인터페이스들은 오리 클래스에서 구체적인 행동을 정의하는 대신, 각각의 행동을 외부 클래스에 맡길 수 있게 해줍니다. 이렇게 하면 오리 클래스는 행동에 대한 구체적인 구현에 의존하지 않으며, 행동을 동적으로 교체하거나 확장할 수 있는 유연성을 확보할 수 있습니다.

</details>



## **오리의 행동을 구현하는 방법 - 구체적 행동 클래스**

위에서 정의한 인터페이스를 기반으로, 오리의 구체적인 행동을 구현할 수 있습니다. 이와 같이 행동을 별도의 클래스로 구현하면, 새로운 행동이 필요할 때 기존 코드를 수정할 필요 없이 새로운 클래스를 추가하는 방식으로 손쉽게 확장할 수 있습니다.

<details>

<summary>수정 코드</summary>

```java
// 날 수 있는 행동을 정의한 FlyWithWings 클래스
public class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I'm flying with wings!");
    }
}

// 날 수 없는 행동을 정의한 FlyNoWay 클래스
public class FlyNoWay implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I can't fly.");
    }
}

// 일반적인 울음소리를 정의한 Quack 클래스
public class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Quack");
    }
}

// 삑삑 소리를 정의한 Squeak 클래스
public class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Squeak");
    }
}
```

이제 오리 클래스들은 각기 다른 `FlyBehavior`와 `QuackBehavior`를 사용하여 다양한 행동을 가질 수 있습니다. 예를 들어, `MallardDuck`은 `FlyWithWings`와 `Quack`을 사용하고, `RubberDuck`은 `FlyNoWay`와 `Squeak`을 사용할 수 있습니다. 새로운 행동이 필요할 때는 단지 새로운 `FlyBehavior` 또는 `QuackBehavior` 클래스를 구현하면 됩니다.

</details>



## **오리 행동 통합하기 - 다양한 행동의 조합**

전략 패턴을 통해 오리의 행동을 다양한 방식으로 통합할 수 있습니다. 이 패턴의 강점은 동일한 오리 클래스가 상황에 따라 서로 다른 행동을 보일 수 있다는 것입니다. 이를 통해 코드의 유연성을 극대화할 수 있습니다.

<details>

<summary>MallardDuck, FlywithWings 클래스</summary>

```java
public class MallardDuck extends Duck {
    public MallardDuck() {
        flyBehavior = new FlyWithWings();
        quackBehavior = new Quack();
    }

    @Override
    public void display() {
        System.out.println("I'm a real Mallard Duck");
    }
}

public class RubberDuck extends Duck {
    public RubberDuck() {
        flyBehavior = new FlyNoWay();
        quackBehavior = new Squeak();
    }

    @Override
    public void display() {
        System.out.println("I'm a rubber duckie");
    }
}
```

`MallardDuck`은 `FlyWithWings`와 `Quack` 행동을 가지며, `RubberDuck`은 `FlyNoWay`와 `Squeak` 행동을 가집니다. 이처럼 행동을 조합하여 다양한 오리 클래스를 만들 수 있으며, 필요에 따라 행동을 동적으로 변경할 수도 있습니다.

</details>



## **오리 코드 테스트 - 단위 테스트 작성**

전략 패턴을 사용하여 설계된 오리 행동은 테스트하기가 훨씬 용이합니다. 각 행동을 독립적으로 테스트할 수 있으며, 오리 클래스에 행동을 주입하여 올바르게 작동하는지 검증할 수 있습니다. 이렇게 함으로써 코드의 유연성을 유지하면서도, 개별 행동들이 제대로 동작하는지를 쉽게 확인할 수 있습니다.

<details>

<summary><code>MallardDuck</code>과 <code>RubberDuck</code> 클래스 테스트 코드</summary>

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

이 테스트 코드는 오리의 다양한 행동을 확인하는 데 사용됩니다. `MallardDuck`은 초기 설정에서 날 수 있지만, 런타임에 `FlyNoWay`로 변경하여 더 이상 날지 못하게 만들 수 있습니다. 마찬가지로, `RubberDuck`은 날 수 없는 설정과 삑삑 소리를 내는 설정을 가진 채로 테스트됩니다.

</details>



## **동적으로 행동 지정하기 - 런타임에서의 행동 변경**

전략 패턴의 또 다른 강점은 런타임에서 행동을 동적으로 변경할 수 있다는 점입니다. 이는 프로그램 실행 중에도 오리의 행동을 자유롭게 바꿀 수 있어 유연한 설계가 가능합니다. 이를 통해 코드의 재사용성과 유연성을 크게 높일 수 있습니다.

<details>

<summary>런타임에서의 행동 변경</summary>

```java
Duck model = new ModelDuck();
model.performFly(); // I can't fly 출력

// 런타임에서 행동 변경
model.setFlyBehavior(new FlyRocketPowered());
model.performFly(); // I'm flying with a rocket! 출력
```

위의 코드에서 `ModelDuck`은 초기 설정에서 날 수 없는(`FlyNoWay`) 상태입니다. 그러나 런타임에 `FlyRocketPowered`로 행동을 변경하면 로켓 추진으로 날 수 있게 됩니다. 이렇게 행동을 동적으로 지정함으로써 다양한 시나리오에 대응할 수 있습니다.

</details>



## **캡슐화된 행동 살펴보기 - 내부 구현의 변화**

캡슐화된 행동을 사용하면, 행동의 내부 구현을 변경해도 외부 클래스에는 영향을 미치지 않습니다. 예를 들어, `FlyBehavior`의 `FlyWithWings` 구현이 변경되어도 `Duck` 클래스나 그 하위 클래스는 전혀 수정할 필요가 없습니다. 이로 인해 코드의 유지보수가 훨씬 쉬워집니다.

<details>

<summary>FlyWithWings 클래스</summary>

```java
class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I'm soaring with wings!"); // 변경된 출력 메시지
    }
}
```

위와 같이 `FlyWithWings`의 구현이 변경되었지만, 오리 클래스는 이 변화를 전혀 알 필요가 없습니다. 이는 코드의 유지보수를 간편하게 하고, 다양한 요구사항에 따라 행동을 변경할 수 있는 유연성을 제공합니다. 또한, 이러한 변경이 시스템 전체에 영향을 미치지 않도록 하여 코드의 안정성을 유지할 수 있습니다.

</details>



## **두 클래스를 합치는 방법 - 합성 대 상속의 선택**

전략 패턴에서는 \*\*합성(Composition)\*\*을 사용하여 오리의 행동을 결합합니다. 합성은 행동을 독립적인 클래스들로 분리하여, 이러한 클래스들을 조합하여 다양한 기능을 구현할 수 있게 합니다. 이는 \*\*상속(Inheritance)\*\*과 달리 클래스 간의 강한 결합도를 피할 수 있습니다.

상속은 클래스 간의 강한 결합을 만들어 내며, 이는 재사용성과 유연성을 제한할 수 있습니다. 반면, 합성은 객체를 조합하여 원하는 행동을 구성할 수 있게 하므로, 코드의 유연성과 재사용성을 극대화할 수 있습니다.

<details>

<summary>예시 코드</summary>

```java
// 상속 대신 합성을 사용하여 유연성 확보
Duck customDuck = new MallardDuck();
customDuck.setFlyBehavior(new FlyNoWay()); // 행동 변경
customDuck.setQuackBehavior(new MuteQuack()); // 행동 변경
customDuck.performFly(); // I can't fly 출력
customDuck.performQuack(); // << Silence >> 출력
```

위의 예제에서 `MallardDuck`은 초기 설정을 변경하여 날 수 없고(`FlyNoWay`), 소리를 내지 않는(`MuteQuack`) 행동을 가집니다. 이렇게 합성을 통해 필요에 따라 오리의 행동을 동적으로 변경할 수 있는 유연성을 제공하며, 코드의 재사용성과 확장성을 극대화할 수 있습니다.

</details>



## 전략패턴을 적용하고의 새로운 행동 추가

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





















