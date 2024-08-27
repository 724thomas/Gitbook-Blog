---
description: 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라
---

# Item 19. Design and Document for Inheritance, or Else Prohibit It

## **1. 상속의 본질과 그 중요성**

상속은 객체지향 프로그래밍의 가장 기본적인 특성 중 하나로, 하위 클래스가 상위 클래스의 속성과 메서드를 물려받아 재사용하거나 확장할 수 있게 합니다. 이를 통해 코드의 중복을 줄이고, 객체 간의 관계를 보다 명확하게 표현할 수 있습니다. 예를 들어, 동물(Animal)이라는 상위 클래스를 정의하고, 이를 상속받아 개(Dog), 고양이(Cat) 같은 하위 클래스를 만드는 것이 전형적인 예입니다. 하위 클래스는 상위 클래스의 공통적인 속성과 동작을 물려받고, 필요에 따라 고유한 특성을 추가할 수 있습니다.

하지만 상속은 단순히 코드를 재사용하는 것을 넘어서 설계의 복잡성을 크게 증가시킬 수 있는 도구이기도 합니다. 잘못된 상속 설계는 시스템의 유지보수성을 떨어뜨리고, 코드의 유연성을 제한하며, 예상치 못한 버그를 유발할 수 있습니다. 이러한 이유로 상속을 사용할 때는 신중한 설계와 철저한 문서화가 필수적입니다.



## **2. 상속을 고려한 설계의 원칙**

상속을 고려한 설계를 할 때는 몇 가지 중요한 원칙을 염두에 두어야 합니다. 이러한 원칙들은 상속의 위험을 최소화하고, 코드의 일관성을 유지하는 데 도움을 줍니다.

### **1. 생성자에서 재정의 가능한 메서드 호출을 피하라**

상속을 염두에 두고 설계할 때 가장 중요한 점 중 하나는 생성자에서 재정의할 수 있는 메서드를 호출하지 않는 것입니다. 생성자 내에서 호출되는 메서드는 하위 클래스에서 재정의될 수 있으므로, 하위 클래스가 완전히 초기화되기 전에 그 메서드가 호출될 수 있습니다. 이는 하위 클래스의 상태가 불완전한 상태에서 메서드가 실행되는 결과를 초래할 수 있으며, 심각한 버그로 이어질 수 있습니다.&#x20;

<details>

<summary>예시코드</summary>

```java
public class Super {
    public Super() {
        overrideMe();
    }

    public void overrideMe() {
        // 이 메서드는 하위 클래스에서 재정의될 수 있음
    }
}

public class Sub extends Super {
    private final Date date;

    public Sub() {
        date = new Date();
    }

    @Override
    public void overrideMe() {
        System.out.println(date);
    }
}
```

위의 코드에서 `Sub` 클래스의 생성자는 `Super` 클래스의 생성자가 호출된 후에 실행됩니다. 하지만 `Super` 클래스의 생성자에서 `overrideMe` 메서드를 호출하기 때문에, `date`가 초기화되지 않은 상태에서 `overrideMe`가 호출될 수 있습니다. 이는 `null` 포인터 예외를 유발할 수 있습니다.

</details>



### **2. 상속을 위한 문서화를 철저히 하라**

상속을 허용하려는 클래스라면, 상속받는 사람이 알아야 할 모든 중요한 사항을 철저히 문서화해야 합니다. 어떤 메서드가 재정의 가능하며, 재정의할 때 주의해야 할 사항은 무엇인지, 상속할 때 어떤 제약이 있는지를 명확하게 명시해야 합니다. 이는 API의 일관성을 유지하고, 사용자에게 올바른 사용법을 안내하는 데 매우 중요합니다.

예를 들어, 자바의 `AbstractList` 클래스는 `get`과 같은 필수적인 메서드를 하위 클래스가 반드시 재정의해야 한다고 문서화하고 있습니다. 이를 통해 `AbstractList`를 상속하는 개발자는 무엇을 해야 하는지 명확하게 이해할 수 있습니다.

### **3. 메서드 호출 순서와 상태를 명확히 하라**

상속을 허용하는 클래스는 메서드 호출 순서와 클래스의 상태를 명확히 정의해야 합니다. 특히, 상위 클래스와 하위 클래스 간의 상태 공유가 일어나는 경우, 상태 변경이 일관성 있게 이루어지도록 설계해야 합니다. 메서드 호출 간에 상태가 예측 가능하게 변경되지 않으면, 하위 클래스의 동작이 불안정해질 수 있습니다.

### **4. 템플릿 메서드 패턴의 활용**

템플릿 메서드 패턴은 상속을 고려한 설계에서 자주 사용되는 패턴 중 하나입니다. 상위 클래스는 전체적인 알고리즘의 구조를 제공하고, 일부 세부적인 구현은 하위 클래스가 결정할 수 있게 하는 패턴입니다. 이를 통해 상속을 보다 명확하고 안전하게 사용할 수 있습니다.

## **3. 상속을 금지해야 하는 상황**

모든 클래스가 상속을 고려해 설계될 필요는 없습니다. 실제로, 많은 경우 상속을 금지하는 것이 더 바람직할 수 있습니다. 상속을 금지해야 하는 일반적인 상황은 다음과 같습니다:

### **1. 클래스가 내부적으로 복잡하고, 자주 변경되는 경우**

만약 클래스의 내부 구현이 복잡하고 자주 변경된다면, 이 클래스를 상속하는 하위 클래스들이 의도치 않게 영향을 받을 수 있습니다. 예를 들어, 상위 클래스의 메서드를 변경했을 때, 이를 재정의한 하위 클래스의 동작이 예기치 않게 변화할 수 있습니다. 이러한 경우, 상속을 허용하는 것은 위험할 수 있으며, 상속을 금지하는 것이 좋습니다.

### **2. 특정 메서드의 호출 순서에 의존하는 경우**

메서드 호출 순서에 따라 클래스의 동작이 달라지는 경우, 상속을 허용하는 것이 복잡성을 증가시킬 수 있습니다. 특히, 상위 클래스에서 특정 순서로 메서드를 호출해야만 정상적으로 동작하는 경우, 하위 클래스에서 이를 재정의하면 문제가 발생할 수 있습니다. 이러한 경우 상속을 금지하는 것이 더 안전합니다.

### **3. 클래스의 불변성을 유지하고자 할 때**

불변 클래스는 상태가 변하지 않는 객체로, 스레드 안전성을 보장하는 데 유용합니다. 불변 클래스를 상속하게 되면 하위 클래스가 상태를 변경할 수 있는 방법을 추가할 가능성이 생기며, 이는 불변성을 깨뜨릴 수 있습니다. 따라서 불변 클래스를 설계할 때는 상속을 금지해야 합니다.

### **4. 외부에서의 접근을 제한해야 하는 경우**

클래스가 특정 기능을 제공하기 위해 설계되었고, 이 기능 외에 추가적인 역할을 부여하지 않도록 하기 위해 상속을 금지할 필요가 있을 수 있습니다. 예를 들어, 특정 유틸리티 클래스나 헬퍼 클래스는 상속을 통해 확장되기보다는, 제공된 기능을 그대로 사용하게 하는 것이 더 적절합니다.

## **4. 상속 금지 방법**

상속을 금지하는 가장 간단하고 명확한 방법은 클래스에 `final` 키워드를 사용하는 것입니다. `final` 클래스는 더 이상 상속될 수 없으며, 이를 통해 상속으로 인한 부작용을 방지할 수 있습니다. 또한, 개별 메서드에도 `final` 키워드를 붙여 해당 메서드가 하위 클래스에서 재정의되지 않도록 할 수 있습니다.

<details>

<summary><strong>예시</strong></summary>

```java
// 상속이 불가능한 클래스
public final class UtilityClass {
    public static void utilityMethod() {
        System.out.println("Utility method");
    }
}

// 상속이 불가능한 메서드
public class BaseClass {
    public final void cannotOverride() {
        System.out.println("This method cannot be overridden");
    }
}
```

이렇게 하면 클래스나 메서드를 재정의하려는 시도가 컴파일 단계에서 차단됩니다. 특히, 상속이 필요하지 않거나 오히려 상속이 시스템에 해를 끼칠 수 있는 경우 `final` 키워드를 사용해 상속을 금지하는 것이 좋은 설계 습관입니다.

</details>



## **5. 상속을 고려한 설계의 실제 예시 - Lombok과 Spring Boot 활용**

이제 상속을 고려한 설계를 Lombok과 Spring Boot를 활용하여 구체적인 예시로 살펴보겠습니다. Lombok은 자바 개발에서 반복적인 코드를 줄이고 코드의 가독성을 높이는 데 유용한 라이브러리입니다. Spring Boot는 복잡한 설정 없이 신속하게 애플리케이션을 개발할 수 있도록 도와주는 프레임워크입니다. 이 두 가지를 활용해 상속을 고려한 설계를 어떻게 할 수 있는지 살펴보겠습니다.

먼저, Lombok을 사용해 상속을 고려한 클래스를 설계하는 예를 들어보겠습니다.

<details>

<summary><strong>예시</strong></summary>

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class BaseEntity {
    private Long id;
    private String name;

    public void printDetails() {
        System.out.println("ID: " + id + ", Name: " + name);
    }

    // 상속할 수 있도록 의도된 메서드
    public void customOperation() {
        System.out.println("Base entity operation");
    }

    // 재정의할 수 없도록 의도된 메서드
    public final void finalOperation() {
        System.out.println("This operation cannot be overridden");
    }
}
```

위의 `BaseEntity` 클래스는 Lombok을 사용해 `getter`와 `setter`를 자동으로 생성합니다. 또한, `customOperation`은 하위 클래스에서 재정의할 수 있지만, `finalOperation`은 재정의할 수 없도록 설계되었습니다.

다음으로, 이 클래스를 상속받아 하위 클래스를 작성해보겠습니다:

```java
public class UserEntity extends BaseEntity {
    private String email;

    @Override
    public void customOperation() {
        System.out.println("User-specific operation");
    }

    // finalOperation 메서드는 재정의할 수 없음
    // @Override
    // public void finalOperation() {
    //     // 컴파일 오류 발생
    // }
}
```

`UserEntity` 클래스는 `BaseEntity`를 상속받아 `email`이라는 속성을 추가하고, `customOperation` 메서드를 재정의합니다. 하지만 `finalOperation` 메서드는 재정의할 수 없으며, 이를 통해 상위 클래스의 중요한 동작을 보호할 수 있습니다.

</details>



또한, Spring Boot를 사용하여 상속을 고려한 구조를 설계할 때도 유사한 원칙이 적용됩니다. Spring Boot에서는 서비스 클래스나 리포지토리 클래스에서 이러한 설계를 자주 활용합니다. 예를 들어, 기본적인 CRUD 기능을 제공하는 추상 클래스에서 상속을 허용하고, 각 엔티티별로 특화된 동작을 하위 클래스에서 구현할 수 있습니다.

<details>

<summary><strong>예시</strong></summary>

```java
public abstract class AbstractService<T, ID> {

    @Autowired
    protected JpaRepository<T, ID> repository;

    public T findById(ID id) {
        return repository.findById(id).orElseThrow(() -> new EntityNotFoundException("Entity not found"));
    }

    public T save(T entity) {
        return repository.save(entity);
    }

    // 하위 클래스에서 특화된 저장 동작을 구현할 수 있음
    public abstract T customSaveOperation(T entity);
}
```

이 예제에서는 `AbstractService` 클래스가 기본적인 CRUD 기능을 제공하며, `customSaveOperation`이라는 추상 메서드를 통해 하위 클래스에서 특화된 동작을 구현할 수 있도록 하고 있습니다.

</details>

<details>

<summary><strong>하위 클래스 예시</strong></summary>

```java
@Service
public class UserService extends AbstractService<User, Long> {

    @Override
    public User customSaveOperation(User user) {
        // 사용자 특화 저장 로직 구현
        return repository.save(user);
    }
}
```

`UserService`는 `AbstractService`를 상속받아 `customSaveOperation`을 재정의합니다. 이를 통해 `User` 엔티티에 특화된 로직을 구현하면서도, 상위 클래스의 공통 동작을 활용할 수 있습니다.

</details>



## **6. 상속을 염두에 둔 설계 패턴 - 템플릿 메서드 패턴**

템플릿 메서드 패턴은 상속을 활용한 대표적인 디자인 패턴 중 하나입니다. 이 패턴은 상위 클래스에서 알고리즘의 골격을 정의하고, 하위 클래스에서 구체적인 구현을 제공하는 방식으로 동작합니다. 이를 통해 공통 로직은 상위 클래스에 위치시키고, 각기 다른 세부 구현은 하위 클래스에서 처리할 수 있습니다.

<details>

<summary><strong>예시</strong></summary>

```java
public abstract class DataProcessor {

    // 템플릿 메서드: 전체 처리 흐름을 정의
    public final void process() {
        loadData();
        processData();
        saveData();
    }

    protected abstract void loadData();
    protected abstract void processData();
    protected abstract void saveData();
}
```

이 클래스에서 `process` 메서드는 데이터 처리의 전체 흐름을 정의하지만, 구체적인 데이터 로딩, 처리, 저장 로직은 하위 클래스에서 구현하게 됩니다.

</details>

<details>

<summary><strong>하위 클래스 예시</strong></summary>

```java
public class CsvDataProcessor extends DataProcessor {

    @Override
    protected void loadData() {
        System.out.println("Loading CSV data");
    }

    @Override
    protected void processData() {
        System.out.println("Processing CSV data");
    }

    @Override
    protected void saveData() {
        System.out.println("Saving processed CSV data");
    }
}
```

이렇게 하위 클래스에서 템플릿 메서드 패턴을 사용하면, 코드의 재사용성과 확장성을 높일 수 있으며, 상위 클래스의 코드 변경 없이 새로운 동작을 손쉽게 추가할 수 있습니다.

</details>



## **7. 상속을 금지해야 할 상황에 대한 자세한 분석**

상속을 금지해야 하는 상황은 앞서 언급한 이유 외에도 다양한 이유가 존재합니다. 이러한 이유들은 특정한 상황에서 상속이 오히려 문제를 일으킬 수 있다는 점을 보여줍니다.

### **1. 다양한 하위 클래스의 동작을 예측할 수 없는 경우**

만약 상위 클래스가 다양한 형태로 상속될 수 있으며, 이러한 하위 클래스의 동작을 상위 클래스에서 완전히 예측하기 어렵다면 상속을 금지하는 것이 좋습니다. 이는 상위 클래스의 유지보수성을 높이고, 코드의 예측 가능성을 확보하는 데 도움이 됩니다.

### **2. 보안상의 이유**

상속을 허용할 경우, 하위 클래스에서 상위 클래스의 중요한 메서드나 데이터를 재정의하거나 접근할 수 있는 가능성이 생깁니다. 이는 시스템의 보안을 위협할 수 있는 요소가 될 수 있으므로, 보안이 중요한 클래스는 상속을 금지하는 것이 바람직합니다.

### **3. 특정 라이브러리나 프레임워크에 강하게 의존하는 경우**

만약 클래스가 특정 라이브러리나 프레임워크에 강하게 의존하고 있으며, 이러한 의존성이 하위 클래스에까지 영향을 미칠 수 있다면 상속을 허용하지 않는 것이 좋습니다. 이는 의존성으로 인해 발생할 수 있는 문제를 줄이고, 코드의 독립성을 유지하는 데 도움이 됩니다.

### **4. 클래스가 불변이거나 변경이 빈번하지 않아야 하는 경우**

예를 들어, 자바의 `String` 클래스는 불변 클래스로 설계되어 있으며, 이 클래스는 상속을 허용하지 않습니다. 이는 `String` 클래스의 무결성을 유지하고, 그 동작을 변경할 수 없도록 보장합니다.

### **5. 정 기능만 제공해야 하는 유틸리티 클래스**

유틸리티 클래스는 특정 기능을 제공하는 데 목적이 있는 경우가 많습니다. 이러한 클래스는 상속을 통해 기능을 확장하기보다는, 제공된 기능을 그대로 사용하게 하는 것이 더 적절할 수 있습니다. 예를 들어, `java.lang.Math` 클래스는 수학 관련 유틸리티 메서드를 제공하며, 상속을 허용하지 않습니다.

## **8. 결론**

이펙티브 자바 3rd Edition의 아이템 19에서 상속을 고려한 설계와 문서화의 중요성은 매우 강조됩니다. 상속은 객체지향 프로그래밍에서 강력한 도구이지만, 잘못된 설계로 인해 시스템의 복잡성을 증가시키고 예기치 않은 버그를 유발할 수 있습니다. 따라서 상속을 허용하는 클래스는 신중하게 설계되고 철저히 문서화되어야 하며, 상속을 금지해야 할 경우에는 `final` 키워드를 통해 이를 명시적으로 차단하는 것이 중요합니다.

또한, Lombok과 Spring Boot를 활용한 실용적인 예시를 통해 상속을 고려한 설계를 구체적으로 살펴보았습니다. 이러한 예시는 실제 개발 환경에서 상속을 적절히 활용하는 데 유용한 지침이 될 것입니다. 상속을 적절히 사용하면 코드의 재사용성과 확장성을 높일 수 있지만, 상황에 따라서는 상속을 금지하는 것이 시스템의 안정성과 유지보수성을 높이는 데 더 유리할 수 있다는 점을 항상 염두에 두어야 합니다.
