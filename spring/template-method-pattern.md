---
description: 템플릿 메서드 패턴
---

# Template Method Pattern

## 정의

템플릿 메서드 패턴은 여러 클래스에서 공통으로 사용하는 메서드를 템플릿화하여 상위 클래스에서 정의하고, 하위 클래스마다 세부 동작 사항을 다르게 구현하는 패턴

변하지 않는 기능은 상위 클래스에 만들어두고, 자주 변경되어 확장할 기능은 하위 클래스에서 만드는것으로, 알고리즘의 골격을 변경해야 할 때, 상위 클래스만 수정하면 됩니다.

<figure><img src="../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

AbstractClass

템플릿 메소드를 구현하고, 템플릿 메서드에서 돌아가는 추상 메서드를 선언합니다. ConcreteClass 역할에 의해 구현됩니다.

ConcreteClass

AbstractClass를 상속하고 추상 메소드를 구체적으로 구현합니다. ConcreteClass에서 구현한 메서드는 AbstractClass의 템플릿 메서드에서 호출됩니다.

<details>

<summary>상위 클래스 (Abstract Class)</summary>

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

<summary>하위 클래스 (Concrete Class)</summary>

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



## 후크

후크 메서드는 부모의 템플릿 메서드의 영향이나 순서를 제어하고 싶을때 사용되는 메서드 형태입니다.

후크 메서드는 하위 클래스에서 선택적으로 Override할 수 있는 메서드로, 템플릿메서드에서 기본적으로는 아무런 동작도 하지 않지만, 필요에 따라 하위 클래스에서 구현하여 특정 동작을 추가/수정합니다.

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



## 장점

* 제어 역전

하위 클래스가 알고리즘의 전체 흐름을 제어하지 않습니다. 상위 클래스에서 전체 흐름을 주도합니다.

* 구현 세부사항 은닉

상위 클래스는 알고리즘의 구조를 알고 있지만, 구체적인 구현 세부사항은 하위 클래스에게 맡깁니다. 상위 클래스는 알고리즘의 전반적인 흐름을 유지하면서도, 세부 구현을 다양하게 변경할 수 있습니다.
