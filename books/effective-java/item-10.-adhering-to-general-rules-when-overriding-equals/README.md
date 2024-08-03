---
description: equals는 일반 규약을 지켜 재정의하라
---

# Item 10. Adhering to General Rules When Overriding equals

자바에서 `equals` 메소드는 객체의 논리적 동등성을 비교하는 데 사용된다. 이는 특히 컬렉션 프레임워크에서 중요한 역할을 한다. 하지만 `equals` 메소드를 잘못 재정의하면 예상치 못한 문제를 일으킬 수 있다.



## **1. 각 인스턴스가 본질적으로 고유하다**

`equals` 메소드는 논리적 동등성을 검사하므로, 값을 포함하는 클래스가 아니라 인스턴스가 고유한 객체에 대해 정의되어야 한다. 예를 들어 `Thread` 클래스와 같은 경우 `Object`의 `equals` 메소드를 그대로 사용한다. 다음은 `Thread` 클래스의 `equals` 메소드 예시이다.

```java
@Override
public boolean equals(Object o) {
    return this == o;
}
```



## **2. 인스턴스의 '논리적 동등성'을 검사할 일이 없다**

논리적 동등성(Logical Equality)을 검사할 필요가 없는 경우 `equals` 메소드를 재정의하지 않는다. 예를 들어, `java.util.regex.Pattern` 클래스는 두 `Pattern` 객체의 논리적 동등성을 검사하지 않는다.



## **3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다**

추상 클래스에서 `equals`를 구현할 때, 모든 하위 클래스가 같은 논리적 동등성 기준을 공유하는 경우 `equals` 메소드를 재정의하지 않는다. 예를 들어, `AbstractList`와 같은 클래스는 상속받는 하위 클래스에서 추가적인 동작 없이 상위 클래스의 `equals` 메소드를 사용할 수 있다.



## **4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다**

이런 경우도 `equals` 메소드를 재정의할 필요가 없다. 이는 메소드가 외부에 노출되지 않기 때문이다.



## 언제 equals를 재정의해야 하는가?

다음은 `equals` 메소드를 재정의해야 하는 경우이다:

* **객체 식별성(Object Identity)**: 두 객체가 동일한 객체인지 확인해야 할 때가 아니라 논리적 동등성을 확인해야 할 때. 예를 들어, 값 클래스(예: `Integer`, `String`) 등은 `equals` 메소드를 재정의하여 논리적 동등성을 비교해야 한다.
* **상위 클래스의 equals가 논리적 동등성을 비교하도록 정의되지 않았을 때**: 상위 클래스의 `equals` 메소드가 논리적 동등성을 검사하지 않는 경우, 하위 클래스에서 이를 재정의해야 한다.
* **값을 표현하는 클래스**: 값 클래스는 동일한 값을 가지는 두 객체를 논리적으로 동일하다고 간주해야 한다. 예를 들어, 두 개의 `Float` 객체가 동일한 값을 가질 때 논리적으로 동등하다고 판단해야 한다.

## equals 메소드 구현 규약

`equals` 메소드는 다음과 같은 규약을 따라야 한다:

1. **반사성(Reflexivity)**: 모든 비null 참조 값 x에 대해, x.equals(x)는 true이다.
2. **대칭성(Symmetry)**: 모든 비null 참조 값 x와 y에 대해, x.equals(y)가 true이면 y.equals(x)도 true이다.
3. **추이성(Transitivity)**: 모든 비null 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)가 true이면 x.equals(z)도 true이다.
4. **일관성(Consistency)**: 모든 비null 참조 값 x와 y에 대해, equals 비교가 일관된 결과를 반환해야 한다.
5. **null-아님(Non-nullity)**: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false를 반환해야 한다.

### 1. 반사성 (Reflexivity)

**정의**

모든 비null 참조 값 x에 대해, x.equals(x)는 true이다.

**설명**

반사성은 객체가 자기 자신과 비교할 때 항상 true를 반환해야 한다는 의미입니다. 이는 논리적으로 타당합니다. 어떤 객체가 자신과 다르다고 판단될 이유는 없기 때문입니다. 반사성을 충족하지 않는 `equals` 메소드는 논리적으로 문제가 발생할 수 있습니다.

**예시**

```java
public class Person {
    private String name;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;  // 자기 자신과 비교하면 항상 true
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
}
```



### 2. 대칭성 (Symmetry)

**정의**

모든 비null 참조 값 x와 y에 대해, x.equals(y)가 true이면 y.equals(x)도 true이다.

**설명**

대칭성은 두 객체가 서로를 비교할 때 결과가 동일해야 한다는 의미입니다. 즉, x.equals(y)가 true라면 y.equals(x)도 true여야 합니다. 이를 통해 객체 비교의 일관성을 유지할 수 있습니다. 대칭성을 충족하지 않는 `equals` 메소드는 서로 다른 객체 간의 비교에서 일관성이 없어질 수 있습니다.

**예시**

```java
public class Person {
    private String name;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
}
```



### 3. 추이성 (Transitivity)

**정의**

모든 비null 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)가 true이면 x.equals(z)도 true이다.

**설명**

추이성은 객체 비교의 결과가 일관되게 전달되어야 한다는 의미입니다. x가 y와 같고 y가 z와 같다면, x도 z와 같아야 합니다. 이를 통해 객체 비교의 체계적인 일관성을 유지할 수 있습니다. 추이성을 충족하지 않는 `equals` 메소드는 객체 간 비교 결과가 혼란스러워질 수 있습니다.

**예시**

```java
public class Person {
    private String name;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
}
```



### 4. 일관성 (Consistency)

**정의**

모든 비null 참조 값 x와 y에 대해, equals 비교가 일관된 결과를 반환해야 한다.

**설명**

일관성은 동일한 객체들 간의 비교 결과가 항상 일관되게 유지되어야 한다는 의미입니다. 만약 x와 y가 같다면, x와 y의 비교는 언제나 같은 결과를 반환해야 합니다. 이를 통해 객체 비교의 신뢰성을 유지할 수 있습니다. 일관성을 충족하지 않는 `equals` 메소드는 동일한 객체들 간의 비교에서 다른 결과를 반환할 수 있습니다.

**예시**

```java
public class Person {
    private String name;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
}
```



### 5. null-아님 (Non-nullity)

**정의**

null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false를 반환해야 한다.

**설명**

null-아님은 어떤 객체도 null과 비교할 때 true를 반환하지 않아야 한다는 의미입니다. 이는 객체 비교에서 null 값을 특별하게 다루기 위해 필요합니다. null과의 비교에서 true를 반환하는 것은 논리적으로 맞지 않으며, 프로그램에서 예기치 않은 오류를 일으킬 수 있습니다.

**예시**

```java
public class Person {
    private String name;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
}
```



### 종합 예시

위의 모든 규약을 충족하는 종합적인 `equals` 메소드 구현 예시를 보면 다음과 같습니다:

```java
public class Person {
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true; // 반사성
        if (o == null || getClass() != o.getClass()) return false; // null-아님 및 타입 체크
        Person person = (Person) o;
        return Objects.equals(name, person.name); // 핵심 필드 비교
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```

이 예시에서는 `equals` 메소드가 반사성, 대칭성, 추이성, 일관성, null-아님의 모든 규약을 충족하도록 구현되었습니다. 또한, `equals`를 재정의할 때는 반드시 `hashCode` 메소드도 함께 재정의해야 함을 보여줍니다.



다음은 위 규약을 준수하는 `equals` 메소드의 예시이다:

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        }
        if (o instanceof String) {
            return s.equalsIgnoreCase((String) o);
        }
        return false;
    }

    // 나머지 코드는 생략
}
```

위 코드는 대칭성을 위반할 수 있다. 다음과 같이 수정할 수 있다:

```java
@Override
public boolean equals(Object o) {
    if (o instanceof CaseInsensitiveString) {
        return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    }
    return false;
}
```

이렇게 하면 `String`과 비교할 때 대칭성 문제가 발생하지 않는다.



## 올바른 equals 메소드 구현 단계

1. **== 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다**: 이는 성능 최적화를 위해 사용된다.
2. **instanceof 연산자로 입력이 올바른 타입인지 확인한다**: 입력이 다른 타입이면 false를 반환한다.
3. **입력을 올바른 타입으로 형변환한다**: 형변환은 안전하다.
4. **입력 객체와 자기 자신의 대응되는 ‘핵심’ 필드들이 모두 일치하는지 하나씩 검사한다**: 모든 필드가 일치하면 true, 그렇지 않으면 false를 반환한다.

다음은 이 단계를 따른 `PhoneNumber` 클래스의 `equals` 메소드 예시이다:

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof PhoneNumber)) {
            return false;
        }
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }

    // 나머지 코드는 생략
}
```

#### equals를 재정의할 때 hashCode도 반드시 재정의하자

`equals`를 재정의할 때 `hashCode` 메소드도 반드시 재정의해야 한다. 그렇지 않으면 `HashSet`, `HashMap` 등 해시 기반 컬렉션의 동작이 불완전해질 수 있다.

## 결론

올바른 `equals` 메소드를 구현하는 것은 자바 프로그래밍에서 매우 중요한 부분이다. 이는 객체의 논리적 동등성을 정의하며, 특히 컬렉션 프레임워크에서 올바르게 동작하는 데 필수적이다. 따라서 `equals` 메소드를 재정의할 때는 반드시 일반 규약을 따르고, 필요에 따라 `hashCode` 메소드도 함께 재정의해야 한다.

핵심 정리:

* `equals` 메소드를 재정의할 때는 일반 규약을 반드시 따라야 한다.
* 필요할 경우 `equals`와 함께 `hashCode`도 재정의해야 한다.
* 값 객체의 경우 모든 핵심 필드를 비교해야 한다.
* 객체 식별성이 아닌 논리적 동등성을 검사하는 경우에만 `equals`를 재정의한다.
