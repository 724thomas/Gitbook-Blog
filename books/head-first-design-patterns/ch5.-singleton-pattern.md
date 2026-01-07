---
description: 싱글톤 패턴
---

# Ch5. Singleton Pattern

## **1. 싱글턴 알아보기**

"싱글턴 패턴은 클래스의 인스턴스가 하나만 생성되도록 제한하며, 이 인스턴스에 대한 전역적인 접근을 제공하는 디자인 패턴이다."

싱글턴 패턴(Singleton Pattern)은 객체 지향 프로그래밍에서 매우 널리 사용되는 패턴 중 하나로, 특정 클래스의 인스턴스를 하나만 생성해야 할 때 사용하는 패턴입니다. 이 패턴을 사용하면 애플리케이션 내에서 해당 클래스의 인스턴스가 오직 하나만 존재하도록 보장할 수 있습니다. 이는 시스템 내에서 공유 자원이나 공통 기능을 제공하는데 유용합니다.

핵심은 **단일 인스턴스**와 **전역 접근**입니다.

**주요 장점:**

* **전역 접근**: 싱글턴 패턴을 사용하면 애플리케이션 내 어디서든 동일한 인스턴스에 접근할 수 있습니다. 이를 통해 전역적으로 상태를 관리해야 하는 경우 매우 유용합니다.
* **메모리 절약**: 클래스의 인스턴스를 하나만 생성하기 때문에 불필요한 메모리 사용을 줄일 수 있습니다. 이는 자원을 효율적으로 관리하는 데 도움이 됩니다.
* **생성 비용 절감**: 객체 생성 비용이 높은 경우, 싱글턴 패턴을 사용하여 인스턴스를 재사용함으로써 시스템의 성능을 향상시킬 수 있습니다.
* **상태 일관성**: 싱글턴 패턴을 사용하면 하나의 인스턴스를 공유하므로, 상태 관리가 일관되게 유지될 수 있습니다. 이는 특히 설정 정보나 로그 관리와 같은 기능에 적합합니다.

**주요 단점:**

* **테스트 어려움**: 싱글턴 패턴은 테스트하기 어려운 구조를 만들 수 있습니다. 특히, 싱글턴 인스턴스가 특정 상태를 유지하는 경우, 테스트 환경에서 이 상태를 초기화하거나 관리하는 것이 어려울 수 있습니다.
* **의존성 문제**: 싱글턴 인스턴스에 너무 많은 기능이 의존하게 되면, 이 인스턴스가 여러 모듈에 걸쳐 종속성이 생기게 됩니다. 이는 모듈 간의 강한 결합을 초래할 수 있습니다.
* **멀티스레드 문제**: 앞서 다뤘듯이, 싱글턴 패턴은 멀티스레딩 환경에서 주의 깊게 구현해야 합니다. 잘못 구현하면 동기화 문제로 인해 심각한 버그가 발생할 수 있습니다.
* **구조적 제한**: 싱글턴 패턴은 클래스 설계에 제한을 가할 수 있습니다. 예를 들어, 여러 인스턴스를 생성해야 하는 경우 싱글턴 패턴이 적합하지 않을 수 있습니다.

## 2. 리틀 싱글턴

리틀 싱글턴은 싱글턴의 개념을 좀 더 작게 만든 변형으로, 전역적인 접근보다는 특정 범위 내에서 하나의 인스턴스만 유지하고 싶은 경우에 사용됩니다. 이를테면, 전역적으로 접근 가능한 객체가 아니라, 특정 클래스 내부나 모듈 내에서만 제한적으로 하나의 인스턴스를 유지하고 싶은 경우에 리틀 싱글턴 패턴이 적용될 수 있습니다.

* **싱글턴:** 애플리케이션 전체에서 하나의 인스턴스만 존재하며, 어디서든 동일한 인스턴스에 접근할 수 있습니다.
* **리틀 싱글턴:** 애플리케이션 전체가 아니라, 특정 영역(예: 클래스, 모듈)에서 하나의 인스턴스만 유지되도록 제한합니다. 전역적인 접근은 필요하지 않지만, 특정 기능 내에서 인스턴스가 공유되어야 할 때 사용됩니다.

<details>

<summary>싱글턴 패턴 예시</summary>

```java
import org.springframework.stereotype.Component;
import lombok.Getter;

@Component
@Getter
public class SingletonService {

    private static final SingletonService instance = new SingletonService();

    private SingletonService() {
        // private constructor to prevent instantiation
    }

    public static SingletonService getInstance() {
        return instance;
    }

    public void doSomething() {
        System.out.println("Doing something in SingletonService");
    }
}
```

위 코드에서는 `@Component`를 사용하여 Spring의 싱글턴 관리 기능을 활용할 수도 있지만, 직접적인 싱글턴 패턴 구현을 보여주기 위해 `getInstance()` 메서드를 통해 인스턴스를 제공하는 방식을 사용했습니다.

</details>

<details>

<summary>리틀 싱글턴 패턴 예시</summary>

리틀 싱글턴 패턴은 스프링 빈으로 특정 클래스 내부에서만 인스턴스를 하나만 유지하고 싶은 경우에 유용합니다.

```java
import org.springframework.stereotype.Service;
import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Service
public class SomeService {

    @Getter
    private final LittleSingleton littleSingleton = new LittleSingleton();

    @RequiredArgsConstructor
    private static class LittleSingleton {
        private final String value = "Hello, I'm a little singleton";

        public void doSomething() {
            System.out.println("Doing something in LittleSingleton");
        }
    }

    public void useLittleSingleton() {
        littleSingleton.doSomething();
    }
}
```

여기서 `LittleSingleton` 클래스는 `SomeService` 클래스 내부에서만 하나의 인스턴스를 유지하며, 외부에서 접근할 수 없게 설계되었습니다. 이는 특정 서비스 내에서만 인스턴스가 필요할 때 유용합니다.

</details>



## **2. 고전적인 싱글턴 패턴 구현법**

싱글턴 패턴의 고전적인 구현 방법은 간단합니다. 주로 다음과 같은 단계를 따릅니다:

1. **생성자를 `private`으로 선언:** 생성자를 외부에서 접근하지 못하도록 막아야 합니다. 이렇게 하면 클래스 외부에서 이 클래스를 사용해 인스턴스를 생성할 수 없습니다.
2. **자신의 타입을 인스턴스로 가지는 `static` 변수를 선언:** 클래스 내에서 유일한 인스턴스를 참조할 수 있도록 `static` 변수를 선언합니다.
3. **인스턴스를 반환하는 `static` 메서드를 작성:** 인스턴스를 반환하는 `getInstance()` 메서드를 작성하여, 인스턴스가 존재하지 않는 경우 생성하고, 이미 존재하는 경우 기존 인스턴스를 반환합니다.

<details>

<summary><strong>코드 예시</strong></summary>

```java
public class Singleton {
    // 유일한 인스턴스를 참조할 static 변수
    private static Singleton instance;

    // private 생성자
    private Singleton() {
    }

    // 인스턴스를 반환하는 static 메서드
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

위 코드는 간단한 싱글턴 패턴의 예시입니다. `getInstance()` 메서드를 통해 오직 하나의 인스턴스만 생성되고, 애플리케이션 어디서든 동일한 인스턴스에 접근할 수 있습니다.

</details>



## **3. 초콜릿 보일러 코드 살펴보기**

싱글턴 패턴의 활용 예로는 초콜릿 보일러(Chocolate Boiler)와 같은 시스템을 들 수 있습니다. 초콜릿 보일러는 초콜릿과 우유를 섞어 데우는 장치로, 이 과정에서 동일한 보일러가 여러 번 생성되지 않도록 해야 합니다. 여러 개의 보일러가 존재하면, 리소스 낭비가 발생하고, 최악의 경우 보일러가 엉망이 될 수 있습니다.

<details>

<summary><strong>초콜릿 보일러 싱글턴 코드 예시</strong></summary>

```java
public class ChocolateBoiler {
    private boolean empty;
    private boolean boiled;

    private static ChocolateBoiler instance;

    private ChocolateBoiler() {
        empty = true;
        boiled = false;
    }

    public static ChocolateBoiler getInstance() {
        if (instance == null) {
            instance = new ChocolateBoiler();
        }
        return instance;
    }

    public void fill() {
        if (isEmpty()) {
            empty = false;
            boiled = false;
            // 초콜릿과 우유를 채우는 로직
        }
    }

    public void boil() {
        if (!isEmpty() && !isBoiled()) {
            // 보일러 가열 로직
            boiled = true;
        }
    }

    public void drain() {
        if (!isEmpty() && isBoiled()) {
            // 초콜릿을 비우는 로직
            empty = true;
        }
    }

    public boolean isEmpty() {
        return empty;
    }

    public boolean isBoiled() {
        return boiled;
    }
}
```

이 코드에서는 `ChocolateBoiler` 클래스가 싱글턴 패턴을 사용하여 구현되었습니다. `getInstance()` 메서드를 통해 오직 하나의 보일러만 사용되도록 보장하며, `fill()`, `boil()`, `drain()` 메서드를 통해 초콜릿 보일러의 동작을 제어할 수 있습니다.

</details>



## **4. 허쉬! 초콜릿 보일러에 문제 발생**

싱글턴 패턴은 편리하고 유용하지만, 문제가 발생할 여지가 있습니다. 특히 멀티스레딩 환경에서는 여러 스레드가 동시에 `getInstance()` 메서드를 호출할 경우, 두 개 이상의 인스턴스가 생성될 수 있는 잠재적 위험이 있습니다.

예를 들어, 초콜릿 보일러에서 두 개의 스레드가 `getInstance()`를 동시에 호출하면, 둘 다 `instance`가 `null`인 상태에서 인스턴스를 생성하려고 시도할 수 있습니다. 이로 인해 두 개의 보일러가 생성되어, 시스템이 엉망이 될 수 있습니다.

<details>

<summary><strong>문제 상황 예시</strong></summary>

```java
public static ChocolateBoiler getInstance() {
    if (instance == null) {
        instance = new ChocolateBoiler();
    }
    return instance;
}
```

위의 코드에서, 두 개의 스레드가 동시에 `instance == null` 체크를 통과하게 되면, 두 개의 초콜릿 보일러가 생성될 수 있습니다.

</details>



## **5. 멀티스레딩 문제 살펴보기**

멀티스레딩 환경에서 싱글턴 패턴을 안전하게 구현하기 위해서는, 스레드 간의 동기화가 필수적입니다. 동기화(synchronization)를 통해 여러 스레드가 동시에 접근하는 상황을 제어해야 합니다. 이를 위해 `synchronized` 키워드를 사용할 수 있습니다.

<details>

<summary><strong>문제 해결을 위한 <code>synchronized</code> 코드</strong></summary>

```java
public static synchronized ChocolateBoiler getInstance() {
    if (instance == null) {
        instance = new ChocolateBoiler();
    }
    return instance;
}
```

`synchronized` 키워드를 사용하면, 동시에 여러 스레드가 접근할 수 없도록 하여 인스턴스가 안전하게 하나만 생성되도록 보장합니다.&#x20;

하지만, 모든 접근이 동기화되기때문에, 생성 이후에도 해당 Synchronized 블록이나 메서드에 접근할때마다 스레드간의 경쟁으로 인해 대기 시간이 발생할 수 있습니다.

동시에  인스턴스를 공유하는 싱글턴이 순차적으로 인스턴스를 공유하게 됩니다.

</details>



## **6. 멀티스레딩 문제 해결하기**

멀티스레딩 문제를 해결하는 또 다른 방법은 이른 초기화(Eager Initialization)와 더블 체크 잠금(Double-Checked Locking) 기법을 사용하는 것입니다.

### **6.1 이른 초기화(Eager Initialization)**&#x20;

클래스가 로드될 때 인스턴스를 미리 생성하여, 멀티스레딩 문제를 회피하는 방법입니다. 즉, 애플리케이션 시작시에 이미 인스턴스가 생성되어 준비되어 있는 방식입니다.

단점: 스레드 세이프 하지만, 애플리케이션이 실행될 때 즉시 인스턴스를 생성하기 때문에, 사용되지 않는다면 메모리 낭비가 발생할 수 있습니다. 특히, 인스턴스 생성에 큰 비용이 들거나, 해당 인스턴스가 애플리케이션의 일부 흐름에서만 필요할 때는 비효율적일 수 있습니다.

<details>

<summary><strong>이른 초기화</strong></summary>

인스턴스 생성 로직을 `static` 키워드와 함께 클래스 내부에서 선언하면 됩니다.

```java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

</details>

### **6.2 더블 체크 잠금(Double-Checked Locking)**

처음 `getInstance()` 호출 시에만 동기화를 적용하고, 이후에는 동기화를 피하는 방법입니다.

<details>

<summary>더블 체크 잠금</summary>

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

이 방법은 성능 저하를 최소화하면서도 안전하게 싱글턴 패턴을 구현할 수 있습니다.

</details>

<details>

<summary>Volatile</summary>

`volatile`은 특정 변수의 값이 여러 스레드에 의해 공유될 때 발생할 수 있는 **메모리 가시성 문제**를 해결하기 위해 사용됩니다.

**메모리 가시성 문제:**

자바는 멀티스레드 환경에서 성능을 최적화하기 위해 각 스레드마다 **캐시**를 사용할 수 있습니다. 즉, 스레드는 변수 값을 메모리에서 직접 읽는 대신, 자신만의 로컬 캐시에 저장하고, 이 캐시된 값을 사용하여 작업을 수행할 수 있습니다. 이렇게 하면 메모리 접근 비용을 줄일 수 있어 성능이 향상됩니다.

<img src="/broken/files/KobMlS218wTXzqK6nICm" alt="" data-size="original">

하지만, 이 로컬 캐시 방식 때문에 **메모리 가시성 문제**가 발생할 수 있습니다. 한 스레드가 변수 값을 변경해도 다른 스레드가 이 변경 사항을 보지 못할 수 있습니다. 이는 로컬 캐시가 최신 상태를 반영하지 않기 때문입니다.

**Volatile의 역할:**

* **직접 메모리에서 읽고 쓰기**: `volatile` 변수는 각 스레드의 캐시가 아닌, 메인 메모리에서 직접 읽고 씁니다. 따라서 한 스레드가 `volatile` 변수의 값을 변경하면, 그 변경 사항이 즉시 메모리에 반영되어 다른 스레드에서도 바로 볼 수 있게 됩니다.
* **쓰기 순서 보장**: `volatile` 키워드는 쓰기 순서를 보장합니다. 즉, 한 스레드가 `volatile` 변수에 쓰기 작업을 한 이후에는, 그 이후의 모든 쓰기 작업이 `volatile` 변수의 변경을 반영한 상태로 이루어집니다. 이는 특정 변수에 대한 쓰기 작업이 일어난 후 다른 작업이 수행되도록 강제하는 데 유용합니다.

</details>

<details>

<summary>Volatile vs Synchronized</summary>

* **`volatile`**:
  * **메모리 가시성 보장**: `volatile` 키워드는 변수의 \*\*가시성(visibility)\*\*을 보장합니다. 즉, 한 스레드가 `volatile`로 선언된 변수의 값을 변경하면, 그 변경 사항이 다른 스레드에게 즉시 반영됩니다.
  * **간단한 상태 플래그에 적합**: 주로 간단한 상태 플래그나 플래그 변수와 같은 읽기/쓰기 작업에 사용됩니다.
* **`synchronized`**:
  * **원자성 보장**: `synchronized`는 메서드나 블록을 임계 영역으로 만들어, 한 번에 하나의 스레드만 해당 코드에 접근할 수 있도록 함으로써 \*\*원자성(atomicity)\*\*을 보장합니다. 즉, 한 스레드가 임계 영역에서 작업하는 동안 다른 스레드들은 해당 블록에 들어갈 수 없습니다.
  * **복잡한 상태 관리에 적합**: 여러 변수를 조작하는 복잡한 연산이나, 원자적(atomic) 연산이 필요한 상황에서 사용됩니다.

#### 2. 동작 방식

* **`volatile`**:
  * **직접 메모리 접근**: `volatile`로 선언된 변수는 각 스레드의 로컬 캐시가 아닌 메인 메모리에서 직접 읽고 쓰기 때문에, 변경된 값이 모든 스레드에 즉시 반영됩니다.
  * **메모리 가시성 보장**: `volatile`은 변수에 대한 읽기와 쓰기 작업의 가시성을 보장하지만, 연산 자체가 원자적이지는 않습니다. 예를 들어, 단순한 값 변경은 안전하지만, `counter++` 같은 복합 연산은 원자성이 보장되지 않습니다.
* **`synchronized`**:
  * **잠금 메커니즘 사용**: `synchronized` 블록이나 메서드에 접근할 때, 해당 블록이 보호하는 객체에 대한 잠금(lock)을 얻어야 합니다. 다른 스레드는 이 잠금이 해제될 때까지 대기해야 합니다.
  * **원자성 보장**: `synchronized`는 코드 블록 내의 모든 연산이 원자적으로 수행되도록 보장합니다. 즉, 블록 내의 연산이 완료될 때까지 다른 스레드가 해당 블록에 접근하지 못합니다.
  * **메모리 가시성 보장**: `synchronized`는 가시성도 보장합니다. 한 스레드가 `synchronized` 블록을 빠져나가면, 다른 스레드가 그 블록을 들어갔을 때, 앞선 스레드의 모든 변경 사항이 메모리에 반영되어 있습니다.

#### 3. 성능

* **`volatile`**:
  * **경량**: `volatile`은 잠금(lock)을 사용하지 않기 때문에, `synchronized`에 비해 오버헤드가 적습니다.
  * **빠른 접근**: `volatile` 변수에 대한 접근은 `synchronized`보다 빠릅니다. 단순한 가시성 문제를 해결할 때 사용하기 좋습니다.
* **`synchronized`**:
  * **비용이 더 큼**: `synchronized`는 잠금 메커니즘을 사용하므로, 성능 저하를 초래할 수 있습니다. 특히, 여러 스레드가 동일한 블록에 자주 접근하는 경우 대기 시간이 길어질 수 있습니다.
  * **데드락 위험**: 잠금(lock)을 잘못 관리하면, 데드락(deadlock)과 같은 문제가 발생할 수 있습니다.

</details>

## **7. 더 효율적인 멀티스레딩 문제 해결하기**

이른 초기화와 더블 체크 잠금 외에도, 다양한 최적화 기법이 존재합니다. 그 중 하나는 내부 클래스(Inner Class)를 이용하는 방법입니다. 내부 클래스는 클래스가 로드될 때 초기화되지 않고, `getInstance()`가 호출될 때 비로소 인스턴스가 생성됩니다.

<details>

<summary><strong>내부 클래스 활용 예시</strong></summary>

```java
public class Singleton {
    private Singleton() {
    }

    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

내부 클래스를 활용한 이 방법은 간결하면서도 효율적입니다. 클래스 로딩 시점과 인스턴스 생성 시점을 분리할 수 있어, 멀티스레딩 환경에서도 안전하게 사용할 수 있습니다.

</details>



## 8. 싱글턴 패턴의 변형

* **멀티톤 패턴(Multiton Pattern):** 싱글턴 패턴의 변형으로, 여러 개의 인스턴스를 관리하지만 인스턴스의 수는 미리 정의된 제한된 개수로 제한됩니다.
* **레이지 싱글턴(Lazy Singleton):** 초기화 비용이 높은 경우, 인스턴스를 미리 생성하지 않고 처음 사용할 때 생성하는 패턴입니다. 더블 체크 잠금이나 이른 초기화가 이러한 패턴에 해당합니다.
* **레지스트리 싱글턴(Registry Singleton):** 싱글턴 인스턴스를 레지스트리에 저장하고, 필요한 경우 레지스트리에서 해당 인스턴스를 가져오는 방식입니다.



## 9. 질문

### 9.1 모든 메서드와 변수를 static으로 선언해서 클래스를 만들면 되지 않나요? 결과적으로 싱글턴 패턴을 사용하는 것과 똑같을 것 같은데요?

모든 메서드와 변수를 `static`으로 선언한 클래스는 싱글턴 패턴과 유사해 보일 수 있지만, 두 가지 중요한 차이가 있습니다. \
첫째, `static`으로 선언된 클래스는 **인스턴스화**가 필요하지 않으며, 전역 상태를 가진 객체가 없기 때문에 **상속**이나 **다형성**을 활용할 수 없습니다. 반면, 싱글턴 클래스는 특정 객체의 인스턴스가 하나만 존재한다는 점에서 다른 객체지향적 설계를 유지할 수 있습니다.&#x20;

<details>

<summary>static 메서드와 상속</summary>

<pre class="language-java"><code class="lang-java"><strong>class Parent {
</strong>    public static void staticMethod() {
        System.out.println("Parent static method");
    }
    
    public void instanceMethod() {
        System.out.println("Parent instance method");
    }
}

class Child extends Parent {
    // static 메서드는 오버라이드가 아니라, 그냥 숨기기(Hiding)
    public static void staticMethod() {
        System.out.println("Child static method");
    }
    
    @Override
    public void instanceMethod() {
        System.out.println("Child instance method");
    }
}

public class Main {
    public static void main(String[] args) {
        Parent parent = new Parent();
        Parent child = new Child();
        
        // static 메서드는 인스턴스 타입에 상관없이 선언된 클래스 타입에 따라 실행됨
        parent.staticMethod(); // "Parent static method"
        child.staticMethod();  // "Parent static method" (Child가 아니라 Parent의 메서드가 호출됨)

        parent.instanceMethod(); // "Parent instance method"
        child.instanceMethod();  // "Child instance method" (상속된 instance method는 오버라이딩됨)
    }
}
</code></pre>

위 예시에서 `staticMethod()`는 오버라이딩되지 않고, **숨겨지기(Hiding)** 때문에 자식 클래스에서 `staticMethod()`를 재정의해도 부모 클래스의 `staticMethod()`가 호출됩니다. 반면에, `instanceMethod()`는 자식 클래스에서 오버라이딩되어 실제로 `Child` 클래스의 메서드가 호출됩니다.

`static` 클래스나 메서드는 특정 인스턴스에 종속되지 않기 때문에, \*\*클래스 계층구조 내에서 상속을 통해 오버라이딩(Overriding)\*\*하는 것이 불가능합니다.

</details>

<details>

<summary>다형성과 static</summary>

`static`으로 선언된 클래스는 인스턴스화되지 않으며, 다형성을 구현하기 위해서는 **객체의 인스턴스**가 필요합니다. 하지만, `static` 클래스 또는 메서드는 클래스 레벨에서 작동하므로, 다형성을 적용할 수 없습니다.

```java
abstract class Animal {
    public abstract void sound();
}

class Dog extends Animal {
    @Override
    public void sound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    @Override
    public void sound() {
        System.out.println("Meow");
    }
}

public class Main {
    public static void makeSound(Animal animal) {
        animal.sound(); // 다형성: 런타임에 적절한 메서드가 호출됨
    }

    public static void main(String[] args) {
        Animal dog = new Dog();
        Animal cat = new Cat();

        makeSound(dog); // "Bark"
        makeSound(cat); // "Meow"
    }
}
```

위 예시에서는 `Animal`이라는 부모 클래스를 상속받은 `Dog`와 `Cat` 클래스가 각각의 `sound()` 메서드를 오버라이드하고 있습니다. `makeSound()` 메서드는 인자로 `Animal` 타입의 객체를 받지만, 실제로는 `Dog`나 `Cat` 객체에 따라 다르게 동작하는 **다형성**을 보여줍니다.

만약 `sound()` 메서드가 `static`으로 선언되었다면, 이러한 다형성을 사용할 수 없게 됩니다. 대신 클래스 타입에 따라 메서드가 정적으로 호출되기 때문에, 항상 부모 클래스의 메서드가 호출되거나, 자식 클래스의 메서드가 호출되더라도 런타임에 동적으로 바뀌지 않습니다.

</details>

\
둘째, `static` 클래스는 언제든지 메모리에 로드될 수 있어 \*\*지연 초기화(lazy initialization)\*\*가 불가능합니다. 싱글턴 패턴은 인스턴스를 실제로 필요할 때 생성할 수 있어, 리소스를 효율적으로 관리할 수 있습니다.





### 9.2 클래스 로더와 관련된 문제는 없나요? 클래스 로더가 각각 다른 싱글턴의 인스턴스를 가질 수 있다는 얘기를 들었거든요.

네, 클래스 로더와 관련된 문제가 있을 수 있습니다. 자바에서 클래스 로더는 JVM이 클래스를 메모리에 로드하는 역할을 합니다.&#x20;

만약 애플리케이션에 여러 개의 클래스 로더가 존재한다면, 각 클래스 로더는 동일한 클래스를 별도로 로드하여 **서로 다른 인스턴스**를 생성할 수 있습니다. 이로 인해, 싱글턴 패턴의 본래 목적이 무색해질 수 있습니다. 특히, 애플리케이션 서버와 같은 환경에서 클래스 로더가 여러 개일 경우, 의도치 않게 여러 개의 싱글턴 인스턴스가 생성될 수 있습니다. 이 문제를 해결하기 위해서는 **정적 팩토리 메서드**나 **서비스 로더**를 사용해 싱글턴 인스턴스를 관리하는 방법을 고려할 수 있습니다.

### 9.3 리플렉션, 직렬화, 역직렬화 문제도 있지 않나요?

*   **리플렉션 문제**: 리플렉션을 사용하면 클래스의 `private` 생성자에 접근할 수 있기 때문에, 싱글턴 클래스의 인스턴스를 여러 개 생성할 수 있는 위험이 있습니다. 이를 방지하려면, 싱글턴 클래스의 생성자에서 리플렉션을 통한 새로운 인스턴스 생성 시도에 대해 예외를 발생시키는 방법이 필요합니다.

    ```java
    private Singleton() {
        if (instance != null) {
            throw new IllegalStateException("이미 인스턴스가 존재합니다.");
        }
    }
    ```
*   **직렬화 및 역직렬화 문제**: 직렬화와 역직렬화는 객체를 바이트 스트림으로 변환하고, 다시 객체로 복원하는 과정입니다. 싱글턴 클래스가 직렬화된 후 역직렬화되면 새로운 인스턴스가 생성되어 **싱글턴 원칙**이 깨질 수 있습니다. 이를 방지하기 위해서는 `readResolve()` 메서드를 오버라이드하여, 역직렬화 시 기존의 싱글턴 인스턴스를 반환하도록 해야 합니다.

    ```java
    private Object readResolve() {
        return getInstance();
    }
    ```

### 9.4 싱글턴은 느슨한 결합 원칙에 위배되지 않나요? Singleton에 의존하는 객체는 전부 하나의 객체에만 결합된 것 아닌가요?

싱글턴 패턴은 종종 **느슨한 결합 원칙**에 위배될 수 있다는 지적을 받습니다. 이는 싱글턴 인스턴스에 의존하는 객체들이 모두 동일한 인스턴스에 강하게 결합되기 때문입니다. 이런 결합은 객체 지향 설계에서 **유연성**을 저하시킬 수 있습니다. 예를 들어, 테스트나 확장성을 고려할 때 문제가 될 수 있습니다. 이를 해결하기 위해서는 싱글턴 객체를 인터페이스로 추상화하거나, 의존성 주입(Dependency Injection)을 통해 결합도를 낮출 수 있습니다. 이렇게 하면, 실제 인스턴스를 테스트 목적으로 \*\*모의 객체(Mock Object)\*\*로 대체할 수 있어 테스트 용이성을 높일 수 있습니다.

## 9.5 싱글턴이 서브클래스를 만들어도 되는 건가요?

싱글턴 클래스의 서브클래스를 만드는 것은 일반적으로 권장되지 않습니다. 싱글턴의 목적은 하나의 인스턴스만 존재하도록 보장하는 것이기 때문에, 서브클래스를 생성하면 이러한 제약이 깨질 수 있습니다. 특히, 서브클래스가 추가로 인스턴스를 만들 수 있는 상황에서는 싱글턴의 의미가 무색해집니다. 만약 서브클래스를 반드시 만들어야 한다면, 서브클래스의 생성자를 `protected`로 선언하고, 서브클래스도 싱글턴으로 구현하여 **상속받은 클래스들 역시 하나의 인스턴스만 가지도록** 해야 합니다. 그러나, 이런 구조는 복잡성을 증가시킬 수 있으므로 가능하면 피하는 것이 좋습니다.

