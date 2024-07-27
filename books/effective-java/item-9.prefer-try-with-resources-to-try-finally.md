---
description: '아이템 9: try-finally 보다는 try-with-resources를 사용하라'
---

# Item 9.Prefer try-with-resources to try-finally

자바에서 리소스를 다룰 때, 파일, 소켓, 데이터베이스 연결 등은 사용 후 반드시 닫아야 합니다. 전통적으로 이를 위해 try-finally 블록을 사용해 왔지만, 이는 코드가 길어지고 오류를 유발할 수 있습니다. 자바 7부터 도입된 try-with-resources 구문은 이를 개선하여 코드의 간결성과 안전성을 높였습니다.



#### try-finally의 문제점

try-finally 구문은 리소스를 안전하게 닫기 위해 사용되지만, 다음과 같은 문제점이 있습니다:

1. **코드가 장황해짐**: 여러 리소스를 닫아야 할 때, 코드가 길어지고 복잡해집니다.
2. **오류 발생 가능성**: 리소스를 닫는 과정에서 예외가 발생하면 원래의 예외가 숨겨질 수 있습니다.
3. **가독성 저하**: 긴 코드 블록은 가독성을 떨어뜨리고, 유지보수를 어렵게 만듭니다.

**예시: try-finally 사용**

```java
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("test.txt"));
    // 파일 읽기
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (br != null) {
        try {
            br.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

위 예시에서 BufferedReader 객체를 사용한 후 닫는 과정이 복잡하고, 예외 처리가 번거롭습니다.



#### try-with-resources의 장점

try-with-resources 구문은 다음과 같은 장점을 제공합니다:

1. **코드 간결성**: 리소스를 자동으로 닫아주므로 코드가 간결해집니다.
2. **명확한 예외 처리**: 여러 예외가 발생해도 원래 예외가 숨겨지지 않고 명확히 처리됩니다.
3. **가독성 향상**: 코드가 짧아지고 구조가 단순해져 가독성이 향상됩니다.

**예시: try-with-resources 사용**

```java
try (BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
    // 파일 읽기
} catch (IOException e) {
    e.printStackTrace();
}
```

위 예시에서는 try-with-resources 구문을 사용하여 BufferedReader 객체를 자동으로 닫아줍니다.



#### 자바에서의 리소스 관리

**AutoCloseable 인터페이스**

try-with-resources 구문은 `AutoCloseable` 인터페이스를 구현한 객체에 대해 자동으로 close 메서드를 호출합니다. 자바 7부터 대부분의 리소스 클래스가 `AutoCloseable` 인터페이스를 구현하고 있습니다.

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

리소스 클래스가 `AutoCloseable` 인터페이스를 구현하면, try-with-resources 구문에서 해당 리소스를 사용할 수 있습니다. 이는 리소스가 더 이상 필요하지 않을 때 자동으로 close 메서드를 호출하여 리소스를 해제합니다.



#### try-with-resources를 사용한 예시

**기본 예시**

```java
try (BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

위 예시에서는 BufferedReader 객체를 try-with-resources 구문으로 사용하여 파일을 읽고, 자동으로 닫습니다. 이는 코드의 길이를 줄이고 예외 처리를 단순화합니다.



**복수 리소스 처리 예시**

여러 리소스를 사용하는 경우에도 try-with-resources 구문을 사용하면 간결하게 처리할 수 있습니다.

```java
try (BufferedReader br = new BufferedReader(new FileReader("test.txt"));
     PrintWriter pw = new PrintWriter(new FileWriter("output.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        pw.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

위 예시에서는 BufferedReader와 PrintWriter 객체를 동시에 사용하고, 자동으로 닫습니다. 이는 코드의 가독성을 높이고 리소스 해제의 안전성을 보장합니다.



Lombok은 자바에서 반복적인 코드를 줄여주는 라이브러리입니다. Lombok의 `@Cleanup` 어노테이션을 사용하여 리소스를 자동으로 닫을 수 있습니다. 이는 try-with-resources 구문과 유사한 방식으로 작동합니다.

```java
java코드 복사import lombok.Cleanup;
import java.io.*;

public class Example {
    public void readFile() throws IOException {
        @Cleanup BufferedReader br = new BufferedReader(new FileReader("test.txt"));
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    }

    public void writeFile() throws IOException {
        @Cleanup PrintWriter pw = new PrintWriter(new FileWriter("output.txt"));
        pw.println("Hello, World!");
    }
}
```

위 예시에서는 Lombok의 `@Cleanup` 어노테이션을 사용하여 BufferedReader와 PrintWriter 객체를 자동으로 닫아줍니다.



### 결론

try-finally 구문은 리소스를 안전하게 닫기 위해 사용되지만, 코드가 장황하고 오류가 발생할 가능성이 높습니다. 자바 7부터 도입된 try-with-resources 구문은 코드의 간결성과 안전성을 높여줍니다. try-with-resources 구문은 `AutoCloseable` 인터페이스를 구현한 객체에 대해 자동으로 close 메서드를 호출하므로, 리소스를 안전하게 관리할 수 있습니다. 또한, Lombok의 `@Cleanup` 어노테이션을 사용하면 더욱 간편하게 리소스를 닫을 수 있습니다.
