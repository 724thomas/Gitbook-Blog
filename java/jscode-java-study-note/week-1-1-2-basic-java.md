---
description: 자바 기본
---

# Week 1(1/2) - Basic Java

<details>

<summary>Java 실행 과정에 대해서 설명해주세요.</summary>

* **소스 코드 작성 (Source Code)**:
  * 개발자는 `.java` 확장자를 가진 파일에 Java 소스 코드를 작성합니다. 이 소스 코드는 일반 텍스트로 작성된 Java 프로그래밍 언어로 작성됩니다.
* **컴파일 (Compilation)**:
  * 작성된 Java 소스 코드는 `javac`라는 Java 컴파일러에 의해 컴파일됩니다. 이 과정에서 `.java` 파일은 바이트코드(Bytecode)로 변환되며, 그 결과는 `.class` 확장자를 가진 파일로 저장됩니다.
  * 바이트코드는 플랫폼 독립적인 코드로, 특정 운영 체제나 하드웨어에 의존하지 않고, 어디서든지 실행될 수 있습니다.
* **클래스 로딩 (Class Loading)**:
  * 프로그램 실행 시, Java Virtual Machine(JVM)은 필요한 `.class` 파일들을 메모리에 로드합니다. 이 과정에서 ClassLoader라는 컴포넌트가 사용되며, 필요한 클래스들을 동적으로 로드합니다.
  * ClassLoader는 기본적으로 Java API에 포함된 클래스뿐만 아니라, 사용자가 정의한 클래스, 외부 라이브러리의 클래스들도 로드할 수 있습니다.
* **바이트코드 검증 (Bytecode Verification)**:
  * 로드된 바이트코드는 JVM에 의해 검증됩니다. 이 단계에서는 코드가 올바르고 안전한지, 즉 Java의 보안 모델에 부합하는지 확인합니다. 바이트코드 검증을 통해 메모리 접근 오류나 보안 문제가 발생하지 않도록 예방합니다.
* **인터프리트 또는 JIT 컴파일 (Interpretation or JIT Compilation)**:
  * JVM은 바이트코드를 인터프리터(Interpreter)를 통해 한 줄씩 읽고 해석하여 실행합니다. 이 방식은 즉시 실행할 수 있지만, 상대적으로 느릴 수 있습니다.
  * 이를 보완하기 위해 JVM은 Just-In-Time (JIT) 컴파일러를 사용하여, 자주 사용되는 바이트코드를 네이티브 머신 코드로 컴파일합니다. 이렇게 하면 프로그램의 실행 속도가 크게 향상됩니다.
* **실행 (Execution)**:
  * JIT 컴파일러에 의해 네이티브 코드로 변환된 바이트코드는 JVM에 의해 실제로 실행됩니다. 이때, 프로그램은 운영 체제에서 제공하는 리소스를 사용하고, 필요에 따라 메모리 관리, 스레드 관리 등을 수행합니다.
* **메모리 관리 및 가비지 컬렉션 (Memory Management and Garbage Collection)**:
  * 프로그램 실행 중 JVM은 자동 메모리 관리를 수행하며, 더 이상 사용되지 않는 객체들을 가비지 컬렉터(Garbage Collector)를 통해 자동으로 제거합니다. 이를 통해 메모리 누수를 방지하고, 애플리케이션의 안정성을 높입니다.

</details>

<details>

<summary>Java Bytecode에 대해서 설명해주세요.</summary>

Java Bytecode는 Java 프로그램이 컴파일된 후 생성되는 중간 코드 형식입니다. Java Bytecode는 플랫폼 독립적이며, JVM(Java Virtual Machine)이 이해하고 실행할 수 있는 형태로 변환된 코드입니다.

</details>

<details>

<summary>Java의 인터프리터(interpreter) 방식과 JIT 컴파일(compile) 방식에 대해서 설명해주세요.</summary>

#### **인터프리터 방식 (Interpreter)**

인터프리터는 Java 바이트코드를 한 줄씩 순차적으로 읽고, 이를 바로 실행합니다. 즉, 프로그램이 실행될 때마다 JVM이 바이트코드 명령을 한 줄씩 해석하여 즉시 실행하는 방식입니다.

#### **JIT 컴파일 방식 (Just-In-Time Compilation)**

* **작동 방식**: JIT 컴파일러는 Java 바이트코드를 인터프리터 방식으로 실행하다가, 자주 실행되는 코드 블록을 발견하면 해당 블록을 네이티브 머신 코드로 컴파일합니다. 이렇게 변환된 네이티브 코드는 이후에 다시 해석할 필요 없이 직접 실행됩니다.

</details>

<details>

<summary>사용해본 Java 버전과 특징 그리고 왜 그 버전을 사용했는지 설명해주세요. <br>Java 8, 11, 17 버전에 대해 아는대로 설명해주세요.</summary>

Java 8

* **람다 표현식**: Java 8에서는 람다 표현식이 도입되어, 코드가 간결해지고, 함수형 프로그래밍 스타일을 도입할 수 있게 되었습니다.
* **스트림 API**: 컬렉션 처리에 있어서 선언형 프로그래밍이 가능해졌으며, 데이터의 필터링, 매핑, 축소 등의 작업을 간편하게 처리할 수 있습니다.
* **Optional 클래스**: NullPointerException을 예방하고, 명시적으로 값이 없음을 표현할 수 있는 `Optional` 클래스가 추가되었습니다.
* **CompletableFuture**: Java 8에서는 비동기 프로그래밍을 쉽게 구현할 수 있는 `CompletableFuture`가 도입되었습니다. 이를 통해 비동기 작업을 관리하고, 여러 작업의 완료를 기다리거나, 비동기 작업 간의 의존성을 쉽게 설정할 수 있습니다.

Java 11

* `var` 키워드를 사용할 수 있게 되어, 코드의 가독성을 높일 수 있습니다.
* `List.of()`, `Set.of()`, `Map.of()` 메서드를 사용하여 불변 컬렉션을 쉽게 생성할 수 있습니다.
* Java 11은 장기 지원(Long-Term Support) 버전으로, 오랜 기간 동안 유지보수와 업데이트가 제공됩니다.

Java 17

* **패턴 매칭**: `instanceof` 연산자를 사용한 패턴 매칭이 추가되어, 타입 검사와 캐스팅이 더욱 간결해졌습니다.
* **Sealed Classes**: 클래스를 상속받을 수 있는 하위 클래스를 제한할 수 있는 `sealed` 키워드가 도입되었습니다.
* **Switch Expressions**: `switch` 문이 표현식으로 사용될 수 있게 개선되었으며, 더 간결한 코드 작성이 가능해졌습니다.

</details>

<details>

<summary>JDK와 JRE에 대해서 설명해주세요.</summary>

#### **JRE (Java Runtime Environment)**

* **정의**: JRE는 Java 애플리케이션을 실행하기 위한 런타임 환경입니다. 개발된 Java 프로그램이 실행될 수 있도록 필요한 라이브러리와 기타 구성 요소들을 포함하고 있습니다.
* **역할**: JRE는 이미 컴파일된 Java 프로그램(.class 파일)을 실행하는 역할을 합니다. Java 애플리케이션을 실행하는 데 필요한 모든 것을 제공하지만, Java 프로그램을 개발하거나 컴파일할 수 있는 도구는 포함되어 있지 않습니다.

#### **JDK (Java Development Kit)**

* **정의**: JDK는 Java 애플리케이션을 개발하기 위한 완전한 개발 도구 세트입니다. JDK에는 Java 프로그램을 개발, 컴파일, 디버깅, 실행하는 데 필요한 모든 도구가 포함되어 있습니다.
* **역할**: JDK는 Java 프로그램을 작성하고, 컴파일하며, 테스트할 수 있는 모든 도구를 제공합니다. 따라서 Java 개발자는 JDK를 사용하여 소프트웨어를 개발합니다.

</details>

<details>

<summary>동일성과 동등성에 대해 설명해 주세요.<br>equals()와 ==의 차이점은 무엇일까요?</summary>

동일성은 두 객체가 **같은 메모리 주소**를 참조하고 있는지를 나타냅니다. 즉, 동일한 객체를 가리키는지 여부를 확인하는 것입니다. `==` 연산자는 \*\*동일성(Identity)\*\*을 비교합니다. 즉, 두 객체가 **같은 메모리 주소**를 참조하고 있는지를 확인합니다.

동등성은 두 객체의 **내용**이 같은지를 나타냅니다. 즉, 객체가 가지고 있는 데이터가 같은지를 확인하는 것입니다. `equals()` 메서드는 \*\*동등성(Equality)\*\*을 비교합니다. 즉, 두 객체의 **내용**이 같은지를 확인합니다.

</details>

<details>

<summary>HashCode를 설명하고, `equals()` 와 `hashCode()` 의 차이점에 대해 설명해 주세요.<br>왜 `equals()` 외에 `hashCode()` 도 재정의해야 하나요?</summary>

`hashCode()`는 Java에서 객체를 식별하기 위한 정수 값을 반환하는 메서드입니다. 이 정수 값은 객체의 메모리 주소나 그 객체의 내용을 기반으로 생성되며, HashTable, HashMap, HashSet과 같은 해시 기반의 컬렉션에서 객체를 빠르게 검색하거나 비교하는 데 사용됩니다.

`equals()` 메서드 외에 `hashCode()` 메서드도 재정의해야 하는 이유는 Java의 해시 기반 컬렉션(`HashMap`, `HashSet`, `Hashtable` 등)에서 객체를 효율적으로 저장하고 검색하기 위해 두 메서드가 함께 사용되기 때문입니다.

* 해시 기반 컬렉션의 일관성 보장
* 효율적인 데이터 저장과 검색

</details>

<details>

<summary>toString()에 대해서 설명해주세요.</summary>

메서드는 Java에서 객체를 문자열로 표현할 때 사용되는 메서드입니다. 이 메서드는 객체의 상태를 사람이 읽을 수 있는 형식의 문자열로 반환합니다.

</details>

<details>

<summary>자바에서 메인 메서드는 왜 static으로 되어 있을까요?</summary>

프로그램이 실행될 때 이 메서드가 클래스의 인스턴스 없이도 호출될 수 있어야 하기 때문입니다. 즉, Java 애플리케이션이 실행될 때 가장 먼저 호출되는 메서드입니다. 이 메서드가 실행되어야 프로그램이 시작되고, 그 안에서 필요한 객체들이 생성되며, 애플리케이션이 본격적으로 작동하게 됩니다.

</details>

<details>

<summary>상수(Constant)와 리터럴(Literal)에 대해서 설명해주세요.</summary>

리터럴은 소스 코드에서 직접 값을 표현하는 방식입니다. 즉, 프로그램 내에서 고정된 값을 나타내는 구체적인 표현입니다. 예를 들어, 숫자 `10`, 문자 `'A'`, 문자열 `"Hello"` 등은 모두 리터럴입니다.

상수는 변하지 않는 값을 가지는 변수로, 보통 `final` 키워드를 사용하여 선언됩니다. 상수는 한 번 초기화되면 이후 값이 변경될 수 없습니다. 상수는 주로 프로그램에서 반복적으로 사용되는 값이나, 변경되어서는 안 되는 값에 사용됩니다.



* **리터럴**은 코드에 직접 쓰여진 고정된 값 그 자체를 의미하며, 변수에 할당되거나 표현식에서 사용됩니다. 리터럴은 변경되지 않지만, 특정 변수에 여러 번 할당될 수 있습니다.
* **상수**는 특정한 값(일반적으로 리터럴)을 가진 변수를 의미하며, `final` 키워드를 사용해 그 값이 변경되지 않도록 보장합니다. 상수는 값을 의미 있게 표현하고, 코드에서 반복적으로 사용되는 값을 하나의 명확한 이름으로 정의할 수 있도록 도와줍니다.

</details>

<details>

<summary>Primitive Type과 Reference Type에 대해서 설명해주세요.</summary>

**Primitive Type**은 Java에서 가장 기본적인 데이터 타입으로, 실제 값을 직접 저장합니다.

**Reference Type**은 객체를 참조하는 데 사용됩니다. 즉, Reference Type 변수는 객체의 메모리 주소를 저장하고, 이 주소를 통해 실제 객체에 접근합니다.

</details>

<details>

<summary>Java는 Call by Value 일까요? 아님 Call by Reference 일까요?</summary>

Java는 **Call by Value** 방식을 사용합니다. 이는 Java에서 메서드에 인수(매개변수)를 전달할 때 인수의 실제 값이 복사되어 전달된다는 의미입니다. 메서드 내부에서 이 복사된 값은 원래 변수와는 독립적으로 존재하므로, 메서드 내에서 이 값을 변경해도 원래 변수에는 아무런 영향을 미치지 않습니다.

</details>

<details>

<summary>Java 직렬화(Serialization)에 대해서 설명해주세요.</summary>

\*\*직렬화(Serialization)\*\*는 Java 객체를 바이트 스트림으로 변환하여 파일, 네트워크, 데이터베이스 등에 저장하거나 전송할 수 있도록 하는 과정입니다. 직렬화된 객체는 나중에 **역직렬화(Deserialization)** 과정을 통해 다시 원래의 객체로 복원될 수 있습니다.

Java 객체는 메모리 내에서만 존재하기 때문에, 객체의 상태를 유지한 채로 파일 시스템에 저장하거나 네트워크를 통해 전송하기 위해서는 객체를 연속된 바이트 형식으로 변환할 필요가 있습니다.

</details>