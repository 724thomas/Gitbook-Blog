---
description: 배열보다는 리스트를 사용하라
---

# Item28. Use Lists Instead of Arrays

자바 프로그래밍에서 배열과 리스트는 자료를 저장하고 관리하는 데 자주 사용되는 컬렉션입니다. 그러나 두 컬렉션은 설계 철학, 사용 방법, 그리고 제네릭과의 호환성 측면에서 큰 차이점을 가지고 있습니다.

배열은 자바에서 성능 면에서 이점이 있지만, **타입 안전성과 유연성**에서 여러 제약이 있습니다. 특히 제네릭과 함께 사용할 때 문제가 발생할 수 있으며, 이는 프로그램의 안정성에 영향을 미칩니다.

반면, 리스트는 제네릭을 통해 컴파일 시점에 타입 오류를 방지하고, 다양한 메서드를 통해 데이터를 유연하게 다룰 수 있게 해줍니다.&#x20;

코드의 안정성과 유지보수성을 높이기 위해 자바에서는 배열보다는 리스트를 사용하는 것이 더 좋은 선택입니다.

## 1. 공변성과 불공변성

### 1.1. 배열의 공변성(Covariance)

배열은 타입 간의 상속 관계를 그대로 따르기 떄문에 서로 상속 관계가 있는 타입끼리 대입이 가능합니다.&#x20;

예를 들어, Integer\[]는 Number\[]로 대입할 수 있습니다. 즉, Number\[] 타입의 변수에 Integer\[] 타입의 값을 넣을 수 있습니다.

```java
Number[] numbers = new Integer[10]; // Integer 배열을 Number 배열로 사용
numbers[0] = 3.14; // 컴파일은 되지만, 실행 시 ArrayStoreException 발생
```

위 예제에서는 `Number[]` 타입 변수에 `Integer[]`를 대입했지만, 실제로는 `Double` 타입 값을 넣으려다 `ArrayStoreException`이 발생합니다. 배열의 공변성은 이런 런타임 오류를 유발할 수 있습니다.

### 1.2. 리스트의 불공변성(Invariance)

리스트는 불공변이기 때문에 타입이 정확히 일치해야만 대입이 가능합니다.

예를 들어, List\<Integer>는 List\<Number>로 대입할 수 없습니다.

```java
List<Number> numbers = new ArrayList<>(); 
// numbers.add(3.14); // 허용
// numbers.add("String"); // 컴파일 오류 발생: 타입 불일치
```

리스트는 컴파일 단계에서 타입 불일치 오류를 잡아주기 때문에 안전하게 사용할 수 있습니다.



## 2. 실체화와 타입 소거

### 2.1. 배열의 실체화 (Reifiable)

배열은 런타임에도 자신이 어떤 타입인지 알고 있는 **실체화** 타입입니다. 배열이 생성될 때 자바는 배열의 타입을 저장해두고, 이후 잘못된 타입의 데이터를 추가하려 하면 런타임에 오류를 발생시킵니다.

### 2.2. 리스트의 타입 소거 (Type Erasure)

제네릭 리스트는 컴파일 시점에 타입 정보를 사용하지만, 런타임에는 타입 정보가 사라집니다. 즉, List\<String>과 List\<Integer>는 컴파일 후 같은 타입으로 처리됩니다. 그러나 컴파일러가 타입 검사를 수행하므로 런타임 오류 없이 안전하게 사용할 수 있습니다.



## 3. 배열 사용의 단점과 문제점

### 3.1. 타입 안전성 문제

배열은 런타임에 타입 검사를 수행합니다. 이는 프로그램이 실행되기 전까지 타입 오류를 감지할 수 없습니다.

```java
Object[] objectArray = new Long[1];
objectArray[0] = "Hello"; // 런타임에 ArrayStoreException 발생
```

위 코드에서 `Long[]` 배열을 `Object[]`로 사용했기 때문에, 컴파일은 되지만 실행 중 `String`을 넣으려 할 때 오류가 발생합니다.

### 3.2. 제네릭과의 비호환성

배열은 제네릭 타입과 잘 호환되지 않습니다. 제네릭을 사용한 배열을 만들려고 하면 컴파일 경고가 발생하며, 안정적인 코드를 작성하기 어렵습니다.

```java
List<String>[] stringLists = new List<String>[1]; // 컴파일 경고 발생
List<Integer> intList = List.of(42);

Object[] objects = stringLists;
objects[0] = intList; // 타입 안전성 문제가 발생할 수 있음
```

배열은 제네릭 타입의 변수를 다룰 때 타입 안전성을 보장하지 않으며, 컴파일러가 타입 정보를 확실히 인지하지 못합니다.



## 4. 리스트 사용의 장점

### 4.1. 컴파일 시점 타입 체크

리스트는 제네릭을 사용하여 컴파일 시점에 타입을 엄격히 검사합니다. 이는 프로그램 실행 전에 오류를 잡을 수 있어 안전한 코드 작성이 가능합니다.

```java
List<Long> list = new ArrayList<>();
// list.add("Hello"); // 컴파일 오류 발생: 타입 불일치
```

위 코드에서 리스트는 컴파일 단계에서 "Hello" 문자열을 추가하려는 시도를 막아주며, 타입 일치를 요구합니다.

### 4.2. 가변 크기와 편리한 메서드 지원

리스트는 요소를 추가하거나 제거할 수 있어 크기 조정이 자유롭습니다. 또한, add, remove, contains 등 다양한 메서드를 지원해 자료를 쉽게 조작할 수 있습니다.

```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.remove("Alice");
System.out.println(names); // [Bob]
```



## 5. 코드 예시: 배열을 리스트로 변환하기

<details>

<summary>배열을 사용한 예시:</summary>

```java
public class ArrayExample {
    private String[] names;

    public ArrayExample() {
        this.names = new String[]{"Alice", "Bob", "Charlie"};
    }

    public void printNames() {
        for (String name : names) {
            System.out.println(name);
        }
    }
}
```

</details>

<details>

<summary>리스트를 사용한 예시:</summary>

```java
import java.util.ArrayList;
import java.util.List;

public class ListExample {
    private List<String> names;

    public ListExample() {
        this.names = new ArrayList<>();
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");
    }

    public void printNames() {
        names.forEach(System.out::println);
    }
}
```

리스트를 사용하면 추가적인 메모리 관리가 필요 없으며, 코드가 더 안전하고 유연하게 작성될 수 있습니다.

</details>
