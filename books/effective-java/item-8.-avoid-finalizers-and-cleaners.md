---
description: '아이템 8: finalizer와 cleaner 사용을 피하라'
---

# Item 8. Avoid Finalizers and Cleaners

자바에서 객체가 더 이상 필요 없을 때 가비지 컬렉터가 이를 회수하여 메모리를 해제합니다. 그러나 가비지 컬렉터만으로는 파일, 소켓, 데이터베이스 연결과 같은 리소스를 적절히 정리할 수 없습니다. 이를 위해 전통적으로 finalizer와 cleaner가 사용되었지만, 이 방법들은 여러 문제점을 가지고 있습니다.



### **finalizer와 cleaner의 개념**

**finalizer**는 자바의 `Object` 클래스에서 제공하는 메서드로, 객체가 가비지 컬렉터에 의해 회수되기 전에 호출됩니다. **cleaner**는 자바 9에서 도입된 기능으로, `java.lang.ref.Cleaner` 클래스를 통해 객체가 더 이상 참조되지 않을 때 실행할 작업을 지정할 수 있습니다. 이 둘은 모두 객체가 더 이상 사용되지 않을 때 자원을 해제하는 데 사용됩니다.



### **finalizer와 cleaner의 문제점**

1. **성능 저하**

finalizer와 cleaner는 성능에 부정적인 영향을 미칩니다. 가비지 컬렉션 과정에서 finalizer와 cleaner를 실행하는 데 추가적인 리소스와 시간이 소요됩니다. 이는 프로그램의 전반적인 성능을 저하시킬 수 있습니다.

```java
@Override
protected void finalize() throws Throwable {
    try {
        // 리소스 정리 작업
    } finally {
        super.finalize();
    }
}
```

2. **예측 불가능한 동작**

finalizer와 cleaner는 언제 실행될지 예측할 수 없습니다. 가비지 컬렉터가 객체를 회수하는 시점에 따라 실행 시점이 달라지므로, 자원이 즉시 해제되지 않을 수 있습니다. 이로 인해 프로그램의 동작이 예측 불가능해질 수 있습니다.

```java
public class Resource {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        @Override
        public void run() {
            // 리소스 정리 작업
        }
    }

    private final State state;
    private final Cleaner.Cleanable cleanable;

    public Resource() {
        this.state = new State();
        this.cleanable = cleaner.register(this, state);
    }
}
```

3. **자원 회수 실패**

finalizer와 cleaner는 자원을 확실히 회수할 수 있다는 보장이 없습니다. 가비지 컬렉터가 객체를 회수하지 않으면 finalizer와 cleaner도 실행되지 않습니다. 이로 인해 중요한 자원이 적절히 해제되지 않을 수 있습니다.



### **자바에서 리소스를 안전하게 회수하는 방법**

**try-with-resources**

try-with-resources는 자바 7부터 도입된 기능으로, 리소스를 자동으로 닫아줍니다. 이를 사용하면 코드가 간결해지고, 예외가 발생해도 리소스가 확실히 해제됩니다.

```java
try (BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
    // 파일 읽기
} catch (IOException e) {
    e.printStackTrace();
}
```



**명시적 종료 메서드**

명시적 종료 메서드는 객체가 더 이상 필요 없을 때 자원을 해제하는 메서드입니다. 일반적으로 `close`나 `dispose`와 같은 이름으로 정의됩니다.

```java
public class Resource {
    public void close() {
        // 자원 해제 로직
    }
}
```



### **대안 방법: try-with-resources와 명시적 종료 메서드 예시**

```java
public class Example {
    public void readFile() {
        try (BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void useResource() {
        Resource resource = new Resource();
        try {
            // 리소스 사용
        } finally {
            resource.close();
        }
    }
}
```

위 예시에서는 `BufferedReader` 객체를 try-with-resources로 사용하고, `Resource` 객체를 명시적으로 종료합니다.



**Lombok을 이용한 예시**

Lombok은 자바에서 반복적인 코드를 줄여주는 라이브러리입니다. Lombok을 이용해 리소스를 안전하게 해제하는 예시를 살펴보겠습니다.

```java
import lombok.Cleanup;
import java.io.*;

public class Example {
    public void readFile() throws IOException {
        @Cleanup BufferedReader br = new BufferedReader(new FileReader("test.txt"));
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
    }

    public void useResource() {
        @Cleanup Resource resource = new Resource();
        // 리소스 사용
    }
}

class Resource {
    public void close() {
        // 자원 해제 로직
    }
}
```

위 예시에서는 Lombok의 `@Cleanup` 어노테이션을 사용하여 `BufferedReader`와 `Resource` 객체를 자동으로 닫아줍니다.

#### 6. 결론

finalizer와 cleaner는 자바에서 자원을 해제하기 위해 사용되지만, 성능 저하, 예측 불가능한 동작, 자원 회수 실패 등의 문제점을 가지고 있습니다. 더 나은 대안으로 try-with-resources와 명시적 종료 메서드를 사용할 수 있습니다. try-with-resources는 자바 7부터 도입된 기능으로, 코드가 간결해지고 리소스를 안전하게 해제할 수 있습니다. 명시적 종료 메서드는 객체가 더 이상 필요 없을 때 자원을 해제하는 메서드입니다. Lombok과 같은 라이브러리는 이러한 작업을 더욱 간편하게 만들어 줄 수 있습니다.
