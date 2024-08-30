---
description: Try-with-reources
---

# Try-with-reources

Java의 **`try-with-resources`** 문법은 Java 7에서 도입된 기능으로, 자동으로 자원을 관리하고 해제할 수 있도록 도와줍니다. 이를 통해 코드가 더 간결해지고, 자원 누수(Resource Leak)를 방지할 수 있습니다. `try-with-resources`는 주로 파일, 네트워크 소켓, 데이터베이스 연결과 같은 자원을 처리할 때 사용됩니다.



`try-with-resources`는 `try` 블록이 끝나면 사용한 자원을 자동으로 닫아주는 구문입니다. 이를 위해 자원 클래스는 `AutoCloseable` 인터페이스를 구현해야 합니다. `AutoCloseable` 인터페이스에는 자원을 해제하는 `close()` 메서드가 포함되어 있습니다.



## 문법

<details>

<summary>기본 문법</summary>

```java
try (ResourceType resource = new ResourceType()) {
    // 자원을 사용한 작업 수행
} catch (ExceptionType e) {
    // 예외 처리
}
```

여기서 `ResourceType`은 `AutoCloseable` 인터페이스를 구현한 자원 클래스입니다. `try` 블록이 끝나면 `resource`의 `close()` 메서드가 자동으로 호출되어 자원이 해제됩니다.

</details>

## 장점

* 자원 누수 방지

기존의 `try-finally` 블록을 사용하여 자원을 해제할 때, 예외가 발생하거나 자원을 명시적으로 닫지 않는 경우 자원 누수가 발생할 수 있습니다. `try-with-resources`는 이 문제를 방지하여 자원이 적절히 해제되도록 보장합니다.

* 코드 간결성

`try-with-resources`를 사용하면 자원을 닫는 코드를 별도로 작성할 필요가 없어 코드가 더 간결하고 읽기 쉬워집니다.

* 중첩된 자원 관리

여러 자원을 동시에 관리할 때도 간단하게 사용할 수 있으며, 모든 자원이 자동으로 안전하게 해제됩니다.



## Try-finally와의 비교

<details>

<summary>Try-finally</summary>

```java
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("example.txt"));
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    System.err.println("파일을 읽는 도중 오류가 발생했습니다: " + e.getMessage());
} finally {
    if (br != null) {
        try {
            br.close();
        } catch (IOException ex) {
            System.err.println("BufferedReader를 닫는 도중 오류가 발생했습니다: " + ex.getMessage());
        }
    }
}
```

#### 설명:

* **복잡성**: `finally` 블록에서 자원을 해제하는 코드가 추가되어 코드가 복잡해집니다.
* **예외 처리**: 자원 해제 중 발생할 수 있는 예외를 처리하는 코드도 필요합니다.

`try-with-resources`를 사용하면 이러한 복잡한 자원 관리 코드가 더 간결하고 명확하게 바뀔 수 있습니다.

</details>

