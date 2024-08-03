---
description: toString을 항상 재정의하라
---

# Item 12. Always Override toString

자바에서 `toString` 메소드는 객체의 문자열 표현을 반환하는 중요한 메소드입니다. `Object` 클래스의 기본 `toString` 메소드는 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우가 거의 없습니다.



## 왜 toString을 재정의해야 하는가?

`toString` 메소드는 객체의 문자열 표현을 반환합니다. 이 메소드를 잘 재정의하면 디버깅, 로깅, 출력 등의 상황에서 유용하게 사용할 수 있습니다. 기본 `toString` 메소드는 클래스 이름과 해시코드를 반환하는데, 이는 유용한 정보를 담고 있지 않습니다. 예를 들어:

```java
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }

    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
}
```

위 예시처럼 `PhoneNumber` 클래스의 `toString` 메소드를 재정의하면 `PhoneNumber@1db9742` 대신 `707-867-5309`와 같은 유용한 정보를 출력할 수 있습니다.



## toString의 규약

`toString` 메소드는 객체의 "읽기 쉬운" 문자열 표현을 반환해야 합니다. 이는 객체의 주요 정보를 포함해야 하며, 가능하면 사람이 이해하기 쉬운 형식이어야 합니다. 다음은 `toString` 메소드의 일반적인 규약입니다:

1. **간결하고 명확한 정보 제공**: 객체의 주요 정보를 포함하여 간결하게 표현합니다.
2. **읽기 쉬운 형식**: 사람이 읽기 쉽고 이해하기 쉬운 형식으로 반환합니다.
3. **모든 필드를 포함할 필요는 없음**: 객체의 상태를 나타내는 주요 필드만 포함합니다.
4. **디버깅과 로깅에 유용**: `toString` 메소드를 잘 재정의하면 디버깅과 로깅 시 유용합니다.



## 좋은 toString 메소드의 예

다음은 `toString` 메소드를 잘 재정의한 예시입니다:

```java
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return String.format("Person[name=%s, age=%d]", name, age);
    }
}
```

이 예시에서는 `name`과 `age` 필드를 포함하여 객체의 주요 정보를 간결하고 명확하게 표현하고 있습니다.

#### toString 메소드를 재정의할 때 주의할 점

1. **중요한 정보를 누락하지 말 것**: 객체의 주요 상태를 나타내는 정보를 포함해야 합니다.
2. **너무 길지 않게 유지**: 너무 많은 정보를 포함하면 가독성이 떨어질 수 있습니다.
3. **형식을 일관되게 유지**: 동일한 형식을 사용하여 일관된 출력을 제공합니다.
4. **데이터 유출 주의**: 민감한 정보를 포함하지 않도록 주의합니다.



## 포맷을 명시하지 않는 경우

때로는 포맷을 명시하지 않고 `toString` 메소드를 재정의해야 할 때가 있습니다. 이 경우에도 객체의 주요 정보를 포함하여 유용한 정보를 제공해야 합니다. 예를 들어:

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public String toString() {
        return "Point(" + x + ", " + y + ")";
    }
}
```

이 예시에서는 포맷을 명시하지 않았지만, 객체의 주요 정보를 간결하게 표현하고 있습니다.



## AutoValue와 toString

`AutoValue`는 구글에서 제공하는 프레임워크로, 자바 클래스의 `equals`, `hashCode`, `toString` 메소드를 자동으로 생성해줍니다. 이를 사용하면 코드의 일관성을 유지하고, 수작업으로 인한 오류를 줄일 수 있습니다. `AutoValue`는 빌드 시점에 코드를 생성하여 컴파일 타임에 포함됩니다.

```java
@AutoValue
public abstract class AutoValueExample {
    public abstract String name();
    public abstract int age();

    public static AutoValueExample create(String name, int age) {
        return new AutoValue_AutoValueExample(name, age);
    }
}
```

위 코드에서는 `AutoValue`를 사용하여 `toString` 메소드를 자동으로 생성합니다. 이는 코드의 일관성을 유지하고, 수작업으로 인한 오류를 줄일 수 있습니다.



## toString 메소드 재정의의 이점

1. **디버깅과 로깅**: 객체의 상태를 명확히 확인할 수 있어 디버깅과 로깅에 유용합니다.
2. **출력의 가독성 향상**: 객체의 주요 정보를 쉽게 읽을 수 있는 형식으로 제공합니다.
3. **코드 유지보수 용이**: 코드의 가독성을 높여 유지보수를 쉽게 합니다.



## 결론

`toString` 메소드를 재정의하는 것은 자바 프로그래밍에서 매우 중요한 부분입니다. 이를 통해 객체의 상태를 명확히 표현하고, 디버깅과 로깅 시 유용한 정보를 제공할 수 있습니다. `toString` 메소드를 재정의할 때는 간결하고 명확한 정보를 제공하며, 일관된 형식을 유지하는 것이 중요합니다. `AutoValue`와 같은 도구를 활용하면 코드의 일관성을 유지하고, 수작업으로 인한 오류를 줄일 수 있습니다.



## 핵심

* `toString` 메소드는 객체의 주요 정보를 간결하고 명확하게 표현해야 합니다.
* 디버깅, 로깅, 출력 등의 상황에서 유용하게 사용됩니다.
* 포맷을 명시하지 않더라도 주요 정보를 포함하여 유용한 문자열을 반환해야 합니다.
* `AutoValue`와 같은 도구를 사용하면 코드의 일관성을 유지하고 오류를 줄일 수 있습니다.



## 질의

**질문 1: toString 메소드의 기본 구현은 어떤 형태인가요?**

**답변:** 자바의 `Object` 클래스에서 기본적으로 제공하는 `toString` 메소드는 클래스 이름과 해시코드의 16진수 표현을 포함하며, 예를 들어 `PhoneNumber@1db9742`와 같은 형태입니다.

**질문 2: 어떤 경우에 toString 메소드를 재정의하지 않아도 되나요?**

**답변:** 상위 클래스에서 이미 적절한 `toString` 메소드를 제공하고 있는 경우, 하위 클래스에서 재정의할 필요가 없습니다. 예를 들어, `AbstractList`와 같은 클래스는 이미 유용한 `toString` 메소드를 제공하므로, 이를 상속받는 클래스에서는 별도로 재정의할 필요가 없습니다.

**질문 3: toString 메소드를 재정의할 때 보안상의 주의사항은 무엇인가요?**

**답변:** `toString` 메소드를 재정의할 때는 민감한 정보를 포함하지 않도록 주의해야 합니다. 예를 들어, 비밀번호, 개인 식별 번호, 금융 정보와 같은 민감한 데이터는 `toString` 메소드에서 제외해야 합니다. 이러한 정보가 로그나 출력에 노출될 경우 보안 문제가 발생할 수 있습니다.

**질문 4: AutoValue 프레임워크는 어떻게 사용하나요?**

**답변:** `AutoValue`는 구글에서 제공하는 프레임워크로, 자바 클래스의 `equals`, `hashCode`, `toString` 메소드를 자동으로 생성합니다. 이를 사용하려면 클래스에 `@AutoValue` 애노테이션을 추가하고, 추상 메소드를 통해 필드를 정의합니다.

```java
@AutoValue
public abstract class AutoValueExample {
    public abstract String name();
    public abstract int age();

    public static AutoValueExample create(String name, int age) {
        return new AutoValue_AutoValueExample(name, age);
    }
}
```

**질문 5: toString 메소드에서 반환하는 문자열에 포맷을 명시하는 것이 중요한가요?**

**답변:** 포맷을 명시하는 것은 매우 중요합니다. 명확하고 일관된 포맷을 사용하면 객체의 상태를 쉽게 이해할 수 있습니다. 특히, 로그나 디버깅 시에 유용합니다. 포맷을 명시하지 않으면 출력 결과가 일관되지 않아 가독성이 떨어질 수 있습니다.

**질문 6: toString 메소드에서 모든 필드를 포함해야 하나요?**

**답변:** 모든 필드를 포함할 필요는 없습니다. `toString` 메소드는 객체의 주요 상태를 나타내는 핵심 필드만 포함하면 됩니다. 너무 많은 필드를 포함하면 가독성이 떨어질 수 있으므로, 중요한 정보를 간결하게 표현하는 것이 좋습니다.

**질문 7: toString 메소드를 자동 생성하는 다른 도구가 있나요?**

**답변:** `AutoValue` 외에도 `Lombok`과 같은 도구가 있습니다. `Lombok`은 `@ToString` 애노테이션을 사용하여 `toString` 메소드를 자동으로 생성합니다.

```java
java코드 복사import lombok.ToString;

@ToString
public class Person {
    private String name;
    private int age;
}
```

**질문 8: toString 메소드를 재정의할 때 형식을 지정하지 않으면 어떻게 되나요?**

**답변:** 형식을 지정하지 않으면 `toString` 메소드의 출력이 일관되지 않을 수 있습니다. 이는 특히 로그나 디버깅 시에 가독성을 떨어뜨리고, 객체의 상태를 이해하기 어렵게 만듭니다. 가능한 한 명확한 형식을 지정하여 일관된 출력을 제공하는 것이 좋습니다.
