---
description: 인터페이스는 타입을 정의하는 용도로만 사용하라
---

# Item22. Use Interfaces Only to define Types

자바에서 인터페이스는 자주 사용되지만 잘못된 방식으로 사용되는 경우가 많습니다.



## 1. 인터페이스의 본질적인 역할

인터페이스는 객체가 가져야 할 행동(메서드)의 집합을 정의하는 역할을 합니다. 인터페이스는 타입을 정의하고 용도이고, 이를 통해 구현체는 해당 인터페이스가 요구하는 메서드를 구현하고 다형성을 활용할 수 있습니다.

<details>

<summary> 타입</summary>

변수나 객체가 가질 수 있는 값의 종류. 데이터의 형태나 동작 방식을 결졍하는 역할을 합니다. 자바는 정적 타입 언어이기 때문에, 모든 변수나 객체는 미리 정의된 타입을 가져야합니다. 타입에는 기본형과 참조형이 있습니다.

</details>



## 2. 인터페이스를 상수 모음으로 오용하지 마라

<details>

<summary> 상수 인터페이스</summary>

```java
public interface Constants {
    String APP_NAME = "MyApplication";
    int MAX_USERS = 100;
}
```

해당 인터페이스는 설계 목적을 벗어난 용도로 사용되고 있습니다. 상수를 모아두기 위해 인터페이스를 사용하면, 해당 인터페이스를 구현한 클래스들이 상속의 의도와 무관하게 이 상수들을 상속받아야 한다는 문제점이 발생합니다.

</details>

### 2.1. 상수 인터페이스의 문제점

* 타입 역할을 하지 않음\
  인터페이스는 타입을 정의하기 위한 용도인데, 상수만 포함된 인터페이스는 타입으로서의 역할을 수행하지 못합니다.
* 구현체가 상수를 노출\
  인터페이스를 구현하는 클래스는 불필요하게 상수를 상속받아야 하며, 이로 인해 상수와 클래스 간의 관계가 모호해집니다. 클래스 내부에서만 필요한 상수들은 외부와 상관없는 **클래스의 세부적인 구현**이므로 인터페이스에 정의하는 것은 부적절하다. 예를 들어, 어떤 클래스가 내부적으로 사용하는 상수는 **해당 클래스의 내부적인 동작을 조절하는 것**일 뿐, 외부에서 이 상수에 접근하거나 의존할 필요가 없다.
* 구현체에 불필요한 의존성 추가\
  인터페이스를 구현하는 클래스는 그 인터페이스의 모든 상수를 무조건 상속받게 됩니다. 상수 인터페이스를 남용하게 되면, 단순히 상수를 가져오기 위한 목적으로 사용되므로 인터페이스 본래의 역할과 목적을 벗어난 오용이 됩니다.
* 복잡성 증가\
  상수 인터페이스를 구현하는 클래스에 구체적인 동작이나 기능을 정의하지 않음에도 인터페이스를 "구현" 하는 형태가 됩니다. 이는 클래스가 실제로 제공하는 기능과 관계없이 불필요하게 많은 인터페이스 목록을 갖게 되며, 실제로 사용하는 개발자나 유지보수하는 사람에게 혼란을 줄 수 있습니다.

<details>

<summary>복잡한 상속 계층 문제</summary>

```java
public interface Constants1 {
    String VALUE1 = "Value1";
}

public interface Constants2 {
    String VALUE2 = "Value2";
}

public interface Constants3 {
    String VALUE3 = "Value3";
}

public class MyApp implements Constants1, Constants2, Constants3 {
    // 많은 인터페이스를 구현하지만, 실제로는 상수만 사용
}
```

위 예시에서 `MyApp` 클래스는 세 개의 상수 인터페이스를 구현하고 있습니다. 이 경우 클래스는 많은 인터페이스를 구현하지만, 실제로는 상수를 상속받아 사용할 뿐입니다. 상수만을 위해 **여러 인터페이스를 상속받아 구현하는 것은 클래스 상속 계층을 불필요하게 복잡하게 만들고**, 유지보수와 코드 가독성에 악영향을 미칩니다.

</details>



## 3. 상수 유틸리티 클래스를 사용하라

상수 유틸리티 클래스는 오직 상수만을 관리하는 정적 필드로 구성된 클래스입니다. 이 클래스는 인스턴스화가 필요 없으며, 단지 상수를 외부에서 쉽게 참조할 수 있도록 정적 필드를 제공하는 역할을 합니다.&#x20;

클래스 내부의 모든 필드는 public static final로 선언되며, 해당 상수를 클래스명.상수명 형식으로 참조합니다.

<details>

<summary> 기본 구조</summary>

```java
public class UtilityClass {
    // 상수를 선언
    public static final String CONSTANT_VALUE = "SomeConstant";

    // 인스턴스화 방지를 위한 private 생성자
    private UtilityClass() {
        throw new AssertionError();  // 인스턴스화 방지
    }
}
```

이 구조에서는 `UtilityClass`가 오직 상수만을 제공하고, **생성자를 `private`으로 선언하여 인스턴스화를 방지**합니다. 이 방식은 **상수 관리**와 **코드의 명확성**을 높이는 데 매우 효과적입니다.

</details>



### 3.1. 상수 유틸리티 클래스의 장점

* 인터페이스 오용방지\
  상수를 클래스 내부에서 관리하고, 인터페이스는 타입과 행동을 정의하는 역할로만 사용할 수 있어서, 상수 인터페이스 안티패턴 문제를 해결할 수 있습니다.
* 인스턴스화 방지\
  상수 유틸리티 클래스는 인스턴스화할 필요가 없으므로, 불필요한 객체 생성을 방지합니다.
* 가독성 및 유지보수성 향상\
  논리적으로 관련된 상수들을 하나의 클래스에서 모아 관리하므로, 가독성이 높아지고 유지보수성도 좋아집니다.

<details>

<summary>@UtilityClass (Lombok)</summary>

```java
import lombok.experimental.UtilityClass;

@UtilityClass
public class AppConstants {
    public static final String APP_NAME = "MyApplication";
    public static final int MAX_USERS = 100;
}
```

이 방식은 **Lombok이 자동으로 상수 유틸리티 클래스의 기본 구조를 생성**해 주기 때문에, 코드를 더 간결하게 만들 수 있습니다. **생성자를 직접 작성할 필요 없이**, Lombok이 이를 처리하여 유틸리티 클래스의 역할을 수행합니다.

</details>



## 4. 상수가 포함된 정석 인터페이스

상수가 포함된 인터페이스를 정석으로 사용하는 경우는 거의 없습니다. 드물에 사용하는 경우에는 특정 상수를 외부에 명시적으로 노출하고, 그 상수를 구현하는 모든 클래스에서 반드시 사용하게 될 때입니다.

<details>

<summary>Example</summary>

```java
public interface HttpStatus {
    // 상수 정의
    int OK = 200;
    int NOT_FOUND = 404;
    int INTERNAL_SERVER_ERROR = 500;

    // HTTP 상태 코드에 대한 설명을 반환하는 메서드
    String getStatusMessage(int statusCode);
}
```

이 경우, **HTTP 상태 코드**라는 특정한 의미를 전달하기 위해 상수를 인터페이스에 포함시키는 것이 합리적입니다. 이러한 상수는 **HTTP 프로토콜에 대한 명확한 규격**을 정의하며, 이를 구현하는 클래스는 이 규격을 따르게 됩니다. 여기서 상수는 그 인터페이스의 **핵심적인 역할**을 하기 때문에 적절하게 인터페이스에 포함된 예시라고 볼 수 있습니다.

</details>

