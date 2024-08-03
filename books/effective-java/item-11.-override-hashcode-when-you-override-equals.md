---
description: equals를 재정의하려거든 hashCode도 재정의하라
---

# Item 11. Override hashCode When You Override equals

자바에서 `equals` 메소드를 재정의할 때는 `hashCode` 메소드도 반드시 재정의해야 한다. 그렇지 않으면 해당 클래스의 인스턴스를 `HashMap`, `HashSet`과 같은 컬렉션의 원소로 사용할 때 문제가 발생할 수 있다.



## 왜 hashCode를 재정의해야 하는가?

`equals` 메소드를 재정의한 클래스에서 `hashCode`를 재정의하지 않으면, 논리적으로 같은 객체가 서로 다른 해시코드를 반환하여 해시 기반 컬렉션에서 올바르게 동작하지 않는다. 자바의 `Object` 클래스는 다음과 같은 `hashCode` 규약을 정의한다:

1. **일관성 (Consistency)**: `equals` 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 객체의 `hashCode` 메소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
2. **동일성 (Equality)**: 두 객체가 `equals(Object)`에 의해 같다고 판단되면, 두 객체의 `hashCode`는 똑같은 값을 반환해야 한다.
3. **불일치 허용 (Inequality)**: 두 객체가 `equals(Object)`에 의해 다르다고 판단되더라도, 두 객체의 `hashCode`가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환하여 해시테이블의 성능이 좋아진다.



## hashCode 규약을 위반할 때 발생하는 문제

`hashCode`를 재정의하지 않으면 `equals` 메소드는 논리적으로 같은 객체를 비교할 때도 다른 해시코드를 반환할 수 있다. 이는 해시 기반 컬렉션에서 같은 객체를 다르게 인식하게 되어 문제가 발생한다. 예를 들어, 다음 코드를 보자:

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
```

위 코드에서 `PhoneNumber` 클래스의 인스턴스를 `HashMap`에 저장하고 있다. 만약 `PhoneNumber` 클래스가 `hashCode`를 재정의하지 않았다면, 동일한 논리적 동등성을 가지는 두 `PhoneNumber` 인스턴스는 서로 다른 해시코드를 반환하여 `HashMap`에서 올바르게 작동하지 않는다. 다음은 잘못된 `hashCode` 구현의 예시이다:

```java
@Override
public int hashCode() {
    return 42;
}
```

위 코드는 모든 객체에서 동일한 해시코드를 반환하여 해시테이블의 성능을 크게 저하시킨다. 해시 기반 컬렉션의 성능은 일반적으로 O(1)이지만, 이 경우 O(n)이 되어 성능이 급격히 저하될 수 있다.



## hashCode 메소드를 재정의하는 방법

**핵심 필드 사용**

`hashCode` 메소드는 객체의 핵심 필드 값을 기반으로 해시코드를 계산해야 한다. 이는 `equals` 메소드에서 비교에 사용되는 모든 필드를 포함해야 한다. 예를 들어, `PhoneNumber` 클래스의 경우 지역 코드, 프리픽스, 가입자 번호가 핵심 필드이다.

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
        if (o == this) return true;
        if (!(o instanceof PhoneNumber)) return false;
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    }

    @Override
    public int hashCode() {
        int result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        return result;
    }
}
```

### **해시 코드 생성 규칙**

해시코드는 가능한 한 고유한 값을 가져야 하므로, 필드의 값을 곱셈과 덧셈을 통해 조합하여 생성한다. 여기서 31과 같은 소수(prime number)를 사용하는 것이 일반적이다. 이는 충돌을 최소화하고 해시테이블의 성능을 최적화한다.

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```



## 해시 코드 계산의 주요 단계

1. **필드의 해시코드 계산**: 기본 타입 필드는 `Type.hashCode(f)`를 사용하고, 참조 타입 필드는 해당 클래스의 `hashCode` 메소드를 호출한다.
2. **결과를 합산**: 각 필드의 해시코드를 31로 곱한 후 결과를 더한다.
3. **결과 반환**: 최종 해시코드를 반환한다.

다음은 위 단계를 적용한 코드 예시이다:

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```



## 올바른 hashCode 메소드 구현의 예

아래는 `PhoneNumber` 클래스의 `hashCode` 메소드의 올바른 구현 예시이다:

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

이 메소드는 `PhoneNumber` 클래스의 각 필드 값을 기반으로 해시코드를 생성하며, 동일한 객체는 동일한 해시코드를 반환하게 된다.



## 추가적인 주의 사항

1. **AutoValue 사용하기**: `AutoValue` 프레임워크를 사용하면 `equals`와 `hashCode` 메소드를 자동으로 생성해준다. 이는 코드의 일관성을 유지하고, 수작업으로 인한 오류를 줄일 수 있다.
2. **캐싱**: 해시코드 계산 비용이 비싼 경우, 해시코드를 계산한 후 캐싱하여 다시 계산하지 않도록 할 수 있다. 이는 특히 불변 객체에서 유용하다.

**예시: 캐싱을 사용하는 hashCode 구현**

```java
private int hashCode; // 0으로 자동 초기화

@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

위 예시는 처음 해시코드를 계산한 후 `hashCode` 필드에 저장하고, 이후에는 캐시된 값을 반환한다.



## 결론

`equals` 메소드를 재정의할 때는 반드시 `hashCode` 메소드도 재정의해야 한다. 이는 해시 기반 컬렉션에서 객체가 올바르게 동작하게 하며, 프로그램의 예기치 않은 동작을 방지할 수 있다. 위에서 설명한 규칙과 방법을 따라 `hashCode` 메소드를 재정의하면, 안전하고 효율적인 자바 프로그램을 작성할 수 있을 것이다.

**핵심 정리**

* `equals` 메소드를 재정의할 때는 `hashCode`도 반드시 재정의해야 한다.
* 해시코드는 객체의 핵심 필드를 기반으로 생성해야 한다.
* 소수를 사용하여 해시코드를 조합하면 충돌을 최소화할 수 있다.
* `AutoValue` 프레임워크를 사용하면 자동으로 `equals`와 `hashCode` 메소드를 생성할 수 있다.
* 캐싱을 사용하면 해시코드 계산 비용을 줄일 수 있다.



## 질의

**질문 1: 왜 hashCode를 재정의할 때 소수(prime number)를 사용하는 것이 좋나요?**

**답변:** 소수(prime number)를 사용하면 해시코드가 더 고르게 분포되도록 도와줍니다. 이는 충돌을 줄이고 해시 기반 컬렉션의 성능을 최적화하는 데 도움이 됩니다. 예를 들어, 31은 자바에서 흔히 사용되는 소수로, 곱셈 연산이 최적화되어 있어 성능 측면에서도 유리합니다.

**질문 2: hashCode를 재정의할 때 모든 필드를 포함해야 하나요?**

**답변:** 모든 필드를 포함할 필요는 없습니다. `equals` 메소드에서 논리적 동등성을 결정하는 데 사용되는 핵심 필드만 포함하면 됩니다. 불필요한 필드까지 포함하면 해시코드 계산이 복잡해지고 성능이 저하될 수 있습니다.

**질문 3: 캐싱을 사용할 때 주의할 점은 무엇인가요?**

**답변:** 캐싱을 사용할 때는 객체가 불변(immutable)인지 확인해야 합니다. 객체의 상태가 변하면 해시코드도 변경되어야 하는데, 캐싱을 사용하면 해시코드가 변경되지 않기 때문에 논리적 오류가 발생할 수 있습니다. 따라서 캐싱은 주로 불변 객체에 사용됩니다.

**질문 4: 왜 hashCode는 0으로 초기화하면 안 되나요?**

**답변:** 0으로 초기화된 해시코드는 모든 객체가 동일한 해시코드를 가지게 만들어 해시 기반 컬렉션의 성능을 저하시킵니다. 이는 해시테이블의 버킷(bucket) 분포가 고르지 않게 되어, 성능이 O(n)으로 떨어질 수 있습니다.

**질문 5: AutoValue는 무엇인가요?**

**답변:** `AutoValue`는 구글에서 제공하는 프레임워크로, 자바 클래스의 `equals`, `hashCode`, `toString` 메소드를 자동으로 생성해줍니다. 이를 사용하면 코드의 일관성을 유지하고, 수작업으로 인한 오류를 줄일 수 있습니다. `AutoValue`는 빌드 시점에 코드를 생성하여 컴파일 타임에 포함됩니다.

**질문 6: hashCode 메소드를 재정의하지 않으면 어떤 일이 발생하나요?**

**답변:** `hashCode` 메소드를 재정의하지 않으면, 동일한 논리적 동등성을 가지는 객체가 다른 해시코드를 반환하여 해시 기반 컬렉션에서 올바르게 동작하지 않습니다. 이는 객체를 검색하거나 저장할 때 예기치 않은 동작을 초래할 수 있습니다. 예를 들어, `HashMap`이나 `HashSet`에서 객체를 찾지 못하거나 중복 저장되는 문제가 발생할 수 있습니다.

**질문 7: equals와 hashCode를 올바르게 재정의했는지 테스트하는 방법은 무엇인가요?**

**답변:** 테스트를 통해 equals와 hashCode의 일관성을 확인할 수 있습니다. JUnit과 같은 단위 테스트 프레임워크를 사용하여 동일한 객체와 다른 객체에 대해 `equals`와 `hashCode`가 올바르게 동작하는지 검증합니다. 또한, `equals`와 `hashCode`가 계약을 준수하는지 확인하는 라이브러리를 사용할 수도 있습니다.

**질문 8: hashCode를 재정의할 때 성능을 고려해야 하나요?**

**답변:** 네, hashCode를 재정의할 때 성능을 고려해야 합니다. 해시코드 계산이 너무 복잡하면 성능 저하를 초래할 수 있습니다. 특히 해시 기반 컬렉션에서 해시코드 계산이 빈번하게 발생하므로, 적절한 해시코드 생성 방법을 사용하여 성능을 최적화해야 합니다. 예를 들어, 필드의 곱셈과 덧셈을 통해 해시코드를 계산하면 성능을 향상시킬 수 있습니다.

**질문 9: equals와 hashCode의 재정의가 잘못된 경우 어떤 문제가 발생할 수 있나요?**

**답변:** equals와 hashCode의 재정의가 잘못된 경우 다음과 같은 문제가 발생할 수 있습니다:

* 해시 기반 컬렉션 (HashMap, HashSet 등)에서 객체를 찾지 못하거나 중복 저장될 수 있습니다.
* 객체의 논리적 동등성 비교가 일관되지 않게 됩니다.
* 컬렉션의 성능이 저하될 수 있습니다.
* 코드의 유지보수와 디버깅이 어려워질 수 있습니다.

**질문 10: hashCode와 equals의 규약을 위반하면 어떤 일이 발생하나요?**

**답변:** hashCode와 equals의 규약을 위반하면 다음과 같은 문제가 발생할 수 있습니다:

* 논리적으로 동일한 객체가 다른 해시코드를 반환하여 해시 기반 컬렉션에서 올바르게 동작하지 않습니다.
* 컬렉션의 효율성이 떨어지고 성능 저하가 발생할 수 있습니다.
* 프로그램의 예기치 않은 동작이나 버그가 발생할 수 있습니다.
* 코드의 일관성이 깨져 유지보수와 디버깅이 어려워집니다.
