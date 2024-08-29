---
description: 템플릿 메서드 패턴
---

# Ch8. Template Method Pattern

개발에서 알고리즘의 구조와 구현 세부 사항을 분리하는 것은 중요한 과제입니다. 이런 문제를 해결하기 위해 자주 사용되는 디자인 패턴 중 하나가 템플릿 메서드 패턴입니다. 템플릿 메서드 패턴은 알고리즘의 기본 골격을 정의하고, 하위 클래스에서 특정 단계들을 재정의하여 다양한 방식으로 알고리즘을 실행할 수 있도록 합니다. 이 패턴을 사용하면 코드의 재사용성을 높이고, 중복을 최소화하며, 코드 유지보수를 쉽게 할 수 있습니다.

여기에 후크 메서드를 더함으로써 알고리즘의 특정 부분을 선택적으로 수정할 수 있는 유연성을 제공합니다. 후크를 활용하면 알고리즘의 주요 흐름을 유지하면서도, 하위 클래스에서 필요에 따라 동작을 쉽게 변경할 수 있습니다.

## 정의

템플릿 메서드 패턴은 상위 클래스에서 알고리즘의 기본 구조를 정의하고, 하위 클래스에서 그 구조의 세부 단계를 구현하는 방법입니다. 즉, 상위 클래스에서는 전체 알고리즘의 흐름을 결정하고, 하위 클래스에서는 그 흐름 내의 일부 단계를 구체적으로 구현하는 방식입니다.

* **템플릿 메서드**: 알고리즘의 골격을 정의하는 메서드입니다. 이 메서드는 일반적으로 상위 클래스에서 정의되며, 알고리즘의 각 단계를 호출합니다.
* **추상 메서드**: 하위 클래스에서 반드시 구현해야 하는 메서드로, 알고리즘의 특정 단계를 구현합니다.
* **구현된 메서드**: 상위 클래스에서 구현된 메서드로, 하위 클래스에서 재사용하거나 필요한 경우 오버라이드할 수 있습니다.



### 예시

<details>

<summary>상위 클래스</summary>

```java
public abstract class ReportGenerator {

    // 템플릿 메서드
    public final void generateReport() {
        fetchData();
        processData();
        formatReport();
        printReport();
    }

    // 각 단계의 메서드들을 정의 (일부는 추상 메서드로 정의)
    protected abstract void fetchData();

    protected abstract void processData();

    protected abstract void formatReport();

    protected void printReport() {
        System.out.println("Report is ready to be printed.");
    }
}
```

위 코드에서 `generateReport` 메서드는 템플릿 메서드로, 보고서 생성의 전체 과정을 정의합니다. `fetchData`, `processData`, `formatReport` 메서드는 각각의 단계에 해당하며, `fetchData`와 `processData`는 각 하위 클래스에서 구체적으로 구현됩니다.

</details>

<details>

<summary>하위 클래스</summary>

```java
public class SalesReportGenerator extends ReportGenerator {

    @Override
    protected void fetchData() {
        System.out.println("Fetching sales data from the database...");
    }

    @Override
    protected void processData() {
        System.out.println("Processing sales data...");
    }

    @Override
    protected void formatReport() {
        System.out.println("Formatting sales report...");
    }
}

public class InventoryReportGenerator extends ReportGenerator {

    @Override
    protected void fetchData() {
        System.out.println("Fetching inventory data from the warehouse...");
    }

    @Override
    protected void processData() {
        System.out.println("Processing inventory data...");
    }

    @Override
    protected void formatReport() {
        System.out.println("Formatting inventory report...");
    }
}
```

여기서 `SalesReportGenerator`와 `InventoryReportGenerator`는 각각 `ReportGenerator`의 하위 클래스이며, 각각의 보고서 유형에 맞게 데이터를 가져오고 처리하는 방식을 구현합니다.

</details>

<details>

<summary>클라이언트 코드</summary>

```java
public class ReportService {

    public void createReport(String type) {
        ReportGenerator reportGenerator;
        
        if ("sales".equalsIgnoreCase(type)) {
            reportGenerator = new SalesReportGenerator();
        } else if ("inventory".equalsIgnoreCase(type)) {
            reportGenerator = new InventoryReportGenerator();
        } else {
            throw new IllegalArgumentException("Unknown report type: " + type);
        }
        
        reportGenerator.generateReport();
    }
}
```

위 코드에서 `ReportService`는 클라이언트 역할을 하며, 사용자가 지정한 보고서 유형에 따라 적절한 `ReportGenerator` 하위 클래스를 선택하여 보고서를 생성합니다.

</details>



### 장점

* **코드 재사용성**: 알고리즘의 공통적인 부분을 상위 클래스에서 정의하여 코드 중복을 줄입니다.
* **유지보수성**: 알고리즘의 골격을 변경해야 할 때, 상위 클래스만 수정하면 되므로 유지보수가 용이합니다.
* **확장성**: 하위 클래스에서 세부 단계를 재정의하여 다양한 방식으로 알고리즘을 확장할 수 있습니다.

### 단점

* **구현 복잡성**: 상위 클래스와 하위 클래스 간의 의존성이 높아지면 코드의 복잡성이 증가할 수 있습니다.
* **하위 클래스 남용**: 상위 클래스에서 정의된 템플릿 메서드를 남용하여 하위 클래스가 불필요하게 많아질 수 있습니다.



## 후크

템플릿 메서드 패턴에서 중요한 개념 중 하나가 "후크(Hook)"입니다. 후크는 하위 클래스에서 선택적으로 오버라이드할 수 있는 메서드로, 템플릿 메서드에서 기본적으로는 아무런 동작도 하지 않지만, 필요에 따라 하위 클래스에서 구현하여 특정 동작을 추가하거나 수정할 수 있게 해줍니다. 후크를 통해 상위 클래스는 알고리즘의 주요 구조를 유지하면서도, 하위 클래스에 추가적인 유연성을 제공할 수 있습니다.

템플릿 메서드 패턴에서 후크는 알고리즘의 특정 단계에서 하위 클래스가 개입할 수 있는 기회를 제공합니다. 후크 메서드는 일반적으로 비어 있거나 기본적인 구현을 가지고 있으며, 하위 클래스에서 이 메서드를 오버라이드하여 원하는 동작을 추가할 수 있습니다.

* **선택적 오버라이딩**: 후크는 하위 클래스에서 반드시 오버라이드할 필요는 없습니다. 필요할 때만 오버라이드하여 동작을 변경할 수 있습니다.
* **알고리즘 흐름의 유연성**: 후크를 통해 알고리즘의 기본 흐름을 유지하면서도, 세부 동작을 하위 클래스에서 조정할 수 있습니다.

<details>

<summary>상위 클래스</summary>

```java
public abstract class ReportGenerator {

    // 템플릿 메서드
    public final void generateReport() {
        fetchData();
        processData();
        if (shouldFormatReport()) {  // 후크 메서드 사용
            formatReport();
        }
        printReport();
    }

    protected abstract void fetchData();

    protected abstract void processData();

    protected abstract void formatReport();

    protected void printReport() {
        System.out.println("Report is ready to be printed.");
    }

    // 후크 메서드: 하위 클래스에서 오버라이드 가능
    protected boolean shouldFormatReport() {
        return true;  // 기본적으로 보고서는 형식을 갖추도록 설정
    }
}
```

위 코드에서 `shouldFormatReport` 메서드가 후크 역할을 합니다. 이 메서드는 기본적으로 `true`를 반환하지만, 하위 클래스에서 이 메서드를 오버라이드하여 보고서를 형식화할지 여부를 결정할 수 있습니다.

</details>

<details>

<summary>후크 메서드를 활용하는 하위 클래스</summary>

```java
public class QuickReportGenerator extends ReportGenerator {

    @Override
    protected void fetchData() {
        System.out.println("Fetching quick data...");
    }

    @Override
    protected void processData() {
        System.out.println("Processing quick data...");
    }

    @Override
    protected void formatReport() {
        System.out.println("Quick report does not require formatting.");
    }

    @Override
    protected boolean shouldFormatReport() {
        return false;  // 이 보고서에서는 형식을 적용하지 않도록 설정
    }
}
```

위 코드에서 `QuickReportGenerator`는 `shouldFormatReport` 후크 메서드를 오버라이드하여 보고서 형식을 건너뛰도록 설정했습니다. 이렇게 하여 기본 알고리즘의 흐름을 유지하면서도, 특정 단계에서는 동작을 변경할 수 있습니다.

</details>

### 장점

* **유연성 향상**: 하위 클래스가 알고리즘의 흐름을 선택적으로 변경할 수 있어 유연성이 증가합니다.
* **코드 재사용성 증대**: 공통적인 알고리즘 구조를 유지하면서도, 필요에 따라 세부 동작을 하위 클래스에서 수정할 수 있습니다.



## 할리우드 원칙

할리우드 원칙은 "우리가 전화할게요, 전화하지 마세요(Don't call us, we'll call you)"라는 문구로 요약할 수 있습니다. 이 원칙은 상위 클래스가 하위 클래스의 메서드를 호출하는 방식으로, 하위 클래스가 상위 클래스의 흐름을 제어하지 않고 상위 클래스가 필요한 시점에 하위 클래스의 메서드를 호출하도록 하는 설계 원칙입니다.

이 원칙을 템플릿 메서드 패턴에 적용하면, 상위 클래스에서 알고리즘의 기본 흐름을 정의하고, 그 흐름 내에서 필요한 시점에 하위 클래스의 메서드를 호출하게 됩니다. 즉, 상위 클래스가 전체 알고리즘을 주도하며, 하위 클래스는 그 흐름에 필요한 부분만 제공하는 형태입니다.

### 템플릿 메서드와 할리우드 원칙의 관계

템플릿 메서드 패턴은 할리우드 원칙의 구현을 잘 보여주는 예입니다. 템플릿 메서드 패턴에서 상위 클래스는 알고리즘의 흐름을 정의하고, 하위 클래스에서 구현해야 할 메서드(추상 메서드 또는 후크 메서드)를 필요할 때 호출합니다. 하위 클래스는 상위 클래스가 호출할 메서드를 제공하지만, 상위 클래스의 흐름을 직접적으로 제어하지는 않습니다.

### 장점

* **제어 역전(Inversion of Control, IoC)**: 하위 클래스가 알고리즘의 전체 흐름을 제어하지 않고, 상위 클래스에서 전체 흐름을 주도합니다. 이는 코드의 구조를 명확하게 하고, 책임을 분산시키며, 유지보수를 쉽게 만들어 줍니다.
* **구현 세부사항의 은닉**: 상위 클래스는 알고리즘의 구조를 알고 있지만, 구체적인 구현 세부사항은 하위 클래스에 맡깁니다. 이렇게 하면 상위 클래스는 알고리즘의 전반적인 흐름을 유지하면서도, 세부 구현을 다양하게 변경할 수 있습니다.



## 템플릿 메서드 패턴과 전략 패턴의 차이

#### 템플릿 메서드 패턴

**템플릿 메서드 패턴**은 상위 클래스에서 알고리즘의 전체 골격을 정의하고, 그 알고리즘의 세부 단계들은 하위 클래스에서 구현하는 패턴입니다. 이 패턴은 알고리즘의 기본적인 구조는 동일하게 유지하면서, 특정 단계들의 구현을 다르게 하고자 할 때 사용됩니다.

* **구현 방법**: 상위 클래스에서 템플릿 메서드를 정의하고, 그 안에서 호출될 추상 메서드를 하위 클래스가 구현합니다.
* **사용 목적**: 알고리즘의 구조는 공통으로 유지하면서, 각 단계의 구체적인 구현을 다르게 하고자 할 때 사용합니다.

#### 전략 패턴

**전략 패턴**은 실행 시점에 사용할 알고리즘을 동적으로 선택할 수 있도록 설계된 패턴입니다. 전략 패턴은 특정 작업을 수행하는 여러 알고리즘을 캡슐화하여, 클라이언트가 필요에 따라 이들 중 하나를 선택하여 사용할 수 있게 합니다.

* **구현 방법**: 알고리즘을 인터페이스나 추상 클래스의 형태로 정의하고, 여러 개의 구체적인 알고리즘 클래스를 구현합니다. 클라이언트는 특정 알고리즘을 실행 시점에 선택하여 사용할 수 있습니다.
* **사용 목적**: 다양한 알고리즘을 캡슐화하고, 실행 시점에 이를 동적으로 선택할 수 있게 하여 알고리즘을 쉽게 교체하거나 확장할 수 있도록 합니다.

### 주요 차이점

**알고리즘의 구조와 유연성**

* **템플릿 메서드 패턴**: 상위 클래스에서 알고리즘의 전체 구조를 고정하고, 그 구조 내에서 일부 단계를 하위 클래스가 구현하도록 합니다. 즉, 알고리즘의 흐름 자체는 상위 클래스에서 결정됩니다.
* **전략 패턴**: 알고리즘 자체를 하나의 인터페이스로 정의하고, 여러 구현체(구체적인 전략 클래스)를 만들어 클라이언트가 상황에 따라 적절한 알고리즘을 선택하도록 합니다. 알고리즘의 흐름은 인터페이스에 정의된 메서드에 따라 달라집니다.

**상속과 구성**

* **템플릿 메서드 패턴**: 상속을 통해 알고리즘을 구현합니다. 상위 클래스에서 템플릿 메서드를 정의하고, 하위 클래스에서 그 메서드의 일부를 구현하는 구조입니다. 이 방식은 상속 관계에 의존하므로, 알고리즘의 유연성이 상속 계층에 제한될 수 있습니다.
* **전략 패턴**: 구성을 통해 알고리즘을 구현합니다. 클라이언트 객체는 전략 객체를 포함(구성)하며, 필요에 따라 이 전략을 변경할 수 있습니다. 즉, 상속이 아닌 구성을 통해 알고리즘을 선택적으로 변경할 수 있어 유연성이 높습니다.

**실행 시점의 알고리즘 선택**

* **템플릿 메서드 패턴**: 상위 클래스에서 알고리즘의 골격이 고정되어 있어, 실행 시점에는 이미 알고리즘의 흐름이 정해져 있습니다. 알고리즘의 특정 단계만 하위 클래스에서 다르게 구현할 수 있습니다.
* **전략 패턴**: 클라이언트는 실행 시점에 사용할 알고리즘(전략)을 동적으로 선택할 수 있습니다. 다양한 알고리즘을 쉽게 교체하거나 새로운 알고리즘을 추가할 수 있습니다.



### 사용 시기

* **템플릿 메서드 패턴**: 알고리즘의 큰 틀은 고정되어 있지만, 세부 구현만 다양하게 적용해야 할 때 적합합니다. 알고리즘의 흐름 자체는 변경할 필요가 없고, 각 단계의 구체적인 동작만 변경하고 싶을 때 사용합니다.
* **전략 패턴**: 알고리즘 자체를 완전히 교체할 수 있도록 유연하게 설계하고자 할 때 적합합니다. 알고리즘의 선택이 실행 시점에 동적으로 결정되어야 하거나, 다양한 알고리즘을 쉽게 교체하고 확장할 수 있는 구조가 필요할 때 사용합니다.
