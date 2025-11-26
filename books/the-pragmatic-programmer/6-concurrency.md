---
description: 동시성
---

# #6 Concurrency

## 📌 Topic 33 ㅡ 시간적 결합(Temporal Coupling)을 깨뜨린다는 것

**— 일하는 순서에 끌려 다니지 않는 소프트웨어를 만들기 위해**

소프트웨어 아키텍처에서 “시간”이라는 요소는 생각보다 훨씬 중요한 문제다.\
우리가 평소에 느끼는 시간은 “일정” 같은 단기적인 것들이지만,\
사실 진짜 중요한 시간은 **설계된 시스템 안에서 작업들이 서로 어떤 ‘순서’에 의존하는가**이다.

많은 개발팀이 기능을 구현할 때 이런 식으로 생각하곤 한다.

> “이 작업 하고 → 그다음에 이것 하고 → 다음에 저것 한다.”

겉보기에는 자연스러워 보이지만, 이렇게 사고하면\
소프트웨어는 **시간적 결합(Temporal Coupling)** 이라는 함정에 빠지게 된다.

즉,\
**A가 끝나야 B를 할 수 있고,**\
**B가 끝나야 C를 할 수 있는** 식으로\
모든 작업이 ‘순차 실행’을 강요하게 되는 것이다.

그러나 실제 세상에서는 여러 일이 동시에 벌어지고,\
사용자는 언제든 버튼을 클릭할 수 있고,\
보고서는 한 번에 도착하지 않으며,\
에러는 예상치 못한 순간에 발생한다.

이런 현실을 반영하지 않고 순차적으로만 생각하면\
시스템은 점점 유연성을 잃고,\
버그는 증가하며,\
변경 비용은 커진다.

***

🔎 동시성을 먼저 보라

책에서는 이렇게 말한다.

> “메시지는 언제나 반드시 ‘보낸 순서’보다 먼저 도착할 수 있다.”

실제로 네트워크나 운영 환경에서는 이런 일이 흔하다.\
따라서 우리는 **‘순차적이어야 하는 것’과 ‘동시에 일어나도 되는 것’을 구분해야 한다.**

많은 프로젝트에서는 이 구분을 거의 하지 않고,\
그냥 “일어나는 순서대로” 코드를 짜버린다.

그래서 나중에 이렇게 된다.

* 왜 버튼을 클릭하면 UI가 멈출까?
* 왜 갑자기 데이터가 꼬이지?
* 왜 이벤트가 순서대로 처리되지 않는 거야?
* 왜 하나 고치면 다른 데서 또 터지는 거지?

답은 대부분 **시간적 결합** 때문이다.

***

🧠 동시성을 찾는 방법

책이 제안하는 방법은 의외로 단순하다.

#### 📌 1. “무조건 순서가 필요한 작업”과

#### 📌 2. “동시에 일어나도 되는 작업”을 구분하라.

이때 유용한 도구가 UML의 **활동 다이어그램(Activity Diagram)** 이다.\
특히 **동기화 막대(Synchronization Bar)** 를 활용하면\
동시에 시작하거나 동시에 끝나야 하는 작업들을 명확히 구분할 수 있다.

***

🍹 예제: 피냐 콜라다 만들기

책에서는 피냐 콜라다 칵테일 제조를 예로 든다.\
보통 우리는 레시피를 이렇게 “순서대로” 읽는다.

1. 믹서를 연다
2. 파인애플 믹스를 붓는다
3. 코코넛 밀크를 넣는다
4. 화이트 럼을 넣는다
5. 얼음을 넣는다
6. 뚜껑을 닫는다
7. 갈아준다
8. 잔을 가져온다
9. 우산 장식을 가져온다
10. 따르고 마무리한다

겉으로 보면 모든 것이 순차적으로 보이지만,\
냉정히 따져보면 동시에 할 수 있는 작업도 많다.

예를 들어:

* “잔 가져오기(8)”는\
  믹싱하는 동안(7) **동시에** 해도 된다.
* “우산 장식 가져오기(9)”도 믹싱 도중에 하고 있어도 된다.

그래서 활동 다이어그램으로 다시 그리면\
동시에 실행될 수 있는 작업 흐름이 눈에 보인다.

→ 이는 소프트웨어에서도 동일하다.\
순서에 종속된 작업만 “순차”로 두고,\
나머지는 “동시 실행 가능”하게 만들어야 한다.

***

🏎 왜 이게 중요한가?

시간적 결합이 심한 시스템은 다음과 같은 문제를 겪는다:

#### ❌ 문제가 되는 부분들

* **하나의 이벤트가 늦어지면 전체 시스템이 지연된다**
* **사용자가 특정 순서를 어기면 에러가 난다**
* **테스트가 어렵고 변경이 고통스럽다**
* **병렬 처리나 확장이 어렵다**

반면, 시간적 결합을 제거하면:

#### ✅ 얻는 이점들

* 시스템이 훨씬 안정적이고 탄력적이 된다
* 병렬 실행으로 성능이 자연스럽게 올라간다
* 변경하기 쉬운 구조가 된다
* Race Condition에 강한 구조가 된다
* 이벤트 기반 아키텍처를 설계하기 쉬워진다

특히 오늘날의 아키텍처—SSE, Webhook, 이벤트 스트림, Kafka, 메시지 큐—에서는\
이 개념이 사실상 **필수**이다.

***

#### ✔ 순서가 필요한 것과 중립적인 것을 반드시 구분하라

#### ✔ 동시 수행 가능한 부분을 찾아 분리하라

#### ✔ 활동 다이어그램 등을 활용해 작업 흐름을 ‘시각화’하라

#### ✔ 순차 실행 때문에 병목이 생기면 병렬화/비동기화를 고려하라

#### ✔ 이를 통해 시스템은 더 안정적이고 유지보수하기 쉬워진다

***









## 🧩 Topic 34 — 공유 상태는 틀린 상태(Shared State Is Wrong State)

레스토랑에서 파이를 하나 남겨두고 두 명의 종업원이 동시에 손님에게 “남은 파이가 있는지” 확인하고 주문을 받는다면 어떤 일이 발생할까?

* 종업원 A: “파이 1조각 남았네! 주문 받습니다!”
* 종업원 B: “파이 1조각 남았네! 주문 받습니다!”

실제로는 1조각만 남았기 때문에 **두 손님 중 한 명은 반드시 실패한다.**

이 상황은 우리가 매일 겪는 \*\*동시성 문제(Concurrency Issue)\*\*의 완벽한 은유다.\
여기서 핵심은 단 하나:

> **공유 상태(shared mutable state)는 언제나 틀린 상태(wrong state)를 만들어낸다.**

***

🍰 레스토랑 예제로 바라본 공유 상태

문제의 본질은 이거다:

* 두 종업원이 **같은 값**(파이 개수)을 공유한다.
* 종업원들은 각각 **자신만의 메모리에서 그 값을 읽는다.**
* 둘 다 “파이가 1개 남았다”고 착각한다.
* 파이 가져가기는 원자적이지 않다(atomic하지 않다).

즉, **병렬 실행이 가능해지는 순간, 공유된 값은 더 이상 신뢰할 수 없다.**

***

📉 비원자적 갱신이 문제를 만든다

책의 Ruby 예제를 **코틀린스럽게 재구성**하면 다음과 같다.

```kotlin
// 공유되는 진열장 데이터
data class DisplayCase(
    var pieCount: Int // 공유 상태
)

class Staff(private val displayCase: DisplayCase) {

    fun servePieToCustomer() {
        if (displayCase.pieCount > 0) {                       // 1. 파이 확인
            promisePieToCustomer()                            // 2. 손님에게 약속
            displayCase.pieCount -= 1                         // 3. 파이 가져감
            givePieToCustomer()                               // 4. 전달
        }
    }

    private fun promisePieToCustomer() { /* ... */ }
    private fun givePieToCustomer() { /* ... */ }
}
```

병렬로 두 스레드가 `servePieToCustomer()`를 실행하면:

1. 두 스레드가 거의 동시에 `pieCount > 0`를 읽는다
2. 둘 다 약속한다
3. 둘 다 pieCount를 감소시킨다
4. 실제 파이는 하나인데, 결과는 파이 **-1 조각** 같은 불가능한 상태가 된다

➡️ 이것이 바로 **공유 상태의 동시성 문제**다.

***

🛑 해결법 1 — 세마포어(Semaphore) 같은 상호 배제(Mutex)

파이 1개를 놓고 **한 명만 접근 가능하도록 잠금을 건다**.

코틀린에서는 `ReentrantLock` 또는 `Semaphore`를 많이 사용한다.

Kotlin 코드 (세마포어 기반)

```kotlin
class SafeDisplayCase(initialCount: Int) {

    private val semaphore = java.util.concurrent.Semaphore(1, true)
    private var pieCount: Int = initialCount

    fun takePieIfAvailable(): Boolean {
        semaphore.acquire() // 🔒 Lock

        return try {
            if (pieCount > 0) {
                pieCount -= 1
                true
            } else {
                false
            }
        } finally {
            semaphore.release() // 🔓 Unlock
        }
    }
}
```

이제 하나의 스레드만 파이 조각을 가져갈 수 있다.\
문제는 해결된 듯 보이지만…

***

⚠️ 세마포어의 문제

세마포어는 강력하지만 다음과 같은 위험이 있다:

* 한 스레드가 Lock을 획득한 후 예외가 발생하면?\
  → Unlock이 안 되고 모든 스레드가 멈춰 버린다 (Deadlock)
* 모든 코드 경로에 확실한 Unlock이 필요하다
* 코드 유지보수 시 Lock이 빠지기 쉽다
* 개발자가 Lock 위치를 실수하면 전체 시스템이 정지

➡️ **Lock 기반 접근은 위험하고, 쉽게 망가진다.**

***

🧱 해결법 2 — 트랜잭션처럼 “값 자체를 보호하는 API” 만들기

이 접근 방식은 훨씬 더 안전하다.

#### ❌ 잘못된 방식

* 스레드가 직접 “파이 개수”를 보고
* 파이가 0 이상이면
* 자신이 pieCount를 바꾼다

➡️ 이 구조 자체가 경쟁 조건을 만든다.

#### ✔ 올바른 방식

* “파이 1개 가져오기”라는 **단일 원자 연산 atomic operation**을 제공한다.
* 외부에서는 개수를 보지 않는다.
* 가져오기 자체가 성공/실패를 알려주도록 한다.

Kotlin 원자적 API 버전 (lock 내부 구현 숨김)

```kotlin
class PieCounter(initialCount: Int) {

    private val lock = java.util.concurrent.locks.ReentrantLock()
    private var count = initialCount

    fun tryTakeOne(): Boolean = lock.withLock {
        if (count > 0) {
            count -= 1
            true
        } else false
    }
}
```

#### 핵심은 이거다:

> **외부에서 count를 직접 읽게 하지 마라.**\
> **읽기 + 쓰기를 조합하면 경쟁 조건이 생긴다.**\
> **대신 "원자적 연산"만 제공해라.**

***

🍨 여러 리소스가 얽힌 문제 → 트랜잭션 필요

만약 파이뿐 아니라 아이스크림도 함께 꺼내야 한다면?

* 파이를 가져왔다 → 성공
* 아이스크림을 가져오는 중 실패
* 파이를 다시 넣어야 한다(skill required)

즉, **여러 공유 리소스의 조합 작업은 반드시 실패 복구가 필요**하다.

실전 코틀린 예:

```kotlin
val pie = pieCounter.tryTakeOne()

if (pie) {
    try {
        val scoop = freezer.tryTakeIceCream()
        if (scoop) {
            giveOrderToCustomer()
        } else {
            freezer.giveBack(scoop) // 실패 복구
        }
    } catch (ex: Exception) {
        pieCounter.giveBack()       // 파이 복구
    }
}
```

보면 알겠지만, 이런 코드는 **빠르게 엉망**이 된다.

* 예외 처리 복잡
* 복구 코드 중복
* 여러 Lock이 섞이면 Deadlock 위험
* 기능 추가 시 수정 포인트 증가

➡️ **공유 상태 + 동시성 + 여러 자원 조합 = 필연적으로 망가진다.**

***

🧨 트랜잭션이 없는 갱신은 언젠가 반드시 실패한다

실제 기업 시스템에서 동시성 문제는 다음에서 자주 발생한다:

* 결제 후 재고 감소
* 주문 수량 업데이트
* 좋아요/조회수 증가
* 예약 좌석 감소
* 알림 중복 방지 카운터
* 혜택 쿠폰 발급 수량 감소

공유 메모리 기반 로직은 **언젠가 반드시 꼬인다.**

***

🧭 그 밖의 언어적 접근

대부분 언어는 다음 두 가지 도구로 공유 리소스를 보호한다:

* Mutex(상호 배제 Lock)
* Monitor(블록 수준 Lock)

하지만 언어 차원에서 동시성을 근본적으로 해결하는 방식도 존재한다:

* Rust: 소유권(Ownership) + Borrowing
* Erlang/Elixir: Actor Model
* Haskell: STM(Software Transactional Memory)

이들은 아예 “공유 상태” 자체를 없애거나 제한한다.\
따라서 동시성 문제를 구조적으로 예방한다.

***

💬 결론: 공유 상태 = 틀린 상태, 피하라

Topic 34의 핵심 결론은 단 하나다.

> **공유 상태를 가능한 한 만들지 말고,**\
> **어쩔 수 없다면 반드시 원자적 연산(API)으로 감춰라.**

#### ✔ 공유 상태를 없애려면?

* 비공유 데이터 구조 사용
* Actor Model
* 메시지 기반 구조
* Immutable state
* 트랜잭션 기반 API

#### ✔ 공유 상태를 써야 한다면?

* 외부에 값 노출 금지
* 원자적 연산 제공
* Lock은 내부에만 숨기기
* 여러 자원 복합 갱신은 반드시 롤백 고려









## 🎭 Topic 35 — 액터와 프로세스: 공유 상태 없이 동시성을 다루는 법

> 작가가 없다면 이야기는 쓰이지 않을 것이다.\
> 배우(actor)가 없다면 이야기는 생명을 얻지 못할 것이다.\
> — 앤지-마리 델산테

공유 상태를 잠그기 위해 락과 세마포어를 사용하는 방식은 너무 위험하고,\
복잡하고, 오류가 생기기 쉽다.

그래서 등장한 개념이 바로 \*\*Actor Model(액터 모델)\*\*이다.

***

🧠 액터란?

액터(actor)는 다음 특성을 가진 동시성 개념이다.

* 자신만의 \*\*고립된 상태(state)\*\*를 가진다
* 외부와의 통신은 \*\*메시지 전달(message passing)\*\*뿐이다
* 공유 메모리가 없다 → Lock, synchronized 필요 없음
* 메시지 처리 중에는 다른 메시지를 받지 않는다 (single-threaded mailbox)

즉, 각 액터는 \*\*"독립된 작은 프로세서"\*\*처럼 동작한다.

***

🧵 프로세스란?

Erlang/Elixir에서 말하는 프로세스는 다음과 같다:

* 매우 가벼운 스레드
* 메시지로만 통신
* 서로 메모리를 공유하지 않음
* 수백만 개도 동시에 실행 가능
* 장애 시 자동 복구(supervision)

즉, 액터를 더 강력하게 확장한 개념이다.

***

🧩 액터는 언제나 동시성을 이긴다

액터 모델에서는 다음이 필요 없다:

* Lock
* atomic
* synchronized
* shared mutable state

왜?

> 액터는 오직 메시지에 의해, 하나씩 순차적으로만 동작하기 때문이다.

***

🥧 레스토랑 예제를 "액터 모델"로 다시 구현해보자

책은 JavaScript + Nact 라이브러리 예제로 설명한다.

우리는 **같은 개념을 코틀린 액터로 재작성**한다.

코틀린에서는 `Coroutine + Channel` 조합을 사용하면 자연스럽게 액터 모델을 구현할 수 있다.

***

🛠️ Kotlin Actor Framework (실전 구현)

우선 간단한 코틀린 액터 시스템을 만든다.

✔️ 액터 베이스 구조

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

interface Actor {
    suspend fun receive(msg: Any)
}

fun <T : Actor> CoroutineScope.actorOf(actor: T): SendChannel<Any> {
    val mailbox = Channel<Any>(Channel.UNLIMITED)

    launch {
        for (msg in mailbox) {
            actor.receive(msg)
        }
    }
    return mailbox
}
```

이제 모든 액터는 `receive(msg)`만 구현하면 된다.

***

👨‍🍳 레스토랑 액터 시스템 구성

액터 1 — CustomerActor

```kotlin
class CustomerActor(
    private val name: String,
    private val waiter: SendChannel<Any>
) : Actor {

    override suspend fun receive(msg: Any) {
        when (msg) {
            is WantPie -> waiter.send(OrderPie(this))
            is TellFoodOnTable -> println("$name 테이블에 나타난 ${msg.food}를 본다.")
            is SorryNoPie -> println("$name: 파이가 없다고? 부루퉁…")
        }
    }

    fun startWantingPie() = WantPie
}

object WantPie
data class TellFoodOnTable(val food: String)
object SorryNoPie
data class OrderPie(val customer: CustomerActor)
```

***

액터 2 — PieCaseActor (진열장)

파이 리스트를 상태로 보유, **공유 상태 없음**

```kotlin
class PieCaseActor(private val slices: MutableList<String>) : Actor {

    override suspend fun receive(msg: Any) {
        when (msg) {
            is TakePie -> {
                if (slices.isEmpty()) {
                    msg.replyTo.send(PieResult(null))
                } else {
                    val slice = slices.removeFirst()
                    msg.replyTo.send(PieResult(slice))
                }
            }
        }
    }
}

data class TakePie(val replyTo: SendChannel<PieResult>)
data class PieResult(val slice: String?)
```

***

액터 3 — WaiterActor (종업원)

고객 → 종업원 → 진열장으로 메시지 라우팅

```kotlin
class WaiterActor(
    private val pieCase: SendChannel<Any>
) : Actor {

    override suspend fun receive(msg: Any) {
        when (msg) {
            is OrderPie -> {
                val reply = Channel<PieResult>()
                pieCase.send(TakePie(reply))

                val result = reply.receive()
                if (result.slice == null) {
                    msg.customer.receive(SorryNoPie)
                } else {
                    msg.customer.receive(TellFoodOnTable(result.slice))
                }
            }
        }
    }
}
```

***

🚀 액터 시스템 실행

```kotlin
fun main() = runBlocking {
    val pieCase = actorOf(PieCaseActor(mutableListOf("사과", "복숭아", "체리")))
    val waiter  = actorOf(WaiterActor(pieCase))

    val c1 = CustomerActor("customer1", waiter)
    val c2 = CustomerActor("customer2", waiter)

    val customer1 = actorOf(c1)
    val customer2 = actorOf(c2)

    // 메시지 보내기
    customer1.send(c1.startWantingPie())
    customer2.send(c2.startWantingPie())
    customer1.send(c1.startWantingPie())

    delay(500)
}
```

***

📝 실행 결과 예시

```
customer1 테이블에 나타난 "사과 파이 한 조각"을 본다.
customer2 테이블에 나타난 "복숭아 파이 한 조각"을 본다.
customer1 테이블에 나타난 "체리 파이 한 조각"을 본다.
```

모든 동작은:

* 공유 상태 없음
* Lock 없음
* synchronized 없음
* 오직 메시지 기반 통신

으로 이루어진다.

***

🌟 왜 액터가 동시성을 이기는가?

#### 1) 공유 메모리가 없다

경쟁 조건(race condition) 불가능

#### 2) Lock과 Atomic이 필요 없다

락을 놓쳤을 때 생기는 지옥이 없다

#### 3) 메시지가 순차적으로 소비된다

액터 내부는 항상 "단일 스레드"처럼 동작

#### 4) 오류가 커지지 않는다

한 액터가 죽어도 전체 시스템이 죽지 않는다 (Erlang supervision)

***

🐦 얼랭(Erlang)에 대한 존경

Erlang은 다음을 제공한다:

* 수백만 개 프로세스를 가볍게 실행
* 메모리 공유 없음
* 항상 메시지 기반
* Supervision(자동 복구)
* 99.999999% 가용성 (“9개 9”)

Elixir는 이 철학을 더 현대적으로 발전시켰다.

액터 모델을 실제 산업에서 가장 성공적으로 구현한 사례가 바로 **Erlang/Elixir**다.

***

📌 결론: 동시성을 다룰 때 공유 상태를 버리고 액터를 사용하라

Topic 35가 말하는 핵심:

> **공유 상태를 보호하는 방법(lock/semaphore)을 고민하는 것이 아니라,**\
> **아예 공유 상태를 만들지 않는 구조(actor)를 고민하라.**

Kotlin에서도 충분히 실천 가능하며,\
특히 코루틴 + 채널 조합은 **액터 모델을 자연스럽게 구현**하게 해준다.







## 📌 Topic 36 ㅡ 칠판(Blackboard) 패턴 — 여러 전문가가 함께 문제를 해결하는 방식

동시성이나 협업이 필요한 복잡한 문제를 해결할 때, 우리는 종종 **모든 정보를 한데 모아두고 다 같이 바라보는 구조**를 필요로 한다.\
범죄 수사에서 흔히 보는 ‘큰 칠판(Blackboard)’이 대표적인 예다.

형사들은 사건 관련 정보 — 용의자, 알리바이, 증언, 단서 — 를 모두 큰 칠판에 적어두고 수십 번을 왔다 갔다 하며 퍼즐을 맞춘다. 어떤 형사는 전화를 뒤지고, 어떤 형사는 현장을 조사하고, 누군가는 CCTV를 훑는다.\
그러나 모두가 **동일한 칠판 위 정보**를 공유한다.

이것이 **Blackboard 패턴**이다.

***

🧩 Blackboard 패턴이 필요한 이유

**🔹 1. 모든 참여자가 동일한 정보에 접근하기 위해**\
사건을 함께 해결하는 형사들처럼, 여러 프로세스·스레드·모듈이 하나의 공통 문제를 해결하고자 할 때 필요하다.

**🔹 2. 각 구성원이 서로의 존재를 몰라도 됨**\
형사 A는 자신의 단서를 적고 갈 뿐, 형사 B가 언제 올지 몰라도 상관없다.\
‘결합도’를 낮춘다.

**🔹 3. 다양한 전문 지식을 가진 여러 참여자가 독립적으로 새로운 단서를 추가할 수 있음**\
서로 다른 알고리즘/서비스가 각자 관측한 정보를 칠판에 덧붙이며 문제 해결에 기여한다.

**🔹 4. 문제는 순서 없이 도착한다**\
데이터가 실시간으로, 랜덤 순서로, 여러 부서에서 도착하는 문제 해결 시스템에 적합하다.

***

📝 컴퓨터 과학의 BlackBoard 시스템

최초의 컴퓨터 기반 Blackboard 시스템은 **린다(Linda: David Gelernter)** 라는 언어였다.\
Linda는 공유 사실(facts)을 튜플 형태로 저장하고, 튜플 매칭으로 문제를 해결했다.

이후에는 JavaSpaces, TSpaces 등 다양한 형태로 확장되었고, 이들은 본질적으로 모두\
➡ **공유 저장소 + 패턴 기반 검색**\
이라는 개념을 유지한다.

***

💡 실무 개발자 관점에서 바라본 Blackboard 패턴

Blackboard 패턴은 아래 같은 현실 세계 문제에서 특히 강력하다.

* 대출 승인 같은 프로세스에서 여러 부서의 데이터를 순서 없이 수집해야 할 때
* 비정형 데이터가 여러 형식으로 도착하고, 이를 한 곳에 모아 규칙을 적용해야 할 때
* 로그, 이벤트, 텔레메트리를 통합 분석해야 할 때
* 여러 개의 ML 모델/전문가들이 순차적으로 데이터를 가공해야 할 때

즉, **여러 출처의 정보를 한데 모아 비동기적으로 처리해야 하는 상황**에 강하다.

***

🏗️ Kotlin으로 구현하는 간단한 Blackboard

책의 예제는 그림, 단서, 메모 중심이지만,\
우리는 **실제 애플리케이션에서 동작하는 형태**로 재구성하자.

아래는 **10년차 Kotlin 개발자가 Actor 기반으로 재구성한 Blackboard 예제**다.

***

✔ 1. Blackboard 모델 정의

```kotlin
/**
 * Blackboard에 저장될 정보 단위
 */
sealed interface Fact {
    val key: String
}

data class TextFact(
    override val key: String,
    val content: String,
) : Fact
```

***

✔ 2. Actor 기반 Blackboard(동시성 안전)

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.ActorScope
import kotlinx.coroutines.channels.Channel
import kotlinx.coroutines.channels.actor

/**
 * Blackboard 액터 메시지 정의
 */
sealed interface BlackboardMsg
data class InsertFact(val fact: Fact) : BlackboardMsg
data class QueryFact(val key: String, val replyTo: CompletableDeferred<List<Fact>>) : BlackboardMsg

/**
 * Blackboard Actor
 */
fun CoroutineScope.blackboardActor() = actor<BlackboardMsg>(capacity = Channel.UNLIMITED) {
    val storage = mutableMapOf<String, MutableList<Fact>>()

    for (msg in channel) {
        when (msg) {
            is InsertFact ->
                storage.computeIfAbsent(msg.fact.key) { mutableListOf() }
                    .add(msg.fact)

            is QueryFact ->
                msg.replyTo.complete(storage[msg.key].orEmpty())
        }
    }
}
```

#### 동시성 문제는?

* 공유 상태는 오직 Actor 내부에서만 수정 가능
* 외부는 메시지를 보내는 방식으로만 접근
* race condition 없음
* lock-free, deadlock-free

***

✔ 3. 여러 전문가(Actor)가 정보 추가하기

```kotlin
sealed interface DetectiveMsg
data object GatherClues : DetectiveMsg

fun CoroutineScope.detectiveActor(
    name: String,
    blackboard: SendChannel<BlackboardMsg>,
) = actor<DetectiveMsg> {

    suspend fun writeClue(key: String, value: String) {
        blackboard.send(InsertFact(TextFact(key, value)))
        println("👮 $name: '$key' 단서 추가")
    }

    for (msg in channel) {
        when (msg) {
            GatherClues -> {
                writeClue("신원", "목격자가 본 인물: 키 180 후드티")
                writeClue("알리바이", "$name 조사 결과: 알리바이 불명확")
                writeClue("증거", "통화 기록 확보")
            }
        }
    }
}
```

***

✔ 4. 실제 실행

```kotlin
fun main(): Unit = runBlocking {
    val board = blackboardActor()

    val d1 = detectiveActor("형사 A", board)
    val d2 = detectiveActor("형사 B", board)

    // 각각 독립적으로 정보를 올림
    d1.send(GatherClues)
    d2.send(GatherClues)

    delay(500)

    // 특정 키를 가진 정보 조회
    val deferred = CompletableDeferred<List<Fact>>()
    board.send(QueryFact("증거", deferred))

    println("📌 '증거' 목록:")
    deferred.await().forEach { println(" - ${(it as TextFact).content}") }
}
```

***

🧭 결론 — Blackboard 패턴을 언제 사용해야 할까?

#### ✔ 적합한 경우

* 여러 전문가/서비스가 동시에 다양한 정보를 수집해야 할 때
* 데이터가 순서 없이 비동기적으로 들어올 때
* 단일 출처가 아닌 여러 출처에서 데이터를 모아야 할 때
* 규칙 기반 추론, 패턴 매칭이 필요한 경우\
  (예: 부정행위 감지, 보안 이벤트 수집)

#### ❌ 적합하지 않은 경우

* 단일 스레드/단일 모듈에서만 해결할 문제
* 순서가 명확하고 직렬화된 처리만 필요한 경우
* 매우 단순한 요청-응답 모델

***

✨ 마무리

칠판(Blackboard) 패턴은 단순한 저장소가 아니다.\
**여러 주체가 비동기적으로 문제 해결에 기여하는 협업 시스템**을 만드는 데 핵심적인 아키텍처다.

특히 최신 애플리케이션에서는

* 이벤트 기반 시스템
* ML 추론 워크플로우
* 대규모 데이터 파이프라인\
  에서 이 패턴의 변형이 매우 많이 사용된다.

