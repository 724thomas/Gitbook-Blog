---
description: 상속보다는 컴포지션을 사용하라
---

# Item18. Composition over inherentance

상속은 객체지향 언어에서 자주 사용되는 개념이지만, 무분별한 상속 사용은 코드의 유연성과 유지보수성을 저하시킬 수 있습니다. 이에 비해, 컴포지션(Composition)은 보다 유연하고 견고한 설계를 가능하게 합니다.

요약 1:

<details>

<summary>상속을 사용하는 코드</summary>

`Vehicle` 클래스를 상속받아 `Car`와 `Truck` 클래스를 구현합니다.

```java
// Vehicle 클래스: 모든 차량의 공통 기능을 정의
public class Vehicle {
    private String brand;
    private String model;
    private int year;

    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
    }

    public void start() {
        System.out.println(brand + " " + model + " is starting.");
    }

    public void stop() {
        System.out.println(brand + " " + model + " is stopping.");
    }
}

// Car 클래스: Vehicle 클래스를 상속받아 승용차의 고유 기능 추가
public class Car extends Vehicle {
    private int trunkCapacity;

    public Car(String brand, String model, int year, int trunkCapacity) {
        super(brand, model, year);
        this.trunkCapacity = trunkCapacity;
    }

    public void openTrunk() {
        System.out.println("Opening trunk with capacity: " + trunkCapacity + " liters.");
    }
}

// Truck 클래스: Vehicle 클래스를 상속받아 트럭의 고유 기능 추가
public class Truck extends Vehicle {
    private int loadCapacity;

    public Truck(String brand, String model, int year, int loadCapacity) {
        super(brand, model, year);
        this.loadCapacity = loadCapacity;
    }

    public void haulLoad() {
        System.out.println("Hauling load with capacity: " + loadCapacity + " kg.");
    }
}
```

</details>

<details>

<summary>상속 중 발생하는 문제</summary>

시간이 지나면서 `Vehicle` 클래스에 새로운 기능을 추가하고 싶어졌습니다. 모든 `Vehicle`에 GPS 기능을 추가했지만, 이 기능이 모든 차량에 동일하게 적용되지 않아 문제가 발생합니다.

```java
// Vehicle 클래스에 GPS 기능 추가
public class Vehicle {
    private String brand;
    private String model;
    private int year;
    private boolean hasGPS;

    public Vehicle(String brand, String model, int year, boolean hasGPS) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.hasGPS = hasGPS;
    }

    public void start() {
        System.out.println(brand + " " + model + " is starting.");
    }

    public void stop() {
        System.out.println(brand + " " + model + " is stopping.");
    }

    public void enableGPS() {
        if (hasGPS) {
            System.out.println("GPS is enabled.");
        } else {
            System.out.println("This vehicle does not have GPS.");
        }
    }
}

// Car 클래스에서 GPS 관련 코드가 불필요해질 수 있음
public class Car extends Vehicle {
    private int trunkCapacity;

    public Car(String brand, String model, int year, int trunkCapacity) {
        super(brand, model, year, true); // GPS를 무조건 사용해야 하는 구조
        this.trunkCapacity = trunkCapacity;
    }

    public void openTrunk() {
        System.out.println("Opening trunk with capacity: " + trunkCapacity + " liters.");
    }
}
```

이 코드에서는 `Vehicle` 클래스에 GPS 기능을 추가하면서, 모든 `Vehicle` 인스턴스에서 GPS 기능을 필수적으로 사용할 수 있게 되었습니다. 하지만, `Car`나 `Truck` 클래스에서 이 기능이 불필요한 경우도 있을 수 있으며, GPS가 없는 차량을 표현하기 어려워집니다.

</details>

<details>

<summary>컴포지션으로 문제 해결</summary>



이제, GPS 기능을 독립적인 컴포넌트로 분리하여 필요에 따라 조합할 수 있도록 컴포지션을 사용해 문제를 해결합니다.

```java
// GPS 클래스: GPS 기능을 별도의 컴포넌트로 정의
public class GPS {
    public void enableGPS() {
        System.out.println("GPS is enabled.");
    }

    public void disableGPS() {
        System.out.println("GPS is disabled.");
    }
}

// Vehicle 클래스: 기본적인 차량 기능만 포함
public class Vehicle {
    private String brand;
    private String model;
    private int year;

    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
    }

    public void start() {
        System.out.println(brand + " " + model + " is starting.");
    }

    public void stop() {
        System.out.println(brand + " " + model + " is stopping.");
    }
}

// Car 클래스: 필요 시 GPS 컴포넌트를 포함하는 방식으로 확장
public class Car {
    private Vehicle vehicle;
    private int trunkCapacity;
    private GPS gps;

    public Car(String brand, String model, int year, int trunkCapacity, GPS gps) {
        this.vehicle = new Vehicle(brand, model, year);
        this.trunkCapacity = trunkCapacity;
        this.gps = gps;
    }

    public void start() {
        vehicle.start();
    }

    public void stop() {
        vehicle.stop();
    }

    public void openTrunk() {
        System.out.println("Opening trunk with capacity: " + trunkCapacity + " liters.");
    }

    public void enableGPS() {
        if (gps != null) {
            gps.enableGPS();
        } else {
            System.out.println("This car does not have GPS.");
        }
    }
}

// Truck 클래스: GPS 사용 여부를 선택 가능
public class Truck {
    private Vehicle vehicle;
    private int loadCapacity;
    private GPS gps;

    public Truck(String brand, String model, int year, int loadCapacity, GPS gps) {
        this.vehicle = new Vehicle(brand, model, year);
        this.loadCapacity = loadCapacity;
        this.gps = gps;
    }

    public void start() {
        vehicle.start();
    }

    public void stop() {
        vehicle.stop();
    }

    public void haulLoad() {
        System.out.println("Hauling load with capacity: " + loadCapacity + " kg.");
    }

    public void enableGPS() {
        if (gps != null) {
            gps.enableGPS();
        } else {
            System.out.println("This truck does not have GPS.");
        }
    }
}
```

#### 코드 설명:

1. **GPS 컴포넌트**: `GPS` 클래스는 GPS 기능을 별도의 컴포넌트로 분리합니다. 이제 `Vehicle` 클래스와 상관없이, GPS가 필요한 차량에만 이 기능을 포함시킬 수 있습니다.
2. **Vehicle 클래스**: `Vehicle` 클래스는 기본적인 차량 기능만 포함하고, GPS 관련 기능은 포함하지 않습니다. 이는 `Vehicle` 클래스가 단순화되고, 다양한 기능을 필요로 하는 경우 확장 가능성을 높여줍니다.
3. **Car와 Truck 클래스**: 이 클래스들은 `Vehicle`을 포함하고, 필요에 따라 `GPS` 객체를 선택적으로 포함합니다. 이를 통해 GPS가 필요한 차량만 해당 기능을 가질 수 있으며, 불필요한 경우에는 이를 제외할 수 있습니다.

</details>

요약 2:

<details>

<summary>상속을 사용하는 예제</summary>

```java
// Shape 클래스: 도형의 기본 속성과 메서드를 정의
public class Shape {
    private String color;

    public Shape(String color) {
        this.color = color;
    }

    public String getColor() {
        return color;
    }

    public void draw() {
        System.out.println("Drawing a shape in color: " + color);
    }
}

// Rectangle 클래스: Shape를 상속받아 사각형을 표현
public class Rectangle extends Shape {
    private int width;
    private int height;

    public Rectangle(String color, int width, int height) {
        super(color);
        this.width = width;
        this.height = height;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a rectangle with width: " + width + ", height: " + height + ", in color: " + getColor());
    }

    public int getArea() {
        return width * height;
    }
}

// Circle 클래스: Shape를 상속받아 원을 표현
public class Circle extends Shape {
    private int radius;

    public Circle(String color, int radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a circle with radius: " + radius + ", in color: " + getColor());
    }

    public double getArea() {
        return Math.PI * radius * radius;
    }
}
```

이 예제에서 `Rectangle`과 `Circle` 클래스는 `Shape` 클래스를 상속받아 공통 속성인 `color`와 메서드 `draw()`를 재사용합니다.

</details>

<details>

<summary>상속을 사용하다가 발생하는 문제</summary>

시간이 지나면서 `Shape` 클래스에 추가적인 도형이 생기고, 특정 도형에만 적용되는 기능이 필요할 때 문제가 발생할 수 있습니다. 예를 들어, `Rectangle`과 `Circle` 클래스에만 특정한 메서드를 추가하려고 할 때, 상속 구조가 문제가 됩니다.

```java
// Shape 클래스에 새로운 메서드 추가 시 문제 발생
public class Shape {
    private String color;

    public Shape(String color) {
        this.color = color;
    }

    public String getColor() {
        return color;
    }

    public void draw() {
        System.out.println("Drawing a shape in color: " + color);
    }

    // 모든 도형에 둘레를 계산하는 메서드 추가 (Rectangle에만 적용될 수 있음)
    public double getPerimeter() {
        // 기본적으로 둘레를 계산할 수 없으므로 예외 발생
        throw new UnsupportedOperationException("Cannot calculate perimeter for generic shape");
    }
}

// Rectangle 클래스에 getPerimeter 메서드 오버라이드
public class Rectangle extends Shape {
    private int width;
    private int height;

    public Rectangle(String color, int width, int height) {
        super(color);
        this.width = width;
        this.height = height;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a rectangle with width: " + width + ", height: " + height + ", in color: " + getColor());
    }

    @Override
    public int getArea() {
        return width * height;
    }

    @Override
    public double getPerimeter() {
        return 2 * (width + height);
    }
}

// Circle 클래스에도 getPerimeter 메서드를 추가해야 하는 경우 문제 발생
public class Circle extends Shape {
    private int radius;

    public Circle(String color, int radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a circle with radius: " + radius + ", in color: " + getColor());
    }

    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }

    @Override
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }
}
```

위 코드에서 `Shape` 클래스에 `getPerimeter()` 메서드를 추가했지만, 이 메서드는 모든 도형에 적합하지 않을 수 있습니다. `Circle`과 `Rectangle` 클래스에 각각의 `getPerimeter()` 메서드를 오버라이드해야 하며, 다른 도형이 추가될 때마다 동일한 작업이 반복됩니다. 이로 인해 코드의 유지보수성이 떨어지고, 상속의 사용이 복잡해집니다.

</details>

<details>

<summary>컴포지션을 사용하여 문제 해결</summary>

이 문제를 해결하기 위해, 상속 대신 컴포지션을 사용하여 기능을 분리하고, 필요할 때만 필요한 기능을 추가할 수 있습니다.

```java
// Color와 Drawable 인터페이스로 기능 분리
public interface Colorable {
    String getColor();
}

public interface Drawable {
    void draw();
}

public interface PerimeterCalculable {
    double getPerimeter();
}

// Color 클래스로 색상 기능을 캡슐화
public class Color implements Colorable {
    private final String color;

    public Color(String color) {
        this.color = color;
    }

    @Override
    public String getColor() {
        return color;
    }
}

// Rectangle 클래스
public class Rectangle implements Drawable, PerimeterCalculable {
    private final Colorable color;
    private final int width;
    private final int height;

    public Rectangle(Colorable color, int width, int height) {
        this.color = color;
        this.width = width;
        this.height = height;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a rectangle with width: " + width + ", height: " + height + ", in color: " + color.getColor());
    }

    @Override
    public double getPerimeter() {
        return 2 * (width + height);
    }
}

// Circle 클래스
public class Circle implements Drawable, PerimeterCalculable {
    private final Colorable color;
    private final int radius;

    public Circle(Colorable color, int radius) {
        this.color = color;
        this.radius = radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a circle with radius: " + radius + ", in color: " + color.getColor());
    }

    @Override
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }
}
```

이 설계에서는 `Color` 클래스가 색상 관련 기능을 담당하고, `Drawable` 인터페이스가 그리기 기능을, `PerimeterCalculable` 인터페이스가 둘레 계산 기능을 제공합니다. 각각의 클래스는 필요한 기능만을 조합하여 사용하며, 상속 구조의 복잡성을 피할 수 있습니다.

</details>



## **1. 상속과 컴포지션의 개념 이해**

\*\*상속(Inheritance)\*\*은 부모 클래스의 속성과 메서드를 자식 클래스가 물려받는 구조입니다. 상속은 코드 재사용성을 높이고, 타입 계층 구조를 구성하는 데 유리합니다. 하지만 상속은 설계가 잘못될 경우, 코드의 변경이 어려워지고, 버그가 발생할 가능성을 높입니다.

\*\*컴포지션(Composition)\*\*은 객체가 다른 객체를 포함하여 재사용성을 높이는 방식입니다. 즉, "has-a" 관계를 나타냅니다. 컴포지션은 클래스 간 결합도를 낮추고, 코드의 유연성을 증가시킵니다. 이는 코드의 유지보수성에 큰 이점을 제공하며, 변경이 필요한 경우에도 상대적으로 용이하게 수정할 수 있습니다.



## **2. 상속의 문제점**

상속을 잘못 사용하면 다음과 같은 문제점들이 발생할 수 있습니다:

1. **강한 결합도(Tight Coupling)**: 자식 클래스는 부모 클래스에 강하게 결합됩니다. 부모 클래스의 구현이 변경되면, 자식 클래스도 영향을 받습니다.
2. **유연성 부족(Lack of Flexibility)**: 상속을 통해 기능을 확장하는 것은 한계가 있습니다. 여러 클래스에서 동일한 기능을 재사용해야 할 때, 중복 코드가 발생할 수 있습니다.
3. **캡슐화 위반(Violation of Encapsulation)**: 상속은 부모 클래스의 내부 구현을 자식 클래스에 노출시킵니다. 이는 객체지향 설계의 중요한 원칙인 캡슐화를 위반할 가능성이 있습니다.



## **3. 컴포지션의 장점**

컴포지션을 사용하면 다음과 같은 이점이 있습니다:

1. **낮은 결합도(Low Coupling)**: 클래스 간 결합도가 낮아져, 하나의 클래스가 변경되더라도 다른 클래스에 미치는 영향이 적습니다.
2. **높은 유연성(High Flexibility)**: 컴포지션은 객체 간 상호작용을 통해 기능을 동적으로 조합할 수 있어, 유연한 설계가 가능합니다.
3. **유지보수 용이성(Maintainability)**: 코드의 변경이 용이하며, 새로운 기능 추가가 상대적으로 간단합니다.



## **4. 상속과 컴포지션의 차이점을 Lombok을 활용한 예제로 설명**

이제 자바 스프링부트에서 Lombok 라이브러리를 사용하여 상속과 컴포지션의 차이점을 설명하는 예제를 살펴보겠습니다.

<details>

<summary><strong>상속을 사용한 코드 예제:</strong></summary>

```java
// 부모 클래스
public class Car {
    private String model;
    private String color;

    public void drive() {
        System.out.println("Driving " + model + " in " + color + " color.");
    }
}

// 자식 클래스
public class ElectricCar extends Car {
    private int batteryCapacity;

    public void chargeBattery() {
        System.out.println("Charging battery to " + batteryCapacity + "%");
    }
}
```

이 예제에서 `ElectricCar` 클래스는 `Car` 클래스를 상속받아 차량의 기본적인 속성(`model`, `color`)을 활용할 수 있습니다. 그러나, 만약 `Car` 클래스의 내부 구현이 변경되면 `ElectricCar` 클래스에도 영향을 미칠 수 있습니다.

</details>

<details>

<summary><strong>컴포지션을 사용한 코드 예제:</strong></summary>

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Car {
    private String model;
    private String color;

    public void drive() {
        System.out.println("Driving " + model + " in " + color + " color.");
    }
}

@Getter
@Setter
public class Battery {
    private int capacity;

    public void charge() {
        System.out.println("Charging battery to " + capacity + "%");
    }
}

public class ElectricCar {
    private final Car car;
    private final Battery battery;

    public ElectricCar(Car car, Battery battery) {
        this.car = car;
        this.battery = battery;
    }

    public void drive() {
        car.drive();
    }

    public void chargeBattery() {
        battery.charge();
    }
}
```

컴포지션을 사용한 이 예제에서는 `ElectricCar`가 `Car`와 `Battery` 객체를 포함하고 있습니다. `ElectricCar` 클래스는 더 이상 `Car` 클래스를 상속받지 않고, 필요한 기능을 컴포지션을 통해 조합합니다. 이는 클래스 간 결합도를 낮추고, 코드의 유연성과 재사용성을 높이는 방법입니다.

</details>



## **5. 컴포지션의 활용 시 고려할 점**

컴포지션을 사용하면 많은 장점이 있지만, 몇 가지 고려해야 할 사항도 있습니다:

1. **설계의 복잡성(Complexity)**: 컴포지션을 사용하면 설계가 복잡해질 수 있습니다. 클래스가 서로 어떻게 상호작용할지 신중하게 설계해야 합니다.
2. **불필요한 오버헤드(Overhead)**: 컴포지션은 더 많은 객체 생성과 메서드 호출을 수반할 수 있어, 성능에 민감한 시스템에서는 주의가 필요합니다.



## **6. 컴포지션을 위한 디자인 패턴**

컴포지션을 효율적으로 활용하기 위해 다양한 디자인 패턴이 사용됩니다. 대표적인 패턴으로는 다음과 같은 것들이 있습니다:

1. **전략 패턴(Strategy Pattern)**: 동작을 클래스로 캡슐화하여 동적으로 행동을 변경할 수 있게 합니다.
2. **데코레이터 패턴(Decorator Pattern)**: 객체에 새로운 행동을 동적으로 추가할 수 있습니다.
3. **어댑터 패턴(Adapter Pattern)**: 클래스의 인터페이스를 다른 인터페이스로 변환하여 호환성을 유지할 수 있습니다.

## **7. 스프링부트와 Lombok을 활용한 컴포지션 예제 확장**

스프링부트와 Lombok을 활용하면 컴포지션 기반의 코드를 더 효율적으로 작성할 수 있습니다. Lombok은 보일러플레이트 코드를 줄여주며, 스프링부트는 의존성 주입을 통해 컴포지션을 자연스럽게 구현할 수 있게 해줍니다.

예를 들어, `@Autowired`를 활용하여 의존성을 주입받는 방식으로 컴포지션을 구현할 수 있습니다:

<details>

<summary>public class ElectricCar</summary>

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ElectricCar {

    private final Car car;
    private final Battery battery;

    @Autowired
    public ElectricCar(Car car, Battery battery) {
        this.car = car;
        this.battery = battery;
    }

    public void drive() {
        car.drive();
    }

    public void chargeBattery() {
        battery.charge();
    }
}
```

이 코드에서는 스프링의 의존성 주입을 통해 `ElectricCar`가 `Car`와 `Battery` 객체를 컴포지션으로 사용할 수 있습니다. 이는 스프링부트의 강력한 DI(Dependency Injection) 기능을 활용한 예시로, 컴포지션 설계를 쉽게 구현할 수 있게 합니다.

</details>



## 8. 상속과 컴포지션의 실무적 적용 사례

### **1. 상속의 실무 적용과 문제점**

상속은 실무에서 코드 재사용과 계층 구조 설계에 자주 사용됩니다. 예를 들어, 공통적인 기능을 여러 클래스에 제공하기 위해 상위 클래스에서 정의한 다음, 하위 클래스에서 이를 상속받아 사용하는 방식입니다.

하지만, 이러한 상속 구조는 시간이 지나면서 다음과 같은 문제를 초래할 수 있습니다:

1. **비정상적인 상속 계층 구조**: 실무에서 요구 사항이 변화하면, 기존의 상속 계층이 비정상적으로 확장될 수 있습니다. 이로 인해 하위 클래스들이 상위 클래스의 불필요한 기능까지 상속받게 되어 불필요한 복잡성이 증가합니다.
2. **테스트 및 유지보수의 어려움**: 상속 구조가 깊어질수록, 한 부분의 수정이 전체 클래스 계층에 영향을 미칠 수 있어 테스트와 유지보수가 어려워집니다. 이는 특히 대규모 프로젝트에서 치명적인 문제로 작용할 수 있습니다.

예를 들어, 상속을 사용해 `Employee` 클래스를 만들고, 이를 상속받아 `Manager`와 `Engineer` 클래스를 생성한다고 가정합니다. 시간이 지나면서 `Engineer` 클래스가 다양한 엔지니어 타입으로 세분화되어, 상속 계층이 점점 복잡해질 수 있습니다. 결국, `Manager`와 `Engineer`는 본질적으로 다른 역할을 수행하지만, 동일한 상위 클래스를 상속받아 설계의 유연성이 저하됩니다.

<details>

<summary>public class Employee<br>public class Manager extends Employee<br>public class Engineer extends Employee</summary>

```java
public class Employee {
    private String name;
    private String department;
    private double salary;

    public void work() {
        System.out.println(name + " is working in " + department);
    }
}

public class Manager extends Employee {
    private int teamSize;

    public void manage() {
        System.out.println("Managing a team of " + teamSize);
    }
}

public class Engineer extends Employee {
    private String specialization;

    public void develop() {
        System.out.println("Developing in the field of " + specialization);
    }
}
```

이러한 구조는 처음에는 간단해 보이지만, 팀 구조나 업무 분담이 변하면서 상속 계층이 복잡해지고, 코드의 유지보수성도 떨어질 수 있습니다.

</details>

### **2. 컴포지션의 실무 적용과 이점**

컴포지션은 이러한 문제를 해결하는 데 매우 유용합니다. 앞서 언급한 문제를 해결하기 위해, `Employee` 클래스를 상속받는 대신, `Manager`와 `Engineer` 클래스를 별도의 클래스들로 분리하고, 필요한 기능을 다른 클래스들로부터 조합하는 방식으로 설계를 변경할 수 있습니다.

예를 들어, 관리 기능을 담당하는 `Management` 클래스를 별도로 두고, 이를 컴포지션으로 `Manager` 클래스에서 사용하도록 설계할 수 있습니다:

<details>

<summary>예시 코드</summary>

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Employee {
    private String name;
    private String department;
    private double salary;

    public void work() {
        System.out.println(name + " is working in " + department);
    }
}

@Getter
@Setter
public class Management {
    private int teamSize;

    public void manage() {
        System.out.println("Managing a team of " + teamSize);
    }
}

public class Manager {
    private final Employee employee;
    private final Management management;

    public Manager(Employee employee, Management management) {
        this.employee = employee;
        this.management = management;
    }

    public void manageTeam() {
        management.manage();
    }
}
```

이 설계는 상속의 단점을 피하고, 컴포지션을 활용하여 유연한 구조를 제공합니다. 또한, `Engineer` 클래스는 별도로 개발 기능을 담당하는 클래스를 컴포지션으로 활용할 수 있으며, 코드의 중복 없이 필요한 기능을 조합할 수 있습니다.

</details>

### **3. 실무에서의 적용 시 고려사항**

컴포지션을 도입할 때 주의해야 할 점은 다음과 같습니다:

1. **객체의 책임 분리**: 각 클래스가 명확한 책임을 가지도록 설계해야 합니다. 객체가 여러 역할을 맡게 되면, 컴포지션의 장점이 약화될 수 있습니다.
2. **인터페이스의 활용**: 인터페이스를 통해 컴포지션된 객체들이 서로 상호작용할 수 있도록 설계하면, 구현체를 변경하거나 확장할 때 더 유연한 대응이 가능합니다.
3. **테스트의 용이성**: 컴포지션을 사용하면, 각 컴포넌트를 독립적으로 테스트할 수 있어 테스트의 용이성이 증가합니다. 이는 특히 단위 테스트를 작성할 때 유리합니다.



## **9. 스프링부트에서의 컴포지션 활용**

스프링부트는 DI(Dependency Injection) 컨테이너를 통해 컴포지션을 자연스럽게 구현할 수 있는 환경을 제공합니다. 스프링의 의존성 주입 기능을 사용하면, 객체 간의 결합도를 낮추고, 애플리케이션의 구성 요소를 유연하게 조립할 수 있습니다.

예를 들어, 다양한 로그 기록 방식(LogStrategy)을 지원하는 애플리케이션을 설계한다고 가정합니다. 로그 기록 방식은 파일, 데이터베이스, 콘솔 등 다양한 방식이 있을 수 있습니다. 이를 컴포지션을 통해 스프링부트에서 유연하게 구현할 수 있습니다.

<details>

<summary>예시 코드</summary>

```java
public interface LogStrategy {
    void log(String message);
}

@Component
public class FileLogStrategy implements LogStrategy {
    @Override
    public void log(String message) {
        System.out.println("Logging to file: " + message);
    }
}

@Component
public class DatabaseLogStrategy implements LogStrategy {
    @Override
    public void log(String message) {
        System.out.println("Logging to database: " + message);
    }
}

@Component
public class ConsoleLogStrategy implements LogStrategy {
    @Override
    public void log(String message) {
        System.out.println("Logging to console: " + message);
    }
}

@Component
public class LogService {

    private final LogStrategy logStrategy;

    @Autowired
    public LogService(@Qualifier("fileLogStrategy") LogStrategy logStrategy) {
        this.logStrategy = logStrategy;
    }

    public void log(String message) {
        logStrategy.log(message);
    }
}
```

이 예제에서는 `LogService`가 `LogStrategy` 인터페이스를 의존성으로 받아들여, 특정 로그 전략에 따라 다르게 동작하도록 설정했습니다. 이는 스프링부트의 DI와 컴포지션을 활용한 좋은 예로, 코드의 유연성과 재사용성을 높여줍니다.

</details>



## **10. Lombok을 활용한 컴포지션 최적화**

Lombok은 자바에서 보일러플레이트 코드를 제거해주는 강력한 도구입니다. 특히 컴포지션을 활용할 때, Lombok의 `@Getter`, `@Setter`, `@RequiredArgsConstructor` 등의 애노테이션을 사용하면 코드가 훨씬 간결해집니다.

<details>

<summary>예시 코드</summary>

```java
import lombok.RequiredArgsConstructor;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@RequiredArgsConstructor
public class Employee {
    private final String name;
    private final String department;
    private double salary;

    public void work() {
        System.out.println(name + " is working in " + department);
    }
}

@Getter
@Setter
@RequiredArgsConstructor
public class Manager {
    private final Employee employee;
    private final Management management;

    public void manageTeam() {
        management.manage();
    }
}
```

이 코드에서는 Lombok을 사용하여 생성자와 게터, 세터를 자동으로 생성함으로써 코드의 가독성과 유지보수성을 높였습니다. Lombok을 활용하면 컴포지션 설계 시 발생하는 반복적인 코드를 줄일 수 있어, 개발 효율성이 크게 향상됩니다.

</details>



## **11. 결론 및 요약**

상속은 간단하고 직관적인 코드 재사용 방법이지만, 장기적으로는 코드의 유연성과 유지보수성을 해칠 수 있습니다. 반면, 컴포지션은 보다 유연하고 확장 가능한 코드를 작성할 수 있는 강력한 도구입니다. 특히, 스프링부트와 Lombok과 같은 도구를 활용하면, 컴포지션 설계를 더욱 효과적으로 구현할 수 있습니다.
