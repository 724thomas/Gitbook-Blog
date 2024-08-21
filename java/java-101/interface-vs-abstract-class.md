---
description: 인터페이스와 추상 클래스의 차이점
---

# Interface vs Abstract Class

## 1. 기본 개념 이해

### **1.1 추상 클래스(Abstract Class)란?**

추상 클래스는 객체 지향 프로그래밍(OOP)에서 일부 메서드가 구현되지 않은 클래스를 말합니다. 이 클래스는 다른 클래스가 상속받아 확장하도록 설계된 클래스로, 공통적인 속성과 메서드를 정의하지만, 일부 메서드는 하위 클래스에서 반드시 구현해야 합니다. 즉, **추상 클래스 자체로는 객체를 생성할 수 없으며,** **주로 공통적인 기능을 공유하는 여러 클래스들을 모듈화하기 위해 사용됩니다.**

<details>

<summary>public abstract class Animal</summary>

```java
public abstract class Animal {
    // 공통 속성
    protected String name;

    // 생성자
    public Animal(String name) {
        this.name = name;
    }

    // 공통 메서드
    public void sleep() {
        System.out.println(name + " is sleeping.");
    }

    // 추상 메서드 (하위 클래스에서 구현 필요)
    public abstract void sound();
}
```

위의 `Animal` 클래스는 공통적인 속성과 메서드를 가지고 있지만, `sound()`라는 메서드는 구현되지 않았습니다. 이는 `Animal` 클래스를 상속받은 하위 클래스들이 각각의 방식으로 `sound()` 메서드를 구현하도록 강제하는 역할을 합니다.

</details>



### **1.2 인터페이스(Interface)란?**

인터페이스는 클래스가 구현해야 하는 메서드의 청사진을 제공하는 일종의 계약(Contract)입니다. 인터페이스는 메서드의 시그니처(이름, 매개변수, 반환 타입)만을 정의하며, 그 구현은 이를 구현하는 클래스에서 이루어집니다. 자바에서 인터페이스는 다중 상속을 지원하기 위한 대안으로 사용되며, 클래스는 여러 개의 인터페이스를 구현할 수 있습니다.

<details>

<summary>public interface Flyable</summary>

```java
public interface Flyable {
    void fly();
}

public interface Swimable {
    void swim();
}

public class Duck implements Flyable, Swimable {
    @Override
    public void fly() {
        System.out.println("Duck is flying.");
    }

    @Override
    public void swim() {
        System.out.println("Duck is swimming.");
    }
}
```

위의 예제에서 `Flyable`과 `Swimable` 인터페이스는 각각 `fly()`와 `swim()` 메서드를 정의하고 있습니다. `Duck` 클래스는 이 두 인터페이스를 구현하며, 각각의 메서드를 실제로 구현하고 있습니다.

</details>



## 2. 추상 클래스와 인터페이스의 주요 차이점

### **2.1 다중 상속 가능 여부**

자바에서는 클래스의 다중 상속을 허용하지 않습니다. 즉, 한 클래스가 두 개 이상의 부모 클래스를 상속받을 수 없습니다. 그러나 인터페이스는 다중 구현이 가능하므로, 하나의 클래스가 여러 개의 인터페이스를 구현할 수 있습니다.

<details>

<summary> 예시:</summary>

```java
public class Animal {
    // 기본 클래스
}

public interface Flyable {
    void fly();
}

public interface Swimable {
    void swim();
}

public class Duck extends Animal implements Flyable, Swimable {
    @Override
    public void fly() {
        System.out.println("Duck is flying.");
    }

    @Override
    public void swim() {
        System.out.println("Duck is swimming.");
    }
}
```

위 코드에서 `Duck` 클래스는 `Animal` 클래스를 상속받고, 동시에 `Flyable`과 `Swimable` 인터페이스를 구현합니다. 이처럼 인터페이스를 사용하면 다중 상속의 장점을 얻을 수 있습니다.

</details>



### **2.2 구현된 메서드의 존재 여부**

추상 클래스는 구현된 메서드와 구현되지 않은 메서드를 모두 가질 수 있습니다. 이는 추상 클래스가 일부 공통 기능을 제공하면서, 하위 클래스에 특정 기능의 구현을 위임할 수 있다는 점에서 유리합니다. 반면, 자바 8 이전의 인터페이스는 모든 메서드가 구현되지 않은 상태로 존재해야 했습니다. 그러나 자바 8 이후부터는 디폴트 메서드(Default Method)라는 기능을 통해 인터페이스에서도 일부 메서드를 구현할 수 있게 되었습니다.

<details>

<summary>public interface Flyable</summary>

```java
public interface Flyable {
    void fly();

    default void takeOff() {
        System.out.println("Taking off.");
    }
}
```

위 예제에서 `takeOff()` 메서드는 디폴트 메서드로, 인터페이스 내에서 구현되었습니다. 이를 통해 인터페이스를 구현하는 클래스들이 공통적으로 사용할 수 있는 기능을 제공할 수 있습니다.

</details>



### **2.3 목적과 사용 사례**

* **추상 클래스**는 계층 구조에서 공통 기능을 공유하면서, 하위 클래스에 특정 기능의 구현을 강제하고 싶을 때 유용합니다. 주로 공통 속성이나 메서드를 공유하는 클래스들이 있을 때 사용됩니다.
* **인터페이스**는 서로 다른 클래스들이 공통의 행동을 할 수 있게끔 설계할 때 사용됩니다. 특히, 구현 클래스들이 전혀 다른 상속 계층에 속하더라도 동일한 인터페이스를 구현하여 공통된 행동을 정의할 수 있습니다.

## 3. 자바 스프링부트에서의 활용

자바 스프링부트에서는 추상 클래스와 인터페이스가 다양한 용도로 사용됩니다. 이들은 스프링의 핵심 기능인 의존성 주입(Dependency Injection)과 서비스 레이어 아키텍처에서 중요한 역할을 합니다.

### **3.1 서비스 레이어에서의 인터페이스 활용**

스프링부트 애플리케이션에서 서비스 레이어는 비즈니스 로직을 담당하며, 인터페이스를 통해 구현됩니다. 이 방식은 구현체를 유연하게 교체할 수 있도록 도와줍니다.

<details>

<summary>public interface UserService</summary>

```java
public interface UserService {
    User findUserById(Long id);
    void createUser(User user);
}

@Service
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;

    @Autowired
    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public User findUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found"));
    }

    @Override
    public void createUser(User user) {
        userRepository.save(user);
    }
}
```

위 예제에서 `UserService` 인터페이스는 사용자 관련 비즈니스 로직을 정의하고 있으며, `UserServiceImpl` 클래스가 이를 구현합니다. 이 구조는 다른 구현체를 쉽게 교체하거나 추가할 수 있도록 해주며, 스프링의 의존성 주입을 통해 인터페이스를 사용하는 곳에 필요한 구현체를 자동으로 주입해줍니다.

</details>



### **3.2 추상 클래스의 활용**

스프링부트에서는 추상 클래스가 공통 기능을 제공하고, 이를 상속받아 구체적인 기능을 구현하는 데 자주 사용됩니다. 예를 들어, 여러 엔터티에서 공통적으로 사용하는 기능을 추상 클래스에서 정의하고, 각 엔터티 클래스가 이를 상속받아 사용할 수 있습니다.

<details>

<summary>public abstract class BaseEntity</summary>

```java
@MappedSuperclass
public abstract class BaseEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        this.updatedAt = LocalDateTime.now();
    }

    // Getters and Setters...
}

@Entity
public class User extends BaseEntity {
    private String name;
    private String email;

    // Getters and Setters...
}
```

위 코드에서 `BaseEntity` 추상 클래스는 모든 엔터티에서 공통적으로 사용될 `id`, `createdAt`, `updatedAt` 필드를 정의하고 있습니다. 또한, `onCreate()`와 `onUpdate()` 메서드는 각각 엔터티가 생성되거나 업데이트될 때 자동으로 호출됩니다. `User` 클래스는 `BaseEntity`를 상속받아 이 기능을 그대로 사용할 수 있습니다.

</details>



## 4. 인터페이스와 추상 클래스의 선택 기준

* **인터페이스를 선택할 때**: 클래스들이 전혀 다른 계층 구조에 있을 수 있으며, 단순히 공통의 행동(메서드)을 정의하고 구현을 강제하고자 할 때. 예를 들어, `Flyable`, `Swimable`과 같이 특정 행동을 정의하는 경우 인터페이스가 적합합니다.
* **추상 클래스를 선택할 때**: 공통적인 상태(필드)를 공유해야 하거나, 일부 구현을 제공하면서 나머지는 하위 클래스에 구현을 위임하고자 할 때. 예를 들어, 공통 속성이나 기본 동작을 포함하는 경우 추상 클래스가 적합합니다.

## 5. 결론

인터페이스와 추상 클래스는 모두 객체 지향 프로그래밍에서 다형성과 캡슐화를 실현하는 중요한 도구입니다. 각자의 특성과 장단점이 있으며, 상황에 맞게 적절히 선택하여 사용해야 합니다. 자바 스프링부트와 같은 프레임워크에서는 이 둘을 적절히 활용하여 유연하고 확장 가능한 구조를 만들 수 있습니다. 인터페이스는 주로 계약을 정의하는 데 사용되고, 추상 클래스는 공통 기능을 제공하면서 확장을 가능하게 합니다.
