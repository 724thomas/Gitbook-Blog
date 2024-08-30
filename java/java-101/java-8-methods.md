---
description: Java 8에서 추가된 기능
---

# Java 8 methods

## 스트림 (Stream)

스트림은 데이터를 저장하지 않고, 일련의 연산을 통해 데이터를 처리하는 추상화된 개념입니다. 이를 통해 대량의 데이터를 간결하고 효율적으로 처리할 수 있습니다.

### 특징

* **선언적 방식**: 스트림 API는 선언적 방식으로 데이터를 처리합니다. 즉, 무엇을 할 것인지에 집중할 수 있게 해줍니다.
* **데이터 변경 없음**: 스트림을 사용한 데이터 처리 과정에서 원본 데이터를 변경하지 않습니다.
* **지연 연산**: 스트림 연산은 기본적으로 지연(Lazy) 연산으로, 실제로 필요한 시점에 연산이 수행됩니다.
* **병렬 처리**: 쉽게 병렬 스트림을 생성하여 병렬 처리를 구현할 수 있습니다.

### 종류

스트림은 크게 \*\*시퀀셜 스트림(Sequential Stream)\*\*과 \*\*병렬 스트림(Parallel Stream)\*\*으로 나뉩니다.

* **시퀀셜 스트림**: 스트림의 연산이 순차적으로 처리됩니다.
* **병렬 스트림**: 스트림의 연산이 여러 스레드에서 병렬로 처리됩니다. 이는 대용량 데이터 처리에 유용하지만, 항상 병렬 스트림이 성능을 개선하는 것은 아닙니다. 데이터의 크기, 연산의 복잡성 등에 따라 오히려 성능이 저하될 수 있습니다.



## 람다 (Lambda)

람다 표현식은 Java 8에서 도입된 함수형 프로그래밍의 핵심 기능 중 하나입니다. 람다 표현식을 사용하면 익명 함수를 간단하게 표현할 수 있으며, 코드의 간결성과 가독성을 높여줍니다.

<details>

<summary>함수형 인터페이스 사용 예시</summary>

```java
Predicate<Integer> isPositive = (n) -> n > 0;

System.out.println(isPositive.test(5));  // true
System.out.println(isPositive.test(-3)); // false
```

이 경우, `isPositive`는 `Predicate<Integer>` 타입의 변수입니다. 이는 하나의 `Integer`를 받아서 `boolean` 값을 반환하는 람다 표현식을 참조합니다.

</details>

### 함수형 인터페이스

람다 표현식은 함수형 인터페이스를 구현하는 데 주로 사용됩니다. 함수형 인터페이스는 하나의 추상 메서드만을 가지는 인터페이스로, Java 8에서는 이 인터페이스들에 `@FunctionalInterface` 애너테이션을 붙여 명시적으로 표현할 수 있습니다.

<details>

<summary>예시</summary>

```java
@FunctionalInterface
interface MyFunction {
    void apply();
}
```

</details>

### 기본 제공 함수형 인터페이스

* **`Predicate<T>`**: `T` 타입의 값을 입력받아 `boolean`을 반환합니다.
* **`Function<T, R>`**: `T` 타입의 값을 받아 `R` 타입의 값을 반환합니다.
* **`Supplier<T>`**: 인자를 받지 않고 `T` 타입의 값을 반환합니다.
* **`Consumer<T>`**: `T` 타입의 값을 받아 어떤 동작을 수행하고 반환 값은 없습니다.

<details>

<summary>커스텀 함수형 인터페이스 활용 예시</summary>

```java
@FunctionalInterface
interface MyPredicate<T> {
    boolean test(T t);
}

public class FunctionalInterfaceExample {
    public static void main(String[] args) {
        MyPredicate<Integer> isEven = n -> n % 2 == 0;

        System.out.println(isEven.test(4)); // 출력: true
        System.out.println(isEven.test(5)); // 출력: false
    }
}
```

이 예제에서는 `MyPredicate` 함수형 인터페이스를 사용하여 숫자가 짝수인지 여부를 검사하는 람다 표현식을 작성하였습니다.

</details>

<details>

<summary>커스텀 함수형 인터페이스 활용 예시2</summary>

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;
import java.util.stream.Collectors;

class User {
    private String name;
    private int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + '}';
    }
}

@FunctionalInterface
interface UserPredicate {
    boolean test(User user);
}

public class CustomFunctionalInterfaceExample {
    public static void main(String[] args) {
        List<User> users = Arrays.asList(
            new User("Alice", 23),
            new User("Bob", 17),
            new User("Anna", 25),
            new User("Charlie", 19)
        );

        // 나이가 18세 이상인지 검사
        UserPredicate isAdult = user -> user.getAge() >= 18;

        // 이름이 'A'로 시작하는지 검사
        UserPredicate startsWithA = user -> user.getName().startsWith("A");

        // 두 조건을 결합하여 검사
        List<User> filteredUsers = users.stream()
            .filter(user -> isAdult.test(user) && startsWithA.test(user))
            .collect(Collectors.toList());

        System.out.println(filteredUsers);
        // 출력: [User{name='Alice', age=23}, User{name='Anna', age=25}]
    }
}
```

* **User 클래스**: 사용자의 `name`과 `age`를 가지고 있는 간단한 데이터 클래스입니다.
* **UserPredicate 함수형 인터페이스**: `User` 객체를 받아 특정 조건을 테스트하는 함수형 인터페이스입니다.
* **isAdult 및 startsWithA**: 각각 나이가 18세 이상인지, 이름이 'A'로 시작하는지를 검사하는 두 개의 람다 표현식입니다.
* **필터링 및 수집**: 스트림 API를 사용해 사용자 리스트에서 두 가지 조건을 모두 만족하는 사용자만 필터링한 후 리스트로 수집합니다.

</details>



## 디폴트 (Default)

이전 버전의 Java에서는 인터페이스에 메서드를 추가할 경우, 그 인터페이스를 구현하는 모든 클래스에서 해당 메서드를 구현해야 했습니다. 하지만 디폴트 메서드를 사용하면 인터페이스에 메서드를 추가하더라도 구현 클래스에서 해당 메서드를 구현할 필요가 없습니다.

<details>

<summary>디폴트 메서드의 사용 예시</summary>

```java
interface Vehicle {
    default void start() {
        System.out.println("Vehicle is starting");
    }
}

class Car implements Vehicle {
    // Car 클래스는 start 메서드를 구현하지 않아도, Vehicle 인터페이스의 기본 구현을 사용합니다.
}

public class DefaultMethodExample {
    public static void main(String[] args) {
        Car car = new Car();
        car.start(); // 출력: Vehicle is starting
    }
}
```

이 예제에서 `Vehicle` 인터페이스는 `start`라는 디폴트 메서드를 가지고 있습니다. `Car` 클래스는 이 인터페이스를 구현하면서 `start` 메서드를 별도로 구현하지 않았지만, `Vehicle` 인터페이스의 디폴트 메서드를 사용할 수 있습니다.

</details>

### 디폴트의 단점

1. **다이아몬드 문제(Diamond Problem)**: 디폴트 메서드를 사용할 때, 다중 상속 구조에서 다이아몬드 문제가 발생할 수 있다. 이는 인터페이스들이 동일한 디폴트 메서드를 제공할 때 발생하며, 이 경우에는 구현 클래스에서 메서드를 오버라이드하여 해결해야 한다.
2. **혼란을 유발할 수 있음**: 디폴트 메서드는 인터페이스의 모든 구현체에서 동일하게 동작해야 한다. 만약 기본 구현이 모든 상황에 적합하지 않다면, 이를 오버라이드해야 할 수도 있다. 이 과정에서 오버라이드할 필요가 있는지 혼란스러울 수 있다.
3. **인터페이스의 일관성**: 인터페이스는 기본적으로 동작을 정의하는 것이 아니라, 동작을 명시하는 역할을 해야 한다. 디폴트 메서드는 이러한 인터페이스의 일관성을 무너뜨릴 수 있다. 따라서 디폴트 메서드를 추가할 때는 이 메서드가 정말로 모든 구현체에 적합한지 신중하게 고려해야 한다.

### 다이아몬드 문제

여러 인터페이스를 구현하는 클래스에서 같은 이름의 디폴트 메서드를 상속받을 때 문제가 발생할 수 있습니다. 이를 **다이아몬드 문제**(Diamond Problem)라고 부릅니다.

<details>

<summary>다이아몬드 문제 예시</summary>

```java
interface InterfaceA {
    default void hello() {
        System.out.println("Hello from InterfaceA");
    }
}

interface InterfaceB {
    default void hello() {
        System.out.println("Hello from InterfaceB");
    }
}

public class DiamondProblemExample implements InterfaceA, InterfaceB {
    public void hello() {
        InterfaceA.super.hello(); // InterfaceA의 hello 메서드 호출
    }

    public static void main(String[] args) {
        DiamondProblemExample example = new DiamondProblemExample();
        example.hello(); // 출력: Hello from InterfaceA
    }
}
```

이 예제에서 `DiamondProblemExample` 클래스는 `InterfaceA`와 `InterfaceB` 모두의 `hello` 메서드를 상속받습니다. 하지만, 두 인터페이스에서 동일한 메서드 이름을 사용하기 때문에 충돌이 발생합니다. 이를 해결하기 위해 특정 인터페이스의 메서드를 호출하도록 명시해야 합니다.

</details>



## 컴플리터블 퓨처 (Completable Future)

비동기 프로그래밍은 처음 자바5에 Future가 추가되면서 가능하게 되었고, 이것을 통해 비동기 작업에 대한 결과값을 반환받을 수 있게 됐습니다. Future에는 단점들이 존재했는데, 단점들을 해결하고자 나온게 자바8에서 CompletableFuture입니다.

### Future의단점

* **외부에서 작업을 중단시키는 기능을 지원하지 않음 (제어 불가)** 비동기 작업이 무한루프나 오래걸릴경우, 강제로 완료 시킬 수가 없습니다.
* **완전한 비동기 작업이 아닙니다.** `Future`는 비동기 작업이지만, 결과를 처리하기 위해서는 **동기화된 블로킹 코드**(’get’)로 돌아가야 합니다.
* **여러 `Future`를 조합할 수 없습니다.** (예: **회원 정보를 가져오는 작업**과 **알림을 발송하는 작업**을 조합하려면 각 `Future`의 결과를 순차적으로 처리해야 하며, 이를 효율적으로 조합할 수 있는 메커니즘이 없습니다.)
* **예외처리가 어렵습니다.** 예: 첫번째 작업의 성공 유무에 따른 두번쨰 작업 실행 방향 처리 구현이 어렵습니다.

### CompletableFuture의 장점

* **유연성**: 여러 비동기 작업을 유연하게 조합할 수 있습니다.
* **비동기 예외 처리**: 예외 처리를 더욱 간단하게 할 수 있습니다.
* **명시적인 비동기 API**: 비동기 작업의 흐름을 명확하게 표현할 수 있습니다.
* **병렬 실행 지원**: 여러 비동기 작업을 병렬로 실행하여 성능을 향상시킬 수 있습니다.

### CompletableFuture의 작업 종류

비동기 작업 실행, 작업 콜백, 작업 조합, 예외처리가 있습니다.

* 비동기 작업 실행
  * `runAsync(Runnable)` - 반환 값이 없는 비동기 작업을 실행합니다.
  * `supplyAsync(Supplier<U>)` - 반환 값이 있는 비동기 작업을 실행합니다.
* 작업 콜백
  * `thenApply(Function<T, U>)` - 결과를 받아서 다른 결과로 변환합니다.
  * `thenAccept(Consumer<T>)` - 결과를 받아서 소비합니다. 반환 값은 없습니다.
  * `thenRun(Runnable)` - 결과를 받지 않고 단순히 실행합니다.
* 작업 조합
  * `thenCompose(Function<T, CompletionStage<U>>)` 이전 작업의 결과를 받아서 새로운 비동기 작업을 실행합니다.
  * `thenCombine(CompletionStage<U>, BiFunction<T, U, V>)` - 두 개의 비동기 작업 결과를 조합합니다.
  * `allOf(CompletableFuture<?>...)` - 여러 비동기 작업을 모두 완료할 때까지 기다립니다.
  * `anyOf(CompletableFuture<?>...)` - 여러 비동기 작업 중 하나라도 완료되면 결과를 반환합니다.
* 예외 처리
  * `exceptionally(Function<Throwable, T>)` - 예외가 발생했을 때 기본값을 반환하거나, 예외를 처리합니다.
  * `handle(BiFunction<T, Throwable, U>)` - 정상 결과와 예외를 모두 처리할 수 있습니다.
  * `handleAsync(BiFunction<T, Throwable, U>)` - 비동기적으로 정상 결과와 예외를 모두 처리할 수 있습니다.



