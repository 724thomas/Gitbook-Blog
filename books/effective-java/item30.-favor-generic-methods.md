---
description: 이왕이면 제네릭 메서드로 만들라
---

# Item30. Favor Generic Methods

자바에서 제네릭 메서드는 메서드의 동작을 호출 시점에 지정된 타입으로 일반화할 수 있게 해줍니다. 제네릭 메서드를 사용하면 특정 타입에 종속되지 않고 다양한 타입을 처리할 수 있어 코드의 재사용성을 높이고, 타입 안전성을 확보할 수 있습니다.



## 1. 제네릭 메서드의 필요성

제네릭 메서드는 타입 매개변수를 받아들여, 메서드 호출 시점에 타입을 결정하게 합니다. 이를 통해 코드 중복을 줄이고, 타입에 독립적인 메서드를 작성할 수 있습니다.

### 1.1. 제네릭 메서드를 사용하지 않는 예시

제네릭을 사용하지 않은 메서드는 여러 타입을 처리하기 위해 오버로딩을 사용하거나, `Object` 타입을 사용하여 타입을 통일하는 경우가 많습니다.

<pre class="language-java"><code class="lang-java"><strong>//제네릭 메서드를 사용하지 않은 경우의 예시
</strong><strong>public class Utils {
</strong>
    // 서로 다른 타입의 두 값을 비교해 더 큰 값을 반환하는 메서드
    public static int max(int a, int b) {
        return (a > b) ? a : b;
    }

    public static double max(double a, double b) {
        return (a > b) ? a : b;
    }
}
</code></pre>

위 코드에서는 `max` 메서드가 두 개의 다른 타입(`int`, `double`)을 처리하기 위해 각각 메서드를 오버로딩하고 있습니다. 이는 코드의 중복을 증가시키고, 새로운 타입을 추가할 때마다 새로운 메서드를 작성해야 합니다.



### 1.2. 제네릭 메서드를 사용한 예시

제네릭 메서드를 사용하면 타입에 의존하지 않고, 더 간결하고 재사용 가능한 코드를 작성할 수 있습니다.&#x20;

<pre class="language-java"><code class="lang-java"><strong>//제네릭 메서드를 사용한 max 메서드의 개선된 예시
</strong><strong>public class Utils {
</strong>
    // 제네릭 메서드를 사용하여 타입에 독립적인 max 메서드 작성
    public static &#x3C;T extends Comparable&#x3C;T>> T max(T a, T b) {
        return (a.compareTo(b) > 0) ? a : b;
    }
}
</code></pre>

`<T extends Comparable<T>>`: 제네릭 타입 매개변수 `T`는 `Comparable` 인터페이스를 구현해야 함을 명시하여, `compareTo` 메서드를 사용할 수 있도록 보장합니다.

<details>

<summary>Comparable 인터페이스 구현 예시</summary>

제네릭 타입 매개변수 `T`가 `Comparable<T>` 인터페이스를 구현하여, `compareTo` 메서드를 사용할 수 있도록 보장하는 코드

```java
// Person 클래스가 Comparable 인터페이스를 구현
class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Comparable 인터페이스의 compareTo 메서드를 구현
    @Override
    public int compareTo(Person other) {
        // 나이를 기준으로 비교 (나이가 적은 순서로 정렬)
        return Integer.compare(this.age, other.age);
    }

    @Override
    public String toString() {
        return name + " (" + age + ")";
    }
}

// 제네릭 메서드: T가 Comparable<T>를 구현해야만 사용할 수 있음
public class Main {

    // 두 값을 비교하여 큰 값을 반환하는 제네릭 메서드
    public static <T extends Comparable<T>> T max(T a, T b) {
        return (a.compareTo(b) > 0) ? a : b;
    }

    public static void main(String[] args) {
        // Integer, String, Person 모두 Comparable을 구현하므로 max 메서드에서 사용 가능
        System.out.println(max(10, 20));                // Integer 비교, 출력: 20
        System.out.println(max("apple", "banana"));     // String 비교, 출력: banana

        // Person 클래스의 비교
        Person person1 = new Person("Alice", 30);
        Person person2 = new Person("Bob", 25);

        // Person의 나이를 비교하여 더 나이가 많은 사람을 반환
        System.out.println(max(person1, person2));      // 출력: Alice (30)
    }
}
```

#### 코드 설명

1. **`Person` 클래스**:
   * `Person` 클래스는 `Comparable<Person>` 인터페이스를 구현하고, `compareTo` 메서드를 오버라이드합니다.
   * `compareTo` 메서드는 `Person` 객체를 나이를 기준으로 비교하여, 정렬이나 크기 비교 시 사용됩니다.
2. **`max` 제네릭 메서드**:
   * `<T extends Comparable<T>>`: `T`는 `Comparable<T>`를 구현해야만 사용할 수 있습니다. 이는 `compareTo` 메서드를 호출할 수 있도록 보장합니다.
   * `max(T a, T b)`: 두 `T` 타입의 객체를 비교하여 더 큰 값을 반환합니다.
3. **`main` 메서드에서의 사용 예시**:
   * `Integer`, `String`, `Person` 등 `Comparable`을 구현한 다양한 타입에서 `max` 메서드를 사용할 수 있습니다.
   * `Person` 객체를 비교할 때는 나이를 기준으로 비교하여 더 큰(나이가 많은) 객체를 반환합니다.

</details>

이렇게 작성된 제네릭 메서드는 `int`, `double`, `String` 등 서로 다른 타입에 대해 작동할 수 있습니다.

```java
// 사용 예시
System.out.println(Utils.max(10, 20));         // 출력: 20
System.out.println(Utils.max(5.5, 2.2));       // 출력: 5.5
System.out.println(Utils.max("apple", "pear")); // 출력: pear
```



## 2. 제네릭 메서드의 이점

* 타입 안전성 제공\
  제네릭 메서드는 컴파일 시점에 타입을 체크하여 타입 안전성을 보장합니다. 잘못된 타입을 전달하면 컴파일러가 오류를 검출하므로, 런타임 오류를 줄일 수 있습니다.
* 코드 중복 제거 및 유지보수성 향상\
  제네릭 메서드는 여러 타입을 처리할 수 있어 코드 중복을 줄여줍니다. 새로운 타입을 추가할 때 메서드를 새로 작성할 필요가 없으며, 유지보수가 용이해집니다.
* 타입 캐스팅 제거\
  제네릭 메서드는 반환 타입을 정확히 알기 때문에, 메서드 호출 시 별도의 타입 캐스팅을 할 필요가 없습니다. 이는 코드의 가독성을 높이고, 실수를 줄이는 데 도움을 줍니다.



## 3. 제네릭 메서드 작성 시 고려 사항

### 3.1. 제네릭 타입 매개변수의 위치

제네릭 메서드의 타입 매개변수는 반환 타입 앞에 위치해야 합니다. 예를 들어, `<T> T methodName(T arg)`와 같이 정의해야 합니다.

```java
// 올바른 제네릭 메서드 정의
public static <T> T identity(T arg) {
    return arg;
}
```

### 3.2. 제네릭 타입의 제한

제네릭 타입을 제한하려면 `extends` 키워드를 사용하여 특정 타입이나 인터페이스를 구현하도록 제약할 수 있습니다. 예를 들어, `T extends Number`는 제네릭 타입 `T`가 `Number`의 하위 타입이어야 함을 의미합니다.

```java
// 제네릭 타입 매개변수를 Number의 하위 타입으로 제한
public static <T extends Number> double sum(T a, T b) {
    return a.doubleValue() + b.doubleValue();
}
```

### 3.3. 와일드카드와 제네릭 메서드

와일드카드(`?`)는 제네릭 타입을 불특정하게 사용하고자 할 때 유용합니다. 메서드 매개변수에 와일드카드를 사용하면, 호출 시점에 타입이 정해지지 않은 다양한 제네릭 타입을 처리할 수 있습니다.

```java
// 와일드카드를 사용하여 제네릭 타입의 리스트 요소 출력
public static void printList(List<?> list) {
    for (Object obj : list) {
        System.out.println(obj);
    }
}
```



## 4. 제네릭 메서드의 예시: 자바 컬렉션 프레임워크

`Collections` 클래스의 다양한 메서드는 제네릭을 사용하여 타입 안전성과 유연성을 제공합니다.

### 4.1. Collections의 binarySearch

`binarySearch` 메서드는 리스트에서 이진 검색을 수행하는 제네릭 메서드입니다.

```java
public static <T extends Comparable<? super T>> int binarySearch(List<T> list, T key);
```

* `<T extends Comparable<? super T>>`: 제네릭 타입 `T`가 `Comparable` 인터페이스를 구현해야 하며, 이는 `T` 또는 그 상위 타입과 비교할 수 있어야 함을 의미합니다.
* 이 메서드는 어떤 타입의 리스트에도 사용될 수 있으며, 타입 안전한 검색을 제공합니다.

### 4.2. Collections의 copy

`copy` 메서드는 한 리스트에서 다른 리스트로 요소를 복사하는 제네릭 메서드입니다.

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src);
```

* `<T>`: 제네릭 타입 매개변수.
* `List<? super T> dest`: 제네릭 타입 `T` 또는 그 상위 타입의 요소를 받을 수 있는 리스트.
* `List<? extends T> src`: 제네릭 타입 `T` 또는 그 하위 타입의 요소를 가진 리스트.
