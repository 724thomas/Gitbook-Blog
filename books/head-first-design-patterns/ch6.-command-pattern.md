---
description: 호출 캡슐화하기. 커멘드 패턴
---

# Ch6. Command Pattern

## 1. 커맨드 패턴 정의와 기본 개념

\*\*커맨드 패턴(Command Pattern)\*\*은 요청을 객체로 캡슐화하여, 요청자와 수신자 간의 결합을 느슨하게 만드는 디자인 패턴이다. 이 패턴은 사용자가 요청을 발송하는 방식과 그 요청을 처리하는 방식 사이에 명확한 구분을 두며, 이를 통해 실행 취소, 작업 기록 및 작업 큐 같은 기능을 손쉽게 구현할 수 있다.

이 패턴의 핵심은 \*\*명령(Command)\*\*이라는 객체를 생성하여, 요청을 포함한 모든 정보를 캡슐화하는 것이다. 명령 객체는 특정 작업을 수행할 수 있는 메서드를 포함하며, 이러한 작업을 언제든지 실행, 취소, 다시 실행할 수 있다.



## 2. 커맨드 패턴의 구조

커맨드 패턴의 구조는 네 가지 주요 요소로 구성된다:

1. **Command(커맨드)**: 실행될 작업을 캡슐화한 인터페이스 혹은 추상 클래스.
2. **ConcreteCommand(구체 커맨드)**: Command 인터페이스를 구현한 클래스. 실제 작업의 구현 내용을 포함.
3. **Invoker(호출자)**: 명령을 요청하는 객체. ConcreteCommand 객체에 작업을 요청한다.
4. **Receiver(수신자)**: 실제로 작업을 수행하는 객체. ConcreteCommand 객체로부터 작업을 전달받아 수행.

이 구조를 통해 작업 요청과 실행이 분리되며, 명령의 실행을 유연하게 관리할 수 있다.

## 3. 음식 주문 과정 자세히 살펴보기

객체마을 식당에서의 음식 주문 과정은 커맨드 패턴의 좋은 예시이다. 고객이 음식을 주문하면, 그 주문은 커맨드 객체로 캡슐화된다. 이 커맨드 객체는 주문 정보를 담고 있으며, 주방 혹은 바텐더 같은 수신자 객체에 전달되어 실제로 주문이 처리된다.

예를 들어, 고객이 "피자 주문"을 요청하면, 해당 요청은 `PizzaOrderCommand`라는 구체 커맨드 객체로 생성된다. 이 객체는 피자 조리법과 같은 정보를 포함하고 있으며, 주방 클래스에 전달되어 피자 조리 작업이 실행된다.

## 4. 객체마을 식당 등장인물의 역할

커맨드 패턴에서는 등장인물들이 명확한 역할을 맡는다:

* **고객(Client)**: 커맨드를 생성하는 역할을 담당한다. 이 예시에서 고객은 특정 음식을 주문하는 사람이다.
* **서빙 담당자(Invoker)**: 커맨드를 실행하도록 요청하는 역할을 한다. 서빙 담당자는 고객의 주문을 받아 주방에 전달한다.
* **주방장(Receiver)**: 실제로 주문을 처리하는 역할을 담당한다. 주방장은 커맨드를 받아 음식을 조리한다.

## 5. 객체마을 식당과 커맨드 패턴의 연관성

객체마을 식당의 시스템은 커맨드 패턴을 완벽하게 반영한다. 고객이 음식을 주문하고, 그 주문이 커맨드 객체로 캡슐화되어, 서빙 담당자와 주방장을 통해 실행된다. 이러한 구조는 커맨드 패턴의 핵심 아이디어인 "호출의 캡슐화"를 잘 보여준다.

<figure><img src="/broken/files/iGo3MvDVzWvg11TAGuD6" alt=""><figcaption></figcaption></figure>



## 6. 첫 번째 커맨드 객체 만들기

커맨드 패턴을 이해하기 위해 먼저 커맨드 객체를 만들어보자. 예를 들어, 객체마을 식당에서 '주문'을 처리하는 커맨드 객체를 만들어 보겠다. 이를 위해 `Command`라는 인터페이스를 정의하고, 이 인터페이스를 구현하는 구체적인 커맨드 클래스를 작성해야 한다.

<details>

<summary><strong><code>Command</code> 인터페이스 정의</strong></summary>

```java
public interface Command {
    void execute();
}
```

이 인터페이스는 `execute()` 메서드를 가지고 있다. 이는 명령을 실행할 때 호출되는 메서드로, 모든 커맨드 객체는 이 메서드를 구현해야 한다.

</details>

<details>

<summary><strong>구체적인 커맨드 클래스 작성</strong></summary>



```java
public class PizzaOrderCommand implements Command {
    private KitchenReceiver kitchen;

    public PizzaOrderCommand(KitchenReceiver kitchen) {
        this.kitchen = kitchen;
    }

    @Override
    public void execute() {
        kitchen.cookPizza();
    }
}
```

위 코드에서 `PizzaOrderCommand` 클래스는 `Command` 인터페이스를 구현하며, 주문이 들어왔을 때 주방에서 피자를 조리하는 명령을 수행하는 역할을 한다.

</details>



## 8. 커맨드 객체 사용하기

이제 생성된 커맨드 객체를 사용하는 방법을 살펴보겠다. 커맨드 객체는 `Invoker`에 의해 호출되어야 한다. 이 `Invoker`는 명령을 실행하는 주체이며, 예를 들어, 리모컨이나 서빙 담당자가 이 역할을 할 수 있다.

<details>

<summary><strong>Invoker 클래스</strong></summary>

```java
public class WaiterInvoker {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void placeOrder() {
        command.execute();
    }
}
```

서빙 담당자(`WaiterInvoker`)는 고객이 주문한 커맨드를 받아서, 이를 주방에 전달하는 역할을 한다. `placeOrder()` 메서드에서 커맨드 객체의 `execute()` 메서드를 호출하여 실제 작업을 수행한다.

</details>



## 9. 커맨드 패턴 클래스 다이어그램 살펴보기

커맨드 패턴의 구조를 시각적으로 이해하기 위해 클래스 다이어그램을 살펴보자.

<figure><img src="/broken/files/1Ic26RgG71vXernGo40W" alt=""><figcaption></figcaption></figure>

다이어그램에는 다음과 같은 클래스들이 포함되어 있다:

* **Command**: 명령의 추상화된 인터페이스.
* **ConcreteCommand**: 명령의 구체적인 구현.
* **Invoker**: 명령을 실행하는 주체.
* **Receiver**: 명령을 실제로 처리하는 객체.

이 다이어그램을 통해 각 클래스가 어떻게 상호작용하는지, 그리고 커맨드 패턴이 구조적으로 어떻게 동작하는지를 이해할 수 있다.

## 10. 슬롯에 명령 할당하기

리모컨처럼 여러 명령을 한 번에 관리할 수 있는 상황을 가정해 보자. 각각의 명령은 리모컨의 슬롯에 할당될 수 있다.

<details>

<summary><strong>리모컨 클래스</strong></summary>

```java
public class RemoteControlInvoker {
    private Command[] onCommands;
    private Command[] offCommands;

    public RemoteControlInvoker() {
        onCommands = new Command[7];
        offCommands = new Command[7];
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void pressOnButton(int slot) {
        onCommands[slot].execute();
    }

    public void pressOffButton(int slot) {
        offCommands[slot].execute();
    }
}
```

이 리모컨 클래스(`RemoteControlInvoker`)는 여러 슬롯을 가지고 있으며, 각 슬롯에 명령을 할당할 수 있다. `pressOnButton()`이나 `pressOffButton()` 메서드를 통해 특정 슬롯에 할당된 명령을 실행할 수 있다.

</details>



## 11. 리모컨 코드 만들기

<details>

<summary>리모컨에 명령을 할당하고, 이를 실행하는 예제</summary>

```java
public class RemoteControlTest {
    public static void main(String[] args) {
        RemoteControlInvoker remote = new RemoteControlInvoker();

        KitchenReceiver kitchen = new KitchenReceiver();
        PizzaOrderCommand pizzaOrder = new PizzaOrderCommand(kitchen);

        remote.setCommand(0, pizzaOrder, null);

        System.out.println("Pressing the On button for slot 0...");
        remote.pressOnButton(0);
    }
}
```

위 코드에서, 리모컨의 첫 번째 슬롯에 피자 주문 명령을 할당하고, 버튼을 눌러 피자를 주문하는 과정을 보여준다. 이로써 커맨드 패턴의 기본 개념과 이를 실제 코드로 구현하는 방법을 확인할 수 있다.

</details>



## 12. 커맨드 클래스 만들기

커맨드 패턴에서 핵심적인 역할을 하는 클래스가 바로 **커맨드 클래스**이다. 이 클래스는 실제로 실행될 작업을 캡슐화하여, 호출자(Invoker)가 특정 작업을 요청할 때 이를 실행하거나 취소할 수 있도록 한다.

커맨드 클래스를 만드는 과정에서 주의해야 할 몇 가지 사항이 있다. 우선, 각 커맨드 클래스는 `Command` 인터페이스를 구현해야 하며, 기본적으로 `execute()`와 `undo()` 메서드를 포함해야 한다. 이는 명령을 실행하고, 필요할 경우 이를 취소할 수 있는 기능을 제공하기 위함이다.

<details>

<summary><strong><code>LightOnCommand</code> 클래스</strong></summary>

```java
public class LightOnCommand implements Command {
    private LightReceiver light;

    public LightOnCommand(LightReceiver light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.on();
    }

    @Override
    public void undo() {
        light.off();
    }
}
```

이 클래스는 `LightReceiver`라는 수신자 객체의 `on()` 메서드를 호출하여 불을 켜는 명령을 수행한다. `undo()` 메서드는 그 반대로 불을 끄는 역할을 한다. 이러한 방식으로, 커맨드 클래스는 작업을 캡슐화하여 호출자와 수신자 간의 결합도를 낮출 수 있다.

</details>



## 13. 리모컨 테스트

이제 커맨드 클래스가 제대로 동작하는지 테스트해보자. 리모컨을 사용하여 여러 명령을 실행하고 취소하는 과정을 구현할 수 있다.

<details>

<summary><strong>리모컨 테스트 코드</strong></summary>

```java
public class RemoteControlTest {
    public static void main(String[] args) {
        RemoteControlInvoker remote = new RemoteControlInvoker();

        LightReceiver livingRoomLight = new LightReceiver("Living Room");
        LightOnCommand lightOn = new LightOnCommand(livingRoomLight);
        LightOffCommand lightOff = new LightOffCommand(livingRoomLight);

        remote.setCommand(0, lightOn, lightOff);

        System.out.println("Turning the light on...");
        remote.pressOnButton(0);
        System.out.println("Undoing the operation...");
        remote.pressUndoButton();
        System.out.println("Turning the light off...");
        remote.pressOffButton(0);
    }
}
```

위 코드는 거실 조명에 대한 커맨드 객체를 리모컨에 할당하고, 버튼을 눌러 조명을 켜거나 끄는 작업을 실행한다. 또한 `pressUndoButton()` 메서드를 통해 직전에 실행된 명령을 취소하는 기능도 확인할 수 있다.

</details>



## 14. API 문서 만들기

커맨드 패턴을 사용한 코드를 작성한 후에는, 그에 대한 **API 문서**를 만드는 것이 중요하다. API 문서는 다른 개발자들이 해당 패턴을 어떻게 사용할지 이해할 수 있도록 돕는다. API 문서를 작성할 때는 각 클래스의 역할과 메서드의 기능을 명확히 설명해야 한다.

**예시: API 문서 구성**

* **Command 인터페이스**: 명령을 실행하는 메서드 `execute()`와 명령을 취소하는 메서드 `undo()`를 정의합니다.
* **ConcreteCommand 클래스**: `LightOnCommand`, `LightOffCommand` 등 구체적인 명령을 실행하는 클래스로, 수신자 객체의 메서드를 호출하여 작업을 수행합니다.
* **Invoker 클래스**: `RemoteControlInvoker` 클래스는 여러 명령을 관리하며, 특정 슬롯에 명령을 할당하고, 이를 실행하거나 취소할 수 있습니다.
* **Receiver 클래스**: `LightReceiver` 클래스는 실제로 작업을 수행하는 클래스입니다. 이 클래스는 `on()` 및 `off()` 메서드를 통해 조명을 제어합니다.

이와 같은 문서를 작성하면 다른 개발자들이 커맨드 패턴을 쉽게 이해하고, 적절하게 활용할 수 있다.

## 15. 작업 취소 기능 추가하기

커맨드 패턴의 주요 장점 중 하나는 \*\*작업 취소 기능(Undo)\*\*을 쉽게 구현할 수 있다는 것이다. 이 기능을 추가하기 위해서는 각 커맨드 객체에 `undo()` 메서드를 구현하고, 호출자가 이를 호출할 수 있도록 해야 한다.

<details>

<summary><strong>Undo 기능 구현</strong></summary>

```java
public class RemoteControlInvoker {
    private Command[] onCommands;
    private Command[] offCommands;
    private Command undoCommand;

    public RemoteControlInvoker() {
        onCommands = new Command[7];
        offCommands = new Command[7];
        undoCommand = new NoCommand(); // 초기에는 아무 명령도 없음
    }

    public void pressOnButton(int slot) {
        onCommands[slot].execute();
        undoCommand = onCommands[slot];
    }

    public void pressOffButton(int slot) {
        offCommands[slot].execute();
        undoCommand = offCommands[slot];
    }

    public void pressUndoButton() {
        undoCommand.undo();
    }
}
```

`RemoteControlInvoker` 클래스에 `undoCommand` 변수를 추가하여, 직전에 실행된 명령을 추적하고, 이를 취소할 수 있도록 구현했다. `pressUndoButton()` 메서드는 이 `undoCommand`에 저장된 명령을 실행하여 작업을 취소한다.

</details>



## 16. 작업 취소 기능 테스트하기

이제 구현한 작업 취소 기능이 제대로 동작하는지 테스트해보자.

<details>

<summary><strong>Undo 기능 테스트 코드</strong></summary>



```java
public class RemoteControlTest {
    public static void main(String[] args) {
        RemoteControlInvoker remote = new RemoteControlInvoker();

        LightReceiver livingRoomLight = new LightReceiver("Living Room");
        LightOnCommand lightOn = new LightOnCommand(livingRoomLight);
        LightOffCommand lightOff = new LightOffCommand(livingRoomLight);

        remote.setCommand(0, lightOn, lightOff);

        System.out.println("Turning the light on...");
        remote.pressOnButton(0);
        System.out.println("Turning the light off...");
        remote.pressOffButton(0);
        System.out.println("Undoing the last operation...");
        remote.pressUndoButton();
    }
}
```

위 코드에서, 불을 켜고 끈 후에 마지막 작업을 취소하여 불이 다시 켜지는 과정을 확인할 수 있다. 이로써 커맨드 패턴의 유용성, 특히 작업 취소 기능의 유용성을 확인할 수 있다.

</details>



## 17. 작업 취소 기능을 구현할 때 상태를 사용하는 방법

커맨드 패턴에서 **작업 취소(Undo)** 기능을 구현할 때, 작업의 상태를 추적하는 방법이 매우 중요하다. 상태를 관리함으로써, 특정 시점으로 돌아가 작업을 되돌리는 기능을 효과적으로 구현할 수 있다.

작업의 상태를 관리하는 일반적인 방법은 수신자(Receiver)의 상태를 보관하고, Undo 작업 시 이전 상태로 복구하는 것이다. 예를 들어, 선풍기의 속도를 제어하는 기능에서 작업 취소 기능을 구현한다고 가정해보자. 이 경우, 선풍기의 속도 상태를 저장해두었다가, Undo 작업 시 이전 속도로 되돌릴 수 있다.

<details>

<summary><strong>예시: 선풍기 상태 관리</strong></summary>

```java
public class CeilingFanReceiver {
    public static final int HIGH = 3;
    public static final int MEDIUM = 2;
    public static final int LOW = 1;
    public static final int OFF = 0;

    private int speed;

    public CeilingFanReceiver() {
        speed = OFF;
    }

    public void high() {
        speed = HIGH;
        System.out.println("Ceiling fan is on high.");
    }

    public void medium() {
        speed = MEDIUM;
        System.out.println("Ceiling fan is on medium.");
    }

    public void low() {
        speed = LOW;
        System.out.println("Ceiling fan is on low.");
    }

    public void off() {
        speed = OFF;
        System.out.println("Ceiling fan is off.");
    }

    public int getSpeed() {
        return speed;
    }
}
```

위 코드에서, `CeilingFanReceiver` 클래스는 선풍기의 속도를 나타내는 `speed` 변수를 관리한다. 속도를 변경할 때마다 해당 상태를 기록하고, 필요시 이전 상태로 복구할 수 있다.

</details>



## 18. 선풍기 명령어에 작업 취소 기능 추가하기

이제 선풍기 명령어에 작업 취소 기능을 추가하겠다. `CeilingFanReceiver`의 상태를 활용해 현재 속도를 기록하고, Undo 시 이전 속도로 복구하는 방식으로 구현할 수 있다.

<details>

<summary><strong>예시: 선풍기 명령어 클래스</strong></summary>

```java
public class CeilingFanHighCommand implements Command {
    private CeilingFanReceiver ceilingFan;
    private int prevSpeed;

    public CeilingFanHighCommand(CeilingFanReceiver ceilingFan) {
        this.ceilingFan = ceilingFan;
    }

    @Override
    public void execute() {
        prevSpeed = ceilingFan.getSpeed();
        ceilingFan.high();
    }

    @Override
    public void undo() {
        if (prevSpeed == CeilingFanReceiver.HIGH) {
            ceilingFan.high();
        } else if (prevSpeed == CeilingFanReceiver.MEDIUM) {
            ceilingFan.medium();
        } else if (prevSpeed == CeilingFanReceiver.LOW) {
            ceilingFan.low();
        } else if (prevSpeed == CeilingFanReceiver.OFF) {
            ceilingFan.off();
        }
    }
}
```

이 클래스에서 `execute()` 메서드는 현재 선풍기의 속도를 기록한 후, 속도를 최고로 올린다. `undo()` 메서드는 기록된 이전 속도를 확인하고, 그 속도로 복구한다.

</details>



## 19. 선풍기 테스트 코드 만들기

이제 선풍기 명령어와 작업 취소 기능이 잘 동작하는지 테스트해보자.

<details>

<summary><strong>예시: 선풍기 테스트 코드</strong></summary>

```java
public class RemoteControlTest {
    public static void main(String[] args) {
        RemoteControlInvoker remote = new RemoteControlInvoker();

        CeilingFanReceiver ceilingFan = new CeilingFanReceiver();
        CeilingFanHighCommand ceilingFanHigh = new CeilingFanHighCommand(ceilingFan);
        CeilingFanMediumCommand ceilingFanMedium = new CeilingFanMediumCommand(ceilingFan);
        CeilingFanOffCommand ceilingFanOff = new CeilingFanOffCommand(ceilingFan);

        remote.setCommand(0, ceilingFanHigh, ceilingFanOff);
        remote.setCommand(1, ceilingFanMedium, ceilingFanOff);

        System.out.println("Setting fan to high...");
        remote.pressOnButton(0);
        System.out.println("Undoing last operation...");
        remote.pressUndoButton();

        System.out.println("Setting fan to medium...");
        remote.pressOnButton(1);
        System.out.println("Undoing last operation...");
        remote.pressUndoButton();
    }
}
```

이 테스트 코드는 선풍기를 최고 속도와 중간 속도로 설정한 후, 각각의 작업을 취소하여 선풍기의 상태가 제대로 복구되는지 확인한다.

</details>



## 20. 선풍기 코드 테스트

위에서 작성한 테스트 코드를 실행하면, 다음과 같은 결과를 얻을 수 있다:

```vbnet
Setting fan to high...
Ceiling fan is on high.
Undoing last operation...
Ceiling fan is off.
Setting fan to medium...
Ceiling fan is on medium.
Undoing last operation...
Ceiling fan is off.
```

이 출력 결과를 통해 작업이 올바르게 실행되고, 취소되는지 확인할 수 있다. 이를 통해 커맨드 패턴을 활용한 상태 관리와 Undo 기능 구현이 성공적으로 이루어졌음을 알 수 있다.

## 21. 여러 동작을 한 번에 처리하기

마지막으로, 커맨드 패턴을 활용해 여러 동작을 한 번에 처리하는 방법을 알아보자. 이 기능은 매크로(Macro) 명령어를 통해 구현할 수 있으며, 여러 개의 커맨드 객체를 순차적으로 실행하거나 취소할 수 있다.

<details>

<summary><strong>매크로 명령어 클래스</strong></summary>

```java
public class MacroCommand implements Command {
    private Command[] commands;

    public MacroCommand(Command[] commands) {
        this.commands = commands;
    }

    @Override
    public void execute() {
        for (Command command : commands) {
            command.execute();
        }
    }

    @Override
    public void undo() {
        for (int i = commands.length - 1; i >= 0; i--) {
            commands[i].undo();
        }
    }
}
```

`MacroCommand` 클래스는 여러 커맨드 객체를 배열로 받아, 순차적으로 실행하거나 취소한다. `execute()` 메서드는 배열의 각 커맨드를 실행하고, `undo()` 메서드는 역순으로 커맨드를 취소한다.

</details>

<details>

<summary><strong>매크로 명령어 테스트</strong></summary>

```java
public class RemoteControlTest {
    public static void main(String[] args) {
        RemoteControlInvoker remote = new RemoteControlInvoker();

        CeilingFanReceiver ceilingFan = new CeilingFanReceiver();
        LightReceiver livingRoomLight = new LightReceiver("Living Room");

        CeilingFanHighCommand ceilingFanHigh = new CeilingFanHighCommand(ceilingFan);
        LightOnCommand lightOn = new LightOnCommand(livingRoomLight);

        Command[] partyOn = { ceilingFanHigh, lightOn };
        MacroCommand partyOnMacro = new MacroCommand(partyOn);

        remote.setCommand(0, partyOnMacro, null);

        System.out.println("Executing macro command...");
        remote.pressOnButton(0);
        System.out.println("Undoing macro command...");
        remote.pressUndoButton();
    }
}
```

이 코드는 매크로 명령어를 사용해 선풍기를 최고 속도로 설정하고, 거실 조명을 켜는 작업을 한 번에 처리한다. 이후, 이 두 작업을 모두 취소하는 기능도 구현된다.

</details>



## 22. 매크로 커맨드 사용하기

이전 Part 4에서 매크로 명령어를 소개했으며, 이제 이 명령어를 활용하는 방법을 더 구체적으로 살펴보겠다. 매크로 커맨드는 여러 개의 명령을 묶어 한 번에 실행하거나 취소할 수 있는 강력한 도구이다.

매크로 커맨드를 통해 한 번에 여러 동작을 수행할 수 있다. 이는 특히 복잡한 작업이 필요한 상황에서 매우 유용하다. 예를 들어, 스마트 홈 시스템에서 여러 장치를 동시에 제어하는 작업이 필요할 때 매크로 커맨드를 사용하면 효과적이다.

## 23. 커맨드 패턴 활용하기

커맨드 패턴을 실전에서 활용할 수 있는 다양한 방법을 알아보자. 이 패턴은 사용자 인터페이스(UI) 이벤트 처리, 작업 큐 관리, 트랜잭션 관리 등에서 널리 사용된다.

**사용자 인터페이스 이벤트 처리**

예를 들어, GUI 프로그램에서 버튼 클릭 이벤트를 처리할 때 커맨드 패턴을 사용하면, 버튼 클릭 시 호출되는 작업을 객체로 캡슐화하여 관리할 수 있다. 이는 이벤트 처리 로직을 깔끔하게 유지하며, 다양한 이벤트를 일관되게 처리할 수 있게 한다.

**작업 큐 관리**

커맨드 패턴은 작업 큐를 관리하는 데에도 유용하다. 작업이 큐에 추가될 때마다 해당 작업을 커맨드 객체로 캡슐화하고, 큐에서 작업을 꺼내 실행하거나 취소할 수 있다. 이를 통해 작업 관리가 더욱 체계적으로 이루어진다.

## 24. 커맨드 패턴 더 활용하기

커맨드 패턴은 단순히 명령을 캡슐화하는 것을 넘어, 복잡한 기능을 더욱 쉽게 구현할 수 있게 돕는다. 다음은 커맨드 패턴을 더 활용할 수 있는 몇 가지 예이다.

**트랜잭션 관리**

여러 작업을 하나의 트랜잭션으로 묶어 관리할 때, 커맨드 패턴을 사용하면 각 작업을 독립적으로 관리하면서도 전체 트랜잭션의 일관성을 유지할 수 있다. 트랜잭션 실패 시 각 작업을 개별적으로 취소할 수 있어, 복구 작업이 간편해진다.

**리모컨 시스템 확장**

스마트 홈 시스템에서 여러 가전제품을 제어하는 리모컨을 예로 들자면, 새로운 가전제품이 추가될 때마다 해당 제품에 맞는 커맨드 클래스를 작성하고, 이를 리모컨에 쉽게 통합할 수 있다. 이는 시스템 확장을 유연하게 처리할 수 있도록 돕는다.

## 25. 실전 적용! 커맨드 패턴

마지막으로, 실제 프로젝트에 커맨드 패턴을 적용하는 방법을 살펴보겠다. 커맨드 패턴을 실전에서 효과적으로 사용하기 위해서는 다음과 같은 사항을 고려해야 한다.

1. **작업의 복잡도**: 커맨드 패턴은 작업이 복잡해질수록 그 진가를 발휘한다. 단순한 작업에는 오히려 과도한 설계가 될 수 있으므로, 적용할 때 작업의 복잡도를 고려해야 한다.
2. **확장성**: 커맨드 패턴은 시스템 확장에 유리하다. 새로운 명령이 필요할 때, 기존 코드를 수정할 필요 없이 새로운 커맨드 클래스를 추가하여 쉽게 확장할 수 있다.
3. **유지보수성**: 커맨드 패턴을 사용하면, 코드의 유지보수가 용이해진다. 각 커맨드가 독립적인 클래스로 구현되기 때문에, 특정 작업의 로직을 수정해야 할 경우 해당 커맨드 클래스만 수정하면 된다.
4. **테스트 용이성**: 각 커맨드 객체를 독립적으로 테스트할 수 있어, 시스템의 각 구성 요소를 개별적으로 검증할 수 있다. 이는 테스트 자동화 및 품질 관리를 더욱 쉽게 만들어준다.



## 종합

<details>

<summary>interface Command</summary>

```java
public interface Command {
    void execute();
    void undo();
}
```

</details>

<details>

<summary>class Light (수신자)</summary>

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class Light {
    public void on() {
        log.info("The light is ON");
    }

    public void off() {
        log.info("The light is OFF");
    }
}
```

#### 3. Li

</details>

<details>

<summary>LightOnCommand 클래스 (전등 켜기 명령)</summary>

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class LightOnCommand implements Command {

    private final Light light;

    @Autowired
    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.on();
    }

    @Override
    public void undo() {
        light.off();
    }
}
```

</details>

<details>

<summary>LightOffCommand 클래스 (전등 끄기 명령)</summary>

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class LightOffCommand implements Command {

    private final Light light;

    @Autowired
    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.off();
    }

    @Override
    public void undo() {
        light.on();
    }
}
```

</details>

<details>

<summary>5. RemoteControl 클래스 (발신자)</summary>

```java
public class RemoteControl {
    private Command[] onCommands;
    private Command[] offCommands;
    private Command undoCommand;

    public RemoteControl() {
        onCommands = new Command[7];
        offCommands = new Command[7];
        undoCommand = new NoCommand(); // 초기에는 아무 명령도 없음
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void pressOnButton(int slot) {
        onCommands[slot].execute();
        undoCommand = onCommands[slot];
    }

    public void pressOffButton(int slot) {
        offCommands[slot].execute();
        undoCommand = offCommands[slot];
    }

    public void pressUndoButton() {
        undoCommand.undo();
    }
}
```

</details>

<details>

<summary>RemoteLoader 클래스 (클라이언트)</summary>

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RemoteLoader implements CommandLineRunner {

    private final RemoteControl remoteControl;
    private final LightOnCommand lightOnCommand;
    private final LightOffCommand lightOffCommand;

    @Autowired
    public RemoteLoader(RemoteControl remoteControl, LightOnCommand lightOnCommand, LightOffCommand lightOffCommand) {
        this.remoteControl = remoteControl;
        this.lightOnCommand = lightOnCommand;
        this.lightOffCommand = lightOffCommand;
    }

    public static void main(String[] args) {
        SpringApplication.run(RemoteLoader.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        remoteControl.setCommand(0, lightOnCommand, lightOffCommand);

        // 전등을 켭니다
        remoteControl.pressOnButton(0);
        // 전등을 끕니다
        remoteControl.pressOffButton(0);
        // 마지막 명령 취소 (전등 켜기)
        remoteControl.pressUndoButton();
    }
}
```

</details>





<details>

<summary>스마트 홈 애플리케이션 개발</summary>

```java
public class SmartHomeRemoteControl {
    private Command[] onCommands;
    private Command[] offCommands;

    public SmartHomeRemoteControl() {
        onCommands = new Command[5];
        offCommands = new Command[5];
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void pressOnButton(int slot) {
        onCommands[slot].execute();
    }

    public void pressOffButton(int slot) {
        offCommands[slot].execute();
    }
}
```

이와 같은 코드 구조를 통해, 스마트 홈 시스템에서 다양한 장치를 손쉽게 제어할 수 있으며, 새로운 장치가 추가되더라도 확장이 용이하다.

</details>

