---
description: 클래스와 멤버의 접근 권한을 최소화하라
---

# Item15. Minimize the Accessibility of Classes and Members

## 1. 정보 은닉의 중요성

정보 은닉은 객체 지향 프로그래밍에서 모듈 간 결합도를 낮추고, 응집도를 높이는 중요한 원칙이다. 클래스 내부의 구현 세부 사항을 외부로부터 숨기고, 필요한 부분만 외부에 노출함으로써 클래스의 재사용성과 유지보수성을 향상시킬 수 있다. 이는 코드 변경 시 영향을 받는 부분을 최소화하여 오류를 줄이고, 코드의 복잡성을 낮출 수 있다.

### 1.1 모듈화와 유연성의 향상

정보 은닉을 통해 각 모듈(클래스)은 자신의 내부 구현에 대한 책임만을 가지며, 외부에 제공하는 인터페이스만을 통해 상호작용하게 된다. 이렇게 하면 모듈 간의 결합도가 낮아져 유연한 설계가 가능해지고, 개별 모듈을 독립적으로 개발하거나 수정할 수 있다.

### 1.2 시스템의 안전성

접근 권한을 적절히 설정함으로써 클래스의 내부 상태가 외부에서 임의로 변경되는 것을 방지할 수 있다. 이는 시스템의 안전성을 높이는 중요한 방법 중 하나이다.

## 2. 자바의 접근 제어자

자바에서는 클래스, 메서드, 필드 등의 접근 범위를 제어하기 위해 네 가지 접근 제어자를 제공한다.

## 2.1 private

`private` 접근 제어자는 가장 강력한 접근 제한을 부여한다. `private`로 선언된 멤버는 해당 클래스 내에서만 접근할 수 있으며, 외부에서는 접근이 불가능하다. 이를 통해 클래스 내부 구현을 완전히 숨길 수 있다.

```java
public class Example {
    private int value;

    private void calculate() {
        // 내부에서만 사용 가능
    }
}
```

### 2.2 package-private (default)

접근 제어자를 명시하지 않으면 해당 멤버는 '패키지 전용'으로 설정된다. 같은 패키지 내의 다른 클래스에서는 접근할 수 있지만, 다른 패키지에서는 접근이 불가능하다.

```java
class PackageExample {
    int value; // package-private
}
```

### 2.3 protected

`protected` 접근 제어자는 `package-private` 범위에 더해, 해당 클래스를 상속받은 하위 클래스에서도 접근할 수 있도록 한다. 상속 관계에서 부모 클래스의 중요한 기능을 하위 클래스에서 사용할 수 있게 해준다.

```java
public class Parent {
    protected int value;

    protected void display() {
        // 자식 클래스에서 접근 가능
    }
}
```

### 2.4 public

`public` 접근 제어자는 가장 넓은 접근 범위를 허용한다. 어디서든 접근이 가능하므로, 반드시 외부에 공개해야 할 멤버에만 사용해야 한다.

```java
public class PublicExample {
    public int value;

    public void printValue() {
        System.out.println(value);
    }
}
```

## 3. 접근 권한을 최소화하는 방법

접근 제어자를 사용할 때는 다음과 같은 원칙을 준수해야 한다.

### 3.1 클래스와 멤버를 가능한 한 숨기기

모든 클래스와 멤버는 기본적으로 `private`으로 선언하고, 필요할 때만 접근 권한을 확장하는 것이 좋다. 이렇게 하면 클래스의 내부 구현을 숨길 수 있고, 외부 코드와의 불필요한 결합을 방지할 수 있다.

### 3.2 public 클래스의 멤버는 반드시 private으로

`public` 클래스를 작성할 때는 멤버 변수와 메서드를 기본적으로 `private`으로 선언해야 한다. 이를 통해 외부에서 접근할 수 있는 경로를 최소화하고, 클래스 내부 구현의 변경이 외부에 영향을 미치지 않도록 할 수 있다.

```java
public class Account {
    private String owner;
    private double balance;

    public double getBalance() {
        return balance;
    }
}
```

### 3.3 패키지 전용(private)을 적절히 활용

패키지 전용 접근 제어자(`package-private`)는 같은 패키지 내에서만 접근이 필요할 때 유용하다. 단, 패키지가 커지면 모듈 간 결합이 증가할 수 있으므로, 가능하면 `private`을 우선적으로 사용하고 필요할 때만 범위를 확장하는 것이 좋다.

### 3.4 불변(immutable) 객체 활용

접근 권한을 최소화하는 또 다른 방법은 불변 객체를 사용하는 것이다. 불변 객체는 상태를 변경할 수 없으므로, 외부에서 객체를 수정하려는 시도를 원천적으로 차단할 수 있다. 이를 통해 객체의 안정성과 예측 가능성을 높일 수 있다.

```java
public final class ImmutablePoint {
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }
}
```

### 3.5 접근 권한 확장의 신중함

특정 상황에서는 클래스나 멤버의 접근 권한을 `protected` 또는 `public`으로 확장해야 할 수 있다. 그러나 이때는 반드시 그 이유를 명확히 하고, 향후 유지보수에 미칠 영향을 고려해야 한다. 불필요한 접근 권한 확장은 클래스의 응집도를 떨어뜨리고, 오류 가능성을 높일 수 있다.

## 4. Lombok과 자바 스프링부트 프로젝트에서의 활용

이제 앞서 설명한 원칙들을 실제 자바 스프링부트 프로젝트에 적용하는 방법을 살펴보겠다. Lombok을 활용하여 코드를 간결하게 유지하면서도, 접근 권한을 최소화하는 설계를 구현해보자.

### 4.1 Lombok을 이용한 기본 클래스 작성

Lombok을 사용하면 반복적인 코드 작성을 줄일 수 있다. 하지만 Lombok을 사용할 때도 접근 권한에 대한 원칙을 준수해야 한다.

```java
import lombok.Getter;
import lombok.Setter;

public class User {
    @Getter
    private final String username;

    @Getter @Setter
    private String email;

    public User(String username) {
        this.username = username;
    }
}
```

위 코드에서 `username` 필드는 `final`로 선언되어 생성 시점 이후에는 변경할 수 없으며, `email` 필드는 외부에서 변경 가능하지만, 접근 권한을 `private`으로 설정하여 직접 수정할 수 없도록 하였다. Lombok의 `@Getter`와 `@Setter` 애너테이션을 사용하였으나, 클래스 외부에서 직접 접근할 수 없도록 `private` 필드로 제한하였다.

### 4.2 스프링부트 서비스 클래스에서의 접근 권한 설정

서비스 클래스는 비즈니스 로직을 처리하는 핵심 역할을 하므로, 접근 권한을 철저히 관리해야 한다. 서비스 클래스 내에서의 멤버 변수나 메서드는 기본적으로 `private`으로 설정하고, 외부에서 호출해야 하는 메서드만 `public`으로 선언하는 것이 좋다.

```java
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findUserByUsername(String username) {
        return userRepository.findByUsername(username)
                .orElseThrow(() -> new UserNotFoundException("User not found"));
    }

    private void validateUser(User user) {
        // 내부에서만 사용
    }
}
```

위 코드에서 `validateUser` 메서드는 외부에 노출될 필요가 없으므로 `private`으로 선언하였다. 반면 `findUserByUsername` 메서드는 외부 요청에 의해 호출될 가능성이 있으므로 `public`으로 설정하였다.

### 4.3 리포지토리 클래스에서의 접근 권한 설정

리포지토리 클래스는 데이터베이스와의 상호작용을 담당한다. 이 클래스 역시 필요한 메서드만 외부에 공개하고, 내부 구현 세부 사항은 숨겨야 한다.

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByUsername(String username);

    private void logQuery(String query) {
        // 내부에서만 사용
    }
}
```

`JpaRepository`를 상속받은 `UserRepository`는 기본적으로 외부에서 사용될 가능성이 있는 메서드만 제공하며, 내부적인 로깅 기능과 같은 세부 사항은 `private`으로 제한하여 외부에 노출되지 않도록 한다.

## 5. 결론

아이템 15에서는 클래스와 멤버의 접근 권한을 최소화하는 것이 얼마나 중요한지 설명한다. 접근 권한을 신중하게 설정함으로써 모듈 간의 결합도를 낮추고, 코드의 유지보수성을 높일 수 있다. 이러한 원칙은 자바 프로그래밍에서 특히 중요하며, 실제 프로젝트에서 이를 실천하기 위해 Lombok과 스프링부트를 적절히 활용하는 방법을 살펴보았다.

클래스와 멤버의 접근 권한을 최소화하는 것은 단순히 코드의 구조를 개선하는 것이 아니라, 장기적으로 시스템의 안정성과 유연성을 확보하는 중요한 전략임을 명심해야 한다. 이 글을 통해 자바 개발자들이 정보 은닉과 접근 권한 관리의 중요성을 더욱 깊이 이해하고, 실무에 적용할 수 있기를 바란다.
