# Ensuring Consistency

### 1. 일관성이란 무엇인가?

일관성은 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻입니다. 가변 객체는 비교 시점에 따라 서로 다를 수도 혹은 같을 수도 있는 반면, 불변 객체는 한 번 다르면 끝까지 다릅니다.

클래스를 작성할 때는 불변 클래스로 만드는 게 나을지를 심사숙고해야 합니다. 불변 클래스로 만들기로 했다면 equals가 한 번 같다고 한 객체가 어느 시점에나 영원히 같다고 답하고, 다르다고 한 객체에는 영원히 다르다고 답하도록 만들어야 합니다.

클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안 된다. 이 제약을 어기면 일반성 조건을 만족시키기가 아주 어렵다.

예컨대 `java.net.URL`의 equals는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교합니다. 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없습니다.

다음은 잘못된 equals 메서드의 예입니다:

```java
@Override
public boolean equals(Object o) {
    if (o == null)
        return false;
}
```

이와 같이 null 검사는 필요하지 않습니다. equals의 대상이 null이라면 false를 반환하면 됩니다.

### 2. MyType 클래스 예제

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
    // 나머지 코드 생략
}
```

equals가 타입을 확인하지 않으면 잘못된 타입의 인스턴스를 주었을 때 ClassCastException을 던져 일반 규약을 위배하게 됩니다. 그런데 instanceOf는 (두 번째 피연산자가 구분하지) 첫 번째 피연산자가 null이면 false를 반환합니다.

### 3. 핵심 필드 비교

입력 객체와 자기 자신의 '핵심' 필드들이 모두 일치하는지 확인합니다. 모든 필드가 일치하면 true를, 하나라도 다르면 false를 반환합니다.

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof PhoneNumber))
        return false;
    PhoneNumber pn = (PhoneNumber) o;
    return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
}
```

이는 모든 필드가 일치하는지 비교하여 equals 메서드를 구현하는 예입니다.

### 4. 결론

1. \== 연산자를 사용해 입력이 자기 자신의 참조인지 확인합니다.
2. instanceOf 연산자로 입력이 올바른 타입인지 확인합니다.
3. 입력을 올바른 타입으로 형변환합니다.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사합니다.

### 핵심 정리

꼭 필요한 경우가 아니라면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다. 재정의해야 할 때는 그 클래스의 핵심 필드를 모두 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.
