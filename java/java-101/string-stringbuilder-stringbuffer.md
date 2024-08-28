---
description: String, StringBuilder, StringBuffer
---

# String, StringBuilder, StringBuffer

## String: 불변(Immutable) 객체

`String`은 Java에서 가장 많이 사용되는 클래스 중 하나로, 문자열을 표현하기 위해 사용됩니다. `String` 객체는 불변(immutable) 객체로, 생성된 후에는 그 값을 변경할 수 없습니다.

```java
String str = "Hello";
str = str.concat(" World");
```

위 코드에서 `str`은 처음에 `"Hello"`를 가리키고, `concat` 메서드를 호출한 후에는 `"Hello World"`라는 새로운 문자열을 가리키게 됩니다. 원래의 `"Hello"` 문자열은 변경되지 않으며, 새로운 문자열 객체가 생성됩니다.

### String의 장점

* **간결함**: 문자열을 다루는 기본적인 클래스이며, 간단한 문자열 조작에 적합합니다.
* **Thread-safe**: `String` 객체는 불변이기 때문에 멀티스레드 환경에서 안전합니다.

### **String의 단점**

* **비효율성**: 문자열 조작이 빈번한 경우, 새로운 객체가 계속 생성되어 메모리와 성능에 영향을 줄 수 있습니다.



## StringBuilder: 가변(Mutable) 객체

`StringBuilder`는 가변(mutable) 객체로, `String`과 달리 동일 객체 내에서 문자열을 변경할 수 있습니다. 즉, 문자열을 추가하거나 수정할 때 새로운 객체를 생성하지 않고 기존 객체를 변경합니다.

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // "Hello World"
sb.insert(5, ","); // "Hello, World"
sb.delete(5, 6); // "Hello World"
sb.reverse(); // "dlroW olleH"
```

위 코드에서는 `StringBuilder` 객체가 생성되고, `append` 메서드를 통해 `" World"`가 추가됩니다. 이 과정에서 새로운 객체가 생성되지 않고, 기존 `StringBuilder` 객체의 값이 변경됩니다.

### **StringBuilder의 장점**

* **성능**: 문자열을 자주 변경해야 하는 경우, `StringBuilder`는 새로운 객체를 생성하지 않으므로 메모리 사용과 성능 면에서 유리합니다.
* **간결성**: `String`처럼 여러 메서드를 체인으로 연결할 수 있어, 코드가 간결해집니다.

### **StringBuilder의 단점**

* **Thread-unsafe**: `StringBuilder`는 동기화를 제공하지 않으므로, 멀티스레드 환경에서는 안전하지 않습니다.



## StringBuffer: 가변(Mutable) 객체 + Thread-safe

`tringBuffer`는 `StringBuilder`와 매우 유사하지만, 멀티스레드 환경에서 안전한(Thread-safe) 가변 객체입니다. `StringBuilder`와 동일하게 문자열을 변경할 수 있지만, 동기화가 추가되어 여러 스레드가 동시에 접근할 경우에도 안전합니다.

```java
StringBuffer sbf = new StringBuffer("Hello");
sbf.append(" World");
```

위 코드에서 `StringBuffer`는 `StringBuilder`와 유사한 방식으로 동작하지만, 멀티스레드 환경에서 안전하게 사용할 수 있습니다.

### **StringBuffer의 장점**

* **Thread-safe**: 동기화가 적용되어 멀티스레드 환경에서 안전하게 사용할 수 있습니다.
* **가변성**: `StringBuilder`처럼 동일 객체 내에서 문자열을 변경할 수 있어, 성능이 우수합니다.

### **StringBuffer의 단점**

* **성능 저하**: 동기화로 인해 `StringBuilder`보다 성능이 약간 떨어질 수 있습니다. 그러나 멀티스레드 환경에서의 안전성을 보장해야 할 때는 이 성능 저하를 감수할 가치가 있습니다.



## 성능 비교

### **단일 스레드 환경**

단일 스레드 환경에서는 `StringBuilder`가 `StringBuffer`보다 성능이 뛰어납니다. 동기화가 필요 없기 때문에 불필요한 오버헤드가 발생하지 않기 때문입니다.

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append("a");
}
```

위와 같은 반복문에서 `StringBuilder`를 사용하면 `String`을 사용할 때보다 훨씬 빠른 성능을 얻을 수 있습니다.

### **멀티스레드 환경**

멀티스레드 환경에서는 `StringBuffer`를 사용하는 것이 안전합니다. 여러 스레드가 동시에 문자열을 수정할 수 있는 상황에서는 동기화가 되어 있는 `StringBuffer`를 사용해야 합니다.

```java
StringBuffer sbf = new StringBuffer();
Runnable task = () -> {
    for (int i = 0; i < 1000; i++) {
        sbf.append("a");
    }
};

Thread thread1 = new Thread(task);
Thread thread2 = new Thread(task);

thread1.start();
thread2.start();
```

이 경우, `StringBuffer`는 동기화를 통해 안전하게 문자열을 수정할 수 있습니다.





## String Pool

ava에서 `String`은 불변(immutable) 객체이기 때문에, 동일한 문자열을 여러 번 사용하더라도 메모리 낭비를 줄이기 위해 **String Pool**이라는 메커니즘을 사용합니다. `String Pool`은 JVM의 Heap 영역에 있는 특별한 공간으로, 동일한 문자열 리터럴을 관리합니다.

```java
String str1 = "Hello";
String str2 = "Hello";
```

위 코드에서 `str1`과 `str2`는 동일한 `"Hello"` 문자열을 참조합니다. 이 문자열은 `String Pool`에 한 번만 생성되고, 이후 같은 리터럴을 사용할 때는 동일한 참조를 반환합니다.

### **String Pool의 장점**

* **메모리 절약**: 동일한 문자열 리터럴이 여러 번 생성되는 것을 방지하여 메모리 사용을 최적화합니다.
* **성능 향상**: 이미 생성된 문자열을 재사용함으로써 성능이 향상됩니다.

### **String Pool의 한계**

* **동적 문자열 생성**: `new` 키워드를 사용해 `String` 객체를 생성하면 `String Pool`에 저장되지 않습니다.

```java
String str3 = new String("Hello");
```

위 코드에서 `str3`은 `new`를 사용해 생성되었기 때문에, `String Pool`과는 별도로 Heap에 새로운 `String` 객체가 생성됩니다.





## intern() 메서드

`intern()` 메서드를 사용하면 동적으로 생성된 `String`을 `String Pool`에 추가할 수 있습니다. 만약 `String Pool`에 이미 동일한 문자열이 존재한다면, 해당 문자열의 참조를 반환합니다.

```java
String str4 = new String("World").intern();
String str5 = "World";
System.out.println(str4 == str5); // true
```

위 코드에서 `str4`는 `intern()`을 사용하여 `String Pool`에 저장된 `"World"` 문자열을 참조하게 됩니다.
