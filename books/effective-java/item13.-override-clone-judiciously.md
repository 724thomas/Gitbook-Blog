---
description: Clone 재정의는 주의해서 진행해라
---

# Item13. Override Clone Judiciously

## 요약

* **`Cloneable`의 문제점**: `Cloneable` 인터페이스는 `clone` 메서드가 `Object` 클래스에 있는 것과 다르게, `protected`로 선언되어 있어 외부에서 접근할 수 없고, 복제를 제대로 수행하지 못하는 경우가 많습니다.
* **`clone` 메서드의 오작동 가능성**: `clone` 메서드는 깊은 복사를 수행하지 않기 때문에, 객체 내부의 참조 필드가 원본과 복제본에서 동일한 객체를 가리키게 되어 문제가 발생할 수 있습니다.
* **코바리언트 반환 타입과 복잡성**: 재정의된 `clone` 메서드는 코바리언트 반환 타입(covariant return typing)을 지원해야 하며, 이를 구현하는 것이 어려울 수 있습니다.
* **깊은 복사의 필요성**: 단순히 얕은 복사를 하는 것보다는, 객체 내의 모든 참조 필드를 복제하여 완전한 독립적인 객체를 만드는 깊은 복사 방식을 사용해야 합니다.
* **다른 대안들**: `Cloneable` 대신 복사 생성자나 복사 팩토리 메서드를 사용하는 것이 더 나은 방법일 수 있습니다. 이 방법들은 `clone` 메서드의 복잡한 문제들을 피할 수 있으며, 객체 복제를 보다 명확하게 할 수 있습니다.



## Cloneable 인터페이스와 Clone 메서드의 개요

`Cloneable` 인터페이스는 자바에서 객체 복제를 지원하기 위해 사용됩니다. `Cloneable`을 구현한 클래스는 `Object`의 `clone` 메서드를 사용할 수 있으며, 이를 통해 객체의 복제본을 생성할 수 있습니다. 하지만, `Cloneable` 인터페이스 자체는 복제를 위한 별도의 메서드를 정의하지 않으며, 단지 `clone` 메서드가 `CloneNotSupportedException`을 발생시키지 않도록 보장할 뿐입니다.

즉, `Cloneable` 인터페이스는 단순히 복제 가능성을 알리는 역할만 하며, `clone` 메서드가 실제로 어떻게 동작할지에 대한 구체적인 정의는 클래스의 책임입니다. 이로 인해 `clone` 메서드를 잘못 구현하면 의도치 않은 버그가 발생할 가능성이 높습니다.



## Clone 메서드의 문제점: 얕은 복사와 깊은 복사

자바의 `clone` 메서드는 기본적으로 얕은 복사(shallow copy)를 수행합니다. 얕은 복사는 객체의 필드 값만 복사하며, 필드가 참조하는 객체들은 복사하지 않습니다. 이러한 얕은 복사는 간단한 데이터 클래스에서는 문제가 없지만, 필드가 다른 객체를 참조하는 경우, 원본 객체와 복제된 객체가 동일한 객체를 참조하게 되어 의도치 않은 결과를 초래할 수 있습니다.

### **얕은 복사의 예시**

예를 들어, `Stack` 클래스에서 `clone` 메서드를 단순히 `super.clone()`을 사용해 구현했다고 가정해보겠습니다.

```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size;

    @Override
    public Stack clone() {
        try {
            return (Stack) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // 일어날 수 없는 일입니다.
        }
    }
}
```

이 경우, `elements` 배열은 얕은 복사만 이루어집니다. 즉, 복제된 스택의 `elements` 배열은 원본 스택과 동일한 배열을 참조하게 됩니다. 따라서 하나의 스택에서 배열의 요소를 변경하면, 다른 스택에서도 그 변경이 반영됩니다.

### **깊은 복사 구현**

이를 방지하려면, `clone` 메서드를 재정의하여 깊은 복사를 구현해야 합니다. 깊은 복사는 객체 내부의 참조 필드까지 모두 새롭게 복사하여, 원본 객체와 복제된 객체가 완전히 독립적인 객체가 되도록 합니다.

```java
j@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // 일어날 수 없는 일입니다.
    }
}
```

위 코드에서는 `elements` 배열도 복제하여 새로운 배열을 생성합니다. 이를 통해 원본과 복제본이 각각 독립적인 배열을 가지게 되어, 하나의 스택에서 배열을 수정해도 다른 스택에 영향을 미치지 않습니다.



## 상속 구조에서의 Clone 메서드 재정의의 복잡성

상속 구조에서 `clone` 메서드를 재정의하는 것은 특히 어렵습니다. 상위 클래스에서 `clone` 메서드를 재정의하고, 이를 하위 클래스에서 사용하면 예상치 못한 결과를 초래할 수 있습니다.

### **코바리언트 반환 타입의 문제**

자바의 `clone` 메서드는 기본적으로 `Object` 타입을 반환합니다. 하지만, 실제로 반환할 때는 하위 클래스의 타입으로 반환해야 합니다. 이를 해결하기 위해, 하위 클래스에서는 `super.clone()`을 호출한 후, 반환 타입을 하위 클래스 타입으로 형변환해야 합니다. 이를 코바리언트 반환 타입(covariant return type)이라고 합니다.

```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

이 방식은 하위 클래스의 `clone` 메서드가 올바른 타입을 반환하도록 보장합니다. 하지만, 하위 클래스가 상위 클래스의 `clone` 메서드를 그대로 사용할 경우, 상위 클래스 타입으로 반환되므로 원하는 결과를 얻지 못할 수 있습니다.



## clone 메서드의 구현에서 발생하는 문제들

`clone` 메서드를 올바르게 구현하기 위해서는 여러 가지 문제를 해결해야 합니다. 특히, 상속 구조에서 이를 구현할 때, 상위 클래스의 상태와 하위 클래스의 상태를 모두 고려해야 합니다.

### **내부 객체의 상태 유지**

객체가 복잡한 내부 구조를 가지고 있을 때, 모든 내부 객체를 정확히 복사해야 합니다. 예를 들어, `HashTable` 클래스는 내부적으로 `Entry` 객체들을 사용하여 해시 테이블을 구성합니다. 이 `Entry` 객체들도 복사해야 하며, 이는 일반적인 `clone` 메서드로는 충분하지 않을 수 있습니다.

```java
public class HashTable implements Cloneable {
    private Entry[] buckets;

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null) {
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    private static class Entry {
        Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }
}
```

위 코드에서 `Entry` 객체들은 재귀적으로 복사되어, 원본과 복제본이 독립적인 객체로 유지됩니다. 하지만, 이 방법은 성능상의 문제가 있을 수 있습니다. 특히, 깊은 복사를 수행할 때, 객체 구조가 복잡하면 성능이 급격히 저하될 수 있습니다.



## Clone 메서드 재정의를 피하는 대안들

위와 같은 문제들로 인해 `Cloneable` 인터페이스와 `clone` 메서드를 사용하지 않고, 다른 방법을 고려하는 것이 좋습니다. 복사 생성자와 복사 팩토리 메서드는 이러한 대안 중 하나로, `clone` 메서드가 가지는 여러 단점을 피할 수 있습니다.

**복사 생성자**

복사 생성자는 기존 객체를 인자로 받아, 그 객체를 복사한 새로운 객체를 생성하는 생성자입니다. 이 방법은 명시적이며, 코드가 명확해지는 장점이 있습니다.

```java
public class Yum {
    private int yum;

    public Yum(Yum yum) {
        this.yum = yum.yum;
    }
}
```

복사 생성자는 객체의 상태를 명확하게 정의할 수 있으며, 깊은 복사를 쉽게 구현할 수 있습니다. 또한, 상속 구조에서도 문제가 발생하지 않으며, 객체의 타입이 명확하게 유지됩니다.

### **복사 팩토리 메서드**

복사 팩토리 메서드는 정적 메서드로, 주어진 객체를 복사하여 새로운 객체를 반환합니다. 이 방법 역시 명시적이고 코드가 명확해지는 장점이 있습니다.

```java
public class Yum {
    private int yum;

    public static Yum newInstance(Yum yum) {
        return new Yum(yum);
    }
}
```

복사 팩토리 메서드는 복사 생성자와 유사하지만, 정적 메서드를 사용함으로써 객체 생성 로직을 좀 더 유연하게 관리할 수 있습니다. 예를 들어, 팩토리 메서드 내부에서 캐싱을 구현하거나, 다른 방법으로 객체 생성을 최적화할 수 있습니다.



## 결론

`Cloneable` 인터페이스와 `clone` 메서드는 자바에서 객체를 복제하는 전통적인 방법이지만, 여러 가지 문제점을 가지고 있습니다. 얕은 복사로 인해 발생하는 참조 공유 문제, 상속 구조에서의 복잡성, 성능 저하 등은 이 방법을 사용하는 데 있어 큰 제약이 됩니다.

따라서, `Cloneable` 인터페이스를 구현하고 `clone` 메서드를 재정의하기보다는, 복사 생성자나 복사 팩토리 메서드를 사용하는 것이 더 나은 선택일 수 있습니다. 이 방법들은 객체 복제를 보다 명확하고 안전하게 수행할 수 있으며, 코드의 가독성과 유지보수성을 높이는 데 기여합니다. 최종적으로, `clone` 메서드를 사용해야 한다면 깊은 복사와 상속 구조에서의 문제를 충분히 이해하고 주의 깊게 재정의하는 것이 중요합니다.
