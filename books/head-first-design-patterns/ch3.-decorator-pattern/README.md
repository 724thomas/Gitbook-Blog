---
description: 데코레이터 패턴
---

# Ch3. Decorator Pattern

**초대형 커피 전문점, 스타벅스**

스타벅스는 커피 주문 시 다양한 커스텀 옵션을 제공하는 것으로 유명합니다. 고객은 원하는 커피 베이스에 우유, 시럽, 휘핑크림 등을 추가하여 자신만의 특별한 음료를 만들 수 있습니다. 이처럼 기본 음료에 다양한 옵션을 자유롭게 추가하는 방식은 소프트웨어 디자인 패턴 중 '데코레이터 패턴'과 유사합니다. 데코레이터 패턴은 기본 객체에 새로운 기능을 동적으로 추가할 수 있도록 설계된 구조로, 객체의 유연한 확장을 가능하게 합니다.

**OCP 살펴보기**

데코레이터 패턴을 이해하기 위해서는 객체지향 설계의 원칙 중 하나인 'OCP(개방-폐쇄 원칙)'를 살펴볼 필요가 있습니다. OCP는 소프트웨어 구성 요소가 확장에는 열려 있어야 하지만, 변경에는 닫혀 있어야 한다는 원칙입니다. 즉, 새로운 기능을 추가할 때 기존 코드에 수정을 최소화해야 한다는 것입니다. 데코레이터 패턴은 이 OCP 원칙을 잘 준수할 수 있도록 도와줍니다. 예를 들어, 커피 주문 시스템에서 새로운 옵션을 추가할 때, 데코레이터 패턴을 사용하면 기존의 커피 클래스는 그대로 두고, 새로운 데코레이터 클래스를 추가함으로써 기능을 확장할 수 있습니다.

**데코레이터 패턴 살펴보기**

데코레이터 패턴은 기존 객체의 인터페이스를 그대로 유지하면서, 객체에 새로운 기능을 추가하는 디자인 패턴입니다. 이 패턴은 상속 대신 조합을 사용하여 기능을 확장합니다. 상속을 사용하면 클래스의 개수가 기하급수적으로 늘어나고, 코드의 복잡성이 증가할 수 있습니다. 하지만 데코레이터 패턴을 사용하면 필요한 기능만 조합하여 사용할 수 있어, 코드의 유연성과 재사용성이 크게 향상됩니다.

**주문 시스템에 데코레이터 패턴 적용하기**

커피 전문점의 주문 시스템을 예로 들어 데코레이터 패턴을 어떻게 적용할 수 있는지 알아보겠습니다. 우선, 기본 커피 클래스를 정의하고, 이 클래스에 데코레이터를 사용해 다양한 옵션(예: 우유, 모카, 휘핑크림 등)을 추가할 수 있도록 설계할 수 있습니다. 이렇게 하면 각 옵션이 독립된 클래스로 구현되므로, 새로운 옵션을 추가할 때 기존 클래스를 수정할 필요가 없습니다. 각 데코레이터는 기본 커피 클래스와 동일한 인터페이스를 구현하여, 기본 객체를 감싸면서도 새로운 기능을 추가할 수 있습니다.

<details>

<summary>abstract class Beverage<br>class Espresso extends Beverage<br>abstract class CondimentDecorator extends Beverage</summary>

```java
abstract class Beverage {
    String description = "Unknown Beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();
}

class Espresso extends Beverage {
    public Espresso() {
        description = "Espresso";
    }

    public double cost() {
        return 1.99;
    }
}

abstract class CondimentDecorator extends Beverage {
    public abstract String getDescription();
}

class Mocha extends CondimentDecorator {
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
```

이 예시에서는 `Espresso` 클래스가 기본 커피 객체로, `Mocha` 클래스가 데코레이터로 사용되었습니다. `Mocha` 클래스는 `CondimentDecorator`를 상속받아, `Espresso` 객체에 모카를 추가하는 역할을 합니다.

</details>



**데코레이터 패턴의 정의**

데코레이터 패턴은 런타임에 객체의 행동을 동적으로 확장할 수 있도록 하는 구조적 패턴입니다. 상속을 통해 기능을 확장하는 대신, 데코레이터 패턴은 객체를 감싸서 추가적인 기능을 부여합니다. 이 패턴은 기존 객체의 코드 변경 없이도 기능을 확장할 수 있어, 코드의 유연성과 확장성을 크게 향상시킵니다.



## 데코레이터 패턴의 실습 및 코드 구현

### **Beverage 클래스 장식하기**

앞서 설명한 기본 커피 클래스와 데코레이터 패턴을 실제 코드로 구현해 보겠습니다. `Beverage` 클래스를 다양한 옵션으로 장식하며 데코레이터 패턴의 유용성을 확인해 보겠습니다. `Beverage` 클래스는 음료의 기본적인 기능과 속성을 정의하고, 각 음료 옵션은 별도의 데코레이터 클래스로 구현됩니다.

기본 `Beverage` 클래스와 `Espresso` 음료를 예제로 들자면, `Beverage`는 모든 음료의 공통 속성과 메서드를 정의하고, `Espresso`는 이를 상속받아 구체적인 음료를 나타냅니다. 데코레이터 클래스는 `Beverage`를 상속받아 특정 옵션을 추가하는 역할을 합니다.

<details>

<summary>class HouseBlend extends Beverage</summary>

```java
class HouseBlend extends Beverage {
    public HouseBlend() {
        description = "House Blend Coffee";
    }

    public double cost() {
        return 0.89;
    }
}

class Soy extends CondimentDecorator {
    Beverage beverage;

    public Soy(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Soy";
    }

    public double cost() {
        return 0.15 + beverage.cost();
    }
}

class Whip extends CondimentDecorator {
    Beverage beverage;

    public Whip(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Whip";
    }

    public double cost() {
        return 0.10 + beverage.cost();
    }
}
```

위 코드에서 `HouseBlend`는 또 다른 기본 음료 클래스로, 소이와 휘핑크림을 각각 추가할 수 있는 `Soy`, `Whip` 데코레이터 클래스와 결합하여 음료를 구성할 수 있습니다.

</details>

### **데코레이터 패턴 적용 연습**

데코레이터 패턴을 통해 새로운 음료 옵션을 추가할 때는, 기존의 클래스를 수정하지 않고도 기능을 확장할 수 있습니다. 예를 들어, 새로운 시럽 옵션을 추가한다고 가정해 보겠습니다. 다음과 같이 `Syrup` 데코레이터 클래스를 추가하기만 하면 됩니다.

<details>

<summary>class Syrup extends CondimentDecorator</summary>

```java
class Syrup extends CondimentDecorator {
    Beverage beverage;

    public Syrup(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Syrup";
    }

    public double cost() {
        return 0.25 + beverage.cost();
    }
}
```

이처럼 새로운 옵션을 추가할 때는, 간단히 새로운 데코레이터 클래스를 정의하여 기존 `Beverage` 객체에 추가하면 됩니다.&#x20;

</details>



### **커피 주문 시스템 코드 만들기**

이제 앞에서 정의한 `Beverage` 클래스와 데코레이터 클래스를 활용해, 실제 커피 주문 시스템을 구현해 보겠습니다. 주문 시스템은 다양한 음료와 옵션을 조합하여 최종 음료의 가격과 설명을 출력할 수 있어야 합니다.

<details>

<summary>class CoffeeShop</summary>

```java
public class CoffeeShop {
    public static void main(String args[]) {
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription() + " $" + beverage.cost());

        Beverage beverage2 = new HouseBlend();
        beverage2 = new Soy(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription() + " $" + beverage2.cost());
    }
}
```

위 코드에서 `CoffeeShop` 클래스의 `main` 메서드를 통해 실제 음료를 주문하고, 선택한 옵션들을 추가하여 최종 가격을 계산할 수 있습니다. 예를 들어, `HouseBlend`에 소이와 휘핑크림을 추가한 음료의 가격과 설명이 출력됩니다.

</details>



## 데코레이터 패턴 적용 사례: 자바 I/O 시스템

### **데코레이터가 적용된 예: 자바 I/O**

자바의 I/O(Input/Output) 시스템은 데코레이터 패턴의 대표적인 사례입니다. 자바의 `java.io` 패키지는 다양한 스트림 클래스를 제공하며, 이들 스트림은 데코레이터 패턴을 사용하여 기능을 확장합니다. 예를 들어, `FileInputStream`은 기본적인 파일 읽기 기능을 제공하며, 이를 데코레이터로 감싸면 추가적인 기능(예: 버퍼링, 데이터 필터링 등)을 추가할 수 있습니다.

```java
FileInputStream fileInput = new FileInputStream("test.txt");
BufferedInputStream bufferedInput = new BufferedInputStream(fileInput);
```

위 코드에서 `BufferedInputStream`은 `FileInputStream`을 감싸서 버퍼링 기능을 추가합니다. 이는 데코레이터 패턴의 전형적인 사용 예입니다. 스트림에 새로운 기능을 추가할 때, 기존 클래스들을 수정할 필요 없이 새로운 데코레이터 클래스를 사용하여 쉽게 기능을 확장할 수 있습니다.

### **java.io 클래스와 데코레이터 패턴**

자바의 `java.io` 패키지에 포함된 클래스들은 데코레이터 패턴을 완벽히 구현하고 있습니다. 이 패턴을 통해 파일 읽기, 쓰기 등의 기본적인 기능에 다양한 부가 기능을 동적으로 추가할 수 있습니다. 예를 들어, `DataInputStream`을 사용하면 기본 입력 스트림에 데이터 변환 기능을 추가할 수 있습니다.

<details>

<summary>DataInputStream</summary>

<pre class="language-java"><code class="lang-java">DataInputStream dataInput = new DataInputStream(
    new BufferedInputStream(
<strong>        new FileInputStream("data.bin")
</strong><strong>    )
</strong><strong>);
</strong></code></pre>

이 코드에서는 파일을 읽을 때, 버퍼링 기능을 추가한 후, `DataInputStream`을 통해 기본 데이터를 자바 기본형으로 변환할 수 있습니다. 이처럼 데코레이터 패턴은 기능을 필요한 만큼 쌓아 올리는 방식을 가능하게 합니다.

</details>



### **자바 I/O 데코레이터 만들기**

기존의 자바 I/O 클래스들을 활용해 데코레이터를 만들어 보겠습니다. 예를 들어, 로그를 남기는 기능을 추가하고 싶다면, 이를 위해 `FilterInputStream`을 상속받아 새로운 데코레이터 클래스를 작성할 수 있습니다.

<details>

<summary>class LoggingInputStream extends FilterInputStream</summary>

```java
class LoggingInputStream extends FilterInputStream {
    public LoggingInputStream(InputStream in) {
        super(in);
    }

    @Override
    public int read() throws IOException {
        int data = super.read();
        System.out.println("Read data: " + data);
        return data;
    }
}
```

`LoggingInputStream` 클래스는 `FilterInputStream`을 상속받아, 데이터가 읽힐 때마다 로그를 출력하는 기능을 추가했습니다. 이렇게 새로운 기능을 추가하려면, 기본 스트림 클래스에 직접적으로 수정할 필요 없이, 데코레이터 패턴을 활용해 기능을 확장하면 됩니다.

</details>



### **새로 만든 자바 I/O 데코레이터 테스트**

앞서 만든 `LoggingInputStream` 데코레이터를 실제 코드에 적용하여 테스트해 보겠습니다.

<details>

<summary>class TestLoggingInputStream</summary>

```java
public class TestLoggingInputStream {
    public static void main(String[] args) throws IOException {
        InputStream input = new LoggingInputStream(new BufferedInputStream(new FileInputStream("test.txt")));
        while (input.read() != -1) {
            // 데이터를 읽으면서 로그가 출력됨
        }
        input.close();
    }
}
```

위 코드에서 파일을 읽는 동안 각 바이트가 콘솔에 로그로 출력됩니다. `LoggingInputStream` 데코레이터를 추가한 덕분에 파일 읽기 기능에 로그 출력 기능이 자연스럽게 통합되었습니다.

</details>

