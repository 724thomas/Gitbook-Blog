---
description: 태그 달린 클래스보다는 클래스 계층구조를 활용하라
---

# Item23. Prefer Class Hierarchies to Tagged Classes

"태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류일 뿐이다."

자바에서 태그 달린 클래스는 하나의 클래스가 여러 역할을 담당하도록 설계된 구조로, 클래스의 특정 필드 값에 따라 동작이 달라지는 형태를 가집니다. 이러한 방식은 유연해 보일 수 있지만, 가독성 측면에서 비효율적이며, 오류 발생 가능성을 높입니다.



## 1. 태그 달린 클래스란?

한 클래스 안에서 여러 역할을 수행하도록 설계된 클래스입니다. 보통 특정 필드나 변수를 통해 객체의 상태나 타입을 구분하고, 이 값을 기반으로 동작을 다르게 처리합니다. 즉, 하나의 클래스가 여러 타입의 행동을 모두 포함하려고 하면서, 이를 태그로 구분하는 방식입니다

<details>

<summary>Example (Circle, Rectangle)</summary>

```java
public class Figure {
    enum Shape { RECTANGLE, CIRCLE }

    // 태그 필드
    final Shape shape;

    // RECTANGLE일 때 사용하는 필드
    double length;
    double width;

    // CIRCLE일 때 사용하는 필드
    double radius;

    // 생성자 (사각형)
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    // 생성자 (원)
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 넓이를 계산하는 메서드
    public double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

위 예시에서 `Figure` 클래스는 `RECTANGLE`(사각형)과 `CIRCLE`(원)의 두 가지 타입을 하나의 클래스에서 처리하기 위해 \*\*태그 필드(shape)\*\*를 사용하고 있습니다. 각 도형에 필요한 필드(`length`, `width`, `radius`)가 공존하며, `shape` 필드의 값에 따라 다른 동작을 수행하도록 설계되어 있습니다.

</details>



## 2. 태그 달린 클래스의 문제점

### 2.1. 가독성 저하

태그 필드를 사용해 여러 역할을 하나의 클래스에서 처리하려다 보니, 코드가 복잡해지고 가독성이 떨어집니다. 각 타입에 따라 필드와 메서드가 불필요하게 중복되며, 코드가 길어지고 이해하기 어려워집니다.

### 2.2. 메모리 낭비

위 예시 클래스에서 RECTANGLE일때 radius 필드는 사용되지 않습니다. 하지만 클래스 구조상 모든 필드가 항상 존재하므로, 필요하지 않은 필드들까지 메모리를 차지하게 되어 불필요한 메모리 낭비가 발생합니다.

### 2.3. 확장성 문제

기능을 확장할 때마다 코드 수정이 많아지는 문제가 발생합니다. 새로운 도형 타입을 추가하려면 태그 필드에 새로운 값을 추가하고, 관련 필드와 메서드도 변경해야합니다.&#x20;



## 3. 클래스 계층구조로의 전환

클래스 계층구조는 상속과 다형성을 활용해 각 도형별로 독립적인 클래스를 정의하고, 공통된 인터페이스나 추상 클래스를 통해 일관된 동작을 제공할 수 있습니다.

<details>

<summary>Example (Circle, Rectangle)</summary>

```java
// 추상 클래스: 모든 도형의 공통적인 동작 정의
public abstract class Figure {
    abstract double area();  // 넓이를 계산하는 추상 메서드
}

// 사각형 클래스
public class Rectangle extends Figure {
    private final double length;
    private final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    public double area() {
        return length * width;
    }
}

// 원 클래스
public class Circle extends Figure {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * (radius * radius);
    }
}
```

이 구조에서 **각 도형은 별도의 클래스**로 분리되어 있습니다. `Figure`는 공통된 추상 클래스로서 도형의 **넓이를 계산하는 `area()` 메서드를 정의**하고, `Rectangle`과 `Circle` 클래스는 이를 **구체적으로 구현**합니다.

</details>

### 3.1. 클래스 계층구조의 장점

### 3.1. 가독성 향상

구조가 단순해지며, 각 클래스는 명확한 역할을 갖게 됩니다.

### 3.2. 메모리 효율성 증가

각 클래스는 해당하는 동형에 맞는 필드만 가지므로, 불피룡한 필드가 존재하지 않게됩니다.

### 3.3. 확장성 개선

새로운 도형을 추가하려면 새로운 클래스를 정의하기만 하면되기 때문에, 기존 코드를 수정할 필요 없이 새로운 도형 타입을 쉽게 확장할 수 있습니다.

### 3.4. 오류 발생 가능성 감소

해당하는 메서드가 적절하게 호출되므로, 조건문을 작성하는 것보다 오류 가능성이 줄어듭니다.



