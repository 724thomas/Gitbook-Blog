---
description: Ch3. 데코레이터 패턴
---

# Ch3. Decorator Pattern

### 데코레이터 패턴의 개념

데코레이터 패턴은 객체의 기능을 동적으로 확장할 수 있게 해주는 구조적 디자인 패턴이다. 이 패턴을 통해 기존 코드를 변경하지 않고도 객체에 새로운 행동을 추가할 수 있다. 데코레이터 패턴은 객체의 구성(Composition)을 활용하여 상속의 단점을 보완한다. 이 패턴은 특히 객체 지향 설계 원칙 중 하나인 OCP(개방-폐쇄 원칙, Open/Closed Principle)를 준수하는 데 유리하다.

### OCP (Open/Closed Principle)와 데코레이터 패턴

OCP는 객체 지향 설계 원칙 중 하나로, "확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다"는 원칙이다. 즉, 소프트웨어 엔티티(클래스, 모듈, 함수 등)는 확장 가능해야 하지만, 기존 코드의 변경 없이도 기능을 추가할 수 있어야 한다.

데코레이터 패턴은 OCP를 충실히 따르는 패턴이다. 데코레이터를 통해 기존 코드를 변경하지 않고도 객체의 기능을 확장할 수 있기 때문이다. 새로운 기능이 필요할 때마다 기존 클래스를 수정하지 않고, 새로운 데코레이터 클래스를 추가하여 기능을 확장할 수 있다. 이로 인해 코드의 재사용성과 유지보수성이 높아진다.

### 예제: 커피 주문 시스템

커피 주문 시스템을 예로 들어 데코레이터 패턴과 OCP를 설명하겠다. 이 시스템에서는 다양한 종류의 커피와 다양한 첨가물을 조합하여 주문할 수 있다. 각각의 커피와 첨가물을 클래스로 표현하고, 데코레이터 패턴을 이용해 첨가물을 동적으로 추가하는 방식으로 구현한다.

#### 1. 기본 구성 요소 정의

먼저 커피와 첨가물을 표현하는 기본 인터페이스와 클래스를 정의한다.

**Beverage.java**

```java
public abstract class Beverage {
    String description = "Unknown Beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();
}
```

`Beverage` 클래스는 커피의 기본 인터페이스 역할을 하며, 모든 커피와 첨가물 클래스는 이 클래스를 확장한다. `getDescription` 메서드는 커피의 설명을 반환하고, `cost` 메서드는 커피의 가격을 반환한다.

**다양한 커피 종류 구현**

```java
public class Espresso extends Beverage {
    public Espresso() {
        description = "Espresso";
    }

    public double cost() {
        return 1.99;
    }
}

public class HouseBlend extends Beverage {
    public HouseBlend() {
        description = "House Blend Coffee";
    }

    public double cost() {
        return 0.89;
    }
}
```

`Espresso`와 `HouseBlend` 클래스는 `Beverage` 클래스를 확장하여 각각 에스프레소와 하우스 블렌드 커피의 가격과 설명을 정의한다.

<figure><img src="/broken/files/RStKJosOlwJ0VsgZainx" alt=""><figcaption></figcaption></figure>

#### 2. 데코레이터 클래스 정의

첨가물은 데코레이터 패턴을 통해 추가할 수 있다. 데코레이터 클래스는 `Beverage` 클래스를 확장한다.

**CondimentDecorator.java**

```java
public abstract class CondimentDecorator extends Beverage {
    public abstract String getDescription();
}
```

`CondimentDecorator` 클래스는 `Beverage` 클래스를 확장하여 첨가물의 기본 기능을 정의한다. 모든 첨가물 클래스는 이 클래스를 확장하여 구현된다.

**다양한 첨가물 구현**

```java
public class Mocha extends CondimentDecorator {
    Beverage beverage;

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }

    public double cost() {
        return 0.20 + beverage.cost();
    }
}

public class Whip extends CondimentDecorator {
    Beverage beverage;

    public Whip(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Whip";
    }

    public double cost() {
        return 0.30 + beverage.cost();
    }
}
```

<figure><img src="/broken/files/16SubuUR7DJ2OUrCrHFI" alt=""><figcaption></figcaption></figure>

`Mocha`와 `Whip` 클래스는 `CondimentDecorator` 클래스를 확장하여 각각 모카와 휘핑 크림 첨가물의 가격과 설명을 정의한다. 데코레이터 패턴을 통해 첨가물은 동적으로 커피에 추가될 수 있다.

#### 3. 데코레이터 패턴 사용 예제

데코레이터 패턴을 활용해 커피 주문을 처리하는 예제를 살펴보자.

```java
public class StarbuzzCoffee {
    public static void main(String args[]) {
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription() + " $" + beverage.cost());

        Beverage beverage2 = new HouseBlend();
        beverage2 = new Mocha(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription() + " $" + beverage2.cost());
    }
}
```

<figure><img src="/broken/files/vi25fBCGGzKUDUGJL6wB" alt=""><figcaption></figcaption></figure>

이 예제에서는 `Espresso`와 `HouseBlend` 커피에 `Mocha`와 `Whip` 첨가물을 추가하는 과정을 보여준다. 각 첨가물은 데코레이터 패턴을 통해 기존 객체에 동적으로 추가된다. 예를 들어, `HouseBlend` 커피에 `Mocha`와 `Whip` 첨가물을 추가하면, 최종 출력은 "House Blend Coffee, Mocha, Whip $1.39"가 된다.

#### 4. OCP와 데코레이터 패턴의 관계

위의 예제에서 알 수 있듯이, 데코레이터 패턴을 통해 새로운 첨가물을 추가할 때 기존 커피 클래스(`Espresso`, `HouseBlend`)를 전혀 수정하지 않았다. 이는 OCP 원칙을 잘 준수한 사례이다. 새로운 기능(첨가물)을 추가할 때 기존 코드를 변경하지 않고도 기능을 확장할 수 있다.

### 데코레이터 패턴의 장점

1. **유연성 증가**: 객체의 기능을 동적으로 추가할 수 있어 유연한 설계가 가능하다.
2. **단일 책임 원칙 준수**: 클래스는 본연의 역할만 수행하고, 추가 기능은 데코레이터를 통해 처리한다.
3. **조합의 다양성**: 다양한 데코레이터를 조합하여 새로운 기능을 쉽게 추가할 수 있다.
4. **OCP 준수**: 기존 코드를 수정하지 않고 새로운 기능을 추가할 수 있다.

### 데코레이터 패턴의 단점

1. **많은 작은 객체 생성**: 데코레이터를 많이 사용하면 객체의 수가 증가하여 복잡도가 높아질 수 있다.
2. **디버깅 어려움**: 여러 레이어의 데코레이터가 적용되면 디버깅이 어려울 수 있다.
3. **런타임 비용 증가**: 데코레이터를 여러 개 적용하면, 각 데코레이터의 `cost` 메서드를 호출할 때마다 비용 계산이 중첩되어 성능에 영향을 줄 수 있다.

### 데코레이터 패턴의 실제 사용 사례

#### 1. Java I/O 라이브러리

<figure><img src="/broken/files/Oti5m0iMiPSRRpdyDnjF" alt=""><figcaption></figcaption></figure>

Java의 I/O 라이브러리는 데코레이터 패턴의 훌륭한 예시이다. `InputStream` 클래스와 다양한 데코레이터 클래스(`BufferedInputStream`, `DataInputStream` 등)는 파일 입출력 기능을 동적으로 확장하는 데 사용된다.

```java
InputStream in = new FileInputStream("test.txt");
InputStream bin = new BufferedInputStream(in);
InputStream din = new DataInputStream(bin);
```

위의 예제에서 `FileInputStream` 객체는 `BufferedInputStream`과 `DataInputStream` 데코레이터를 통해 확장된다. 이를 통해 파일 입출력 성능을 높이고 다양한 데이터를 처리할 수 있게 된다.

#### 2. GUI 라이브러리

GUI 컴포넌트 라이브러리에서도 데코레이터 패턴이 자주 사용된다. 예를 들어, `JComponent` 클래스는 다양한 데코레이터를 통해 기능이 확장될 수 있다. 스크롤 패널, 테두리 등은 데코레이터 패턴을 사용하여 구현된다.

```java
JComponent component = new JTextArea();
component = new JScrollPane(component);
component = new BorderFactory.createTitledBorder("Title", component);
```

위의 예제에서 `JTextArea` 객체는 `JScrollPane`과 `TitledBorder` 데코레이터를 통해 확장된다. 이를 통해 텍스트 영역에 스크롤 기능과 테두리를 추가할 수 있다.

### 결론

데코레이터 패턴은 객체의 기능을 확장하는 데 매우 유용한 디자인 패턴이다. 상속 대신 구성을 사용하여 유연한 설계를 가능하게 하고, 단일 책임 원칙을 준수할 수 있게 한다. 특히 OCP 원칙을 준수하여 기존 코드를 변경하지 않고도 기능을 확장할 수 있다는 점에서 큰 장점을 가진다. 그러나 과도한 사용은 코드의 복잡도를 증가시킬 수 있으므로 적절하게 활용하는 것이 중요하다.
