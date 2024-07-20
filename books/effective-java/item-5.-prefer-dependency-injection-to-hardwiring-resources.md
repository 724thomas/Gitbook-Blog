---
description: 자원을 직접 명시하지 말고 의존 객체 주입을 사용해라
---

# Item 5. Prefer Dependency Injection to Hardwiring Resources

자바 애플리케이션에서 객체 간의 의존성을 관리하는 것은 매우 중요한 일이다. 의존성 관리를 잘못하면 코드의 유연성이 떨어지고, 테스트와 유지보수가 어려워진다. **이펙티브 자바**의 아이템 5에서는 이러한 문제를 해결하기 위해 자원을 직접 명시하지 말고 의존 객체 주입(Dependency Injection, DI)을 사용할 것을 권장하고 있다.

### 의존 객체 주입의 개념

의존 객체 주입은 객체가 사용할 의존 객체를 직접 생성하지 않고 외부에서 주입받는 방식을 말한다. 이를 통해 객체 간의 결합도를 낮추고, 코드의 유연성과 재사용성을 높일 수 있다. 의존 객체 주입은 주로 생성자 주입, 세터 주입, 필드 주입의 세 가지 방식으로 구현된다.

#### 생성자 주입

생성자 주입은 객체 생성 시 생성자를 통해 의존 객체를 주입받는 방식이다. 이 방식은 객체의 불변성을 보장하고, 의존성을 명확히 할 수 있다는 장점이 있다.

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        // 사전에서 단어를 검사하는 로직
        return dictionary.contains(word);
    }
}
```

위 예제에서 `SpellChecker` 클래스는 `Lexicon` 객체를 의존성으로 가지며, 생성자를 통해 주입받는다. 이렇게 하면 `SpellChecker` 객체는 항상 유효한 `Lexicon` 객체를 가지고 있게 되어 코드의 안정성이 높아진다.

#### 세터 주입

세터 주입은 객체 생성 후 세터 메서드를 통해 의존 객체를 주입받는 방식이다. 이 방식은 의존성을 동적으로 변경할 수 있다는 장점이 있지만, 객체의 일관성이 깨질 수 있다는 단점이 있다.

```java
public class SpellChecker {
    private Lexicon dictionary;

    public void setDictionary(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        // 사전에서 단어를 검사하는 로직
        return dictionary.contains(word);
    }
}
```

#### 필드 주입

필드 주입은 필드에 직접 의존 객체를 주입하는 방식이다. 주로 프레임워크에 의해 자동으로 주입될 때 사용된다. 예를 들어, 스프링에서는 `@Autowired` 어노테이션을 사용하여 필드 주입을 구현할 수 있다.

```java
public class SpellChecker {
    @Autowired
    private Lexicon dictionary;

    public boolean isValid(String word) {
        // 사전에서 단어를 검사하는 로직
        return dictionary.contains(word);
    }
}
```

### 의존 객체 주입의 장점

의존 객체 주입을 사용하면 다음과 같은 장점이 있다.

#### 유연성과 재사용성

의존 객체 주입을 사용하면 객체 간의 결합도가 낮아져 유연성과 재사용성이 높아진다. 예를 들어, `SpellChecker` 클래스는 다양한 `Lexicon` 구현체를 사용할 수 있게 되어, 필요에 따라 쉽게 변경할 수 있다.

#### 테스트 용이성

의존 객체 주입을 사용하면 모의 객체(Mock Object)를 사용하여 단위 테스트를 쉽게 작성할 수 있다. 이는 테스트의 독립성을 보장하고, 테스트 코드의 유지보수성을 높이는 데 기여한다.

```java
public class SpellCheckerTest {
    private SpellChecker spellChecker;
    private Lexicon mockLexicon;

    @Before
    public void setUp() {
        mockLexicon = mock(Lexicon.class);
        when(mockLexicon.contains(anyString())).thenReturn(true);
        spellChecker = new SpellChecker(mockLexicon);
    }

    @Test
    public void testIsValid() {
        assertTrue(spellChecker.isValid("test"));
    }
}
```

위 예제에서 `SpellCheckerTest` 클래스는 `mockLexicon` 모의 객체를 사용하여 `SpellChecker`를 테스트한다. 이를 통해 `SpellChecker`의 동작을 독립적으로 검증할 수 있다.

#### 코드의 가독성과 유지보수성

의존 객체 주입을 사용하면 코드의 가독성과 유지보수성이 높아진다. 객체 간의 의존성을 명확히 정의하고, 변경이 필요한 부분만 수정하면 되기 때문에 코드 관리가 용이하다.

### 실세계 예제: 스프링부트와 의존 객체 주입

스프링부트는 의존 객체 주입을 간편하게 구현할 수 있는 프레임워크로, 자바 애플리케이션 개발에서 널리 사용되고 있다. 스프링부트에서 의존 객체 주입을 사용하는 예제를 통해 이를 어떻게 적용할 수 있는지 살펴보자.

#### 서비스와 리포지토리 클래스

먼저, `UserService`와 `UserRepository` 클래스를 정의하자.

```java
import org.springframework.stereotype.Service;

@Service
public class UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }
}
```

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

위 예제에서 `UserService` 클래스는 `UserRepository`를 의존성으로 가지며, 생성자 주입을 통해 주입받는다. 이는 `UserService` 클래스가 `UserRepository`와의 결합도를 낮추고, 쉽게 테스트할 수 있게 한다.

#### 컨트롤러 클래스

다음으로, `UserController` 클래스를 정의하자.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {
    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findUserById(id);
    }
}
```

위 예제에서 `UserController` 클래스는 `UserService`를 의존성으로 가지며, 생성자 주입을 통해 주입받는다. 이를 통해 `UserController`는 `UserService`와의 결합도를 낮추고, 쉽게 테스트할 수 있게 한다.

#### 테스트 클래스

마지막으로, `UserService` 클래스의 단위 테스트를 작성하자.

```java
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

public class UserServiceTest {
    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testFindUserById() {
        User user = new User();
        user.setId(1L);
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));

        User result = userService.findUserById(1L);

        assertNotNull(result);
        assertEquals(1L, result.getId());
    }
}
```

위 예제에서 `UserServiceTest` 클래스는 `Mockito`를 사용하여 `UserRepository`의 모의 객체를 생성하고, `UserService`의 동작을 테스트한다. 이를 통해 `UserService`의 동작을 독립적으로 검증할 수 있다.

### 결론

의존 객체 주입은 객체 간의 결합도를 낮추고, 코드의 유연성과 재사용성을 높이는 데 중요한 설계 원칙이다. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하면, 코드의 가독성과 유지보수성이 높아지며, 테스트 용이성도 향상된다. 스프링부트와 같은 프레임워크를 사용하면 의존 객체 주입을 간편하게 구현할 수 있으며, 이를 통해 실제 프로젝트에서 유용하게 활용할 수 있다.
