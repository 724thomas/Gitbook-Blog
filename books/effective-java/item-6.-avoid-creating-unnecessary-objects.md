---
description: 불필요한 객체 생성을 피하라
---

# Item 6. Avoid Creating Unnecessary Objects

자바 개발에서 객체 생성을 최소화하는 것은 성능 최적화와 메모리 관리에 중요한 역할을 합니다. 조슈아 블로크(Joshua Bloch)의 "이펙티브 자바(Effective Java)" 3판에서는 이에 대한 구체적인 권장 사항을 제시하고 있습니다. 특히, 아이템 6에서는 불필요한 객체 생성을 피하는 방법에 대해 다루고 있습니다. 이번 글에서는 이 주제에 대해 자세히 살펴보겠습니다.

### 객체 생성의 비용

객체를 생성하는 것은 메모리 할당과 생성자 호출을 포함한 작업입니다. 이는 간단해 보이지만, 다수의 객체를 빈번히 생성하고 폐기하는 작업이 반복될 경우 시스템의 성능 저하를 초래할 수 있습니다. 이러한 비용을 최소화하기 위해 우리는 객체를 효율적으로 관리해야 합니다.

### 불필요한 객체 생성을 피하는 방법

불필요한 객체 생성을 피하기 위한 여러 가지 방법이 있습니다. 이 중 일부는 코드의 설계 단계에서, 다른 일부는 코드 구현 단계에서 적용할 수 있습니다. 이제 몇 가지 주요 방법을 살펴보겠습니다.

#### 1. 불변 객체 사용하기

불변 객체(Immutable Object)는 한 번 생성되면 그 상태가 변하지 않는 객체를 말합니다. 불변 객체를 사용하면 동일한 인스턴스를 재사용할 수 있어 불필요한 객체 생성을 줄일 수 있습니다. 자바에서는 `String` 클래스가 대표적인 불변 객체입니다.

```java
// 새로운 String 객체를 생성하지 않음
String str1 = "Hello";
String str2 = "Hello";

// str1과 str2는 같은 인스턴스를 참조
```

#### 2. 정적 팩토리 메서드 사용하기

정적 팩토리 메서드는 객체 생성의 제어권을 개발자에게 넘겨줍니다. 이를 통해 캐싱(caching)과 같은 최적화 작업을 쉽게 구현할 수 있습니다. 예를 들어, `Boolean` 클래스의 `valueOf` 메서드는 불필요한 객체 생성을 피하는 좋은 예입니다.

```java
// Boolean 객체를 재사용하여 불필요한 객체 생성을 피함
Boolean trueValue = Boolean.valueOf(true);
Boolean falseValue = Boolean.valueOf(false);
```

#### 3. 오토박싱 피하기

오토박싱(Autoboxing)은 기본형(primitive type) 데이터를 그에 대응하는 래퍼 클래스(wrapper class) 객체로 자동 변환하는 기능입니다. 이 과정에서 불필요한 객체가 생성될 수 있습니다. 따라서 성능이 중요한 코드에서는 이를 주의해야 합니다.

```java
// 불필요한 객체 생성
Integer sum = 0;
for (int i = 0; i < 100; i++) {
    sum += i;
}

// 기본형을 사용하여 불필요한 객체 생성을 피함
int sum = 0;
for (int i = 0; i < 100; i++) {
    sum += i;
}
```

#### 4. 생성 비용이 큰 객체의 재사용

생성 비용이 큰 객체는 한 번 생성한 후 재사용하는 것이 좋습니다. 예를 들어, 정규 표현식(Regular Expression) 객체는 생성 비용이 크기 때문에, 반복 사용이 예상되는 경우 재사용하는 것이 좋습니다.

```java
j// 비효율적인 방법: 매번 새로운 Pattern 객체를 생성
public boolean isValid(String input) {
    return input.matches("[a-z]+");
}

// 효율적인 방법: Pattern 객체를 재사용
private static final Pattern pattern = Pattern.compile("[a-z]+");

public boolean isValid(String input) {
    return pattern.matcher(input).matches();
}
```

#### 5. 방어적 복사 피하기

객체를 전달할 때 방어적 복사(defensive copy)를 사용하면 불필요한 객체가 생성될 수 있습니다. 대신, 객체를 불변으로 설계하거나, 방어적 복사가 꼭 필요한 경우 최소화하는 것이 좋습니다.

```java
// 불변 객체 사용
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // 객체를 불변으로 유지하여 방어적 복사 피함
}
```

### 예제: Lombok을 활용한 객체 생성 최적화

Lombok은 자바 개발에서 보일러플레이트 코드를 줄여주는 라이브러리로, 이를 활용하여 객체 생성 최적화를 더욱 쉽게 구현할 수 있습니다. 예를 들어, Lombok의 `@Builder` 어노테이션을 사용하여 불변 객체를 손쉽게 생성할 수 있습니다.

```java
import lombok.Builder;
import lombok.Value;

@Value
@Builder
public class Person {
    String name;
    int age;
}

// 사용 예시
Person person = Person.builder()
                      .name("John Doe")
                      .age(30)
                      .build();
```

이와 같이 Lombok을 사용하면 불변 객체를 쉽게 생성하고, 불필요한 객체 생성을 피할 수 있습니다.

### 결론

불필요한 객체 생성을 피하는 것은 자바 애플리케이션의 성능을 최적화하는 중요한 방법입니다. 불변 객체 사용, 정적 팩토리 메서드 활용, 오토박싱 피하기, 생성 비용이 큰 객체의 재사용, 방어적 복사 최소화 등의 방법을 통해 불필요한 객체 생성을 줄일 수 있습니다. 또한, Lombok과 같은 도구를 활용하여 이러한 최적화 작업을 더욱 쉽게 구현할 수 있습니다. 이러한 원칙들을 준수하면 메모리 사용을 줄이고, 애플리케이션의 성능을 향상시킬 수 있습니다.
