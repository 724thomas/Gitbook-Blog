---
description: 문자열, 예외, 제네릭
---

# Week 2(1/2) - String, Exception, Generic

## 1. String 리터럴과 `new String("")`의 차이

자바에서 `String`은 불변 객체(immutable object)입니다. 즉, 한 번 생성된 문자열은 변경할 수 없습니다. 이를 통해 자바에서 문자열을 다루는 두 가지 주요 방식인 _String 리터럴_과 `new String("")`에 대해 알아보겠습니다.

**1.1 String 리터럴**

```java
String str1 = "Hello";
String str2 = "Hello";
```

위 코드에서 `str1`과 `str2`는 둘 다 동일한 리터럴 `"Hello"`를 참조합니다. 이는 자바의 _String 상수 풀_ 덕분입니다. String 리터럴은 JVM의 String 상수 풀에 저장되며, 동일한 리터럴이 이미 존재할 경우, 새롭게 메모리를 할당하지 않고 기존 객체를 참조합니다. 따라서 `str1 == str2`는 `true`를 반환합니다.

**1.2 `new String("")`**

```java
String str3 = new String("Hello");
String str4 = new String("Hello");
```

위 코드에서 `str3`과 `str4`는 각각 `new` 키워드를 사용하여 새롭게 `String` 객체를 힙 메모리에 생성합니다. 이 경우, 서로 다른 두 객체가 생성되므로 `str3 == str4`는 `false`를 반환합니다. 하지만 두 문자열의 내용은 동일하기 때문에 `str3.equals(str4)`는 `true`입니다.

**1.3 차이점 정리**

* _메모리 사용_: String 리터럴은 상수 풀을 이용하여 메모리를 절약할 수 있지만, `new String()`은 힙 메모리에 매번 새로운 객체를 생성합니다.
* _성능_: String 리터럴 방식이 메모리와 성능 면에서 효율적입니다.
* _참조 비교_: String 리터럴은 상수 풀에 저장된 동일한 문자열을 참조하지만, `new String()`은 매번 새로운 객체를 생성하므로 참조 비교(`==`)에서 다른 결과가 나옵니다.

## 2. String, StringBuilder, StringBuffer의 차이점

자바에서 문자열을 다루기 위한 대표적인 클래스 세 가지는 `String`, `StringBuilder`, `StringBuffer`입니다. 각 클래스의 주요 차이점과 사용 사례를 살펴보겠습니다.

**2.1 String**

* **불변성(Immutable)**: `String` 객체는 한 번 생성되면 변경할 수 없습니다. 새로운 문자열이 필요할 경우, 기존 객체를 수정하는 것이 아니라 새로운 객체를 생성합니다.
* **사용 사례**: 변경되지 않는 문자열, 상수 문자열, 간단한 문자열 처리.

**2.2 StringBuilder**

* **가변성(Mutable)**: `StringBuilder` 객체는 문자열을 변경할 수 있습니다. 문자열을 수정할 때마다 새로운 객체를 생성하지 않고, 기존 객체를 수정합니다.
* **동기화(Synchronization)**: `StringBuilder`는 동기화되지 않으므로 멀티스레드 환경에서 안전하지 않습니다.
* **사용 사례**: 단일 스레드 환경에서 빈번한 문자열 조작이 필요한 경우. 예를 들어, 루프 내에서 문자열을 여러 번 연결하거나 변경할 때 사용됩니다.

**2.3 StringBuffer**

* **가변성(Mutable)**: `StringBuffer`도 문자열을 변경할 수 있습니다.
* **동기화(Synchronization)**: `StringBuffer`는 동기화가 지원되므로 멀티스레드 환경에서 안전하게 사용할 수 있습니다.
* **사용 사례**: 멀티스레드 환경에서 빈번한 문자열 조작이 필요한 경우.

**2.4 차이점 정리**

| 클래스             | 불변성 | 동기화 지원 | 주요 사용 사례         |
| --------------- | --- | ------ | ---------------- |
| `String`        | 불변  | 불필요    | 변경되지 않는 문자열      |
| `StringBuilder` | 가변  | 미지원    | 단일 스레드에서의 문자열 조작 |
| `StringBuffer`  | 가변  | 지원     | 멀티스레드에서의 문자열 조작  |



## 3. Exception과 Error의 차이

**3.1 Exception**

`Exception`은 프로그램에서 발생할 수 있는 예외적인 상황을 나타냅니다. 대부분의 경우, 개발자가 이러한 예외를 처리하여 프로그램이 정상적으로 동작할 수 있도록 할 수 있습니다. 예를 들어, 파일을 읽다가 파일이 존재하지 않는 경우(FileNotFoundException)와 같은 예외가 발생할 수 있습니다.

**3.2 Error**

`Error`는 프로그램이 복구할 수 없는 심각한 문제를 나타냅니다. 일반적으로 JVM 레벨에서 발생하며, 개발자가 이를 처리할 수 없습니다. 예를 들어, OutOfMemoryError는 JVM이 메모리를 할당할 수 없을 때 발생하며, 이 경우 프로그램은 중단될 수밖에 없습니다.

**3.3 차이점 정리**

* **복구 가능성**: `Exception`은 복구 가능하지만, `Error`는 복구할 수 없습니다.
* **발생 원인**: `Exception`은 코드 로직이나 외부 환경의 문제로 발생할 수 있으며, `Error`는 JVM 레벨의 심각한 문제로 인해 발생합니다.

## 4. Exception 클래스의 예시

자바에서 `Exception` 클래스는 여러 하위 클래스로 세분화되어 있습니다. 대표적인 예시들을 살펴보겠습니다.

**4.1 Checked Exception 예시**

* `IOException`: 입출력 작업 중 발생하는 예외로, 파일이 존재하지 않거나 네트워크 연결이 끊어진 경우 발생합니다.
* `SQLException`: 데이터베이스 작업 중 발생하는 예외로, SQL 구문이 잘못되었거나 데이터베이스에 접근할 수 없는 경우 발생합니다.

**4.2 Unchecked Exception 예시**

* `NullPointerException`: 객체가 `null`인 상태에서 해당 객체의 메서드를 호출하거나 속성에 접근하려고 할 때 발생합니다.
* `ArrayIndexOutOfBoundsException`: 배열의 인덱스가 잘못된 경우 발생하는 예외입니다.

**4.3 사용자 정의 예외**

```java
public class InvalidAgeException extends Exception {
    public InvalidAgeException(String message) {
        super(message);
    }
}
```

사용자 정의 예외는 `Exception` 클래스를 상속받아 개발자가 필요에 따라 정의할 수 있습니다.

## 5. Checked Exception과 Unchecked Exception의 차이

자바의 예외는 `Checked Exception`과 `Unchecked Exception`으로 구분됩니다. 각 예외의 차이점과 처리 방식을 살펴보겠습니다.

**5.1 Checked Exception**

* **정의**: 컴파일 시점에서 검사되는 예외입니다. `Exception` 클래스의 하위 클래스 중 `RuntimeException`을 상속받지 않는 모든 예외가 Checked Exception에 해당합니다.
* **처리 요구사항**: 이러한 예외는 반드시 `try-catch` 블록으로 처리하거나, 메서드 시그니처에 `throws` 키워드를 사용하여 선언해야 합니다.
* **예시**: `IOException`, `SQLException`

**5.2 Unchecked Exception**

* **정의**: 컴파일 시점에서 검사되지 않는 예외입니다. `RuntimeException`과 그 하위 클래스들이 여기에 해당합니다.
* **처리 요구사항**: 이러한 예외는 명시적으로 처리하지 않아도 되지만, 발생할 경우 프로그램이 비정상적으로 종료될 수 있습니다.
* **예시**: `NullPointerException`, `ArrayIndexOutOfBoundsException`

**5.3 차이점 정리**

| 분류                    | 처리 요구사항          | 예시                                            |
| --------------------- | ---------------- | --------------------------------------------- |
| `Checked Exception`   | 반드시 처리하거나 선언해야 함 | `IOException`, `SQLException`                 |
| `Unchecked Exception` | 처리 요구사항 없음       | `NullPointerException`, `ArithmeticException` |

## 6. throw와 throws의 차이

`throw`와 `throws`는 예외 처리에서 사용되는 중요한 키워드입니다. 이 둘의 차이점과 사용 방법을 살펴보겠습니다.

**6.1 `throw`**

* **정의**: `throw` 키워드는 명시적으로 예외를 발생시킬 때 사용됩니다.
* **사용 예시**:

```java
public void checkAge(int age) {
    if (age < 18) {
        throw new IllegalArgumentException("나이는 18세 이상이어야 합니다.");
    }
}
```

**6.2 `throws`**

* **정의**: `throws` 키워드는 메서드가 `Checked Exception`을 던질 수 있음을 선언할 때 사용됩니다.
* **사용 예시**:

```java
public void readFile(String fileName) throws IOException {
    FileReader fileReader = new FileReader(fileName);
    // 파일 읽기 로직
}
```

**6.3 차이점 정리**

* **사용 목적**: `throw`는 예외를 발생시키는 데 사용되며, `throws`는 메서드가 어떤 예외를 던질 수 있는지를 선언합니다.
* **위치**: `throw`는 메서드 내에서 사용되며, `throws`는 메서드 시그니처에 사용됩니다.

## 7. try-catch-finally 구문에서 finally의 역할

`finally` 블록은 `try-catch` 구문에서 예외 발생 여부와 관계없이 항상 실행되는 부분입니다. 주요 역할과 사용 사례는 다음과 같습니다.

**7.1 리소스 해제**

파일, 네트워크 연결, 데이터베이스 연결 등의 리소스는 사용 후 반드시 해제해야 합니다. 이때 `finally` 블록을 사용하여 예외가 발생하더라도 리소스를 확실하게 해제할 수 있습니다.

<pre class="language-java"><code class="lang-java"><strong>FileInputStream inputStream = null;
</strong>try {
    inputStream = new FileInputStream("example.txt");
    // 파일 읽기 로직
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (inputStream != null) {
        try {
            inputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
</code></pre>

**7.2 예외 발생 시 행동 보장**

예외가 발생하더라도 반드시 실행되어야 하는 코드(예: 로그 작성, 데이터 저장)를 `finally` 블록에 넣어 처리할 수 있습니다.

**7.3 주요 특징**

* `finally` 블록은 `try` 또는 `catch` 블록이 종료된 후 항상 실행됩니다.
* `return` 문이 `try` 또는 `catch` 블록에 있더라도, `finally` 블록은 실행됩니다.
* `finally` 블록은 선택적이며, 반드시 사용해야 하는 것은 아닙니다.

## 8. Throwable과 Exception의 차이

`Throwable`과 `Exception`은 자바의 예외 처리에서 중요한 역할을 하는 클래스입니다. 이들의 차이점을 살펴보겠습니다.

**8.1 Throwable**

* **정의**: `Throwable`은 자바의 모든 에러와 예외의 최상위 클래스입니다. 즉, 모든 예외와 오류는 `Throwable` 클래스를 상속받습니다.
* **하위 클래스**: `Throwable`의 주요 하위 클래스는 `Error`와 `Exception`입니다.
* **역할**: `Throwable` 클래스는 예외 또는 오류가 발생했을 때 해당 정보를 캡처하고, 이를 상위 호출 스택으로 전달할 수 있는 메커니즘을 제공합니다.

**8.2 Exception**

* **정의**: `Exception`은 `Throwable`의 하위 클래스이며, 대부분의 예외 상황을 나타냅니다.
* **하위 클래스**: `IOException`, `SQLException` 등 여러 `Checked Exception` 및 `Unchecked Exception` 클래스가 `Exception`을 상속받습니다.
* **역할**: `Exception` 클래스는 프로그램에서 발생할 수 있는 다양한 예외 상황을 나타내며, 대부분의 경우 이를 처리하여 프로그램이 계속 실행될 수 있도록 합니다.

**8.3 차이점 정리**

* **상속 관계**: `Throwable`은 최상위 클래스이며, `Exception`은 그 하위 클래스입니다.
* **주요 역할**: `Throwable`은 예외와 오류를 모두 포함하며, `Exception`은 주로 예외를 처리합니다.

## 9. 제네릭(Generic)이란 무엇이고, 왜 사용할까요?

**9.1 제네릭(Generic)의 정의**

제네릭은 자바에서 타입을 일반화하여 클래스나 메서드를 작성할 수 있는 기능입니다. 즉, 제네릭을 사용하면 데이터 타입에 구애받지 않는 코드를 작성할 수 있습니다.

**9.2 제네릭의 주요 목적**

* **타입 안정성**: 컴파일 시점에서 타입 체크를 통해 런타임 에러를 방지할 수 있습니다.
* **재사용성**: 데이터 타입에 독립적인 코드를 작성함으로써, 다양한 타입에서 사용할 수 있는 코드 재사용성이 높아집니다.
* **가독성**: 명시적인 타입 지정으로 코드 가독성이 향상됩니다.

**9.3 제네릭의 예시**

```java
public class Box<T> {
    private T item;

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }
}
```

위 코드에서 `Box` 클래스는 `T`라는 타입 파라미터를 사용하여 작성되었습니다. 이 클래스는 어떤 타입의 객체든 저장할 수 있으며, 컴파일 시점에 타입을 지정할 수 있습니다.

**9.4 제네릭의 장점**

* **타입 안정성 보장**: 제네릭을 사용하면 컴파일 시점에 타입 오류를 검출할 수 있어 런타임 에러를 줄일 수 있습니다.
* **코드 중복 감소**: 다양한 타입을 처리하기 위해 중복되는 코드를 줄일 수 있습니다.
* **유연성**: 제네릭을 통해 여러 타입에 대해 동일한 로직을 적용할 수 있습니다.

**9.5 사용 예시: 타입 안정성을 보장한 경험**

제네릭을 사용하여 타입 안정성을 보장한 예로, `ArrayList<String>`을 사용한 경험을 들 수 있습니다. 제네릭을 적용하지 않은 `ArrayList`는 `Object` 타입의 요소를 받아들이기 때문에, 요소를 꺼낼 때마다 타입 캐스팅을 해야 합니다. 이 과정에서 `ClassCastException`이 발생할 수 있지만, 제네릭을 사용하면 이러한 문제를 컴파일 시점에 방지할 수 있습니다.

## 10. 제네릭을 사용한 경험을 소개해 주세요.

**10.1 제네릭을 통한 타입 안정성 보장**

제네릭을 사용함으로써 타입 안정성을 보장한 사례는 여러 가지가 있습니다. 예를 들어, `ArrayList`를 사용할 때 제네릭을 적용하면, 요소의 타입을 명확히 지정할 수 있어 컴파일 타임에 타입 오류를 미리 확인할 수 있습니다.

```java
ArrayList<String> stringList = new ArrayList<>();
stringList.add("Hello");
stringList.add("World");
// stringList.add(10); // 컴파일 오류 발생
```

위 코드에서 `stringList`는 `String` 타입의 요소만을 받을 수 있습니다. 만약 잘못된 타입의 데이터를 추가하려고 하면, 컴파일 타임에 오류가 발생하여 프로그램이 안정적으로 동작할 수 있도록 합니다.

**10.2 제네릭을 사용한 리팩토링 경험**

프로젝트에서 반복적으로 사용되는 코드를 리팩토링하면서 제네릭을 활용한 경험이 있습니다. 초기 코드에서는 다양한 타입의 데이터를 처리하는 여러 메서드가 존재했는데, 제네릭을 사용하여 코드 중복을 줄이고 유지보수성을 높일 수 있었습니다.

```java
public <T> void printArray(T[] array) {
    for (T element : array) {
        System.out.println(element);
    }
}
```

이와 같은 제네릭 메서드는 배열의 타입에 관계없이 모든 배열을 출력할 수 있어 코드의 재사용성을 극대화했습니다.
