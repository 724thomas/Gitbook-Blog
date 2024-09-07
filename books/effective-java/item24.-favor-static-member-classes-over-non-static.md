---
description: '아이템 24: 멤버 클래스는 되도록 Static으로 만들라'
---

# Item24. Favor Static Member Classes Over Non-Static

**"멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자"**

자바에서 클래스 안에 선언된 클래스는 멤버 클라스라고 하고, 멤버 클래스는 바깥 클래스의 일부로 정의됩니다. 멤버 클래스는 크게 static 중첩 클래스와 non-static 중첩 클래스(내부 클래스)로 나뉩니다.



## 1. 멤버 클래스의 종류

멤버 클래스는 외부클래스 안에 정의된 중첩 클래스입니다.

### 1.1. 정적 멤버 클래스 (Static Member Class)

정적 멤버 클래스는 외부 클래스의 인스턴스와 **독립적**으로 전재할 수 있습니다. 바깥 클래스와 직접적인 연관이 없으며, 주로 외부 클래스의 논리적인 하위 개념을 표현하는 데 사용됩니다.

Static으로 선언된 중첩 클래스는 외부 클래스의 인스턴스에 접근할 수 없습니다.

<details>

<summary>Example</summary>

```java
public class OuterClass {
    private static String staticMessage = "Hello from OuterClass";

    // 정적 멤버 클래스
    public static class StaticInnerClass {
        public void printMessage() {
            System.out.println(staticMessage);  // 외부 클래스의 정적 멤버에만 접근 가능
        }
    }

    public static void main(String[] args) {
        // 외부 클래스 인스턴스 없이 정적 멤버 클래스 사용
        OuterClass.StaticInnerClass inner = new OuterClass.StaticInnerClass();
        inner.printMessage();  // 출력: Hello from OuterClass
    }
}
```

</details>

### 1.2. 비정적 멤버 클래스 (Non-static Member Class)

비정적 멤버 클래스는 외부 클래스의 인스턴스에 속한 클래스로, 외부 클래스의 인스턴스와 연결되어 있습니다.

<details>

<summary>Example</summary>

```java
public class OuterClass {
    private String instanceMessage = "Hello from OuterClass";

    // 비정적 멤버 클래스
    public class InnerClass {
        public void printMessage() {
            System.out.println(instanceMessage);  // 외부 클래스의 인스턴스 필드에 접근 가능
        }
    }

    public static void main(String[] args) {
        OuterClass outer = new OuterClass();      // 외부 클래스 인스턴스 생성
        OuterClass.InnerClass inner = outer.new InnerClass();  // 외부 인스턴스를 통해 내부 클래스 생성
        inner.printMessage();  // 출력: Hello from OuterClass
    }
}
```

</details>

* 외부 클래스의 인스턴스 없이 생성할 수 없습니다.
* 외부 클래스의 인스턴스에 자동으로 숨겨진 참조를 가집니다.\
  비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있습니다. (클라스명.this)
* 외부 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없습니다. \
  이 관계는 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할때 자동으로 만들어지는게 보통이지만, 가끔 클래스.new Member Class(arg)를 호출하는 경우도 있습니다.
* 어댑터를 정의할 때 자주 쓰입니다. \
  외부 클래스의 인스턴스와 강하게 결합된 상태에서, 해당 클래스의 기능을 적절하게 변환해 다른 인터페이스에 맞게 사용할 수 있기 떄문입니다. 비정적 멤버 클래스는 **외부 클래스의 인스턴스 필드나 메서드**에 접근할 수 있으므로, 외부 클래스의 데이터를 변환하거나 조작해야 할 때 유리합니다.

<details>

<summary>Adapter</summary>

```java
// 기존의 호환되지 않는 인터페이스
interface MediaPlayer {
    void play(String audioType, String fileName);
}

// 새로운 인터페이스
class AdvancedMediaPlayer {
    public void playMp4(String fileName) {
        System.out.println("Playing mp4 file: " + fileName);
    }

    public void playVlc(String fileName) {
        System.out.println("Playing vlc file: " + fileName);
    }
}

// MediaPlayer 인터페이스를 구현하는 외부 클래스
public class MediaAdapter {
    private AdvancedMediaPlayer advancedPlayer;

    public MediaAdapter() {
        this.advancedPlayer = new AdvancedMediaPlayer();  // 외부 클래스에서 사용
    }

    // 비정적 멤버 클래스를 어댑터로 사용
    public class InnerMediaPlayer implements MediaPlayer {
        @Override
        public void play(String audioType, String fileName) {
            if (audioType.equalsIgnoreCase("mp4")) {
                advancedPlayer.playMp4(fileName);  // AdvancedMediaPlayer의 메서드를 사용
            } else if (audioType.equalsIgnoreCase("vlc")) {
                advancedPlayer.playVlc(fileName);
            } else {
                System.out.println("Invalid media format: " + audioType);
            }
        }
    }
}

// 클라이언트 코드
public class AdapterPatternExample {
    public static void main(String[] args) {
        // 어댑터의 비정적 멤버 클래스 인스턴스를 사용
        MediaAdapter adapter = new MediaAdapter();
        MediaPlayer player = adapter.new InnerMediaPlayer();

        player.play("mp4", "video.mp4");  // 출력: Playing mp4 file: video.mp4
        player.play("vlc", "movie.vlc");  // 출력: Playing vlc file: movie.vlc
        player.play("avi", "film.avi");   // 출력: Invalid media format: avi
    }
}
```

**비정적 멤버 클래스**를 사용해 외부 클래스의 메서드를 **다른 인터페이스에 맞게 변환**하는 어댑터

</details>



### 1.3. 익명 클래스 (Anonymous Class)

익명 클래스는 이름이 없는 클래스로, 주로 인터페이스나 클래스의 인스턴스를 즉석에서 생성할 때 사용됩니다.

<details>

<summary>Example</summary>

```java
public class AnonymousClassExample {
    public static void main(String[] args) {
        // Runnable 인터페이스를 구현하는 익명 클래스
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello from Anonymous Class");
            }
        };

        runnable.run();  // 출력: Hello from Anonymous Class

        // 한 번만 사용되는 간단한 로직을 익명 클래스로 작성
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Running in a separate thread!");
            }
        }).start();  // 출력: Running in a separate thread!
    }
}
```

</details>

### 1.4. 지역 클래스 (Local Class)

지역 클래스는 메서드 내부에 정의된 클래스입니다. 메서드의 지역 변수처럼 작동하며, 메서드가 호출될 때만 존재할 수 있습니다.

<details>

<summary>Example</summary>

```java
public class LocalClassExample {

    public void greet(String name) {
        // 메서드 내부에 정의된 지역 클래스
        class Greeter {
            public void sayHello() {
                System.out.println("Hello, " + name);  // 메서드 매개변수에 접근 가능
            }
        }

        // 지역 클래스의 인스턴스 생성 및 사용
        Greeter greeter = new Greeter();
        greeter.sayHello();  // 출력: Hello, Alice
    }

    public static void main(String[] args) {
        LocalClassExample example = new LocalClassExample();
        example.greet("Alice");
    }
}
```

</details>



## 2. 비정적 멤버 클래스의 문제점

### 2.1. 외부 클래스의 인스턴스와 강한 결합

비정적 멤버 클래스는 **외부 클래스의 인스턴스**와 강하게 결합됩니다. 이로 인해 **외부 클래스 인스턴스가 존재해야만 멤버 클래스의 인스턴스를 생성**할 수 있습니다.

### 2.2. 메모리 누수 위험

비정적 멤버 클래스는 외부 클래스에 대한 숨겨진 참조를 갖기 때문에, 외부 클래스의 인스턴스가 GC에 의해 수거되지 않는 문제가 발생할 수 있습니다.



## 3. 정적 멤버 클래스의 장점

### 3.1. 외부 클래스와 독립적

정적 멤버 클래스는 외부 클래스의 인스턴스에 접근할 수 없기 때문에, 외부 클래스와 강한 결합이 발생하지 않습니다. 이는 설계에서 책임을 분리하는 데 유리하며, 코드를 더욱 모듈화할 수 있습니다.

### 3.2. 메모리 누수 방지

정적 멤버 클래스는 외부 클래스의 인스턴스에 대한 참조를 가지지 않으므로, 외부 클래스가 수거되지 않아서 발생하는 메모리 누수 문제가 없습니다.



## 4. 언제 비정적 멤버 클래스를 사용해야 할까?

비정적 멤버 클래스는 외부 클래스의 인스턴스에 대한 참조가 필요할 때만 사용해야 합니다. 즉, 멤버 클래스가 외부 클래스의 인스턴스와 밀접한 관계를 갖고 있거나, 외부 클래스의 인스턴스에 의존하는 경우에만 비정적 멤버 클래스를 사용하는 것이 적절합니다.



## 5. 외부 클래스에 Static은 왜 안될까?

`public static class MathUtils`처럼 **클래스 자체에 `static`을 붙이는 것**은 **문법적으로 허용되지 않습니다**. 자바에서 `static`은 **내부 클래스**나 **정적 멤버**에 사용할 수 있지만, \*\*최상위 클래스(top-level class)\*\*에는 `static`을 붙일 수 없습니다.

* **최상위 클래스**는 이미 독립적으로 존재할 수 있기 때문에, **`static`이 불필요**합니다. 자바에서는 클래스가 \*\*최상위(top-level)\*\*일 경우, 그 자체로 **독립적인 엔터티**입니다. 즉, **외부 클래스의 인스턴스에 의존하지 않기 때문에** `static`이 필요하지 않습니다.
* `static`은 \*\*내부 클래스(inner class)\*\*가 외부 클래스의 인스턴스와 **독립적**으로 존재할 수 있음을 나타내기 위해 사용하는 것이므로, **최상위 클래스**에서는 사용할 필요가 없습니다.
