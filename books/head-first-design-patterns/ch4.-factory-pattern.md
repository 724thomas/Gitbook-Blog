---
description: 팩토리 패턴
---

# Ch4. Factory Pattern

## **최첨단 피자 코드 만들기 (Creating Advanced Pizza Code)**

피자 주문 시스템을 구축하기 위해 가장 먼저 해야 할 일은 피자 객체를 생성하는 코드 작성입니다. 일반적인 피자 클래스는 다양한 속성을 가질 수 있습니다. 예를 들어, 피자의 종류, 크기, 토핑 등이 포함될 수 있습니다.

<details>

<summary>class Pizza</summary>

```java
public class Pizza {
    private String name;
    private String dough;
    private String sauce;
    private List<String> toppings;

    // Constructor, getters, and setters
}
```

위 코드에서는 `Pizza` 클래스가 기본적인 피자의 속성을 담고 있습니다. 이 클래스는 이후 팩토리 패턴을 활용하여 다양한 피자 객체를 생성하는 기반이 됩니다.

</details>

### **피자 코드 추가하기 (Adding Pizza Code)**

이제 `Pizza` 클래스에 다양한 종류의 피자를 추가할 수 있도록 코드를 확장해 보겠습니다. 예를 들어, `MargheritaPizza`와 `PepperoniPizza` 같은 서브클래스를 추가하여 피자 종류별로 클래스를 정의할 수 있습니다.

<details>

<summary>class MargheritaPizza, PepperoniPizza</summary>

```java
public class MargheritaPizza extends Pizza {
    public MargheritaPizza() {
        name = "Margherita Pizza";
        dough = "Thin crust";
        sauce = "Tomato basil";
        toppings.add("Fresh Mozzarella");
        toppings.add("Parmesan");
    }
}

public class PepperoniPizza extends Pizza {
    public PepperoniPizza() {
        name = "Pepperoni Pizza";
        dough = "Regular crust";
        sauce = "Marinara sauce";
        toppings.add("Mozzarella");
        toppings.add("Pepperoni");
    }
}
```

이처럼 각 피자에 맞는 속성을 정의함으로써 코드의 확장성을 높일 수 있습니다.

</details>

### **객체 생성 부분 캡슐화하기 (Encapsulating Object Creation)**

위와 같이 서브클래스를 이용해 다양한 피자를 정의했지만, 객체 생성 부분이 여전히 코드 곳곳에 흩어져 있습니다. 이를 캡슐화하기 위해 팩토리 패턴을 적용할 수 있습니다. 팩토리 패턴을 사용하면 객체 생성 로직을 한 곳에 집중시켜 관리하기가 용이해집니다.

<details>

<summary>class PizzaFactory</summary>

```java
public class PizzaFactory {
    public static Pizza createPizza(String type) {
        Pizza pizza = null;
        if (type.equals("Margherita")) {
            pizza = new MargheritaPizza();
        } else if (type.equals("Pepperoni")) {
            pizza = new PepperoniPizza();
        }
        return pizza;
    }
}
```

이제 클라이언트 코드는 단순히 `PizzaFactory.createPizza("Margherita")`를 호출하기만 하면 `MargheritaPizza` 객체를 생성할 수 있습니다. 이렇게 객체 생성 부분을 캡슐화함으로써 코드의 유지 보수성과 가독성을 크게 향상시킬 수 있습니다.

</details>

### **객체 생성 팩토리 만들기 (Creating Object Creation Factory)**

위에서 구현한 `PizzaFactory` 클래스는 가장 기본적인 팩토리 패턴의 예시입니다. 실제로 팩토리 패턴은 다양한 방식으로 확장될 수 있습니다. 예를 들어, 팩토리 메서드를 인터페이스나 추상 클래스로 정의하여 더욱 유연한 설계가 가능합니다.

<details>

<summary>abstract class PizzaFactory</summary>

```java
public abstract class PizzaFactory {
    public abstract Pizza createPizza(String type);
}
```

이렇게 함으로써 다양한 팩토리 구현체를 만들 수 있으며, 팩토리 메서드를 통해 생성 로직을 더욱 세분화할 수 있습니다.

</details>

### **클라이언트 코드 수정하기 (Modifying Client Code)**

팩토리 패턴을 적용한 후 클라이언트 코드는 다음과 같이 단순화됩니다. 클라이언트는 더 이상 피자 생성에 대한 세부사항을 알 필요 없이, 팩토리의 메서드를 호출하기만 하면 됩니다.

<details>

<summary>class PizzaStore</summary>

```java
public class PizzaStore {
    private PizzaFactory pizzaFactory;

    public PizzaStore(PizzaFactory factory) {
        this.pizzaFactory = factory;
    }

    public Pizza orderPizza(String type) {
        Pizza pizza = pizzaFactory.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```

이렇게 클라이언트 코드를 수정하면 객체 생성 로직이 완전히 분리되어 코드의 응집도와 유지 보수성이 크게 향상됩니다.

</details>



## **간단한 팩토리의 정의 (Defining a Simple Factory)**

간단한 팩토리(Simple Factory)는 특정 클래스나 메서드에서 객체 생성을 담당하는 디자인 패턴입니다. 간단한 팩토리는 팩토리 메서드 패턴과는 달리 별도의 인터페이스나 추상 클래스를 필요로 하지 않습니다. 주로 정적인 메서드를 사용하여 객체를 생성합니다.

<details>

<summary>class SimplePizzaFactory</summary>

```java
public class SimplePizzaFactory {
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        if (type.equals("Margherita")) {
            pizza = new MargheritaPizza();
        } else if (type.equals("Pepperoni")) {
            pizza = new PepperoniPizza();
        }
        return pizza;
    }
}
```

위의 `SimplePizzaFactory` 클래스는 단일 메서드를 통해 피자 객체를 생성합니다. 이 방법은 코드가 간단하고 직관적이지만, 객체 생성 로직이 복잡해질 경우 유지 보수에 어려움이 생길 수 있습니다.

</details>



### **다양한 팩토리 만들기 (Creating Various Factories)**

앞서 설명한 간단한 팩토리는 기본적인 객체 생성을 처리하기에는 충분하지만, 더 복잡한 시나리오에서는 문제가 될 수 있습니다. 이를 해결하기 위해 팩토리 메서드 패턴(Factory Method Pattern)을 활용할 수 있습니다. 이 패턴은 객체 생성을 서브클래스에서 처리하도록 위임하여 확장성과 유연성을 높입니다.

<details>

<summary>abstract class PizzaFactory</summary>

```java
public abstract class PizzaFactory {
    public abstract Pizza createPizza(String type);
}

public class NYPizzaFactory extends PizzaFactory {
    @Override
    public Pizza createPizza(String type) {
        if (type.equals("Cheese")) {
            return new NYStyleCheesePizza();
        } else if (type.equals("Veggie")) {
            return new NYStyleVeggiePizza();
        } else {
            return null;
        }
    }
}

public class ChicagoPizzaFactory extends PizzaFactory {
    @Override
    public Pizza createPizza(String type) {
        if (type.equals("Cheese")) {
            return new ChicagoStyleCheesePizza();
        } else if (type.equals("Veggie")) {
            return new ChicagoStyleVeggiePizza();
        } else {
            return null;
        }
    }
}
```

위 코드에서 `NYPizzaFactory`와 `ChicagoPizzaFactory`는 각각 뉴욕 스타일과 시카고 스타일의 피자를 생성하는 팩토리입니다. 이와 같이 다양한 팩토리를 구현함으로써 코드의 유연성을 크게 높일 수 있습니다.

</details>



### **피자 가게 프레임워크 만들기 (Building a Pizza Store Framework)**

다양한 팩토리를 사용해 피자 가게 프레임워크를 설계할 수 있습니다. 이 프레임워크는 고객이 피자를 주문하면 해당 지역에 맞는 팩토리를 사용하여 피자를 생성하는 구조로 설계됩니다.

<details>

<summary>class PizzaStore</summary>

```java
public class PizzaStore {
    private PizzaFactory pizzaFactory;

    public PizzaStore(PizzaFactory factory) {
        this.pizzaFactory = factory;
    }

    public Pizza orderPizza(String type) {
        Pizza pizza = pizzaFactory.createPizza(type);
        if (pizza != null) {
            pizza.prepare();
            pizza.bake();
            pizza.cut();
            pizza.box();
        }
        return pizza;
    }
}
```

`PizzaStore` 클래스는 다양한 팩토리 객체를 주입받아, 그에 따라 다른 종류의 피자를 생성합니다. 이 방식은 지역별로 다른 피자 스타일을 쉽게 추가하거나 수정할 수 있는 장점이 있습니다.

</details>



### **서브클래스가 결정하는 것 알아보기 (Understanding What Subclasses Decide)**

팩토리 메서드 패턴에서 중요한 점은 객체 생성의 결정을 서브클래스가 내린다는 것입니다. 이는 객체 생성의 유연성을 극대화하고, 코드의 재사용성을 높이는 중요한 요소입니다. 예를 들어, 각 지역의 피자 스타일을 책임지는 서브클래스들은 각자의 로직에 따라 피자를 생성할 수 있습니다.

### **피자 스타일 서브클래스 만들기 (Creating Pizza Style Subclasses)**

이제 다양한 피자 스타일을 지원하는 서브클래스를 추가해보겠습니다. 예를 들어, 뉴욕 스타일과 시카고 스타일의 피자를 각각 서브클래스로 정의할 수 있습니다.

<details>

<summary>class NYStyleCheesePizza, ChicagoStyleCheesePizza extends Pizza</summary>

```java
public class NYStyleCheesePizza extends Pizza {
    public NYStyleCheesePizza() {
        name = "NY Style Sauce and Cheese Pizza";
        dough = "Thin Crust Dough";
        sauce = "Marinara Sauce";
        toppings.add("Grated Reggiano Cheese");
    }
}

public class ChicagoStyleCheesePizza extends Pizza {
    public ChicagoStyleCheesePizza() {
        name = "Chicago Style Deep Dish Cheese Pizza";
        dough = "Extra Thick Crust Dough";
        sauce = "Plum Tomato Sauce";
        toppings.add("Shredded Mozzarella Cheese");
    }
}
```

각 서브클래스는 특정 피자 스타일에 맞게 정의되며, 이로 인해 피자 가게는 고객의 요청에 따라 올바른 스타일의 피자를 제공할 수 있습니다.

</details>



### **팩토리 메소드 선언하기 (Declaring Factory Methods)**

이제 팩토리 메서드 패턴을 완성하기 위해 각 서브클래스에서 팩토리 메서드를 선언합니다. 이 메서드들은 피자 스타일에 맞는 피자를 생성하도록 설계됩니다.

<details>

<summary>abstract class PizzaStore</summary>

```java
public abstract class PizzaStore {
    protected abstract Pizza createPizza(String type);

    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```

이 패턴에서는 `PizzaStore` 클래스가 객체 생성을 위한 추상 메서드를 제공하고, 구체적인 생성 로직은 서브클래스에서 구현됩니다.

</details>



### **팩토리 메서드로 피자 주문하기 (Ordering Pizza Using Factory Methods)**

팩토리 메서드를 사용하면 피자 주문 프로세스가 더욱 효율적으로 관리됩니다. 예를 들어, 뉴욕 스타일 피자 가게와 시카고 스타일 피자 가게를 각각 구현할 수 있습니다.

<details>

<summary>class NYPizzaStore, ChicagoPizzaStore extends PizzaStore</summary>

```java
public class NYPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("Cheese")) {
            return new NYStyleCheesePizza();
        } else if (type.equals("Veggie")) {
            return new NYStyleVeggiePizza();
        } else {
            return null;
        }
    }
}

public class ChicagoPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("Cheese")) {
            return new ChicagoStyleCheesePizza();
        } else if (type.equals("Veggie")) {
            return new ChicagoStyleVeggiePizza();
        } else {
            return null;
        }
    }
}
```

이처럼 팩토리 메서드를 통해 피자 주문 시 스타일에 맞는 피자를 자동으로 선택하고 생성할 수 있습니다.

</details>



### **피자가 만들어지기까지 (From Pizza Order to Creation)**

팩토리 메서드를 통해 피자가 만들어지기까지의 과정은 다음과 같습니다. 클라이언트가 `orderPizza` 메서드를 호출하면, 적절한 팩토리 메서드가 호출되어 피자가 생성됩니다. 이 후, 생성된 피자는 준비, 조리, 자르기, 포장 등의 과정을 거치게 됩니다.

이 과정을 통해 클라이언트는 특정 피자 생성 방식에 대해 알 필요 없이, 피자 객체를 얻을 수 있습니다. 팩토리 패턴을 통해 객체 생성 과정을 캡슐화하고, 코드의 결합도를 낮출 수 있습니다.



## **팩토리 메서드 패턴 살펴보기 (Exploring the Factory Method Pattern)**

팩토리 메서드 패턴(Factory Method Pattern)은 객체 생성의 책임을 서브클래스로 위임하여, 객체 생성 과정에서 생기는 결합도를 낮추는 디자인 패턴입니다. 이를 통해 클라이언트는 구체적인 클래스의 인스턴스를 직접 생성하지 않고도 객체를 생성할 수 있습니다.

예를 들어, `NYPizzaStore`와 `ChicagoPizzaStore` 클래스에서 각각 `createPizza` 메서드를 오버라이드하여, 구체적인 피자 객체를 생성하게 했습니다. 이 패턴을 사용하면 새로운 피자 스타일이 추가되더라도 기존 코드를 수정할 필요가 없어집니다. 이는 객체 생성 과정에서의 유연성과 확장성을 극대화할 수 있는 방법입니다.

### **병렬 클래스 계층 구조 알아보기 (Understanding Parallel Class Hierarchies)**

팩토리 메서드 패턴을 활용하는 경우, 병렬 클래스 계층 구조(Parallel Class Hierarchies)를 만드는 상황이 발생할 수 있습니다. 예를 들어, 피자 클래스와 피자 팩토리 클래스가 각각 별도의 계층 구조를 형성할 수 있습니다. 이러한 병렬 구조는 각 클래스가 자신의 역할에 맞게 책임을 분담함으로써 시스템의 유연성과 유지 보수성을 강화합니다.

<details>

<summary>abstract class Pizza</summary>

```java
public abstract class Pizza { 
    String name;
    String dough;
    String sauce;
    List<String> toppings = new ArrayList<>();

    public void prepare() {
        System.out.println("Preparing " + name);
    }
    public void bake() {
        System.out.println("Baking " + name);
    }
    public void cut() {
        System.out.println("Cutting " + name);
    }
    public void box() {
        System.out.println("Boxing " + name);
    }

    public String getName() {
        return name;
    }
}

public abstract class PizzaStore {
    protected abstract Pizza createPizza(String type);

    public Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);
        System.out.println("--- Making a " + pizza.getName() + " ---");
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```

위 코드에서는 `Pizza` 클래스와 `PizzaStore` 클래스가 각각 독립적인 계층 구조를 형성하고 있습니다. `PizzaStore`는 팩토리 메서드를 통해 `Pizza` 객체를 생성하고, 이를 조작합니다. 이는 병렬 클래스 계층 구조의 대표적인 예입니다.

</details>



### **팩토리 메서드 패턴의 정의 (Defining the Factory Method Pattern)**

팩토리 메서드 패턴은 객체 지향 디자인에서 매우 중요한 패턴 중 하나로, 다음과 같은 특징을 가지고 있습니다:

1. **서브클래스에게 객체 생성 책임을 위임**: 객체 생성의 구체적인 과정을 서브클래스가 담당하게 하여, 클라이언트는 객체 생성의 복잡성을 신경 쓰지 않아도 됩니다.
2. **코드의 유연성 증가**: 새로운 객체를 추가할 때, 기존 코드를 수정할 필요 없이 새로운 서브클래스를 추가함으로써 확장성을 유지할 수 있습니다.
3. **객체 생성의 통제**: 팩토리 메서드를 통해 객체 생성 과정을 중앙에서 통제함으로써, 객체 생성을 보다 효율적으로 관리할 수 있습니다.

### **객체 의존성 살펴보기 (Examining Object Dependencies)**

팩토리 패턴에서 중요한 점은 객체 간의 의존성을 어떻게 관리하느냐입니다. 객체 간의 결합도가 높아지면 시스템의 유지 보수성이 떨어지고, 코드 수정 시 많은 부분에 영향을 미칠 수 있습니다. 팩토리 패턴은 이러한 의존성을 줄이기 위해 유용한 방법을 제공합니다.

예를 들어, 클라이언트가 특정 피자 클래스에 직접 의존하지 않도록 팩토리 패턴을 사용하여 피자 객체를 생성하는 경우, 클라이언트는 `Pizza` 클래스의 서브클래스에 대한 지식 없이도 객체를 생성할 수 있습니다. 이를 통해 클라이언트와 피자 클래스 간의 결합도를 낮추고, 코드의 유연성을 높일 수 있습니다.

### **의존성 뒤집기 원칙 (Dependency Inversion Principle)**

의존성 뒤집기 원칙(Dependency Inversion Principle, DIP)은 객체 지향 설계에서 매우 중요한 원칙입니다. 이 원칙에 따르면, 고수준 모듈(High-level modules)은 저수준 모듈(Low-level modules)에 의존해서는 안 됩니다. 대신, 두 모듈 모두 추상화된 인터페이스에 의존해야 합니다.

<details>

<summary>interface Pizza</summary>

```java
public interface Pizza {
    void prepare();
    void bake();
    void cut();
    void box();
}

public class MargheritaPizza implements Pizza {
    @Override
    public void prepare() {
        System.out.println("Preparing Margherita Pizza");
    }
    // 기타 메서드 구현...
}

public class PizzaStore {
    private Pizza pizza;

    public PizzaStore(Pizza pizza) {
        this.pizza = pizza;
    }

    public void orderPizza() {
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
    }
}
```

위 코드에서 `PizzaStore`는 구체적인 `Pizza` 클래스가 아닌 `Pizza` 인터페이스에 의존합니다. 이는 의존성 뒤집기 원칙을 적용한 예로, `PizzaStore`는 어떤 구체적인 피자 클래스가 들어와도 상관없이 피자를 조작할 수 있습니다.

</details>



### **의존성 뒤집기 원칙 적용하기 (Applying the Dependency Inversion Principle)**

의존성 뒤집기 원칙을 적용하면 코드의 유연성과 재사용성이 크게 향상됩니다. 특히 팩토리 패턴과 결합하여 사용하면, 객체 생성과 의존성 관리가 더욱 효과적으로 이루어집니다.

<details>

<summary>class NYPizzaFactory extends PizzaFactory</summary>

```java
public class NYPizzaFactory extends PizzaFactory {
    @Override
    public Pizza createPizza(String type) {
        if (type.equals("Cheese")) {
            return new NYStyleCheesePizza();
        } else if (type.equals("Veggie")) {
            return new NYStyleVeggiePizza();
        } else {
            return null;
        }
    }
}
```

위 코드에서 `PizzaStore`는 `PizzaFactory` 인터페이스를 통해 객체를 생성합니다. 이로 인해 `PizzaStore`는 구체적인 팩토리 구현체에 의존하지 않으며, 새로운 팩토리 구현체가 추가되더라도 `PizzaStore` 코드를 수정할 필요가 없습니다.

</details>



### **의존성 뒤집기 원칙을 지키는 방법 (How to Maintain the Dependency Inversion Principle)**

의존성 뒤집기 원칙을 지키기 위해서는 다음과 같은 전략을 사용할 수 있습니다:

1. **인터페이스 도입**: 고수준 모듈이 저수준 모듈에 직접 의존하지 않도록 인터페이스나 추상 클래스를 도입합니다.
2. **팩토리 패턴 적용**: 객체 생성을 팩토리 패턴을 통해 관리함으로써, 고수준 모듈이 구체적인 객체 생성 과정에 대한 지식을 가지지 않도록 합니다.
3. **의존성 주입**: 의존성을 외부에서 주입받음으로써 객체 간 결합도를 낮춥니다.

이러한 방법을 통해 시스템의 유연성과 유지 보수성을 유지할 수 있습니다.



## **원재료 종류 알아보기 (Exploring Ingredient Types)**

피자 제조 과정에서 다양한 재료가 필요합니다. 도우, 소스, 치즈 등 피자마다 사용하는 원재료가 다를 수 있습니다. 이러한 원재료들을 객체로 표현하고, 이를 조합하는 과정이 복잡해질 수 있습니다. 예를 들어, 뉴욕 스타일 피자와 시카고 스타일 피자는 서로 다른 도우, 소스, 치즈를 사용합니다.

```java
public interface Dough {}
public interface Sauce {}
public interface Cheese {}

public class ThinCrustDough implements Dough {}
public class ThickCrustDough implements Dough {}

public class MarinaraSauce implements Sauce {}
public class PlumTomatoSauce implements Sauce {}

public class ReggianoCheese implements Cheese {}
public class MozzarellaCheese implements Cheese {}
```

위와 같이 원재료를 인터페이스로 정의하고, 각 스타일에 맞는 구체적인 클래스를 구현할 수 있습니다. 이렇게 함으로써 피자의 구성 요소를 분리하여 코드의 유연성과 재사용성을 높일 수 있습니다.

### **원재료군으로 묶기 (Grouping Ingredients)**

원재료들이 다양해지면서, 이를 효율적으로 관리할 필요성이 생깁니다. 원재료들을 하나의 그룹으로 묶어 관리하는 방법을 생각해 볼 수 있습니다. 이는 원재료 팩토리(Abstract Factory) 패턴을 활용하여 구현할 수 있습니다. 이 패턴을 사용하면 관련된 객체들을 하나의 팩토리에서 생성하여, 일관된 구성 요소를 제공할 수 있습니다.

<details>

<summary>interface PizzaIngredientFactory <br>class NYPizzaIngredientFactory, chicagoPizzaIngredientFactory implements PizzaIngredientFactory</summary>

```java
public interface PizzaIngredientFactory {
    Dough createDough();
    Sauce createSauce();
    Cheese createCheese();
}

public class NYPizzaIngredientFactory implements PizzaIngredientFactory {
    @Override
    public Dough createDough() {
        return new ThinCrustDough();
    }

    @Override
    public Sauce createSauce() {
        return new MarinaraSauce();
    }

    @Override
    public Cheese createCheese() {
        return new ReggianoCheese();
    }
}

public class ChicagoPizzaIngredientFactory implements PizzaIngredientFactory {
    @Override
    public Dough createDough() {
        return new ThickCrustDough();
    }

    @Override
    public Sauce createSauce() {
        return new PlumTomatoSauce();
    }

    @Override
    public Cheese createCheese() {
        return new MozzarellaCheese();
    }
}
```

위 코드에서 `PizzaIngredientFactory` 인터페이스와 그 구현체들인 `NYPizzaIngredientFactory`와 `ChicagoPizzaIngredientFactory`는 피자 스타일에 맞는 원재료를 제공합니다. 이를 통해 피자 구성 요소의 일관성을 유지하고, 재사용 가능성을 높일 수 있습니다.

</details>



### **원재료 팩토리 만들기 (Creating Ingredient Factories)**

원재료 팩토리 패턴을 사용하면 피자 객체 생성 시 일관된 원재료를 사용하여 피자를 조립할 수 있습니다. 이를 위해 피자 클래스가 원재료 팩토리를 이용해 필요한 구성 요소를 생성하도록 설계할 수 있습니다.

<details>

<summary>abstract class Pizza<br>class CheesePizza extends Pizza</summary>

<pre class="language-java"><code class="lang-java"><strong>public abstract class Pizza {
</strong>    String name;
    Dough dough;
    Sauce sauce;
    Cheese cheese;

    abstract void prepare();

    public void bake() {
        System.out.println("Baking " + name);
    }

    public void cut() {
        System.out.println("Cutting " + name);
    }

    public void box() {
        System.out.println("Boxing " + name);
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

public class CheesePizza extends Pizza {
    PizzaIngredientFactory ingredientFactory;

    public CheesePizza(PizzaIngredientFactory ingredientFactory) {
        this.ingredientFactory = ingredientFactory;
    }

    @Override
    void prepare() {
        System.out.println("Preparing " + name);
        dough = ingredientFactory.createDough();
        sauce = ingredientFactory.createSauce();
        cheese = ingredientFactory.createCheese();
    }
}
</code></pre>

이제 `CheesePizza` 클래스는 원재료 팩토리를 이용하여 피자 제작에 필요한 모든 재료를 준비합니다. 이로 인해 특정 피자 스타일에 맞는 재료를 일관되게 사용할 수 있습니다.

</details>



### **뉴욕 원재료 팩토리 만들기 (Creating New York Ingredient Factory)**

위에서 구현한 원재료 팩토리 중 뉴욕 스타일의 피자 재료를 담당하는 팩토리를 좀 더 구체화해 보겠습니다. `NYPizzaIngredientFactory`는 뉴욕 스타일 피자를 위해 얇은 도우와 마리나라 소스, 레지아노 치즈를 제공합니다.

<details>

<summary>class NYPizzaIngredientFactory implements PizzaIngredientFactory</summary>

```java
public class NYPizzaIngredientFactory implements PizzaIngredientFactory {
    @Override
    public Dough createDough() {
        return new ThinCrustDough();
    }

    @Override
    public Sauce createSauce() {
        return new MarinaraSauce();
    }

    @Override
    public Cheese createCheese() {
        return new ReggianoCheese();
    }
}
```

이 팩토리를 사용하여 뉴욕 스타일 피자를 만드는 피자 가게는 항상 동일한 원재료를 사용하여 피자를 제작하게 됩니다. 이는 제품의 일관성을 유지하는 데 중요한 역할을 합니다.

</details>



### **Pizza 클래스 변경하기 (Modifying the Pizza Class)**

원재료 팩토리 패턴을 도입하면 `Pizza` 클래스는 보다 유연하고 재사용 가능한 구조로 바뀝니다. 피자 클래스는 더 이상 구체적인 재료 클래스에 의존하지 않고, 팩토리를 통해 원재료를 조립합니다.

<details>

<summary>class VeggiePizza extends Pizza</summary>

```java
public class VeggiePizza extends Pizza {
    PizzaIngredientFactory ingredientFactory;

    public VeggiePizza(PizzaIngredientFactory ingredientFactory) {
        this.ingredientFactory = ingredientFactory;
    }

    @Override
    void prepare() {
        System.out.println("Preparing " + name);
        dough = ingredientFactory.createDough();
        sauce = ingredientFactory.createSauce();
        cheese = ingredientFactory.createCheese();
    }
}
```

이처럼 `Pizza` 클래스와 그 하위 클래스들은 이제 원재료 팩토리에 의존하여 피자 제조에 필요한 구성 요소들을 준비합니다. 이는 피자 스타일에 따라 원재료를 쉽게 교체할 수 있게 해줍니다.

</details>



### **올바른 재료 공장 사용하기 (Using the Right Ingredient Factory)**

피자 가게는 특정 피자 스타일에 맞는 원재료 팩토리를 사용해야 합니다. 예를 들어, 뉴욕 피자 가게는 `NYPizzaIngredientFactory`를, 시카고 피자 가게는 `ChicagoPizzaIngredientFactory`를 사용합니다.

<details>

<summary>class NYPizzaStore extends PizzaStore</summary>

```java
public class NYPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String item) {
        Pizza pizza = null;
        PizzaIngredientFactory ingredientFactory = new NYPizzaIngredientFactory();

        if (item.equals("cheese")) {
            pizza = new CheesePizza(ingredientFactory);
            pizza.setName("New York Style Cheese Pizza");
        } else if (item.equals("veggie")) {
            pizza = new VeggiePizza(ingredientFactory);
            pizza.setName("New York Style Veggie Pizza");
        }

        return pizza;
    }
}
```

이와 같은 방식으로 피자 가게는 특정 원재료 팩토리를 사용하여 피자를 제조하며, 이를 통해 일관된 제품 품질을 유지합니다.

</details>



## **바뀐 내용 되돌아보기 (Reviewing Changes)**

원재료 팩토리 패턴을 도입한 후, 코드 구조가 어떻게 변했는지 살펴봅시다. 원재료 팩토리는 피자 제조에 필요한 구성 요소들을 중앙에서 관리할 수 있게 해주며, 이는 코드의 유지 보수성과 확장성을 크게 높입니다.

* **코드의 유연성 증가**: 원재료 팩토리 패턴을 도입함으로써, 새로운 피자 스타일이나 원재료를 쉽게 추가할 수 있습니다.
* **객체 간 결합도 감소**: 피자 클래스가 구체적인 원재료 클래스에 의존하지 않으므로, 시스템의 결합도가 줄어듭니다.
* **일관성 유지**: 원재료 팩토리를 통해 피자 제조에 필요한 모든 원재료를 일관되게 제공할 수 있습니다.

이제 피자 주문 시스템은 더욱 유연하고 관리하기 쉬운 구조를 가지게 되었습니다.



### **새로운 코드로 또 피자 주문하기 (Ordering Pizza with New Code)**

피자 주문 시스템에 새로운 피자 스타일을 추가해야 할 경우, 기존 팩토리 패턴을 확장하여 쉽게 대응할 수 있습니다. 예를 들어, 새로운 스타일의 피자를 도입할 때, 기존의 팩토리 메서드에 새로운 조건을 추가하거나 새로운 팩토리 클래스를 만들면 됩니다.

<details>

<summary>class CaliforniaPizzaFactory extends PizzaFactory</summary>

```java
public class CaliforniaPizzaFactory extends PizzaFactory {
    @Override
    public Pizza createPizza(String type) {
        if (type.equals("Cheese")) {
            return new CaliforniaStyleCheesePizza();
        } else if (type.equals("Veggie")) {
            return new CaliforniaStyleVeggiePizza();
        } else {
            return null;
        }
    }
}
```

위 코드에서 `CaliforniaPizzaFactory`는 캘리포니아 스타일 피자를 생성하기 위해 새롭게 추가된 팩토리 클래스입니다. 이렇게 새로운 스타일의 피자를 추가할 때 기존의 구조를 크게 변경하지 않고도 유연하게 대응할 수 있습니다.

</details>



### **추상 팩토리 패턴의 정의 (Defining the Abstract Factory Pattern)**

추상 팩토리 패턴(Abstract Factory Pattern)은 서로 연관된 객체들의 집합을 생성하는 인터페이스를 제공합니다. 이 패턴은 팩토리 메서드 패턴의 확장판으로, 다양한 팩토리 메서드를 제공하여 여러 종류의 객체를 생성할 수 있게 합니다. 예를 들어, 피자뿐만 아니라 피자와 관련된 모든 원재료도 함께 생성할 수 있습니다.

추상 팩토리 패턴의 주요 특징은 다음과 같습니다:

* **서로 관련된 객체 집합의 일관된 생성**: 특정 스타일에 맞는 객체들을 함께 생성하여 일관된 구성을 유지할 수 있습니다.
* **구체적인 클래스를 노출하지 않음**: 클라이언트는 구체적인 클래스를 알 필요 없이 인터페이스를 통해 객체를 생성할 수 있습니다.
* **확장성**: 새로운 스타일이나 새로운 구성 요소를 추가할 때 기존 코드를 최소한으로 수정할 수 있습니다.

### **팩토리 메소드 패턴과 추상 팩토리 패턴 (Factory Method Pattern vs. Abstract Factory Pattern)**

팩토리 메소드 패턴과 추상 팩토리 패턴은 객체 생성에 초점을 맞춘다는 점에서 유사하지만, 적용되는 범위와 방법에는 차이가 있습니다. 팩토리 메소드 패턴은 개별 객체 생성에 중점을 두고, 추상 팩토리 패턴은 관련된 객체 집합의 생성에 중점을 둡니다.

예를 들어, 팩토리 메소드 패턴을 사용하면 피자 객체 하나를 생성하는 데 중점을 두지만, 추상 팩토리 패턴은 피자와 함께 사용되는 도우, 소스, 치즈 등의 원재료도 함께 생성할 수 있습니다.

<details>

<summary>class CaliforniaPizzaStore extends PizzaStore</summary>

```java
public class CaliforniaPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String item) {
        Pizza pizza = null;
        PizzaIngredientFactory ingredientFactory = new CaliforniaPizzaIngredientFactory();

        if (item.equals("cheese")) {
            pizza = new CheesePizza(ingredientFactory);
            pizza.setName("California Style Cheese Pizza");
        } else if (item.equals("veggie")) {
            pizza = new VeggiePizza(ingredientFactory);
            pizza.setName("California Style Veggie Pizza");
        }

        return pizza;
    }
}
```

위 코드에서 `CaliforniaPizzaStore`는 캘리포니아 스타일의 피자를 생성하기 위해 추상 팩토리 패턴을 사용하여 모든 구성 요소를 관리합니다.

</details>



### **추상 팩토리 패턴과 팩토리 메서드의 조합 (Combining the Abstract Factory Pattern and Factory Method Pattern)**

추상 팩토리 패턴과 팩토리 메서드 패턴을 결합하여 객체 생성의 유연성을 극대화할 수 있습니다. 이 조합은 특히 복잡한 객체 구성 및 다양한 제품 계열을 지원하는 시스템에서 유용합니다.

<details>

<summary>interface PizzaIngredientFactory<br>class CaliforniaPizzaIngredientFactory implements PizzaIngredientFactory</summary>

```java
public interface PizzaIngredientFactory {
    Dough createDough();
    Sauce createSauce();
    Cheese createCheese();
}

public class CaliforniaPizzaIngredientFactory implements PizzaIngredientFactory {
    @Override
    public Dough createDough() {
        return new WholeWheatDough();
    }

    @Override
    public Sauce createSauce() {
        return new BruschettaSauce();
    }

    @Override
    public Cheese createCheese() {
        return new GoatCheese();
    }
}
```

위 코드에서 `CaliforniaPizzaIngredientFactory`는 캘리포니아 스타일 피자에 필요한 원재료들을 생성합니다. 이를 통해 클라이언트는 피자 객체 생성 과정에서 모든 구성 요소를 쉽게 관리할 수 있습니다.

</details>



### **디자인 도구 상자 안에 들어가야 할 도구들 (Tools to Include in Your Design Toolbox)**

여러 디자인 패턴을 활용해 피자 주문 시스템을 개발하면서, 코드 재사용성과 확장성을 크게 높일 수 있었습니다. 이러한 패턴들을 체계적으로 정리하여 디자인 도구 상자를 구성하면, 다른 프로젝트에서도 이 패턴들을 쉽게 적용할 수 있습니다.

디자인 도구 상자에 포함되어야 할 주요 도구는 다음과 같습니다:

1. **팩토리 메소드 패턴**: 객체 생성의 책임을 서브클래스에 위임하여 코드의 유연성을 높임.
2. **추상 팩토리 패턴**: 서로 관련된 객체 집합을 일관되게 생성할 수 있도록 관리함.
3. **의존성 주입(DI)**: 객체 간의 결합도를 낮추고 코드의 유지 보수성을 높임.
4. **전략 패턴(Strategy Pattern)**: 행동을 캡슐화하여 런타임 시에 행위를 변경할 수 있게 함.
5. **템플릿 메소드 패턴(Template Method Pattern)**: 기본 알고리즘의 구조를 정의하고 하위 클래스가 특정 단계들을 재정의할 수 있도록 함.

이러한 도구들을 활용하여 복잡한 문제를 해결하고, 다양한 요구사항에 유연하게 대응할 수 있습니다.

### **디자인 도구 상자 구성하기 (Building Your Design Toolbox)**

마지막으로, 디자인 도구 상자를 구성하는 방법을 정리해보겠습니다. 각 패턴을 프로젝트의 요구사항에 맞게 적절히 선택하고, 이를 쉽게 적용할 수 있는 코드와 문서를 준비하는 것이 중요합니다. 디자인 도구 상자를 잘 구축하면, 새로운 프로젝트를 시작할 때마다 일관된 품질의 코드를 빠르게 작성할 수 있습니다.

1. **패턴 선택**: 프로젝트의 요구사항에 맞는 디자인 패턴을 선택합니다.
2. **코드 템플릿 준비**: 자주 사용하는 패턴의 코드 템플릿을 준비하여 빠르게 적용할 수 있도록 합니다.
3. **문서화**: 각 패턴의 사용 방법과 적용 예제를 문서화하여, 팀원들이 쉽게 이해하고 활용할 수 있게 합니다.
4. **정기적인 업데이트**: 새로운 패턴이나 개선된 방법론이 있을 경우, 디자인 도구 상자를 정기적으로 업데이트하여 최신 상태를 유지합니다.

디자인 도구 상자를 잘 활용하면, 프로젝트 진행 속도와 코드 품질을 동시에 높일 수 있습니다.

#### 결론

팩토리 패턴과 추상 팩토리 패턴은 객체 지향 프로그래밍에서 매우 중요한 역할을 합니다. 이 패턴들은 객체 생성의 유연성을 높이고, 코드의 재사용성을 극대화하는 데 기여합니다. 피자 주문 시스템을 통해 이러한 패턴들을 어떻게 적용할 수 있는지 살펴보았으며, 이를 바탕으로 다른 프로젝트에서도 디자인 도구 상자를 활용하여 복잡한 문제를 효과적으로 해결할 수 있을 것입니다.
