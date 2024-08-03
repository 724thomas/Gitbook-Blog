---
description: toString을 항상 재정의하라
---

# Always Override toString

### 1. 기본 toString의 한계

Object의 기본 toString 메서드는 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없습니다. 이 메서드는 다음과 같이 단순히 클래스 이름과 해시코드를 반환할 뿐입니다.

```java
@Override
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

이러한 기본 구현은 대부분의 경우 유용한 정보를 제공하지 않습니다. 따라서 대부분의 클래스에서는 toString을 재정의하여 적절한 정보를 반환해야 합니다.

### 2. toString 재정의의 중요성

toString 메서드를 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용하는 시스템을 디버깅하기 쉽습니다. toString 메서드는 객체를 print나 println, 혹은 assert 구문에 넘길 때, 또는 디버깅 시 자동으로 호출됩니다. 다른 엔티티에서 사용할 때 유용한 정보를 제공하여 개발자가 객체의 상태를 쉽게 파악할 수 있게 합니다.

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

위의 예제는 PhoneNumber 클래스를 toString으로 재정의하여, 객체를 문자열로 표현할 때 전화번호 형식으로 나타내도록 합니다.

### 3. 일관성과 명확한 포맷

toString을 구현할 때 반환값의 포맷을 문서화하고 명확하게 정해야 합니다. 이는 사용자에게 일관된 정보를 제공하고, 그 객체가 어떤 상태인지 명확히 보여줄 수 있게 합니다.

```java
j@Override
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

반환값의 포맷을 명확히 하지 않으면, 다음과 같이 코드만 보고는 어떤 정보를 담고 있는지 파악하기 어렵습니다:

```java
@Override
public String toString() {
    return "Area Code: " + areaCode + ", Prefix: " + prefix + ", Line Number: " + lineNum;
}
```

이와 같이 명확한 포맷을 사용하면, 객체를 사용하는 프로그램에서 일관된 출력을 기대할 수 있습니다.

### 4. 주요 정보 포함

toString이 그 객체가 가진 주요 정보 모두를 반환하는 게 좋습니다. 객체의 상태를 명확히 표현하여, 예를 들어 전화번호 객체라면 전화번호 전체를 반환하도록 합니다.

```java
@Override
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
}
```

위 예제는 PhoneNumber 객체의 모든 주요 정보를 포함하여 출력합니다.

### 결론

모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 말끔하게 재정의한 경우는 예외다. toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용하는 시스템을 디버깅하기 쉽게 해준다. toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.
