# DIP. Dependency Inversion Principal

## 1. DIP란

DIP는 SOLID 원칙 중 역전관계 제어 원칙을 의미합니다. 이는 고수준 모듈은 저수준 모듈에 의존해서는 안된다는 의미입니다. 다시 말해서, 구현체에 의존하지 않고, 추상화를 참조하라는 원칙입니다.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 2. DIP를 지키지 않은 코드

직접 구현체를 가져다 쓰게 된다면?

핵심 비즈니스 로직(클라이언트)에서 어떤 구현체를 쓸지 알게 된다면(의존 상태), 새로운 구현체를 교체할때 클라이언트도 같이 수정을 해야하기 떄문에 유지보수, 확장성 모두 저하됩니다.

예시 코드:

```java
// 구현체에 직접 의존하는 OrderService(클라이언트)
public class OrderService {

    private final TestNotificationService notificationService;

    public OrderService() {
        this.notificationService = new TestNotificationService(); // 구현체에 의존
    }

    public void processOrder(String message) {
        notificationService.send(message);
    }
}

public class TestNotificationService {
    public void send(String message) {
        System.out.println("메시지 전송: " + message);
    }
}
```

위 코드는 알림을 보내는 TestNotificationService를 임시적으로 작성한 상태입니다. 추후 이메일/카톡/슬랙 등의 구현체로 변경될 수 있습니다. 이는 OCP를 위베하며, 유지보수 및 확장이 어렵습니다.

만약 이메일로 알림을 보내기로 결정을 했을때에는 클라이언트에서\
`this.notificationService = new EmailNotificationService();` 로 수정해야합니다.



## 3. DIP를 지키는 코드

직접 구현체를 가져다 쓰지 않고 인터페이스(추상화)를 사용하게 된다면?

```java
// 공통 인터페이스 도입
public interface NotificationService {
    void send(String message);
}
```

```java
// 구현체
public class TestNotificationService {
    public void send(String message) {
        System.out.println("메시지 전송: " + message);
    }
}

public class EmailNotificationService {
    public void send(String message) {
        // 이메일 전송 로직...
    }
}
```

```java
// 클라이언트 리팩토링
public class OrderService {
    private final NotificationService notificationService;

    // 생성자 주입을 통해 어떤 구현체가 들어올지 외부에서 결정
    public OrderService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    public void processOrder(String message) {
        notificationService.send(message);
    }
}
```

```java
public class AppMain {
    public static void main(String[] args) {
        NotificationService service = new EmailNotificationService(); // 생성자 주입
        OrderService orderService = new OrderService(service);
        orderService.processOrder("주문이 완료되었습니다!");
    }
}
```

현재 구조는:

* `OrderService` 클래스는 어떤 구현체가 들어올지 전혀 모르게 됩니다. -> DIP 만족
* 어떤 구현체로 바꾸든 `OrderService` 수정이 필요가 없습니다. -> OCP 만족
* 테스트 시, `FakeNotificationService`를 주입하면 단위 테스트가 가능합니다.

즉, 새로운 구현체로 교체하려고 할때는,&#x20;

* `EmailNotificationService` 구현체를 작성하고,
* `AppMain`에서 생성자 주입으로 넣게됩니다.



## 4. 정적으로 구현체를 교체 (스프링X)

AppConfig 이라는 클래스를 하나 만들어서 의존성 주입의 역할을 하게 할 수 있습니다.

```java
// 스프링 도입 없이 구현체를 정적으로 바꾸는 구조
public class AppConfig {
    public OrderService orderService() {
        return new OrderService(notificationService());
    }

    public NotificationService notificationService() {
        return new EmailNotificationService(); // 여기만 바꾸면 끝!
    }
}
```

```java
public class AppMain {
    public static void main(String[] args) {
        AppConfig appConfig = new AppConfig();
        OrderService orderService = appConfig.orderService();
        orderService.processOrder("이메일로 주문 알림 전송!");
    }
}
```

위 방식은 싱글톤이 아니라는 점을 제외하고, 실제로 스프링 프레임워크가 내부적으로 의존성을 관리하는 방식과 유사합니다.



## 5. 정적으로 구현체를 교체 (스프링O)

스프링을 도입했을때 더욱 우아하게 DIP를 실현할 수 있습니다.

```
디렉터리 구조
/src/main/java/com/example/notification
├── domain
│   └── NotificationService.java               // 추상화 (인터페이스)
├── infrastructure
│   ├── TestNotificationService.java         // 구현체 1
│   └── EmailNotificationService.java         // 구현체 2
├── config
│   └── NotificationConfig.java               // 구현체 선택을 위한 config
└── application
    └── OrderService.java                     // 클라이언트 코드
```

```java
// 추상화 (인터페이스)
public interface NotificationService {
    void send(String message);
}
```

```java
// 구현체1, 2
@Component("test")  // 이 이름이 Bean 이름으로 사용됨
public class TestNotificationService {
    public void send(String message) {
        System.out.println("메시지 전송: " + message);
    }
}

@Component("email") // 이 이름이 Bean 이름으로 사용됨
public class EmailNotificationService implements NotificationService {
    public void send(String message) {
        //이메일 전송 로직...
    }
}
```

<pre class="language-java"><code class="lang-java"><strong>// 구현체를 위한 config
</strong><strong>@Configuration
</strong>public class NotificationConfig {

    @Bean
    public NotificationService notificationService() {
        return new EmailNotificationService(); // 구현체 교체는 여기서만 함
    }
}
</code></pre>

<pre class="language-java"><code class="lang-java">// 클라이언트 코드
<strong>@RequiredArgsConstructor
</strong>@Service
public class OrderService {
    private final NotificationService notificationService;

    public void processOrder(String message) {
        notificationService.send(message);
    }
}
</code></pre>

* 각 클래스가 한 가지 책임을 가짐 -> SRP 만족
* 구현체 변경 시 클라이언트 변경 없음 -> OCP 만족
* 어떤 구현체도 자유롭게 치환 가능 -> LSP 만족
* 필요한 기능만 담은 인터페이스 사용 -> ISP 만족
* 클라이언트가 추상화에만 의존 -> DIP 만족



## 6. 정리

* 정적으로 구현체를 갈아끼워야 할 때, DIP를 지키는 구조는 유지보수와 확장성의 핵심입니다.
* 스프링을 사용하면 생성, 주입, 관리 모두 프레임워크에 위임하여 훨씬 편리하게 DIP를 실현할 수 있습니다.
* AppConfig를 활용한 수동 설정, `@Profile`, `@ConditionalOnProperty`, `@Primary`, `@Bean` 방식 등 다양한 전략이 존재하며, 상황에 맞게 선택하면 됩니다.
