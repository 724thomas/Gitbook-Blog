---
description: 다이나믹 프록시
---

# Dynamic Proxy

## 다이나믹 프록시와 리플렉션(Part 1)

### 1. 출발점 — 프록시는 이해되는데, “런타임 생성”이 이해되지 않았다

프록시 패턴 자체는 어렵지 않습니다.

우리는 이미 이런 코드를 이해하고 있었습니다.

```kotlin
interface UserService {
    fun add(user: String)
    fun upgradeLevels()
}

class UserServiceImpl : UserService {
    override fun add(user: String) {
        println("사용자 등록: $user")
    }

    override fun upgradeLevels() {
        println("사용자 등급 업그레이드")
    }
}

class UserServiceTxProxy(
    private val target: UserService
) : UserService {

    override fun add(user: String) {
        target.add(user)
    }

    override fun upgradeLevels() {
        println("[TX] 트랜잭션 시작")
        try {
            target.upgradeLevels()
            println("[TX] 커밋")
        } catch (e: Exception) {
            println("[TX] 롤백")
            throw e
        }
    }
}
```

이 구조는 명확합니다.

* 프록시는 같은 인터페이스를 구현한다.
* 내부에 동일 타입의 target을 가진다.
* 실제 호출은 target에 위임한다.
* 앞뒤에 부가기능을 삽입한다.

여기까지는 아무 문제가 없습니다.

하지만 Spring에서 `@Transactional`을 붙였을 때,\
우리는 `UserServiceTxProxy`를 만들지 않았습니다.

그런데 트랜잭션이 동작합니다.

이 시점에서 이해가 깨지기 시작합니다.

> “프록시는 코드가 있어야 호출되는 것 아닌가?”
>
> “런타임에 생성된다는 건 대체 무슨 의미인가?”

이 질문을 풀기 위해 가장 먼저 **컴파일과 런타임을 분리해야 했습니다.**

***

### 2. 컴파일 시점 — 아무 일도 일어나지 않는다

```
@Service
class UserServiceImpl : UserService {

    @Transactional
    override fun upgradeLevels() {
        println("비즈니스 로직 실행")
    }
}
```

여기서 많이 착각합니다.

> “@Transactional을 붙이면 컴파일 단계에서 바이트코드가 바뀌는 것 아닌가?”

아닙니다.

컴파일 단계에서 일어나는 일은 단 하나입니다.

* Kotlin → JVM 바이트코드(.class) 생성
* @Transactional은 클래스 메타데이터 영역에 기록

끝입니다.

이 시점의 클래스는 그냥 평범한 클래스입니다.

```kotlin
public void upgradeLevels() {
    System.out.println("비즈니스 로직 실행");
}
```

트랜잭션 코드? 없습니다.\
프록시? 없습니다.\
AOP? 없습니다.

컴파일은 정적 단계입니다.\
Spring은 아직 등장하지 않았습니다.

***

### 3. 애플리케이션 실행 — Spring이 개입하기 시작한다

```kotlin
fun main() {
    runApplication<App>()
}
```

이제 런타임입니다.

여기서 중요한 구분이 등장합니다.

| 단계  | 수행 주체        | 역할                 |
| --- | ------------ | ------------------ |
| 컴파일 | 컴파일러         | .class 파일 생성       |
| 실행  | JVM + Spring | 객체 생성, 후처리, 프록시 적용 |

Spring은 실행 시점에 동작합니다.

ApplicationContext가 생성되면서 다음이 진행됩니다.

1. Component Scan
2. BeanDefinition 등록
3. Bean 인스턴스 생성
4. BeanPostProcessor 실행



> “Bean 초기화는 컴파일 시점 아닌가?”

아닙니다.

Bean 초기화는 **런타임**입니다.\
Spring이 객체를 실제로 생성하고 가공하는 단계입니다.

***

### 4. Bean 생성 — 아직은 순수 객체다

Spring이 실제로 객체를 만듭니다.

```kotlin
val original = UserServiceImpl()
```

이 시점에서의 상태:

* 프록시 아님
* 트랜잭션 기능 없음
* 그냥 순수 객체

많은 사람들이 여기서 오해합니다.

> “그럼 프록시는 언제 생기는 거지?”

바로 다음 단계입니다.

***

### 5. BeanPostProcessor — AOP 적용 판단

Spring 내부에는 `BeanPostProcessor`라는 확장 포인트가 있습니다.

그중 AOP 관련 핵심이 바로 이것입니다.

```kotlin
AnnotationAwareAspectJAutoProxyCreator
```

이 객체가 Bean이 생성될 때마다 개입합니다.

#### 내부에서 하는 일

1. 이 Bean에 AOP가 필요한가?
2. @Transactional이 붙어 있는가?
3. 프록시로 감싸야 하는가?

만약 필요하다면, Spring은 원본 Bean을 그대로 컨테이너에 넣지 않습니다.

대신 새로운 객체를 만듭니다.

그것이 바로 **다이나믹 프록시**입니다.

***

### 6. 다이나믹 프록시 — 언제 만들어지는가?

Spring이 내부적으로 호출하는 코드의 핵심은 이것입니다.

```kotlin
Proxy.newProxyInstance(...)
```

이 메서드가 호출되는 순간,\
JDK 내부에서 다음 일이 벌어집니다.

***

#### 6.1 인터페이스 분석

예를 들어:

```kotlin
interface UserService {
    fun add(user: String)
    fun upgradeLevels()
}
```

JDK는 이 인터페이스의 메서드 목록을 읽습니다.

* add(String)
* upgradeLevels()

이 단계는 메타정보 분석 단계입니다.



> “리플렉션이 여기서 쓰이는 건가?”

아닙니다.

이건 바이트코드 생성 준비 단계입니다.\
우리가 말하는 “리플렉션 호출”은 아닙니다.

***

#### 6.2 프록시 바이트코드 생성

JDK 내부에는 ProxyGenerator가 있습니다.

이 클래스는 바이트코드를 직접 조립합니다.

결과적으로 이런 구조의 클래스가 만들어집니다.

```kotlin
class $Proxy0 implements UserService {

    InvocationHandler h;

    public void upgradeLevels() {
        h.invoke(this, METHOD_upgradeLevels, null);
    }
}
```

중요한 점:

* 이 클래스는 소스코드가 없습니다.
* 디스크에 .class 파일도 없습니다.
* 메모리에만 존재합니다.



> “JVM이 직접 만든다는 건 무슨 의미인가?”
>
> “그 만드는 방법은 코드로 존재하는가?”

정확히는:

* JDK 내부에 바이트코드를 생성하는 코드가 존재한다.
* newProxyInstance 호출 시 그 코드가 실행된다.
* 바이트 배열이 생성된다.
* defineClass()로 JVM에 등록된다.

즉, 마법이 아닙니다.\
코드가 존재합니다.

***

#### 6.3 defineClass — 실행 중 클래스 정의

```kotlin
defineClass(name, byteArray, ...)
```

이 메서드는 JVM에게 말합니다.

> “이 바이트 배열을 새로운 클래스로 등록하라.”

여기서 핵심 개념이 등장합니다.

JVM은:

* 컴파일된 파일만 로딩하는 것이 아니다.
* 실행 중에도 클래스를 정의할 수 있다.

이게 “런타임 생성”의 정확한 의미입니다.

***

#### 6.4 프록시 인스턴스 생성

이제 생성된 클래스의 인스턴스를 만듭니다.

```kotlin
new $Proxy0(handler)
```

그리고 이 프록시 객체가 Spring 컨테이너에 등록됩니다.

구조는 이렇게 됩니다.

```
UserService 타입
    ↓
프록시 객체
    ↓
target = UserServiceImpl
```

컨테이너에는 원본이 아니라 프록시가 들어갑니다.



## 다이나믹 프록시와 리플렉션 (Part 2)

### 요약

Part 1에서 우리는 “프록시가 언제 만들어지는지”를 끝까지 따라갔습니다. 이번 Part 2는 그 다음 단계, 즉 **만들어진 프록시가 실제로 메서드를 어떻게 가로채고**, 그 과정에서 **InvocationHandler와 Method 객체가 어떤 역할을 하며**, 마지막에 **리플렉션이 정확히 어디서 발생하는지**를 한 단계도 생략하지 않고 연결합니다.

핵심은 두 가지입니다.

* 프록시가 가로채는 순간, 호출 정보(어떤 메서드인지)는 **InvocationHandler가 찾는 게 아니라 프록시가 전달**합니다.
* 리플렉션은 “프록시를 만들기 위해서”가 아니라, **실제 target 메서드를 실행하기 위해 `method.invoke()`를 호출하는 순간** 발생합니다.

***

### 1) “메서드를 호출하면 프록시가 대신 호출된다”는 말의 정확한 의미

Part 1의 마지막 상태는 이 구조였습니다.

* Spring 컨테이너에 등록된 Bean은 원본이 아니라 프록시 객체
* 원본 객체는 프록시 내부에 `target`으로 존재

즉, 사용자 코드에서 이렇게 보이더라도,

```kotlin
val userService: UserService = applicationContext.getBean(UserService::class.java)
userService.upgradeLevels()
```

실제로 `userService`는 `UserServiceImpl`이 아니라 **프록시 인스턴스**입니다. 이게 “프록시가 대신 호출된다”의 첫 번째 의미입니다. 호출한 대상이 바뀐 것이고, 그 덕분에 프록시는 호출 흐름의 맨 앞에서 로직을 삽입할 수 있습니다.

여기서 중요한 감각은 이것입니다.

* 컴파일 시점의 호출 코드는 `userService.upgradeLevels()`로 고정되어 있습니다.
* 하지만 런타임에 `userService`가 가리키는 실제 인스턴스가 “원본”이 아니라 “프록시”라서, 동일한 호출 문장이 다른 경로로 실행됩니다.

이 지점이 Part 1에서 “프록시는 코드가 있어야 호출되는 것 아닌가?”라는 혼란이 생겼던 이유와 연결됩니다. 코드는 그대로인데, **인스턴스가 바뀌었기 때문에** 실행 흐름이 바뀝니다.

***

### 2) 프록시 클래스 내부는 어떤 모양인가

우리가 작성한 정적 프록시(`UserServiceTxProxy`)는 소스 코드가 눈에 보였습니다. 다이나믹 프록시는 소스 코드가 보이지 않습니다. 하지만 논리적으로는 이런 형태의 코드가 “메모리 안”에 존재한다고 이해하면 됩니다.

```kotlin
// 개념적 의사 코드(실제 생성물은 바이트코드)
class ProxyLikeUserService(
    private val handler: java.lang.reflect.InvocationHandler
) : UserService {

    override fun add(user: String) {
        // add 호출이 들어오면 곧바로 invoke로 위임
        handler.invoke(this, /* add에 해당하는 Method */, arrayOf(user))
    }

    override fun upgradeLevels() {
        // upgradeLevels 호출이 들어오면 곧바로 invoke로 위임
        handler.invoke(this, /* upgradeLevels에 해당하는 Method */, null)
    }
}
```

여기서 핵심은 프록시 클래스의 모든 메서드 구현이 사실상 한 문장으로 요약된다는 점입니다.

* “어떤 메서드가 호출되든 handler.invoke(...)로 넘긴다.”

이 구조 때문에 “invoke에서 모든 메서드를 호출하는 거야?”라는 질문이 나올 수밖에 없습니다. 결론은 그렇습니다. 프록시의 각 메서드는 결국 `invoke()`라는 공통 관문을 거쳐서 실행됩니다. 다만 정확히 표현하면 “invoke가 모든 메서드를 호출한다”라기보다, “프록시의 모든 메서드 구현이 invoke로 위임된다”가 더 정확합니다.

***

### 3) InvocationHandler는 무엇을 하는가 (그리고 무엇을 하지 않는가)

이제 InvocationHandler 쪽을 봅니다. 이해를 위해 우리가 직접 만든 핸들러를 그대로 사용하겠습니다.

```kotlin
import java.lang.reflect.InvocationHandler
import java.lang.reflect.Method

class TxInvocationHandler(
    private val target: Any
) : InvocationHandler {

    override fun invoke(
        proxy: Any,
        method: Method,
        args: Array<Any>?
    ): Any? {
        println("[TX] 트랜잭션 시작 - method=${method.name}")

        return try {
            // 핵심: 타깃 메서드 실행은 method.invoke(...)로 수행됨
            val result = method.invoke(target, *(args ?: emptyArray()))
            println("[TX] 커밋 - method=${method.name}")
            result
        } catch (t: Throwable) {
            println("[TX] 롤백 - method=${method.name}")
            throw t
        }
    }
}
```

여기서 InvocationHandler의 책임은 딱 두 가지입니다.

1. 공통 관문에서 “부가기능(트랜잭션 시작/커밋/롤백)”을 수행한다.
2. 그 다음 실제 타깃 메서드를 실행한다.

반대로 InvocationHandler가 **하지 않는 일**도 중요합니다.

* InvocationHandler는 “다음에 어떤 메서드가 호출될지” 미리 알지 못합니다.
* InvocationHandler는 “method를 이름으로 검색해서 찾아오지 않습니다.”
* InvocationHandler는 “프록시 클래스를 생성하지 않습니다.” (프록시 생성은 `Proxy.newProxyInstance` 내부의 역할)

이 정리가 중요한 이유는 다음 질문으로 자연스럽게 이어집니다.

> “InvocationHandler는 이미 메서드를 알고 있다?”

이 질문에 대한 답은 한 문장으로 끝내면 오해가 남습니다. 따라서 문맥까지 포함해서 풀어야 합니다.

InvocationHandler는 **미리** 메서드를 알고 있지 않습니다. 대신, **호출 시점**에 프록시가 `Method` 객체를 인자로 넘겨주기 때문에 “그 순간에는” 어떤 메서드인지 알 수 있습니다. 즉, Handler가 메서드를 ‘외워서’ 알고 있는 게 아니라, ‘전달받아서’ 알고 있습니다. 이 전달 구조 덕분에 Handler는 특정 인터페이스/특정 메서드에 종속되지 않고, 어떤 인터페이스든 공통 방식으로 처리할 수 있습니다. 그래서 다이나믹 프록시는 “프록시를 찍어내는 공장”처럼 동작할 수 있고, Handler는 “공통 관문에서 할 일만 하는 로직”으로 유지됩니다. 결과적으로 메서드가 늘어나도 Handler를 다시 작성할 필요가 없고, 프록시가 자동으로 모든 메서드를 invoke로 모아줍니다.

***

### 4) Method 객체는 무엇인가 (문자열이 아니다)

우리가 초반에 리플렉션을 헷갈렸던 가장 큰 이유는 “메서드를 문자열로 가져온다”라는 설명 때문이었습니다. 실제로 `getMethod("upgradeLevels")` 같은 API는 문자열을 쓰기 때문에, 이런 질문이 나옵니다.

> “메서드 이름을 문자열로 가져오는데, 다른 클래스에 동일한 메서드 이름이 있으면?”

이 질문은 매우 타당합니다. 하지만 다이나믹 프록시의 invoke 흐름에서는, 보통 사용자가 `getMethod("...")`로 직접 찾는 게 아니라 **JVM/프록시가 정확한 Method를 넘겨줍니다.** 그리고 그 `Method`는 단순히 이름 문자열이 아니라, “어떤 클래스/인터페이스에 선언된, 어떤 시그니처의 메서드인지”를 모두 담고 있는 객체입니다.

확인용으로 이런 로그를 찍어보면 감각이 잡힙니다.

```kotlin
import java.lang.reflect.InvocationHandler
import java.lang.reflect.Method

class DebugHandler(private val target: Any) : InvocationHandler {
    override fun invoke(proxy: Any, method: Method, args: Array<Any>?): Any? {
        println("declaringClass=${method.declaringClass.name}")
        println("name=${method.name}")
        println("paramTypes=${method.parameterTypes.joinToString { it.simpleName }}")
        return method.invoke(target, *(args ?: emptyArray()))
    }
}
```

여기서 `declaringClass`가 핵심입니다. 메서드 이름이 같아도 “어디에 선언된 메서드인지”가 다르면 Method 객체 자체가 다릅니다. 또한 오버로딩이 있으면 파라미터 타입 배열까지 포함되므로 이름만 같은 경우는 구분됩니다.

정리하면, “문자열로 찾으면 충돌하지 않나?”라는 걱정은 리플렉션을 직접 사용할 때의 걱정이고, 다이나믹 프록시의 실행 흐름에서는 대부분 “이미 특정된 Method 객체”가 흘러들어오기 때문에 충돌 문제가 구조적으로 줄어듭니다. 그리고 설령 사용자가 직접 `getMethod`를 쓰더라도, 그 API는 파라미터 타입까지 함께 넘겨서 시그니처로 찾는 형태를 기본으로 제공하므로, 이름 하나만으로 애매하게 실행되는 형태가 아닙니다. 이 관점이 잡히면 “리플렉션은 문자열 기반이라 위험하다”라는 인식이 많이 정리됩니다.

***

### 5) 리플렉션은 정확히 어디서 발생하는가

이제 가장 핵심 질문으로 돌아옵니다.

* “리플렉션이 아직도 이해가 잘 안된다.”
* “리플렉션은 InvocationHandler가 필요한 Method 정보를 가져오는 방식인가?”
* “어떻게 가져오는데?”

이 질문을 풀기 위해서는, 리플렉션을 “정보 조회”와 “동적 실행”으로 나눠 생각해야 합니다.

* **정보 조회 관점의 리플렉션**: Class/Method 같은 메타정보를 읽는 행위
* **동적 실행 관점의 리플렉션**: `method.invoke(target, ...)`로 실제 메서드를 실행하는 행위

우리가 대화에서 계속 강조했던 “핵심 리플렉션”은 두 번째입니다. 이유는 간단합니다. 성능/동작/흐름 관점에서 사람들이 말하는 리플렉션은 보통 `invoke`를 의미하기 때문입니다. 그리고 다이나믹 프록시에서 리플렉션이 “정확히 일어나는 지점”도 바로 이 줄입니다.

```kotlin
val result = method.invoke(target, *(args ?: emptyArray()))
```

여기서 일어나는 일이 무엇이냐면,

* 컴파일 타임에 `target.upgradeLevels()`라고 “직접 호출”하지 않고,
* 런타임에 전달받은 `Method` 객체를 통해
* “어떤 메서드를 실행할지”를 동적으로 결정하고 실행합니다.

즉 리플렉션은 “메서드 정보를 가져오는 과정”이라기보다, “이미 전달받은 Method를 이용해 실제 실행을 하는 과정”입니다. Method 정보 자체는 프록시가 `invoke(proxy, method, args)` 형태로 넘겨주기 때문에, Handler는 찾을 필요가 없고 전달받은 것을 사용합니다. 따라서 “InvocationHandler가 Method 정보를 어떻게 가져오나?”라는 질문의 답은 “Handler가 가져오는 게 아니라 프록시가 전달한다”입니다. 그리고 “리플렉션은 어디서 발생하나?”의 답은 “Handler가 target을 호출할 때 `method.invoke`로 실행하는 순간”입니다. 이 두 문장을 분리해 이해하면 혼란이 크게 줄어듭니다.

***

### 6) 호출 흐름을 코드로 끝까지 재현해보기

아래는 “정적 프록시” 느낌을 유지하면서도, “다이나믹 프록시 → invoke → method.invoke” 흐름을 실제로 재현하는 최소 예시입니다.

```kotlin
import java.lang.reflect.InvocationHandler
import java.lang.reflect.Method
import java.lang.reflect.Proxy

interface UserService {
    fun add(user: String)
    fun upgradeLevels()
}

class UserServiceImpl : UserService {
    override fun add(user: String) {
        println("사용자 등록: $user")
    }

    override fun upgradeLevels() {
        println("사용자 등급 업그레이드")
    }
}

class TxInvocationHandler(private val target: Any) : InvocationHandler {
    override fun invoke(proxy: Any, method: Method, args: Array<Any>?): Any? {
        // 1) 프록시가 메서드를 가로챈 뒤, 모든 호출은 invoke로 들어옵니다.
        println("[TX] 시작 - ${method.name}")

        return try {
            // 2) 여기서부터가 리플렉션 기반의 '동적 실행'입니다.
            //    어떤 메서드를 실행할지(method)는 프록시가 이미 전달해준 상태입니다.
            val result = method.invoke(target, *(args ?: emptyArray()))

            // 3) 예외가 없으면 커밋(여기서는 로그로 표현)
            println("[TX] 커밋 - ${method.name}")
            result
        } catch (t: Throwable) {
            // 4) 예외가 있으면 롤백(여기서는 로그로 표현)
            println("[TX] 롤백 - ${method.name}")
            throw t
        }
    }
}

fun main() {
    val target: UserService = UserServiceImpl()

    val proxy = Proxy.newProxyInstance(
        UserService::class.java.classLoader,
        arrayOf(UserService::class.java),
        TxInvocationHandler(target)
    ) as UserService

    // 클라이언트는 UserService 타입만 알고 호출합니다.
    // 하지만 실제 인스턴스는 target이 아니라 proxy입니다.
    proxy.add("wonjoon")
    proxy.upgradeLevels()
}
```

이 코드를 기준으로 실행 흐름을 다시 한 번 “문장으로” 붙이면 이렇습니다.

1. `proxy.upgradeLevels()`를 호출합니다.
2. 실제로는 프록시 클래스의 `upgradeLevels()`가 실행됩니다.
3. 프록시 클래스는 내부에서 `handler.invoke(this, method_upgradeLevels, null)`을 호출합니다.
4. Handler는 트랜잭션 시작 로그를 찍고, `method.invoke(target, ...)`로 타깃을 실행합니다.
5. 그 `method.invoke(...)`가 리플렉션이며, 이 순간 런타임 동적 실행이 일어납니다.
6. 예외 여부에 따라 커밋/롤백 로직이 실행되고 호출이 종료됩니다.
