---
description: 이왕이면 제네릭 타입으로 만들라
---

# Item29. Prefer Generic Types

자바에서는 제네릭(Generics)을 사용하여 타입을 안전하게 관리하고, 코드의 재사용성을 높일 수 있습니다. 제네릭은 데이터 타입을 일반화하여, 특정 타입에 종속되지 않고 다양한 타입을 처리할 수 있게 해줍니다.

특히 타입 안전성을 보장하여 런타임 오류를 줄일 수 있으며, 컴파일 시점에 오류를 잡을 수 있어 더 안정적인 코드를 작성할 수 있습니다.



## 1. 제네릭 타입의 필요성

제네릭을 사용하지 않고 모든 객체를 `Object` 타입으로 처리하게 되면, 객체를 사용할 때마다 타입 캐스팅을 해야 하는 불편함이 생깁니다. 이 과정에서 잘못된 타입을 캐스팅하게 되면 `ClassCastException`이 발생할 위험이 있습니다. 제네릭은 이러한 위험을 줄이고, 타입 안정성을 보장하여 안전한 코드를 작성할 수 있도록 도와줍니다.

### 1.1. 제네릭을 사용하지 않은 클래스의 문제점

제네릭이 적용되지 않은 코드는 다음과 같은 문제를 야기할 수 있습니다.&#x20;

<details>

<summary>제네릭을 사용하지 않은 Stack 클래스</summary>

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object element) {
        ensureCapacity();
        elements[size++] = element;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 메모리 누수 방지
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위 `Stack` 클래스는 모든 데이터를 `Object` 타입으로 처리합니다. 이 때문에 `pop()` 메서드를 사용할 때마다 반환된 객체를 원하는 타입으로 캐스팅해야 하고, 잘못된 타입으로 캐스팅할 경우 런타임 에러가 발생할 수 있습니다.

</details>

### 1.2. 제네릭을 사용한 Stack 클래스의 개선

제네릭을 사용하면 위에서 언급된 타입 안전성 문제를 해결할 수 있습니다.

<details>

<summary>제네릭을 사용한 Stack 클래스</summary>

```java
public class Stack<E> {  // E는 Element의 약자로 타입 매개변수
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; // 제네릭 배열 생성시 캐스팅 필요
    }

    public void push(E element) {
        ensureCapacity();
        elements[size++] = element;
    }

    public E pop() {
        if (size == 0)
            throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // 메모리 누수 방지
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

위 코드에서 `Stack<E>`는 제네릭 타입으로, `E`는 Stack이 저장할 요소의 타입을 나타냅니다. 제네릭을 사용함으로써 `Stack` 클래스는 모든 타입에 대해 안전하게 사용할 수 있으며, 잘못된 타입 캐스팅으로 인한 런타임 에러를 방지할 수 있습니다.

</details>

### 1.3. 제네릭의 이점

* **타입 안정성**: 제네릭을 사용하면 컴파일 시점에 타입 검사가 이루어지므로, 잘못된 타입이 사용될 위험이 줄어듭니다. 런타임 에러보다 컴파일 타임에 에러를 잡는 것이 훨씬 안전합니다.
* **중복 코드 제거**: 제네릭을 사용하면 여러 타입을 처리하는 코드에서 중복을 제거할 수 있어, 코드의 재사용성이 높아집니다.
* **가독성과 유지보수성 향상**: 제네릭을 사용하면 코드의 의도가 명확해지고, 타입 캐스팅으로 인한 복잡성이 줄어들어 가독성이 향상됩니다.



## 2. 제네릭 타입 사용 시 고려 사항

### 2.1. 제네릭 배열 생성 시 주의

자바에서는 제네릭 배열을 직접 생성하는 것을 허용하지 않습니다. 이는 제네릭의 타입 정보가 런타임에 소거되기 때문에 안전하지 않기 때문입니다. 따라서 제네릭 배열을 생성할 때는 반드시 `Object` 배열을 생성한 후 타입 캐스팅을 해야 하며, 이 과정에서 `@SuppressWarnings("unchecked")` 어노테이션을 사용하여 경고를 무시해야 합니다.

```java
@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

### 2.2. 제네릭과 가변 인수(varargs)

제네릭과 가변 인수를 함께 사용할 때는 주의가 필요합니다. 제네릭 가변 인수 메서드는 힙 오염(Heap Pollution)을 일으킬 수 있으므로 `@SafeVarargs` 어노테이션을 사용하여 컴파일러 경고를 제거해야 합니다.

```java
@SafeVarargs
public static <T> void addToList(List<T> list, T... elements) {
    for (T element : elements) {
        list.add(element);
    }
}
```

### 2.3. 제네릭 타입의 한계

제네릭은 타입 소거로 인해 런타임에 타입 정보를 잃어버립니다. 따라서 제네릭 타입으로는 다음과 같은 작업을 할 수 없습니다.

1. **인스턴스 생성**: `new E()`와 같이 제네릭 타입 매개변수로 객체를 생성할 수 없습니다.
2. **정적 멤버 사용**: 제네릭 타입 매개변수는 정적 멤버(필드, 메서드)와 함께 사용할 수 없습니다.
3. **타입 검사와 형변환**: `instanceof` 연산자는 제네릭 타입에 대해 사용할 수 없으며, 제네릭 타입으로의 형변환도 제한됩니다.



## 3. 코드 예제: 제네릭 타입을 사용한 클래스 만들기

### 3.1. 제네릭 없이 작성한 Box 클래스

<details>

<summary>제네릭을 사용하지 않은 Box 클래스</summary>

```java
public class Box {
    private Object object;

    public void set(Object object) {
        this.object = object;
    }

    public Object get() {
        return object;
    }
}
```

이 코드는 단순히 `Object` 타입을 사용하기 때문에 모든 종류의 데이터를 담을 수 있습니다.&#x20;

그러나 데이터를 가져올 때마다 타입을 캐스팅해야 하는 불편함이 있고, 잘못된 타입으로 캐스팅하면 런타임 오류가 발생할 수 있습니다.

</details>

### 3.2. 제네릭을 사용한 Box 클래스

<details>

<summary>제네릭을 사용한 Box 클래스</summary>

```java
public class Box<T> {  // T는 타입 매개변수로, 박스가 담을 타입을 의미
    private T t;

    public void set(T t) {
        this.t = t;
    }

    public T get() {
        return t;
    }
}
```

위 `Box<T>` 클래스는 제네릭을 사용하여 박스가 담을 타입을 `T`로 지정했습니다. 이렇게 하면 데이터를 담을 때와 가져올 때 타입이 보장되므로 타입 캐스팅이 필요 없고, 안전하게 사용할 수 있습니다.

```java
// 사용 예시
Box<Integer> integerBox = new Box<>();
integerBox.set(123);
Integer number = integerBox.get(); // 타입 캐스팅 없이 안전하게 사용 가능

Box<String> stringBox = new Box<>();
stringBox.set("Hello");
String text = stringBox.get(); // 역시 타입 캐스팅 없이 안전하게 사용 가능
```

이처럼 제네릭을 사용하면 타입 안전성을 보장하면서도 다양한 타입을 처리할 수 있어, 코드의 재사용성과 유지보수성이 크게 향상됩니다.

</details>

