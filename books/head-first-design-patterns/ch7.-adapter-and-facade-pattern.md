---
description: 어댑터 패턴과 퍼사드 패턴
---

# Ch7. Adapter and Facade Pattern

어댑터 패턴(Adapter Pattern)과 퍼사드 패턴(Facade Pattern)은 구조적 디자인 패턴(Structural Design Pattern)에 속합니다. 이 패턴들은 기존 코드나 시스템의 구조를 바꾸지 않고도 기능을 추가하거나 시스템을 확장할 수 있게 도와줍니다.



## 어댑터 패턴

### 정의

어댑터 패턴은 서로 다른 인터페이스를 가진 두 개의 클래스가 함께 작동할 수 있도록 중간에서 '어댑터' 라는 객체를 통해 호환성을 제공하는 구조적 디자인 패턴입니다. 이 패턴은 기존 클래스를 수정하지 않고도 클라이언트가 기대하는 인터페이스를 맞춰줄 수 있습니다.

어댑터 패턴은 새로운 시스템이 기존의 레거시 시스템과 상호작용해야하는 경우에 주로 사용합니다. 기존 코드를 수정하지 않고 새로운 요구사항에 맞게 적응시키기 위해 어댑터를 사용하여 두 시스템 간의 불일치를 해결합니다.

### 구조

* 클라이언트: 타깃 인터페이스를 사용하여 요청을 보내는 역할을 합니다.
* 타깃: 클라이언트가 원하는 인터페이스를 정의합니다.
* 어댑터: 타깃 인터페이스를 구현하고, 클라이언트의 요청을 어댑티의 인터페이스에 맞게 변환합니다.
* 어댑티: 어댑터에 의해 적응되어야 하는 기존 클래스

<details>

<summary>예시</summary>

```java
// 타깃 인터페이스
public interface Target {
    void request();
}

// 어댑티 클래스
public class Adaptee {
    public void specificRequest() {
        System.out.println("Adaptee's specific request");
    }
}

// 어댑터 클래스
public class Adapter implements Target {
    private Adaptee adaptee;

    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        adaptee.specificRequest();
    }
}

// 클라이언트 코드
public class Client {
    public static void main(String[] args) {
        Adaptee adaptee = new Adaptee();
        Target adapter = new Adapter(adaptee);
        adapter.request(); // 출력: Adaptee's specific request
    }
}
```

이 예제에서 `Adapter` 클래스는 `Target` 인터페이스를 구현하여 `Adaptee`의 `specificRequest()` 메서드를 `request()`로 변환합니다. 클라이언트는 `Target` 인터페이스를 사용하여 `Adaptee`의 기능을 호출할 수 있습니다.

</details>



### 유형

* 클래스 어댑터 패턴: 어댑터가 두 인터페이스를 상속받아 변환하는 방식. 다중 상속이 가능한 언어에서만 사용됩니다.
* 객체 어댑터 패턴: 어댑터가 하나의 인터페이스를 구현하고, 다른 인터페이스를 포함하는 방식으로 변환을 수행합니다.

<details>

<summary>예시</summary>

자바에서 사용되는 `InputStreamReader` 클래스는 바이트 스트림을 문자 스트림으로 변환하는 어댑터로 작동합니다. 이 클래스는 어댑터 패턴의 전형적인 예로, 바이트 기반의 `InputStream`을 문자 기반의 `Reader` 인터페이스로 적응시킵니다.

```java
InputStream inputStream = new FileInputStream("file.txt");
Reader reader = new InputStreamReader(inputStream);
BufferedReader bufferedReader = new BufferedReader(reader);
```

이 코드에서 `InputStreamReader`는 바이트 스트림을 문자 스트림으로 변환하여 `BufferedReader`가 사용할 수 있도록 합니다.

</details>



### 장점

* **호환성:** 기존 코드의 변경 없이 새로운 시스템과의 통합을 가능하게 합니다.
* **유연성:** 인터페이스의 불일치를 해결하여 다양한 클래스를 통합할 수 있습니다.
* **재사용성:** 기존 클래스를 재사용할 수 있어, 코드 중복을 줄일 수 있습니다.

### 단점

* **복잡성 증가:** 새로운 어댑터 클래스를 도입함으로써 코드의 복잡성이 증가할 수 있습니다.
* **성능 저하:** 어댑터를 통해 호출되는 메서드들은 직접 호출되는 메서드에 비해 성능이 떨어질 수 있습니다.



## 퍼사드 패턴

퍼사드 패턴은 복잡한 시스템의 서브시스템의 단순화된 인터페이스로 감싸, 클라이언트가 복잡한 세부사항을 알지 않고도 서브시스템을 쉽게 사용할 수 있게 해주는 구조적 디자인 패턴입니다.

복잡한 시스템에서는 여러 서브시스템이 함께 작동하는데, 이러한 시스템을 사용하는 클라이언트는 각 서브시스템의 세부사항을 모두 이해하고, 사용할 필요 없이, 단순화된 인터페이스를 통해 필요한 작업을 수행할 수 있습니다. 퍼사드 패턴은 이러한 단순화된 인터페이스를 제공합니다.



### 구조

* 퍼사드: 클라이언트가 사용할 단순화된 인터페이스를 제공합니다.
* 서브시스템 클래스들: 실제 작업을 수행하는 클래스들입니다. 이들 클래스를 감싸서 단일 인터페이스로 제공합니다.

<details>

<summary>예시</summary>

```java
// 서브시스템 클래스들
class SubsystemA {
    public void operationA() {
        System.out.println("Subsystem A operation");
    }
}

class SubsystemB {
    public void operationB() {
        System.out.println("Subsystem B operation");
    }
}

// 퍼사드 클래스
class Facade {
    private SubsystemA subsystemA;
    private SubsystemB subsystemB;

    public Facade() {
        subsystemA = new SubsystemA();
        subsystemB = new SubsystemB();
    }

    public void performOperations() {
        subsystemA.operationA();
        subsystemB.operationB();
    }
}

// 클라이언트 코드
public class Client {
    public static void main(String[] args) {
        Facade facade = new Facade();
        facade.performOperations();
    }
}
```

이 예제에서 `Facade` 클래스는 `SubsystemA`와 `SubsystemB`의 기능을 하나의 메서드(`performOperations()`)로 통합하여 클라이언트가 서브시스템의 복잡한 세부사항을 신경 쓰지 않고 사용할 수 있게 합니다.

</details>

<details>

<summary>예시2</summary>

스프링 프레임워크의 `JdbcTemplate` 클래스는 퍼사드 패턴의 좋은 예입니다. `JdbcTemplate`은 JDBC의 복잡한 세부사항(예: 커넥션 관리, 예외 처리)을 단순화된 인터페이스로 감싸 사용자가 간단하게 데이터베이스 작업을 수행할 수 있게 합니다.

```java
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);

String sql = "SELECT name FROM users WHERE id = ?";
String name = jdbcTemplate.queryForObject(sql, new Object[]{1}, String.class);
```

이 코드에서 `JdbcTemplate`은 JDBC API를 감싸고, 클라이언트는 복잡한 JDBC 작업을 직접 수행할 필요 없이 간단한 메서드 호출만으로 원하는 작업을 수행할 수 있습니다.

</details>

### 장점

* **단순성:** 복잡한 서브시스템을 단순화하여 사용하기 쉽게 만듭니다.
* **결합도 감소:** 클라이언트와 서브시스템 간의 결합도를 낮춰 시스템 유지보수를 용이하게 합니다.
* **유연성:** 퍼사드를 통해 서브시스템의 내부 구조를 변경하더라도 클라이언트 코드에 영향을 주지 않습니다.

### 단점

* **추가 추상화:** 퍼사드를 추가하면 추가적인 추상화 레이어가 생기므로, 불필요하게 복잡성이 증가할 수 있습니다.
* **단순화의 위험:** 퍼사드가 너무 많은 기능을 단순화하면 서브시스템의 유연성을 잃을 수 있습니다.



## 어댑터 패턴과 퍼사드 패턴의 비교

어댑터 패턴과 퍼사드 패턴은 모두 시스템의 구조를 단순화하거나 시스템 간의 통합을 돕는 역할을 합니다.

* 어댑터 패턴: 기존 클래스의 인터페이스를 변경하여 클라이언트가 요구하는 인터페이스에 맞춥니다. 주로 두 개의 불일치하는 인터페이스 간의 호환성을 제공하는데 사용됩니다.
* 퍼사드 패턴: 복잡한 서브시스템을 단순화된 인터페이스로 감싸 클라이언트가 쉽게 사용할 수 있게 만듭니다. 복잡한 시스템을 보다 쉽게 관리하고 사용하기 위해 사용됩니다.



## 최소 지식 원칙

OOP에서 객체들이 서로 상호작용할 때, 다른 객체에 대해 가능한 적은 정보만 알아야 한다는 원칙입니다. 시스템의 모듈화와 캡슐화를 촉진하여 코드의 유지보수성과 확장성을 높이는 데 중요한 역할을 합니다.

<details>

<summary>예시</summary>

최소 지식 원칙은 "친구에게만 이야기하라"라는 비유로 설명될 수 있습니다. 즉, 한 객체는 자신이 직접적으로 참조하는 객체들(친구들)에 대해서만 메서드를 호출해야 하며, 다른 객체들(친구의 친구)에는 직접 접근하지 말아야 한다는 것입니다.

```java
public class Person {
    private Address address;

    public Address getAddress() {
        return address;
    }
}

public class Address {
    private City city;

    public City getCity() {
        return city;
    }
}

public class City {
    private String cityName;

    public String getCityName() {
        return cityName;
    }
}

public class Client {
    public String getPersonCityName(Person person) {
        return person.getAddress().getCity().getCityName();
    }
}
```

위의 코드에서는 `Client` 클래스가 `Person`, `Address`, `City`의 내부 구조에 대해 지나치게 많은 정보를 알고 있습니다. 이는 최소 지식 원칙에 위배됩니다.

이 원칙을 적용하여 코드를 개선하면 다음과 같이 변경할 수 있습니다:

```java
public class Person {
    private Address address;

    public String getCityName() {
        return address.getCityName();
    }
}

public class Address {
    private City city;

    public String getCityName() {
        return city.getCityName();
    }
}

public class City {
    private String cityName;

    public String getCityName() {
        return cityName;
    }
}

public class Client {
    public String getPersonCityName(Person person) {
        return person.getCityName();
    }
}
```

이 개선된 코드에서 `Client` 클래스는 `Person` 객체에만 의존하며, `Address`와 `City` 객체의 내부 구조에 대해 알 필요가 없습니다. 각 객체는 자신이 관리하는 데이터에만 접근하고, 필요한 정보는 외부로 노출되는 메서드를 통해 제공됩니다. 이로써 코드의 의존성을 줄이고, 변경이 필요한 경우에도 변경 범위를 최소화할 수 있습니다.

</details>



## 데코레이터 vs 어댑터 vs 퍼사드 비교

#### 1. 데코레이터 패턴 (Decorator Pattern)

**목적**

데코레이터 패턴은 객체에 새로운 기능을 동적으로 추가할 때 사용됩니다. 객체의 인터페이스를 변경하지 않고, 기존 객체를 감싸는(wrapper) 방식으로 새로운 기능을 부여합니다. 이 패턴은 상속 대신 조합을 활용해 기능을 확장할 수 있게 해줍니다.

**구조**

* **컴포넌트(Component):** 기본 인터페이스나 추상 클래스입니다.
* **구체 컴포넌트(Concrete Component):** 기본 기능을 제공하는 클래스입니다.
* **데코레이터(Decorator):** 컴포넌트 인터페이스를 구현하고, 기존 컴포넌트를 감싸며, 새로운 기능을 추가하는 클래스입니다.
* **구체 데코레이터(Concrete Decorator):** 추가적인 기능을 제공하는 데코레이터의 구체 구현입니다.

<details>

<summary><strong>예시</strong></summary>

```java
public interface Coffee {
    String getDescription();
    double cost();
}

public class SimpleCoffee implements Coffee {
    public String getDescription() {
        return "Simple Coffee";
    }

    public double cost() {
        return 2.0;
    }
}

public abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;

    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }

    public String getDescription() {
        return decoratedCoffee.getDescription();
    }

    public double cost() {
        return decoratedCoffee.cost();
    }
}

public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    public String getDescription() {
        return decoratedCoffee.getDescription() + ", Milk";
    }

    public double cost() {
        return decoratedCoffee.cost() + 0.5;
    }
}
```

위 코드에서 `MilkDecorator`는 `SimpleCoffee`에 우유를 추가하는 기능을 동적으로 추가합니다.

</details>

**사용 시점**

* 여러 클래스에서 공통 기능을 재사용하고 싶을 때
* 상속을 통해 기능을 확장하기보다 유연하게 기능을 추가하고 싶을 때

***

#### 2. 어댑터 패턴 (Adapter Pattern)

**목적**

어댑터 패턴은 호환되지 않는 인터페이스를 가진 클래스들이 함께 작동할 수 있도록, 중간에 어댑터 객체를 두어 호환성을 제공하는 데 사용됩니다. 즉, 클라이언트가 기대하는 인터페이스와 실제 제공하는 인터페이스 사이의 불일치를 해결합니다.

**구조**

* **타깃(Target):** 클라이언트가 기대하는 인터페이스입니다.
* **어댑터(Adapter):** 타깃 인터페이스를 구현하고, 어댑티(Adaptee)의 메서드를 호출하여 변환 작업을 수행합니다.
* **어댑티(Adaptee):** 어댑터가 적응시켜야 하는 기존 클래스입니다.
* **클라이언트(Client):** 타깃 인터페이스를 통해 어댑터를 사용하는 객체입니다.

<details>

<summary><strong>예시</strong></summary>

```java
public interface MediaPlayer {
    void play(String audioType, String fileName);
}

public class Mp3Player implements MediaPlayer {
    public void play(String audioType, String fileName) {
        System.out.println("Playing mp3 file. Name: " + fileName);
    }
}

public interface AdvancedMediaPlayer {
    void playVlc(String fileName);
    void playMp4(String fileName);
}

public class VlcPlayer implements AdvancedMediaPlayer {
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file. Name: " + fileName);
    }

    public void playMp4(String fileName) {
        // Do nothing
    }
}

public class MediaAdapter implements MediaPlayer {
    AdvancedMediaPlayer advancedMusicPlayer;

    public MediaAdapter(String audioType) {
        if(audioType.equalsIgnoreCase("vlc")) {
            advancedMusicPlayer = new VlcPlayer();
        } 
    }

    public void play(String audioType, String fileName) {
        if(audioType.equalsIgnoreCase("vlc")) {
            advancedMusicPlayer.playVlc(fileName);
        }
    }
}
```

위 예시에서 `MediaAdapter`는 `MediaPlayer` 인터페이스를 `AdvancedMediaPlayer`와 호환되도록 변환합니다.

</details>

**사용 시점**

* 기존 클래스를 수정할 수 없고, 인터페이스가 다른 클래스와 협력해야 할 때
* 새로운 인터페이스로 기존 클래스를 재사용하고자 할 때

***

#### 3. 퍼사드 패턴 (Facade Pattern)

**목적**

퍼사드 패턴은 복잡한 서브시스템의 인터페이스를 단순화하여, 클라이언트가 보다 쉽게 사용할 수 있도록 돕는 패턴입니다. 퍼사드는 여러 클래스의 복잡한 상호작용을 감추고 단순한 인터페이스를 제공함으로써 시스템의 사용성을 개선합니다.

**구조**

* **퍼사드(Facade):** 클라이언트에게 단순화된 인터페이스를 제공합니다.
* **서브시스템(Subsystems):** 복잡한 기능을 수행하는 클래스들입니다. 클라이언트는 이들 클래스와 직접 상호작용하지 않고 퍼사드를 통해 접근합니다.

<details>

<summary><strong>예시</strong></summary>

```java
class CPU {
    void freeze() { System.out.println("CPU freezing..."); }
    void jump(long position) { System.out.println("CPU jumping to position " + position); }
    void execute() { System.out.println("CPU executing..."); }
}

class Memory {
    void load(long position, byte[] data) { System.out.println("Loading memory..."); }
}

class HardDrive {
    byte[] read(long lba, int size) { 
        System.out.println("Reading data from hard drive...");
        return new byte[size]; 
    }
}

class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;

    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }

    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}
```

이 예시에서 `ComputerFacade`는 CPU, 메모리, 하드 드라이브와 같은 서브시스템을 감싸고, 클라이언트가 쉽게 컴퓨터를 시작할 수 있도록 `start()` 메서드를 제공합니다.

</details>

**사용 시점**

* 복잡한 서브시스템을 단순화하고 클라이언트가 쉽게 접근할 수 있게 하고자 할 때
* 특정 서브시스템의 사용을 제한하거나 제어하고자 할 때

***

#### 4. 패턴 비교

<table><thead><tr><th width="128">패턴</th><th>목적</th><th width="238">사용 상황</th><th>주요 특징</th></tr></thead><tbody><tr><td><strong>데코레이터</strong></td><td>객체에 동적으로 새로운 기능을 추가</td><td>런타임 시 객체에 기능을 추가하고 제거하고 싶을 때</td><td>객체를 감싸서 기능을 확장, 상속 대신 조합 활용</td></tr><tr><td><strong>어댑터</strong></td><td>호환되지 않는 인터페이스 간의 불일치를 해결</td><td>기존 클래스를 수정하지 않고 새로운 인터페이스에 맞추고 싶을 때</td><td>기존 클래스의 인터페이스를 변환, 클래스 재사용성 높임</td></tr><tr><td><strong>퍼사드</strong></td><td>복잡한 서브시스템을 단순화된 인터페이스로 감싸서 사용하기 쉽게 만듦</td><td>클라이언트가 서브시스템의 복잡성을 숨기고 단순하게 사용할 수 있게 하고 싶을 때</td><td>서브시스템을 감싸서 단순한 인터페이스 제공</td></tr></tbody></table>

이 세 가지 패턴은 모두 서로 다른 목적을 가지고 있으며, 서로 다른 상황에서 사용됩니다. 데코레이터는 객체의 기능을 동적으로 확장하는 데 유용하고, 어댑터는 기존 시스템과 새로운 시스템 간의 불일치를 해결하며, 퍼사드는 복잡한 시스템을 단순화하여 사용성을 높입니다. 이 패턴들은 모두 객체지향 설계에서 중요한 역할을 하며, 적절한 상황에서 사용하면 코드의 유지보수성과 확장성을 크게 향상시킬 수 있습니다.
