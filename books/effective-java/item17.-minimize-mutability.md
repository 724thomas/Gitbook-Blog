---
description: 아이템 17 변경 가능성을 최소화하라
---

# Item17. Minimize Mutability

불변 객체는 그 상태를 변경할 수 없기 때문에, 프로그램의 안전성과 신뢰성을 높이는 데 크게 기여할 수 있다. 이러한 불변 객체는 다중 스레드 환경에서도 특별한 동기화 없이 안전하게 사용할 수 있으며, 에러 발생 가능성을 최소화할 수 있다.



## 1. 불변 객체란 무엇인가?

불변 객체(Immutable Object)는 생성된 후 그 상태가 절대로 바뀌지 않는 객체를 말한다. 이는 객체의 모든 필드가 초기화 후에는 변경될 수 없음을 의미한다. 불변 객체를 사용하면 예상치 못한 변경으로 인한 오류를 방지할 수 있고, 객체 간 공유가 안전하게 이루어질 수 있다.

### 1.1 불변 객체의 특징

* **상태 불변성**: 객체가 생성된 후 그 상태가 변경되지 않는다.
* **객체 공유 가능성**: 불변 객체는 안전하게 공유될 수 있으며, 여러 스레드가 동기화 없이 접근할 수 있다.
* **방어적 복사 필요 없음**: 불변 객체는 외부에서 그 상태를 변경할 수 없으므로, 방어적 복사(defensive copy)가 필요 없다.

## 2. 불변 객체의 이점

불변 객체는 다양한 장점을 제공한다. 특히 다중 스레드 환경에서 유용하며, 코드의 유지보수성과 안정성을 크게 향상시킨다.

### 2.1 스레드 안전성

불변 객체는 스레드 안전(thread-safe)하다. 즉, 여러 스레드가 동시에 접근해도 객체의 상태가 변경되지 않기 때문에 동기화(synchronization) 없이 안전하게 사용할 수 있다. 이는 성능 최적화와 코드의 단순화에도 기여한다.

### 2.2 오류 감소

불변 객체는 상태가 변하지 않기 때문에 예상치 못한 변경으로 인한 오류를 방지할 수 있다. 특히, 객체가 여러 메서드나 클래스에서 사용될 때 이러한 장점은 더욱 두드러진다. 프로그램의 일관성을 유지하고, 디버깅과 테스트를 쉽게 할 수 있게 된다.

### 2.3 코드의 단순화

불변 객체는 객체를 생성하는 시점 이후에 상태를 변경할 필요가 없기 때문에, 불변 객체를 사용하는 코드는 일반적으로 가변 객체를 사용하는 코드보다 간단하다. 상태 변경에 따른 복잡한 로직이 제거되므로 코드가 더욱 명료해진다.

## 3. 불변 객체 설계 방법

불변 객체를 설계하려면 몇 가지 규칙을 따라야 한다. 아래는 불변 객체를 설계하기 위한 5가지 기본 원칙이다.

#### 3.1 객체 상태를 변경하는 메서드를 제공하지 않는다

불변 객체의 가장 중요한 원칙은 객체의 상태를 변경하는 메서드를 제공하지 않는 것이다. 이 말은, 객체의 필드를 변경할 수 있는 메서드(setter)를 포함하지 않는 것을 의미한다.

<details>

<summary>public class ImmutableClass</summary>

```java
public class ImmutableClass {
    private final int value;

    public ImmutableClass(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

위 예제에서 `ImmutableClass`는 `value` 필드를 초기화하는 생성자만 제공하고, 그 값을 변경하는 메서드를 제공하지 않는다.

</details>

### 3.2 클래스를 확장할 수 없도록 한다

불변 객체로 설계된 클래스는 확장할 수 없도록 해야 한다. 만약 클래스를 상속받아 하위 클래스에서 상태를 변경할 수 있다면, 불변성을 유지할 수 없기 때문이다. 이를 위해 클래스에 `final` 키워드를 사용한다.

```java
public final class ImmutableClass {
    // 클래스 내용 생략
}
```

### 3.3 모든 필드를 `final`로 선언한다

클래스의 모든 필드는 `final`로 선언하여, 객체가 생성된 후에 필드가 변경되지 않도록 한다. `final` 키워드는 필드의 값이 한 번 초기화된 후에는 변경될 수 없음을 보장한다.

```java
private **final** int value;
```

#### 3.4 모든 필드를 `private`로 선언한다

모든 필드를 `private`로 선언하여 외부에서 접근하지 못하도록 해야 한다. 이렇게 하면 객체의 내부 상태가 외부에 노출되지 않으며, 불변성을 유지할 수 있다.

```java
**private** final int value;
```

#### 3.5 가변 객체를 참조하는 필드는 방어적으로 복사한다

불변 객체가 가변 객체를 참조할 때는 그 가변 객체를 방어적으로 복사하여 내부에 저장해야 한다. 이렇게 하면 외부에서 가변 객체를 변경하더라도 불변 객체의 상태가 변하지 않도록 보호할 수 있다.

<details>

<summary>public final class ImmutableClass</summary>

```java
import java.util.Date;

public final class ImmutableClass {
    private final Date date;

    public ImmutableClass(Date date) {
        this.date = new Date(date.getTime()); // 방어적 복사
    }

    public Date getDate() {
        return new Date(date.getTime()); // 방어적 복사
    }
}
```

위 예제에서 `Date` 객체는 가변 객체이기 때문에, `ImmutableClass`에서 해당 객체를 직접 참조하지 않고 방어적으로 복사하여 사용한다.

</details>

## 4. Lombok을 활용한 불변 객체의 구현

Lombok은 자바에서 반복적인 코드를 줄여주는 유용한 라이브러리다. 특히 불변 객체를 쉽게 구현할 수 있도록 도와주는 `@Value`와 `@Builder` 등의 어노테이션을 제공한다. 이를 활용하여 불변 객체를 더 간단하게 작성할 수 있다.

### 4.1 `@Value` 어노테이션

`@Value` 어노테이션을 사용하면, 불변 객체를 간단하게 생성할 수 있다. `@Value`는 클래스의 모든 필드를 `final`과 `private`으로 선언하고, 생성자, `getter` 메서드, `toString`, `equals`, `hashCode` 메서드를 자동으로 생성해준다.

```java
import lombok.Value;

@Value
public class ImmutableClass {
    int value;
}
```

위 예제에서 `ImmutableClass`는 Lombok의 `@Value` 어노테이션을 통해 불변 객체로 쉽게 선언되었다.

### 4.2 `@Builder` 어노테이션

Lombok의 `@Builder` 어노테이션은 빌더 패턴을 사용하여 객체를 생성할 수 있도록 도와준다. 빌더 패턴은 불변 객체를 생성할 때 특히 유용한 패턴으로, 복잡한 객체를 단계적으로 생성할 수 있다.

```java
import lombok.Builder;
import lombok.Value;

@Value
@Builder
public class ImmutableClass {
    int value;
    String name;
}
```

위 예제에서 `ImmutableClass`는 빌더 패턴을 통해 불변 객체로 생성되었다. 빌더 패턴을 사용하면 코드의 가독성을 높이고, 더 유연하게 객체를 생성할 수 있다.

## 5. 불변 객체의 예시: `String` 클래스

Java의 `String` 클래스는 대표적인 불변 객체이다. `String` 클래스는 한 번 생성되면 그 값을 변경할 수 없으며, 모든 `String` 메서드는 새로운 `String` 객체를 반환한다. 이러한 불변성 덕분에 `String` 클래스는 안전하게 공유될 수 있으며, 여러 스레드가 동시에 사용할 수 있다.

```java
String str = "Hello";
str.toUpperCase(); // 새로운 "HELLO" 객체 반환, 원본 "Hello"는 변경되지 않음
```

위 예제에서 `toUpperCase()` 메서드는 새로운 `String` 객체를 반환하며, 원래의 `str`은 여전히 "Hello" 값을 유지한다.

### 6. 결론

불변 객체는 자바 프로그램에서 안전성과 신뢰성을 높이는 중요한 개념이다. 불변 객체를 설계하고 사용하는 것은 오류를 줄이고, 유지보수성을 높이며, 특히 다중 스레드 환경에서 큰 이점을 제공한다. 이펙티브 자바의 아이템 17에서는 이러한 불변 객체의 중요성을 강조하며, 불변 객체를 효과적으로 설계하기 위한 가이드라인을 제공하고 있다. 또한, Lombok을 사용하여 불변 객체를 더 쉽게 구현할 수 있으며, 이를 통해 반복적인 코드를 줄이고 코드의 가독성을 높일 수 있다.

불변 객체를 설계할 때는 이 가이드라인을 따르고, 코드의 안정성과 일관성을 유지하는 데 주의를 기울여야 한다. 이를 통해 보다 견고하고 유지보수하기 쉬운 프로그램을 작성할 수 있을 것이다.



## 7. 함수형 프로그래밍과 절차형 프로그래밍: 객체와의 연관성

불변 객체의 개념을 이해하려면, 함수형 프로그래밍과 절차형 프로그래밍이 객체를 어떻게 다루는지 알아보는 것이 중요하다. 두 프로그래밍 패러다임은 객체의 상태 관리 방식에서 근본적인 차이를 보이며, 이는 프로그램의 안정성과 유지보수성에 큰 영향을 미친다.

## 8. 절차형 프로그래밍과 객체

절차형 프로그래밍은 프로그램을 일련의 명령어 또는 절차로 구성하며, 객체의 상태는 프로그램 실행 중 계속해서 변경될 수 있다. 이 패러다임에서는 객체가 상태를 가지며, 해당 상태를 변경하는 메서드가 일반적으로 사용된다.

### **8.1 절차형 프로그래밍에서의 객체 사용**

절차형 프로그래밍에서 객체는 주로 상태를 변경하기 위해 사용된다. 예를 들어, 객체가 필드를 가지고 있으며, 이 필드는 메서드를 통해 변경될 수 있다. 이러한 방식은 프로그램이 동작하는 동안 객체의 상태가 여러 번 변경될 수 있음을 의미한다.

<details>

<summary>public class Account</summary>

```java
public class Account {
    private double balance;

    public Account(double initialBalance) {
        this.balance = initialBalance;
    }

    public void deposit(double amount) {
        balance += amount;
    }

    public void withdraw(double amount) {
        balance -= amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

위 예제에서 `Account` 클래스는 `balance`라는 상태를 가지고 있으며, `deposit`과 `withdraw` 메서드를 통해 이 상태를 변경할 수 있다. 이러한 절차형 프로그래밍 스타일에서는 객체의 상태가 여러 번 변경될 수 있으며, 프로그램의 로직에 따라 그 상태가 바뀌는 것이 일반적이다.

</details>



## 9. 함수형 프로그래밍과 객체

함수형 프로그래밍은 불변성을 중심으로 설계된다. 함수형 프로그래밍에서는 객체의 상태를 변경하지 않고, 새로운 객체를 생성하여 변경된 상태를 반영한다. 이는 객체가 한번 생성되면 그 상태가 절대 변하지 않는다는 것을 의미하며, 불변 객체를 사용하여 프로그램의 예측 가능성과 안정성을 높인다.

### **9.1 함수형 프로그래밍에서의 객체 사용**

함수형 프로그래밍에서 객체는 상태를 변경하지 않는 불변 객체로 사용된다. 상태를 변경할 필요가 있을 때는 기존 객체를 변경하는 대신, 변경된 값을 가진 새로운 객체를 반환한다.

<details>

<summary>public final class Account</summary>

```java
public final class Account {
    private final double balance;

    public Account(double initialBalance) {
        this.balance = initialBalance;
    }

    public Account deposit(double amount) {
        return new Account(this.balance + amount);
    }

    public Account withdraw(double amount) {
        return new Account(this.balance - amount);
    }

    public double getBalance() {
        return balance;
    }
}
```

위 예제에서 `Account` 클래스는 불변 객체로 설계되었다. `deposit`과 `withdraw` 메서드는 새로운 `Account` 객체를 반환하며, 기존 객체의 상태를 변경하지 않는다. 이러한 방식은 객체가 불변성을 유지하도록 하며, 프로그램의 상태 관리가 더욱 간결하고 예측 가능해진다.

</details>



## 10. 절차형 프로그래밍의 한계와 함수형 프로그래밍의 장점

절차형 프로그래밍에서 객체의 상태가 자주 변경되면, 프로그램의 흐름을 추적하고 디버깅하기 어려워진다. 상태 변경이 많아질수록 예상치 못한 부작용(side effects)이 발생할 가능성도 높아진다. 특히 다중 스레드 환경에서는 동기화 문제로 인해 상태 관리가 복잡해지며, 버그 발생 가능성이 커진다.

반면, 함수형 프로그래밍은 객체의 상태를 변경하지 않으므로, 프로그램이 더욱 안정적이고 예측 가능해진다. 불변 객체는 다중 스레드 환경에서도 안전하게 공유될 수 있으며, 상태 변경으로 인한 부작용을 피할 수 있다. 이는 코드의 가독성을 높이고, 유지보수를 용이하게 만든다.

## 11. 객체지향 프로그래밍과 함수형 프로그래밍의 결합

Java와 같은 객체지향 프로그래밍 언어에서는 절차형 프로그래밍과 함수형 프로그래밍의 장점을 결합할 수 있다. 객체지향 언어에서 불변 객체를 사용하면, 객체지향의 캡슐화(encapsulation)와 상속(inheritance) 같은 이점을 누리면서도 함수형 프로그래밍의 불변성과 순수 함수의 장점을 활용할 수 있다.

<details>

<summary>public class Account</summary>

```java
import lombok.Value;

@Value
public class Account {
    double balance;

    public Account deposit(double amount) {
        return new Account(balance + amount);
    }

    public Account withdraw(double amount) {
        return new Account(balance - amount);
    }
}
```

위 예제에서는 Lombok의 `@Value` 어노테이션을 사용하여 불변 객체를 더 쉽게 생성할 수 있다. 이처럼 객체지향 프로그래밍과 함수형 프로그래밍을 결합하면, 코드의 안정성을 높이고 유지보수를 쉽게 하면서도 객체지향의 유연성을 유지할 수 있다.

</details>

```java
```
