---
description: 원시타입과 참조타입
---

# Primitive vs Object Type

## 원시타입

원시타입은 자바에서 제공하는 기본 데이터 타입입니다. 이는 'int', 'char', 'boolean', 'byte', 'short', 'long', 'float', 'double'의 8가지로 구성됩니다.

* 메모리 할당: 원시타입 변수는 실제 값을 스택 메모리에 저장합니다. 예를 들어, 'int x=10;'과 같이 선언하면 변수 'x'에 10이라는 값이 직접 저장됩니다.
* 특징: 원시타입 변수는 값을 직접 담고 있기 때문에, 값이 복사되더라도 다른 변수에 영향을 주지 않습니다. 즉, 원시타입 변수 간의 할당은 값 자체의 복사입니다.



## 참조타입

참조타입은 객체를 참조하는 변수입니다. 예를 들어, 배열, 클래스, 인터페이스 등이 이에 해당합니다.

* 매모리 할당: 참조타입 변수는 스택 메모리에 객체의 참조(메모리 주소)를 저장하고, 실제 객체는 힙 메모리에 할당됩니다. 예를 들어, 'String str = "Hello";'의 경우, 'str' 변수는 'Hello"라는 객체가 힙 메모리의 어느 위치에 저장되어 있는지를 참조합니다.
* 특징: 참조타입 변수의 복사는 참조 주소의 복사입니다. 즉, 하나의 객체를 여러 참조 변수가 가리킬 수 있으며, 그 중 하나를 통해 객체의 상태를 변경하면, 다른 참조 변수에도 그 변경이 반영됩니다.

#### 예시 코드:

```java
int a = 5;
int b = a; // b는 a의 값을 복사하여 가집니다.

String s1 = new String("Hello");
String s2 = s1; // s2는 s1이 참조하는 객체를 동일하게 참조합니다.
```



Q. 자바에서 문자열 비교 시 `==` 대신 `equals()`를 사용하는 이유는 무엇인가요?

A. `==` 연산자는 두 객체가 **동일한 메모리 주소**를 참조하는지 비교합니다. 즉, 두 문자열 객체가 동일한 객체를 가리키는지 확인하는 것입니다.

하지만, 자바에서 문자열(`String`)은 **객체**이기 때문에, 동일한 문자열 값이라도 `new` 키워드로 생성된 문자열 객체들은 각각 다른 메모리 주소를 가질 수 있습니다. 따라서, `==` 연산자는 서로 다른 메모리 주소를 가리키는 두 `String` 객체에 대해 `false`를 반환할 수 있습니다.

자바에서 `String`은 **불변(immutable)** 객체입니다. 한 번 생성된 `String` 객체는 그 값을 변경할 수 없습니다. 이러한 불변성 덕분에 자바는 **String Pool**이라는 메모리 영역을 활용하여 같은 문자열 값을 가진 객체들을 효율적으로 관리할 수 있습니다.
