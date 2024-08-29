---
description: Checked Exception과 Unchecked Exception
---

# Checked Exceptions and Unchecked Exceptions

## Checked Exception

Checked Exception은 컴파일 타임에 확인되는 예외입니다. 이러한 예외가 발생할 가능성이 있는 코드를 작성할 때 반드시 try-catch 블록을 사용하여 예외를 처리하거나, throws 키워드를 사용하여 해당 메서드가 예외를 던질 수 있음을 선언해야합니다.

대표적인 Checked Exception에는, IOException, SQLException, ClassNotFoundException 등이 있습니다.

<details>

<summary>예시</summary>

```java
public void readFile(String fileName) throws IOException {
    FileReader file = new FileReader(fileName);
    BufferedReader fileInput = new BufferedReader(file);
    fileInput.readLine();
}
```

</details>

### 장점

* 예외가 명확히 정의됩니다. 개발자가 예외 가능성을 인지하고 적절한 처리를 강제할 수 있습니다.
* 예외 처리 로직이 명확하며, 코드의 안전성을 높일 수 있습니다.

### 단점

* 코드가 복잡해지고 장황해질 수 있습니다.
* 모든 예외를 처리하는 것이 번거로울 수 있으며, 특히 작은 프로그램에서는 불필요하게 느껴질 수 있습니다.



## Unchecked Exception

Unchecked Exception은 런타임에 발생하는 예외입니다. 런타임에서 예외가 발생하면 프로그램은 비정상적으로 종료될 수 있습니다.

대표적인 Unchecked Exception에는 ArithmeticException, NullPointerException, ArrayIndexOutOfBoundsException 등이 있습니다.

<details>

<summary>예시</summary>

```java
public int divide(int a, int b) {
    return a / b;
}
```

위의 코드에서 `b`의 값이 0일 경우 `ArithmeticException`이 발생합니다. 이 예외는 Unchecked Exception으로, 컴파일러는 예외 처리를 강제하지 않지만, 실행 시 발생할 수 있는 위험이 존재합니다.

</details>

### 장점

* 코드가 간결해지고 불필요한 예외 처리 코드를 줄일 수 있습니다.
* 예외 처리에 대한 부담을 줄일 수 있어 더 직관적인 코드 작성이 가능합니다.

### 단점

* 예외가 발생했을 때 비정상적으로 종료될 가능성이 있습니다.
* 예외가 명확히 처리되지 않으면, 디버깅이 어려워질 수 있습니다.





## 스프링 트랜잭션 추상화

스프링의 트랜잭션 추상화는 **트랜잭션 관리의 복잡성을 숨기고, 다양한 데이터 접근 기술에 대해 일관된 방식으로 트랜잭션을 처리할 수 있도록 해주는 것**입니다. 스프링 트랜잭션 추상화를 사용하면, 개발자는 이러한 차이를 신경 쓸 필요 없이 동일한 방식으로 트랜잭션을 관리할 수 있습니다.

## 트랜잭션롤백 대상

스프링 프레임워크에서 트랜잭션(Transaction)은 데이터의 일관성을 유지하기 위해 사용됩니다. 트랜잭션 관리의 핵심은 특정 작업이 완료되었을 때 데이터가 일관되게 반영되도록 하고, 실패 시 데이터가 이전 상태로 롤백되도록 합니다.

### 기본 정책

스프링에서는 특정 예외가 발생했을 때 트랜잭션을 롤백할지 여부를 결정할 수 있습니다. 기본적으로 스프링은 Unchecked Exception(런타임 예외)과 `Error`가 발생하면 트랜잭션을 롤백하고, Checked Exception이 발생하면 트랜잭션을 커밋합니다.

롤백 정책은 커스터마이징이 가능합니다.

<details>

<summary>예시</summary>

```java
@Transactional(rollbackFor = IOException.class)
public void someMethod() throws IOException {
    // IOException 발생 시에도 롤백 처리
}
```

이 예시는 Checked Exception인 `IOException`이 발생하더라도 트랜잭션을 롤백

</details>

<details>

<summary>예시2</summary>

```java
@Transactional(rollbackFor = CustomException.class)
public void someMethod() throws CustomException {
    // CustomException 발생 시 롤백 처리
}
```

이와 같이, 특정 Custom Exception이 발생했을 때에도 트랜잭션을 롤백

</details>

