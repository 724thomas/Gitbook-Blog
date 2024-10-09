---
description: '@Transactional'
---

# @Transactional

## 1. @Transactional 이란?

Transactional은 스프링에서 제공하는 어노테이션입니다. 메서드나 클래스에 적용하여 해당 범위 내에서 트랜잭션을 관리합니다. 이를 통해 일관성을 유지하고, 예외 발생시 롤백할 수 있습니다.&#x20;

클래스에 @Transactional을 적용하게 되면, 해당 클래스의 모든 public 메서드에 트랜잭션이 적용됩니다. 각 메서드 호출 시마다 트랜잭션이 시작되고, 메서드 실행 후 커밋 또는 롤백이 이루어집니다.&#x20;

### 1.1. @Transactional의 기본 속성

* propagation: 전파 방식. 기본: REQUIRED
* isolation: 격리 수준. 기본: Repeatable Read(MySQL)
* timeout: 트랜잭션 타임아웃. 기본: 제한없음
* readonly: 읽기 전용 여부. 기본: false
* rollbackFor/noRollbackFor: 롤백/커밋 대상 예외 지정





## 2. 작동 원리

트랜잭션은 AOP를 기반으로 동작합니다. 스프링은 프록시 객체를 생성하여 실제 메서드 호출 전에 트랜잭션을 시작하고, 실행 후에는 트랜잭션을 커밋하거나 롤백합니다. 개발자 대신 트랜잭션을 관리하게 됩니다.

AOP가 프록시 객체를 생성하여 동작하기 때문에, private이나 protected 메서드 처럼 외부에서 접근할 수 없는 접근 제어자를 사용하게되면 @Transactional이 적용되지 않습니다.

프록시 기반 AOP는 동일 클래스 내에서 자기 자신의 메서드를 호출할 경우 프록시가 적용되지 않는다는 제한이 있습니다. 따라서 같은 클래스 내의 메서드 호출 시에는 @Transactional이 제대로 적용되지 않을 수 있습니다. (이 경우, 구조를 재설계 / 트랜잭션 관리를 명시적으로 처리)





## 3. 트랜잭션 전파(Propagation.\~\~)

트랜잭션 전파는 현재 실행중인 트랜잭션이 있을 때, 새로운 메서드가 호출되었을 때의 트랜잭션 동작 방식을 정의하는 것입니다. (여러 트랜잭션이 어떻게 상호작용할지 정의하는 것입니다.)

```java
@Transactional
public void parent(User user) {
    userRepository.save(user);
    child(user.getStats);
}

@Transactional(Propagation.MANDATORY)
public void child(UserStats userStats) {
    userStatsRepository.save(userStats);
}
```

위 코드 처럼, parent내의 child가 실행될시 어떻게 처리를 해야할지 정의를 하는 것입니다.\
위 예시는 parent 트랜잭션이 항상 존재해야한다는 것입니다.

* Propagation.REQUIRED\
  부모 트랜잭션이 존재하면 자식 트랜잭션은 부모에게 포함된다. (자식 트랜잭션이 실패하면 부모도 롤백)\
  부모 트랜잭션이 존재하지 않으면, 새로운 트랜잭션을 생성한다. (자식이 부모가 된다.)
* Propagation.REQUIRES\_NEW\
  부모 트랜잭션의 존재 유무와 관계 없이, 새로운 트랜잭션을 생성한다.\
  자식은 쓰기 작업 가능. 부모가 읽기 전용이고 자식이 쓰기 전용일때 사용.
* Propagation.NESTED\
  부모 트랜잭션이 존재할 경우, 부모 트랜잭션안에 새로운 트랜잭션을 생성한다.\
  자식 트랜잭션은 부모 트랜잭션의 영향(커밋, 롤백)을 받지만, 부모는 자식의 영향을 받지 않는다.
* Propagation.MANDATORY\
  부모에 무조건 포함시키고, 부모가 없다면 예외 발생(IllegalTransactionStateException)
* Propagation.SUPPORTS\
  부모가 존재하면 부모에 포함\
  부모가 존재하지 않는다면 자식은 @Transaction 적용 없이 동작(위험하지만 성능 이점)
* Propagation.NOT\_SUPPORTED\
  부모가 존재하면 부모 트랜잭션을 일시 중지하고, 트랜잭션 없이 동작 (부모 트랜잭션이 롤백되더라도 자식은 영향을 받지 않습니다.)\
  부모가 존재하지 않는다면 트랜잭션 없이 동작
* Propagation.NEVER\
  부모 트랜잭션이 존재하면 안됩니다.\
  트랜잭션이 아예 적용이 안됩니다.

<div data-full-width="true">

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

</div>



## 4. 트랜잭션 격리 수준(isolation = Isolation.\~\~)

트랜잭션 격리 수준은 동시에 실행되는 트랜잭션 간의 데이터 일관성을 유지하기 위한 설정입니다.

* READ\_UNCOMMITTED: Dirty Read, Non-Repeatable Read, Phantom Read
* READ\_COMMITTED: Non-Repeatable Read, Phantom Read
* REPEATABLE\_READ: Phantom Read
* SERIALIZABLE

트랜잭션 격리 수준별 현상 정의

* **Dirty Read**: 다른 트랜잭션에서 아직 커밋하지 않은 변경 내용을 읽는 것
* **Non-Repeatable Read**: 동일한 쿼리를 두 번 실행할 때 결과가 다른 것 (다른 트랜잭션이 데이터를 수정하고 커밋한 경우)
* **Phantom Read**: 동일한 조건의 조회에서 레코드 수가 달라지는 것 (다른 트랜잭션이 데이터를 삽입하거나 삭제하고 커밋한 경우)





## 5. 예외 처리와 롤백(rollbackFor / noRollbackFor = \~\~)

기본적으로 Unchecked 예외가 발생하면 롤백되고, Checked 예외는 롤백되지 않습니다. 필요시 지정할 수 있습니다.

만약 체크 예외 발생시 트랜잭션을 롤백하기 위해서는, \
@Transactional(rollbackFro = SQLException.class) 같이 설정.\
SQLException 발생시 롤백됩니다.

반대로, 특정 Unchecked 예외시 롤백하지 않게 하려면, \
@Transactional(noRollbackFor = IllegalArgumentException.class).\
IllegalArgumentException이 발생해도 트랜잭션이 롤백되지 않고 커밋됩니다.

여러 예외를 지정할 수도 있습니다. \
@Transactional(rollbackFor = {SQLException.class, IOException.class})





## 6. 읽기 전용 트랜잭션 설정(readOnly = true/false)

트랜잭션에서 readOnly=true 옵션을 통해서 읽기 전용 모드로 동작하게 할 수 있습니다. 성능, 메모리 최적화를 위해 사용되며 데이터를 조회하는 작업에 사용됩니다.

```java
@Service
@RequiredArgsConstructor
public class ParentService {

    private final UserRepository userRepository;
    private final ChildService childService;

    @Transactional(readOnly = true)  // 부모 트랜잭션: 읽기 전용
    public void readAndModifyUser(Long userId) {
        User user = userRepository.findById(userId)
                                  .orElseThrow(() -> new RuntimeException("User not found"));

        // 자식 트랜잭션에서 쓰기 작업을 수행
        childService.updateUserStats(user);
    }
}

@Service
@RequiredArgsConstructor
public class ChildService {

    private final UserStatsRepository userStatsRepository;

    @Transactional(propagation = Propagation.REQUIRES_NEW)  // 새로운 쓰기 트랜잭션 생성
    public void updateUserStats(User user) {
        UserStats stats = user.getStats();
        stats.incrementLoginCount();
        userStatsRepository.save(stats);
    }
}
```

* 메모리 이점: JPA 영속성 컨텍스트가 스냅샷 저장소를 유지하지 않게 됩니다.
* 불필요한 플러시 방지: 자동으로 Hibermate에서 FlushMode.MANUAL로 변경하여, 트랜잭션이 끝날때 자동으로 DB에 반영되는 flush를 하지 않습니다.&#x20;
* 데이터베이스 잠금 최적화: 동시성 성능을 높입니다.
* 가독성: 읽기 전용인것을 단번에 알 수 있습니다.

따라서, 읽기 작업의 트랜잭션의 경우 예외없이 무조건사용하는 것이 좋습니다.





## 7. 트랜잭션의 타임아웃 설정(timeout = \~)

타임아웃 설정은 트랜잭션이 시작된 후 지정된 시간 내에 완료되지 않으면 자동으로 롤백되는 기능입니다.

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional(timeout = 5) // 5초로 설정
    public void deleteUser(Long userId) {
        User user = userRepository.findById(userId)
                                  .orElseThrow(() -> new EntityNotFoundException("User not found"));
        userRepository.delete(user); //cascade가 걸려있다고 가정
    }
}
```

* 과도한 락 점유 방지: 다른 트랜잭션이 대기 상태로 빠지며 성능 저하
* 데드락:&#x20;

### 7.1. 동작 원리

1. 트랜잭션 시작: 트랜잭션 매니저는 해당 시간 동안 트랜잭션이 완료되기를 기다립니다.
2. 정상 종료: 타임아웃 시간 내에 정상적으로 완료되면 트랜잭션은 커밋됩니다.
3. 타임아웃 초과: 트랜잭션 매니저는 트랜잭션을 롤백하고, 변경 사항은 모두 취소





## 8. 트랜잭션 이벤트와 리스너(phase = TransactionPhase.\~\~)

트랜잭션의 상태 변화에 따라 이벤트를 발생시키는 것. 커밋이나 롤백 시점에 특정 작업을 수행할 수 있습니다.

**@TransactionalEventListener의 단계**

* **BEFORE\_COMMIT**: 트랜잭션 커밋 직전에 호출
* **AFTER\_COMMIT**: 트랜잭션 커밋 후에 호출 (가장 많이 사용됨)
* **AFTER\_ROLLBACK**: 트랜잭션 롤백 후에 호출
* **AFTER\_COMPLETION**: 트랜잭션 완료 후에 호출 (커밋 또는 롤백 상관없이)

```java
@Transactional
public void performTransactionalOperation() {
    // 트랜잭션 내 작업
    someDatabaseOperation();

    // 트랜잭션 작업 후 바로 메서드 호출
    sendEmail();
}
```

위 상황의 경우, 트랜잭션이 커밋되기전에 항상 이메일을 보내게 됩니다.

```java
public class TransactionalEvent {
    private String message;

    public TransactionalEvent(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}

@Service
public class TransactionalService {

    private final ApplicationEventPublisher eventPublisher;

    public TransactionalService(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    @Transactional
    public void performTransactionalOperation() {
        // 트랜잭션 내 작업
        someDatabaseOperation();

        // 트랜잭션 커밋 후 실행될 이벤트 발행
        eventPublisher.publishEvent(new TransactionalEvent("이메일 전송"));
    }
}

@Component
public class TransactionEventListener {

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT) //
    public void handleAfterCommit(TransactionalEvent event) {
        // 트랜잭션 커밋 후 실행할 로직
        sendEmail(event.getMessage());
    }

    private void sendEmail(String message) {
        System.out.println("트랜잭션 후 이메일 전송: " + message);
    }
}
```

EventListener를 사용하게되면, 커밋이 되기 전에 이메일 전송을 하게 되지만, \
@TransactionalEventListener(phase = TransactionPhase.AFTER\_COMMIT)로 인해 커밋 후에 실제로 실행되게 됩니다.

* 데이터 일관성 유지
* 불필요한 작업 방지
* 롤백 시점에 따른 보정 작업





## 9. 테스트에서의 @Transactional 사용

테스트 클래스나 메서드에 @Transactional을 적용하면, 각 테스트가 완료된 후에 자동으로 롤백됩니다. 이렇게 사용하게 되면, 테스트 간 데이터 간섭이 없으므로 안정적인 테스트가 가능합니다.

```java
@SpringBootTest
@Transactional
public class UserServiceTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    //@Commit 을 사용하여 데이터베이스에 실제로 커밋되게 할 수 있습니다.
    public void testCreateUser() {
        // 트랜잭션 내에서 사용자 추가
        User user = new User("testUser");
        userRepository.save(user);

        // 사용자 추가 확인
        assertNotNull(userRepository.findByUsername("testUser"));

        // 테스트가 끝나면 트랜잭션이 롤백되어 데이터베이스에 반영되지 않음
    }
}
```
