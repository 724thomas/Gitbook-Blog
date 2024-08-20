---
description: Public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라
---

# Item 16. Accessor Methods Over Public Fields

자바에서의 클래스 설계 시, 객체 지향 원칙을 준수하는 것이 중요합니다. 이 원칙 중 하나는 클래스의 상태를 외부에 노출하지 않고, 접근자 메서드를 통해 접근하게 하는 것입니다. 이러한 설계는 클래스의 유연성을 높이고, 유지보수성을 향상시킵니다.



## 1. Public 필드의 문제점

클래스의 필드를 public으로 선언하게 되면, 그 필드에 직접 접근이 가능합니다. 이 방법은 코드 작성이 간단하고, 초기 학습 단계에서는 매력적으로 보일 수 있습니다. 하지만, 이 방식에는 다음과 같은 문제점들이 존재합니다:

1. **캡슐화의 위반**:\
   객체 지향 프로그래밍에서 중요한 원칙 중 하나인 캡슐화(encapsulation)는 객체의 상태를 외부에서 직접 변경하지 못하도록 보호하는 것입니다. 그러나 필드가 public으로 선언되면, 외부에서 직접 해당 필드를 조작할 수 있어 객체의 상태가 불안정해질 수 있습니다. 이로 인해 버그가 발생할 가능성이 높아집니다.
2. **변경에 취약**:\
   public 필드를 사용하는 경우, 해당 필드를 사용하는 모든 코드가 그 필드의 내부 구조에 의존하게 됩니다. 이로 인해 필드의 타입이나 의미를 변경하려면 그 필드를 참조하는 모든 코드를 수정해야 합니다. 반면, 접근자 메서드를 사용하면 메서드 내부에서 필드에 대한 변경 사항을 감출 수 있습니다.
3. **불변성을 보장하기 어려움**:\
   불변 객체는 다중 스레드 환경에서 안전하게 사용할 수 있어 자바에서 중요한 개념입니다. 하지만 public 필드를 사용하면 객체의 불변성을 보장하기 어렵습니다. 필드를 public으로 노출하면, 외부에서 해당 필드를 변경할 수 있기 때문에, 객체의 상태가 변하게 됩니다.

<details>

<summary><strong>예시: Public 필드 사용의 문제점</strong></summary>

```java
public class Point {
    public int x;
    public int y;
}

Point p = new Point();
p.x = 5;
p.y = 10;
```

위의 코드는 직관적이고 간단해 보일 수 있지만, 문제의 소지가 있습니다. `x`와 `y`의 값을 아무나 변경할 수 있으며, 이러한 변경이 예상치 못한 버그를 일으킬 수 있습니다.

</details>



***

## 2. 접근자 메서드를 사용해야 하는 이유

접근자 메서드를 사용하면, 객체의 내부 구현을 숨기고 외부와의 상호작용을 제어할 수 있습니다. 다음은 접근자 메서드를 사용해야 하는 몇 가지 주요 이유입니다:

1. **캡슐화**:\
   필드를 private으로 선언하고, 접근자(getter)와 설정자(setter) 메서드를 제공하면, 클래스 외부에서는 이 필드에 직접 접근할 수 없습니다. 이로 인해 클래스의 내부 구현이 외부로부터 보호됩니다.
2. **유연성**:\
   접근자 메서드를 사용하면, 필드의 값이 읽히거나 쓰일 때 추가적인 로직을 삽입할 수 있습니다. 예를 들어, 값을 읽을 때 캐싱을 사용하거나, 값을 설정할 때 유효성 검사를 추가할 수 있습니다.
3. **불변성 강화**:\
   클래스의 필드를 final로 선언하고, 생성자에서만 초기화하게 하면, 해당 클래스는 불변 객체가 됩니다. 이는 다중 스레드 환경에서 안전성을 제공합니다.
4. **인터페이스와의 일관성**:\
   접근자 메서드를 사용하면, 클래스의 필드를 외부에 공개하지 않고도 인터페이스를 통해 일관된 방법으로 접근할 수 있습니다. 이는 코드의 가독성과 유지보수성을 높이는 데 도움이 됩니다.



<details>

<summary><strong>예시: 접근자 메서드를 사용한 개선된 코드</strong></summary>

```java
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }
}
```

위의 코드에서는 `x`와 `y` 필드가 private으로 선언되어 있으며, 이를 통해 캡슐화가 이루어집니다. 외부에서는 `getX()`와 `getY()` 메서드를 통해서만 `x`와 `y`의 값을 읽을 수 있습니다.

</details>



***

#### 3. 불변 클래스의 설계

불변 클래스는 객체가 생성된 이후 그 상태를 변경할 수 없는 클래스를 의미합니다. 이러한 클래스는 다중 스레드 환경에서 안전하게 사용될 수 있어, 자바에서 특히 중요합니다. 불변 클래스를 설계할 때 접근자 메서드를 사용하는 것은 필수적입니다.

<details>

<summary><strong>예시: 불변 클래스 설계</strong></summary>

```java
public final class ImmutablePoint {
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }
}
```

위의 `ImmutablePoint` 클래스는 불변 객체로, `x`와 `y` 필드는 final로 선언되어 있으며, 생성자를 통해 초기화됩니다. 필드를 외부에 노출하지 않고, 접근자 메서드를 통해서만 값을 읽을 수 있습니다. 이로써, 해당 클래스는 불변성을 보장하며, 안전하게 사용할 수 있습니다.

</details>

***

## 4. Lombok을 사용한 접근자 메서드 자동 생성

자바에서 접근자 메서드를 수동으로 작성하는 것은 반복적이고 번거로울 수 있습니다. 이럴 때, Lombok 라이브러리를 사용하면 접근자 메서드를 자동으로 생성할 수 있습니다. Lombok을 사용하면 코드가 간결해지고, 유지보수성이 향상됩니다.

<details>

<summary><strong>Lombok 예시</strong></summary>

```java
import lombok.Getter;

public class Point {
    @Getter
    private int x;
    @Getter
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

위 코드에서 Lombok의 `@Getter` 애노테이션을 사용하여 `x`와 `y` 필드에 대한 접근자 메서드가 자동으로 생성됩니다. 이로 인해 코드를 더 간결하게 유지할 수 있습니다.

</details>



## 6. 캡슐화와 정보 은닉의 중요성

캡슐화(encapsulation)와 정보 은닉(information hiding)은 객체 지향 프로그래밍의 핵심 개념으로, 클래스 설계에서 중요한 역할을 합니다. 이 두 개념은 클래스 내부의 구현 세부 사항을 외부로부터 숨기고, 외부와의 상호작용을 제한함으로써 코드의 안정성과 유지보수성을 높이는 데 기여합니다.

public 필드를 사용하는 것은 캡슐화와 정보 은닉의 원칙을 위반하는 행위입니다. 반면, 접근자 메서드를 통해 필드에 접근하도록 설계하면, 클래스 내부의 구현을 보호할 수 있습니다.

<details>

<summary><strong>캡슐화의 예시</strong></summary>

캡슐화의 주요 목표는 객체의 내부 상태를 보호하는 것입니다. 예를 들어, 계좌의 잔액을 다루는 `BankAccount` 클래스가 있다고 가정해 봅시다.

```java
public class BankAccount {
    private double balance;

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        }
    }
}
```

위의 코드에서 `balance` 필드는 private으로 선언되어 있으며, 외부에서 직접 접근할 수 없습니다. 대신 `deposit()`과 `withdraw()` 메서드를 통해 잔액을 변경할 수 있습니다. 이렇게 하면 잘못된 값이 잔액으로 설정되는 것을 방지할 수 있습니다.

</details>

<details>

<summary><strong>정보 은닉의 예시</strong></summary>

정보 은닉은 캡슐화와 밀접한 관련이 있으며, 클래스 내부의 세부 구현을 외부에서 숨기는 것을 목표로 합니다. 예를 들어, `TemperatureSensor` 클래스가 온도를 측정하는 방식이 변경될 가능성이 있다고 가정해 봅시다.

```java
public class TemperatureSensor {
    private double temperatureInCelsius;

    public double getTemperatureInCelsius() {
        return temperatureInCelsius;
    }

    public void updateTemperature(double newTemperature) {
        this.temperatureInCelsius = newTemperature;
    }
}
```

이 클래스의 내부 구현이 변경되어 온도를 섭씨에서 화씨로 저장해야 한다고 해도, 외부 코드에는 영향을 주지 않습니다. 외부에서는 여전히 `getTemperatureInCelsius()` 메서드를 통해 섭씨 온도를 읽을 수 있습니다. 이러한 설계는 정보 은닉을 통해 코드의 유연성과 확장성을 높입니다.

</details>



***

## 7. 접근자 메서드의 유효성 검사 및 추가 로직

접근자 메서드를 사용하는 또 다른 중요한 이유는 필드 값에 대한 유효성 검사와 추가 로직을 적용할 수 있다는 점입니다. 필드에 직접 접근하면 이러한 로직을 적용할 수 있는 기회가 사라지지만, 접근자 메서드를 사용하면 필요에 따라 다양한 검사를 추가할 수 있습니다.

<details>

<summary><strong>유효성 검사의 예시</strong></summary>

아래의 예시는 유효성 검사를 통해 잘못된 데이터가 설정되는 것을 방지하는 예시입니다.

```java
public class Rectangle {
    private int width;
    private int height;

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        if (width > 0) {
            this.width = width;
        } else {
            throw new IllegalArgumentException("Width must be positive");
        }
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        if (height > 0) {
            this.height = height;
        } else {
            throw new IllegalArgumentException("Height must be positive");
        }
    }
}
```

위 코드에서 `setWidth()`와 `setHeight()` 메서드는 폭과 높이가 0보다 큰지 확인합니다. 이러한 유효성 검사를 통해 잘못된 값이 필드에 설정되는 것을 방지할 수 있습니다.

</details>

<details>

<summary><strong>추가 로직의 예시</strong></summary>

```java
public class Employee {
    private double salary;

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        if (salary < 0) {
            throw new IllegalArgumentException("Salary cannot be negative");
        }
        this.salary = salary;
        updateTaxInformation();
    }

    private void updateTaxInformation() {
        // 세금 정보를 업데이트하는 추가 로직
    }
}
```

`setSalary()` 메서드는 급여를 설정하면서 추가적으로 세금 정보를 업데이트하는 로직을 포함합니다. 이렇게 하면 필드 값의 변경과 관련된 모든 작업을 중앙 집중화할 수 있습니다.

</details>

***

## 8. 접근자 메서드와 객체 직렬화

객체 직렬화는 객체의 상태를 저장하거나 네트워크를 통해 전송할 때 사용됩니다. 자바에서는 `Serializable` 인터페이스를 사용하여 객체를 직렬화할 수 있습니다. 하지만, 직렬화 가능한 객체의 필드를 public으로 선언하는 것은 위험할 수 있습니다. 필드가 외부에 노출되면 객체의 내부 구현이 고정되어 버리기 때문에, 향후 변경이 어려워집니다.

<details>

<summary><strong>접근자 메서드와 직렬화의 관계</strong></summary>

접근자 메서드를 사용하면, 필드의 직렬화 여부를 제어할 수 있으며, 직렬화 과정에서 필드의 노출을 방지할 수 있습니다. 예를 들어, 민감한 데이터가 포함된 객체를 직렬화할 때는 이러한 필드를 transient로 선언하여 직렬화에서 제외할 수 있습니다.

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    private String username;
    private transient String password; // 직렬화에서 제외됨

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

위의 코드에서 `password` 필드는 `transient` 키워드를 사용하여 직렬화에서 제외되었습니다. 이를 통해 민감한 정보가 직렬화될 때 유출되는 것을 방지할 수 있습니다.

</details>

***

## 9. 접근자 메서드의 성능 고려사항

일부 개발자들은 성능상의 이유로 필드를 public으로 선언하기도 합니다. 접근자 메서드를 호출하는 것보다 필드에 직접 접근하는 것이 더 빠르다고 생각할 수 있기 때문입니다. 하지만, 현대의 자바 컴파일러와 JIT(Just-In-Time) 컴파일러는 이러한 차이를 최소화하며, 실제로는 접근자 메서드를 사용하는 것이 더 안전하고 권장됩니다.

**성능 최적화와 접근자 메서드**

실제 애플리케이션에서 성능 문제가 발생하는 경우는 드뭅니다. 대부분의 경우, 접근자 메서드를 사용하는 것이 성능에 미치는 영향은 미미하며, 코드의 가독성과 유지보수성을 높이는 데 더 큰 이점을 제공합니다. 따라서 성능 최적화보다는 안전성과 유연성을 우선시하는 것이 좋습니다.
