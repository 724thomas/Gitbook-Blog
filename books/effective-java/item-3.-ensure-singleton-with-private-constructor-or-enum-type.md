---
description: private 생성자나 열거 타입으로 싱글턴임을 보증하라
---

# Item 3. Ensure Singleton with Private Constructor or Enum Type

싱글턴(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 의미한다. 싱글턴 패턴은 무상태(stateless) 객체나 시스템 컴포넌트에서 유용하다. 그러나 싱글턴을 올바르게 구현하지 않으면 다양한 문제가 발생할 수 있다. 여기서는 싱글턴 패턴을 안전하게 구현하는 방법을 소개한다.

**싱글턴 패턴의 문제점**

싱글턴 패턴은 일반적으로 두 가지 방식으로 구현된다. 첫 번째는 public static final 필드를 사용하는 방식이고, 두 번째는 정적 팩터리 메서드를 사용하는 방식이다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```

이 방법의 장점은 단순하고 직관적이라는 것이다. 그러나 리플렉션을 사용하면 private 생성자를 호출할 수 있어 싱글턴 패턴이 깨질 수 있다. 이를 방지하려면 생성자를 수정하여 두 번째 인스턴스 생성 시 예외를 던지도록 해야 한다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private static boolean instanceCreated = false;

    private Elvis() {
        if (instanceCreated) {
            throw new IllegalStateException("Already created.");
        }
        instanceCreated = true;
    }

    public void leaveTheBuilding() { ... }
}
```

또한, 싱글턴 클래스가 직렬화 가능한 클래스라면 readResolve 메서드를 제공해야 한다. 그렇지 않으면 역직렬화 과정에서 새로운 인스턴스가 생성될 수 있다.

```java
private Object readResolve() {
    return INSTANCE;
}
```

**정적 팩터리 메서드를 사용하는 싱글턴 패턴**

정적 팩터리 메서드를 사용하면 더 유연하게 싱글턴을 구현할 수 있다. API를 변경하지 않고도 싱글턴 방식에서 변경할 수 있으며, 메서드 참조를 공급자(supplier)로 사용할 수 있다.

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public static Elvis getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() { ... }
}
```

정적 팩터리 메서드는 공급자 인터페이스를 사용할 수 있으며, 유일한 인스턴스를 반환하기 때문에 단위 테스트 작성 시 가짜(mock) 객체로 대체하기 쉽다.

**열거 타입으로 싱글턴 구현하기**

가장 간단하고 안전한 싱글턴 구현 방법은 열거 타입을 사용하는 것이다. 이 방법은 직렬화와 리플렉션 공격에서도 안전하다.

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

이 방법은 다른 방법들보다 간결하고 추가 작업 없이 직렬화가 가능하다. 열거 타입은 인스턴스가 하나뿐임을 보장하며, 복잡한 직렬화 상황에서도 제2의 인스턴스가 생성되지 않는다.

**결론**

싱글턴 패턴을 구현할 때는 주의해야 할 점이 많다. private 생성자를 사용하거나 열거 타입을 사용하면 대부분의 문제를 해결할 수 있다. 특히 열거 타입을 사용하면 싱글턴을 안전하고 쉽게 구현할 수 있다. 이러한 방법을 통해 싱글턴 패턴의 장점을 최대한 활용할 수 있을 것이다.
