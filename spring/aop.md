# AOP

## Q. Aspect, Advice, Pointcut, JoinPoint, Target Object

* **Aspect**는 여러 객체에 공통으로 적용할 관심사(예: 로깅, 트랜잭션)를 모듈화한 단위입니다.
* **Advice**는 실제로 실행되는 부가 기능 코드입니다. (메서드 실행 전·후·예외 시점 등에 개입할 수 있습니다.)
* **JoinPoint**는 실제로 Advice가 실행될 수 있는 시점(메서드 실행, 예외 발생 등)입니다.
* **Pointcut**은 어떤 JoinPoint에 Advice를 적용할지 조건을 정의한 표현식입니다.
* **Target Object**는 실제 비즈니스 로직을 담고 있는 원본 객체로, 프록시에 의해 감싸져 AOP가 적용됩니다.



## Q. Advice 종류와 동작 순서

* Spring AOP에서 Advice는 크게 5가지: **Before, After, AfterReturning, AfterThrowing, Around**가 있습니다.
* 실행 순서는 Around → Before → Target 메서드 실행 → AfterReturning/AfterThrowing → After입니다.
* Around는 메서드 실행 전후를 모두 제어할 수 있어, 가장 강력한 형태입니다.
* 예외 발생 시에는 AfterThrowing이 실행되고, 정상 반환 시에는 AfterReturning이 실행됩니다.
* 마지막으로 After는 성공/실패 여부와 관계없이 무조건 실행됩니다.



## Q. Proxy 기반 동작 구조

* Spring AOP는 기본적으로 **프록시 패턴**을 사용합니다. Target 객체를 직접 호출하지 않고 Proxy 객체를 호출하도록 DI 컨테이너가 관리합니다.
* **JDK Dynamic Proxy**는 인터페이스 기반 프록시를 생성하고, **CGLIB Proxy**는 클래스 기반으로 상속받아 프록시를 만듭니다.
* 인터페이스가 있으면 JDK Dynamic Proxy, 없으면 CGLIB을 기본적으로 사용합니다. (Spring 5 이후 기본 정책)
* 프록시 객체는 **빈 등록 시점**에 생성되어, 클라이언트는 항상 Target이 아닌 Proxy를 의존성 주입받습니다.

<details>

<summary>JDK Dynamic Proxy가 하는 일 (런타임에 자동 생성되는 가짜 클래스의 예시 느낌)</summary>

```java
class $Proxy0 implements MemberService { // JDK가 자동 생성
    private MemberService target; // 진짜 객체 (MemberServiceImpl)
    
    public $Proxy0(MemberService target) {
        this.target = target;
    }

    @Override
    public void join() {
        System.out.println("[프록시] Advice 실행 - 로그 찍기");
        target.join(); // 진짜 객체 호출
        System.out.println("[프록시] Advice 실행 - 트랜잭션 종료");
    }
}

MemberServiceImpl이 아닌 $Proxy0 (JDK Proxy 객체)이 등록됩니다.
하지만 $Proxy0이 MemberService 인터페이스를 구현하고 있으므로
@Autowired MemberService memberService; 주입이 문제없이 동작합니다.
```

</details>

<details>

<summary>CGLIB 프록시의 실제 생성 구조 예시 코드</summary>

```java
class MemberService {
    public void join() {
        System.out.println("회원 가입 로직 실행");
    }
}
```

```java
// CGLIB이 런타임에 만드는 "가짜 자식 클래스"의 느낌
class MemberService$$Proxy extends MemberService {
    private final MemberService target;

    public MemberService$$Proxy(MemberService target) {
        this.target = target;
    }

    @Override
    public void join() {
        System.out.println("[프록시] Advice - 시작");
        try {
            // 부모(=원본) 메서드를 호출하는 느낌
            target.join(); // 또는 super.join() 과 유사한 효과
            System.out.println("[프록시] Advice - 정상 반환");
        } catch (Throwable t) {
            System.out.println("[프록시] Advice - 예외 처리");
            throw t;
        } finally {
            System.out.println("[프록시] Advice - 종료");
        }
    }
}
```

핵심 느낌:

* **원본 클래스를 상속**한 자식 프록시 클래스를 만들고,
* **메서드를 오버라이드**해서 호출을 가로챈 뒤(Advice),
* 마지막에 원본 로직을 호출한다는 흐름이에요.

</details>



## Q. 프록시 클래스 캐스팅 문제

* JDK Dynamic Proxy는 인터페이스 기반이므로 **구현체로 캐스팅할 수 없습니다.**
* 예를 들어, `((MemberServiceImpl) proxy)`는 `ClassCastException`을 유발합니다.
* 반면 CGLIB Proxy는 구체 클래스를 상속하므로 캐스팅이 가능합니다.
* 따라서 AOP를 적용할 때는 가급적 **인터페이스 타입을 의존성 주입 받는 방식**을 권장합니다.

<details>

<summary>인터페이스는 캐스팅이 안되니까?</summary>

\*\*캐스팅이 되느냐는 “실제 객체의 런타임 타입”\*\*에 달려 있어요.

* **성공하는 경우**: 변수 타입은 인터페이스지만, **실제 객체**가 구현체면 구현체로 캐스팅 가능
* **실패하는 경우**: 변수 타입은 인터페이스지만, **실제 객체**가 구현체가 아니라 “프록시 클래스”면 구현체로 캐스팅 불가

JDK 동적 프록시(JDK Dynamic Proxy)는 \*\*인터페이스만 구현하는 ‘별도의 클래스’\*\*를 런타임에 만들기 때문에, 실제 객체가 구현체가 아닙니다. 그래서 구현체로 다운캐스팅하면 `ClassCastException`이 나는 거예요. 반면 CGLIB은 **구체 클래스를 상속한 자식 프록시**를 만들므로, 구현체 타입으로 캐스팅이 됩니다.

#### 예시로 비교

```java
// JDK Dynamic Proxy: 실패 예
MemberService proxy = appContext.getBean(MemberService.class); // 실제로는 $Proxy0 같은 클래스
MemberServiceImpl impl = (MemberServiceImpl) proxy; // ❌ ClassCastException
```

```java
// CGLIB Proxy: 성공 예 (구체 클래스를 상속한 프록시)
MemberService proxy = appContext.getBean(MemberService.class); // 실제로는 MemberService$$EnhancerByCGLIB...
MemberServiceImpl impl = (MemberServiceImpl) proxy; // ⭕ 캐스팅 OK
```

핵심 정리:

* **인터페이스라서** 캐스팅이 안 되는 게 아니라, **프록시의 런타임 타입이 구현체가 아니기 때문에**(JDK 방식) 캐스팅이 실패합니다.
* 실무에선 **항상 인터페이스 타입으로 주입**하면 JDK/CGLIB 어느 쪽이든 안전하게 동작합니다.
* 구현체로 꼭 캐스팅해야 할 이유가 있다면(지양) `proxy-target-class=true`로 **CGLIB 강제**를 고려하세요.

</details>

***

## Q. Self-invocation 문제

* 같은 클래스 내부에서 `this.methodA()` → `this.methodB()`처럼 자기 자신의 다른 메서드를 호출하면 Proxy를 거치지 않습니다.
* Proxy를 거치지 않으므로 Advice가 적용되지 않는 문제가 발생합니다.
* 이를 해결하기 위해서는 `AopContext.currentProxy()`를 사용하거나, 내부 호출을 분리해 다른 빈으로 위임하는 방식이 필요합니다.
* 이 문제는 AOP의 구조적 한계이자, “프록시를 통한 호출만 Advice 적용” 원칙 때문입니다.

***

## Q. Proxy 내부 call 구조 분석

* 클라이언트는 Proxy를 호출하고, Proxy 내부에서 MethodInterceptor(Advice 체인)가 실행됩니다.
* Advice 체인은 `Around → Before → Target → AfterReturning/AfterThrowing → After` 순서로 흘러갑니다.
* 각 Advice는 `ProceedingJoinPoint.proceed()`를 호출해야 Target 메서드 실행이 진행됩니다.
* 결국 Target 객체는 Proxy에 의해 감싸져 있고, Proxy → Advice Chain → Target → Advice Chain 역순 반환 구조로 실행됩니다.

***

## Q. Target Object 직접 호출과 AOP 적용 차이

* Target Object를 직접 호출하면 Proxy가 개입하지 않기 때문에 Advice가 전혀 적용되지 않습니다.
* 이는 스프링 컨테이너가 반환하는 Bean이 Proxy 객체라는 사실과 관련이 있습니다.
* 만약 `new` 키워드로 직접 객체를 생성하면 AOP 적용을 받을 수 없습니다.
* 따라서 항상 Spring 컨테이너가 관리하는 Bean(Proxy)을 통해 호출해야 AOP 기능이 보장됩니다.

***

## Q. 왜 Spring AOP는 메서드 실행 시점만 JoinPoint로 지원하나요?

* AspectJ는 필드 접근, 생성자 호출 등 다양한 JoinPoint를 지원합니다.
* 그러나 Spring AOP는 **프록시 기반**이라서 "메서드 실행"만 가로챌 수 있습니다.
* Proxy는 메서드 호출을 위임하는 구조라서 필드 접근 같은 low-level 조작은 할 수 없습니다.

***

## Q. AOP와 트랜잭션(@Transactional) 관계

* `@Transactional`은 사실상 AOP 기반의 Advice로 구현됩니다.
* 트랜잭션 시작, 커밋, 롤백 로직이 Proxy를 통해 Target 실행 전후로 적용됩니다.
* Self-invocation 문제가 여기서도 동일하게 발생합니다 → 같은 클래스 내부 메서드 호출은 트랜잭션이 적용되지 않습니다.

***

## Q. AOP 적용 범위를 설정하는 방법

* `@Pointcut` 표현식으로 특정 패키지, 클래스, 메서드, 어노테이션을 필터링할 수 있습니다.
* 예: `execution(* com.example.service..*(..))` 은 service 패키지의 모든 메서드에 Advice를 적용합니다.
* `@annotation(Transactional)`을 사용하면 특정 어노테이션이 붙은 메서드에만 적용할 수 있습니다.
* 이처럼 Pointcut 표현식은 AOP 적용 범위를 세밀하게 제어하는 핵심 도구입니다.

