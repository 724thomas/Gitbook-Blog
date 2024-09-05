---
description: 가변 vs 불변
---

# Mutable vs Immutable

## 1. 정의

### 1.2 Mutable

* 가변
* 상태 변경시 재사용 가능하여 메모리 효율적
* Data race 문제 발생 가능
* 값을 수정할 수 있어서 유연합니다

### 1.3 Immutable

* 불변
* 상태 변경시 새로운 객체 생성되어 메모리 비효율적
* Thread-safe
* 유연성이 떨어지지만, 예측 가능한 동작



## 질문

### 왜 Immutable 객체는 Thread-safe 한가요?

Immutable 객체는 한 번 생성되면 상태가 절대로 변하지 않으며, 이를 보장하기 위해 필드들이 **final**로 선언되어 있습니다. 이로 인해 객체의 초기 상태가 설정된 후에는 변경될 수 없기 때문에, **여러 스레드가 동시에 이 객체를 참조하더라도 상태 변동으로 인한 충돌이나 불일치가 발생하지 않습니다**. 즉, 스레드 간에 **동기화를 할 필요 없이** 객체를 안전하게 공유할 수 있습니다. 이로 인해 Immutable 객체는 기본적으로 스레드 안전(Thread-safe)하다고 할 수 있습니다.



### Immutable 객체는 성능 문제를 일으킬 수 있다는 단점이 있습니다. 이유가 뭔가요?

Immutable 객체는 불변이기 때문에 상태 변경이 필요할 때마다 **새로운 객체를 생성해야** 합니다. 이는 메모리 사용량이 증가하고, 특히 **상태 변경이 빈번한 경우** 객체가 자주 생성되어 **가비지 컬렉션이 자주 발생**함으로써 성능 저하를 초래할 수 있습니다. 또한, **객체 생성 자체가 성능 상의 비용**을 초래할 수 있습니다. 그러나 **변경이 자주 발생하지 않는 값**이나 **캐싱을 활용한 재사용**을 통해 성능 문제를 최소화할 수 있습니다.



### Java에서 String은 왜 Immutable로 설계됐나요?

첫째, **String Pool**을 활용하여 **메모리 사용을 최적화. 새로운 객체를 생성하지 않고** 동일한 참조 주소를 공유함으로써 메모리 낭비를 줄입니다.

둘째, `String`의 불변성은 **보안성**을 제공합니다.

셋째, **해시값을 캐싱**할 수 있어 `String` 객체가 많이 사용되는 **HashMap** 등의 자료구조에서 성능을 향상. Hash라는 정수형 필드를 가지고 있고, 처음 hashCode()가 호출될때 해시값을 필드에 저장합니다. (HashMap, HashSet, HashTable같은 해시 기반 자료구조에 저장되거나 검색될때 사용)



### 자바에서 불변 객체를 만들기 위해 어떤 방법들을 사용할 수 있나요?

클래스를 final로 선언하여 상속을 통해 변경되지 않도록 하거나, 모든 필드를 private하고 final로 선언하여 한 번 초기화된 후 값을 변경할 수 없게 합니다.

또는, 생성자에서 모든 필드를 초기화하고, 객체가 생성된 이후에는 필드를 변경할 수 없게 합니다. Setter 메서드도 제공하지 않습니다.

가변 필드(Mutable Object)를 참조하는 경우, 해당 필드를 복사하여 내부에서 사용하거나 방어적 복사(객체의 복사본을 반환)를 적용하여 외부에서 수정할 수 없도록 합니다.&#x20;



### 가변 객체를 불변 객체처럼 사용하기 위해 어떤 기법을 사용할 수 있을까요?

방어적 복사를 사용할 수 있습니다.

불변 래퍼(Immutable Wrapper)를 사용하여 가변 객체를 불변 객체처럼 감싸는 래퍼 클래스를 만들어 가변 객체의 메서드를 제한하여 상태를 변경할 수 없게 합니다.

<details>

<summary>Collections.unmodifiableList()</summary>

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ImmutableWrapperExample {
    public static void main(String[] args) {
        // 가변 리스트 생성
        List<String> mutableList = new ArrayList<>();
        mutableList.add("Apple");
        mutableList.add("Banana");

        // 불변 리스트로 감싸기 (불변 래퍼)
        List<String> immutableList = Collections.unmodifiableList(mutableList);

        // 리스트 내용을 읽을 수는 있지만
        System.out.println("불변 리스트: " + immutableList);

        // 수정하려고 하면 UnsupportedOperationException 발생
        try {
            immutableList.add("Orange");
        } catch (UnsupportedOperationException e) {
            System.out.println("리스트는 불변이라 수정할 수 없습니다.");
        }
    }
}
```

위 코드에서 `mutableList`는 가변 객체지만, `Collections.unmodifiableList()` 메서드를 사용하여 **불변 래퍼**로 감싸서 `immutableList`는 더 이상 수정할 수 없는 리스트가 됩니다. 수정 시도는 `UnsupportedOperationException`을 발생시킵니다.

</details>

불변 인터페이스를 구현하여 가변 객체의 상태를 숨기고, 상태를 직접 변경할 수 없도록 인터페이스나 추상 클래스를 통해 불변 인터페이스를 제공합니다.

<details>

<summary>Immutable Interface</summary>

```java
interface ReadOnlyUser {
    String getName();
    int getAge();
}

class User implements ReadOnlyUser {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 상태를 변경할 수 있는 setter 메서드는 제공되지 않음

    @Override
    public String getName() {
        return name;
    }

    @Override
    public int getAge() {
        return age;
    }

    // 상태를 변경할 수 있는 메서드를 내부적으로만 사용
    public void setAge(int age) {
        this.age = age;
    }
}

public class ImmutableInterfaceExample {
    public static void main(String[] args) {
        ReadOnlyUser user = new User("John", 25);

        // 읽기만 가능
        System.out.println("이름: " + user.getName());
        System.out.println("나이: " + user.getAge());

        // ReadOnlyUser 타입에서는 상태 변경 불가
        // user.setAge(30); // 컴파일 에러 발생
    }
}
```

위 예시에서 `ReadOnlyUser` 인터페이스는 **getter**만을 제공하여 객체의 상태를 읽을 수 있지만, **setter** 메서드를 노출하지 않으므로 외부에서 상태를 변경할 수 없습니다. `User` 클래스는 내부적으로는 `setAge()` 메서드를 사용할 수 있지만, **인터페이스를 통해 외부에 노출된 객체는 불변**처럼 사용할 수 있습니다.

</details>



### 가변 객체와 불변 객체를 사용하는 상황에서 성능을 어떻게 고려해야할까요?

**여러 스레드가 동일한 객체를 공유하는 상황**에서는 데이터 레이스 문제가 발생할 수 있기 때문에, 불변 객체가 선호됩니다. 불변 객체는 상태 변경이 없으므로 **스레드 안전성을 보장**하고, 동기화 비용을 절약할 수 있습니다.

그러나, 불변 객체는 **상태가 변경될 때마다 새로운 객체를 생성**해야 하기 때문에, **메모리 사용량 증가**와 **객체 생성 비용**이 성능에 영향을 미칠 수 있습니다. 이런 성능 문제를 완화하기 위해 **객체 재사용**이나 **캐싱** 기법을 활용하거나, **상태 변화를 최소화하는 설계**를 적용할 수 있습니다.

**예를 들어**, 상태 변경이 빈번한 상황에서는 가변 객체를 사용하고, 멀티스레드 환경에서는 불변 객체를 사용하여 성능과 안정성을 적절히 균형 있게 고려하는 것이 중요합니다. 또한, **스트림 기반 처리**나 **Lombok의 빌더 패턴**을 활용하여 불변 객체를 효율적으로 생성하는 방법도 성능 최적화에 도움이 됩니다.
