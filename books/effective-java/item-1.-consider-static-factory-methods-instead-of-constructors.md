---
description: 생성자 대신 정적 팩터리 메서드를 고려하라
---

# Item 1. Consider Static Factory Methods Instead of Constructors

클라이언트가 클래스의 인스턴스를 얻는 전통적인 방법은 public 생성자를 이용하는 것입니다. 하지만 모든 프로그래머가 알아야 할 중요한 개념 중 하나는 바로 정적 팩터리 메서드(static factory method)입니다. 정적 팩터리 메서드를 사용하면 생성자에 비해 여러 가지 장점을 누릴 수 있습니다. 이 글에서는 정적 팩터리 메서드의 정의와 장점, 그리고 단점을 상세히 살펴보겠습니다.

**정적 팩터리 메서드란?**

정적 팩터리 메서드는 클래스의 인스턴스를 반환하는 단순한 정적 메서드입니다. 예를 들어, `Boolean` 클래스에는 기본 타입의 `boolean` 값을 받아 `Boolean` 객체 참조로 변환하는 `valueOf` 메서드가 있습니다.

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

이 메서드는 Boolean 객체를 생성하지 않고도 캐싱된 인스턴스를 반환하여 메모리 사용량을 줄일 수 있습니다. 이처럼 정적 팩터리 메서드는 객체 생성과 관련된 다양한 최적화 및 편리함을 제공할 수 있습니다.

**정적 팩터리 메서드의 장점**

1. **이름을 가질 수 있다**: 생성자와 달리 정적 팩터리 메서드는 이름을 가질 수 있어 반환될 객체의 특성을 명확히 설명할 수 있습니다. 예를 들어, `BigInteger(int, int, Random)` 생성자는 그 목적이 불분명하지만, `BigInteger.probablePrime`은 소수를 반환한다는 의미를 명확히 전달합니다. 이는 코드의 가독성을 크게 향상시킵니다.\

2.  **호출할 때마다 인스턴스를 새로 생성하지 않아도 된다**: 정적 팩터리 메서드는 요청할 때마다 동일한 인스턴스를 반환할 수 있습니다. 이는 불변 클래스(immutable class)에 특히 유용합니다. 대표적인 예로 `Boolean.valueOf(boolean)` 메서드는 불필요한 객체 생성을 피하기 위해 미리 생성된 인스턴스를 재사용합니다.

    ```java
    Boolean trueInstance = Boolean.valueOf(true);
    Boolean falseInstance = Boolean.valueOf(false);
    ```

    이와 같은 패턴은 메모리 사용을 최적화하고 성능을 높이는 데 기여합니다. 또한 플라이웨이트 패턴(Flyweight pattern)과 유사한 방식으로 메모리를 절약할 수 있습니다.\

3.  **반환 타입의 하위 타입 객체를 반환할 수 있다**: 정적 팩터리 메서드는 반환할 객체의 하위 타입을 반환할 수 있습니다. 이는 API 설계 시 유연성을 높이는 중요한 기능입니다. 예를 들어, `java.util.EnumSet` 클래스는 정적 팩터리 메서드를 사용하여 여러 하위 타입의 인스턴스를 반환할 수 있습니다.

    ```java
    EnumSet<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
    ```

    이 방법을 사용하면 특정 상황에 맞게 객체를 최적화할 수 있습니다.\

4.  **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다**: 정적 팩터리 메서드는 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있습니다. 예를 들어, `EnumSet` 클래스는 원소의 수에 따라 `RegularEnumSet`이나 `JumboEnumSet`을 반환합니다.

    ```java
    EnumSet<Rank> regularSet = EnumSet.of(Rank.JACK);
    EnumSet<Rank> jumboSet = EnumSet.allOf(Rank.class);
    ```

    이는 클라이언트 코드의 복잡성을 줄이고, 상황에 맞는 최적의 객체를 반환할 수 있게 합니다.\

5.  **작성할 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다**: 정적 팩터리 메서드는 서비스 제공자 프레임워크(service provider framework)를 구현할 때 특히 유용합니다. 클라이언트는 실제 구현 클래스를 몰라도 인터페이스를 통해 객체를 사용할 수 있습니다. 예를 들어, JDBC(Java Database Connectivity) API는 데이터베이스 연결을 얻기 위해 정적 팩터리 메서드를 사용합니다.

    ```java
    Connection conn = DriverManager.getConnection(url);
    ```

    이러한 방식은 서비스 제공자의 유연성과 확장성을 높이는 데 기여합니다.





**정적 팩터리 메서드의 단점**

1.  **상속을 하려면 public이나 protected 생성자가 필요하다**: 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없습니다. 이는 상속이 필요한 경우 유연성을 제한할 수 있습니다. 이 문제를 해결하려면 상속용 public 또는 protected 생성자를 추가로 제공해야 합니다.

    ```java
    public class MyClass {
        private MyClass() { }
        public static MyClass newInstance() {
            return new MyClass();
        }
    }

    public class MySubclass extends MyClass {
        public MySubclass() {
            super();
        }
    }
    ```
2. **프로그램이 커지면 찾기 어렵다**: 생성자와 달리 정적 팩터리 메서드는 API 문서에 명시적으로 드러나지 않기 때문에 사용자가 찾기 어려울 수 있습니다. 따라서 API 문서와 클래스 설명에 정적 팩터리 메서드를 명확히 문서화하는 것이 중요합니다.

**결론**

정적 팩터리 메서드는 객체 생성의 유연성을 제공하며, 이름을 통해 가독성을 높이고, 같은 객체를 재사용하는 등의 장점을 제공합니다. 특히 반환 타입의 하위 타입 객체를 반환하거나, 입력 매개변수에 따라 다른 클래스의 객체를 반환하는 능력은 매우 유용합니다. 그러나 상속 문제와 문서화의 중요성을 간과해서는 안 됩니다. 이러한 점을 염두에 두고 상황에 맞춰 정적 팩터리 메서드와 생성자를 적절히 활용하면 더 나은 코드와 API를 설계할 수 있습니다.

마지막으로, 정적 팩터리 메서드를 사용할 때의 몇 가지 주의 사항을 정리하면 다음과 같습니다:

* 상속을 고려해야 하는 경우에는 public 또는 protected 생성자를 제공하는 것이 좋습니다.
* 정적 팩터리 메서드는 반드시 API 문서에 명확히 명시하여 사용자가 쉽게 이해할 수 있도록 해야 합니다.
* 반복되는 객체 생성을 피하고, 재사용 가능한 인스턴스를 제공하는 데 유리한 패턴을 적용합니다.

이러한 사항들을 고려하여 정적 팩터리 메서드를 적절히 활용하면 더 효율적이고 유지 보수하기 쉬운 코드를 작성할 수 있습니다.
