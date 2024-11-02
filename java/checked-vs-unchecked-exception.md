# Checked vs Unchecked Exception

## 1. 기본 개념

자바 예외는 Throwable 클래스를 기반으로 합니다. 그리고 이를 상속받는 Exception과 Error로 나뉩니다. 이 중에서 Exception은 프로그램(애플리케이션)내에 발생하는 오류입니다.

**Checked Exception**은 외부 환경과의 상호작용에서 발생할 수 있는 예외로, 컴파일 시점에 예외 처리를 강제하여 안정성을 높입니다. 주로 `try-catch`나 `throws`로 명시적인 예외 처리가 필요합니다.

**Unchecked Exception**은 프로그래밍 오류에서 발생하며, 개발자가 필요한 경우에만 예외 처리를 할 수 있습니다. 필수가 아니므로 코드가 간결해지는 장점이 있지만, 발생 시 프로그램이 비정상 종료될 가능성이 있습니다.

<figure><img src="../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

## 2. Checked Exception

컴파일 시점에 발생 가능성을 검사하는 예외입니다. 보통 외부 자원(파일, 네트워크, 데이터베이스 등)과의 작업에서 주로 발생하며, 예상치 못한 오류를 방지하기 위해 사용해야합니다.&#x20;

### 2.1. 특징

* 컴파일러 검사: Checked Exception은 컴파일 시점에 컴파일러가 검사합니다.
* 예외 처리 강제: try-catch 문으로 예외를 처리하거나, throws를 사용해 예외를 상위 메서드로 전달해야 합니다.
* 외부 리소스 의존: 파일 입출력, 네트워크 연결, 데이터베이스 접근 등 외부 리소스를 다룰 때 자주 사용됩니다.

### 2.2. 장단점

* 장점: 예외 처리 여부를 강제하기 때문에 안정적인 코드 작성이 가능해집니다. 외부 리소스와 통신 시도시 실패할 가능성이 높기때문에 문제 예방이 가능합니다.
* 단점: 예외 처리 코드가 많아지면 코드의 복잡성이 높아지고 가독성이 떨어집니다.



## 3. Unchecked Exception

Unchecked Exception은 런타임 시점에 발생하며, 컴파일러가 예외 처리를 강제하지 않는 예외입니다. 주로 프로그래밍 로직 오류로 인해 발생하며, 개발자가 필요한 경우에만 예외 처리를 합니다.

### 3.1. 특징

* 컴파일러 비검사: 컴파일시에 오류가 발생하지 않습니다.
* 주로 로직 오류: NullPointerException, ArithmaticException, ArrayIndexOutOfBoundsException 등이 포함되며, 프로그래밍 오류로 발생합니다.
* 선택적 예외 처리: 필수가 아니기 때문에 필요한 경우에만 예외 처리를 추가합니다.

### 3.2. 장단점

* 장점: 예외 처리를 선택적으로 할 수 있어서, 코드가 복잡해지지 않으며 가독성이 높습니다.
* 단점: 예외 처리를 강제하지 않기때문에, 프로그램이 비정상적으로 종료될 가능성이 있습니다.



## 4. Checked Exception과 Unchecked Exception의 실무

**Checked Exception**

* 외부 자원을 다루는 작업에서는 Checked Exception이 발생할 가능성이 높으므로, **적절한 예외 처리를 통해 오류를 방지**하는 것이 좋습니다.
* `try-catch`로 예외를 처리하거나 `throws`로 상위 메서드로 전달하여 처리할 수 있습니다.
* 외부 리소스 사용 시에는 `try-with-resources` 문법을 활용해 자원 반납을 자동화하는 것이 좋습니다.

**Unchecked Exception**

* 프로그래밍 오류로 인해 발생할 수 있는 Unchecked Exception은, 코드 리뷰와 테스트를 통해 **가능한 한 사전에 오류를 방지**하는 것이 좋습니다.
* NullPointerException이나 잘못된 인덱스 참조 등의 오류는 코드 작성 단계에서 방지하는 것이 우선입니다.
* 필요할 경우 예외 처리를 추가하여 오류 발생 시 사용자에게 의미 있는 메시지를 제공하거나, 에러 로그를 기록할 수 있습니다.



## 5. try-with-resources

자바에서 자원을 안전하게 관리하는 것은 애플리케이션의 안정성과 성능에 큰 영향을 미칩니다. 파일, 데이터베이스, 네트워크 소켓 등 외부 자원은 사용이 끝난 후 반드시 닫아야 하며, 그렇지 않으면 자원 누수와 같은 문제가 발생할 가능성이 있습니다. `try-with-resources`는 이러한 자원들을 자동으로 닫아주는 기능을 제공하여, **보다 안전하고 간결한 코드 작성**을 가능하게 해줍니다.



<details>

<summary>try-catch</summary>

```java
public class TryCatchExample {
    public void readFile(String fileName) {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader(fileName));
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            System.err.println("파일 읽기 중 오류 발생: " + e.getMessage());
        } finally {
            // 자원을 수동으로 닫아야 함
            try {
                if (reader != null) reader.close();
            } catch (IOException e) {
                System.err.println("자원 닫기 실패: " + e.getMessage());
            }
        }
    }
}
```

</details>

<details>

<summary>try-with-resources</summary>

```java
public class FileReaderExample {
    public void readFile(String fileName) {
        try (BufferedReader reader = new BufferedReader(new FileReader(fileName))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            System.err.println("파일 읽기 중 오류 발생: " + e.getMessage());
        }
    }
}
```



</details>

