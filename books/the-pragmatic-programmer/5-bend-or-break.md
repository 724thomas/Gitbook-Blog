---
description: 5장 구부러지거나 부러지거나
---

# #5 Bend, or Break

### 💻 실용주의 프로그래머: Topic 28

#### 결합도 줄이기 (Reduce Coupling)

***

#### 1. 핵심 개념 — “서로 덜 연결될수록 더 바꾸기 쉽다”

> “우리가 어떤 것 하나만을 골라내려 해도,\
> 그것이 우주의 다른 모든 것과 얽혀 있음을 깨닫게 된다.”\
> — 존 뮤어(John Muir)

결합도(Coupling)가 높은 코드는 바꾸기 어렵다.\
그 이유는 **한 부분의 변화가 다른 부분까지 연쇄적으로 영향을 미치기 때문**이다.\
코드를 수정할 때마다 여러 모듈을 동시에 수정해야 한다면, 이미 결합도가 높다는 신호다.

> 💡 좋은 설계란, **변경의 파급 범위가 좁은 설계**다.

***

#### 2. 개념 비유 — “단단한 연결 vs 유연한 연결”

* 단단하게 얽힌 구조(삼각형 그물망)는 하나만 바꿔도 전체가 흔들린다.
* 반대로 유연하게 연결된 구조(선형 연결)는 일부만 교체 가능하다.

소프트웨어도 마찬가지다.\
모듈 간 연결을 최소화해 **독립적 변경이 가능한 형태**로 만들어야 한다.

***

#### 3. Tip 44 — “결합도가 낮은 코드는 바꾸기 쉽다”

결합도가 높은 코드에서는 다음 문제가 반복적으로 발생한다.

* **연쇄 호출(Train Wreck):** 객체를 계속 점으로 연결하며 내부 상태에 접근
* **글로벌 상태(Global State):** 전역 변수나 싱글톤으로 인해 의존성 확산
* **상속 남용:** 부모 클래스 변경이 모든 자식에게 파급

> 💬 “한 모듈의 변경이 시스템 전체로 퍼질 때,\
> 그건 ‘결합도 폭발’이다.”

***

#### 4. 사례 — “열차 사고(Train Wreck)”

```java
public void applyDiscount(customer, order_id, discount) {
    totals = customer.orders.find(order_id).getTotals();
    totals.grandTotal = totals.grandTotal - discount;
}
```

이 코드의 문제:

* 고객(Customer), 주문(Order), 총합(Totals) 객체가 **직렬로 연결되어 있음**
* 변경이 발생하면 전부 수정해야 함
* 한 모듈의 수정이 여러 객체에 의존함

***

#### 5. Tip 45 — “묻지 말고 말하라 (Tell, Don’t Ask)”

객체의 내부 상태를 **묻고 직접 조작하지 말라.**\
대신 객체 스스로에게 **무엇을 하라(Tell)** 고 요청하라.

```java
// 잘못된 방식
public void applyDiscount(customer, order_id, discount) {
    totals = customer.orders.find(order_id).getTotals();
    totals.applyDiscount(discount);
}

// 개선된 방식
public void applyDiscount(customer, order_id, discount) {
    customer.applyDiscountToOrder(order_id, discount);
}
```

이 원칙은 **캡슐화(encapsulation)** 를 강화하고,\
연쇄 호출을 없애며, **한 객체의 책임을 명확히 분리**한다.

***

#### 6. 데메테르 법칙 (Law of Demeter, LoD)

> “메서드는 오직 가까운 친구와만 이야기해야 한다.”

**LoD 원칙:**\
메서드 내에서는 다음 대상에게만 메시지를 보낼 수 있다.

* 같은 클래스의 메서드
* 파라미터로 받은 객체
* 자신이 직접 생성한 객체
* 전역 변수 (지양)

즉,

```java
customer.getOrders().get(0).getTotals().getAmount();
```

같은 코드는 LoD 위반이다.

***

#### 7. Tip 46 — “메서드 호출을 얽지 말라”

> “`.` 하나를 줄이는 것이 결합도를 낮추는 시작이다.”

```java
// 🚫 나쁜 방식
amount = customer.orders.last().totals.amount;

// ✅ 개선된 방식
amount = customer.getLastOrderTotal();
```

이처럼 **중간 접근을 캡슐화**하면 결합도가 줄고\
코드 변경 시 영향 범위가 작아진다.

***

#### 8. 글로벌 상태의 해악

전역 변수나 싱글톤은 **모든 코드가 공유하는 결합 지점**이다.\
이로 인해 테스트가 어려워지고, 변경 시 시스템 전체가 흔들린다.

```java
// 나쁜 예시
Config.logLevel = "debug";
```

> 💡 전역 데이터는 “코드의 진흙탕”이다.\
> 모든 함수가 엉켜 있기 때문에 어디서 문제가 발생했는지 추적하기 어렵다.

***

#### 9. Tip 47 — “전역 데이터를 피하라”

* 전역 변수 대신 **의존성 주입(Dependency Injection)** 을 사용하라.
* 테스트 중에는 **Mock 객체**로 대체하라.
* 전역 데이터를 쓸 수밖에 없다면, **접근을 제한하는 래퍼(wrapper)** 를 만들어라.

***

#### 10. 외부 리소스도 전역 데이터다

데이터베이스, 파일 시스템, 외부 API 등은 모두 **전역 상태**를 가진 리소스다.\
이 리소스를 사용할 때는 항상 명시적으로 **관리 객체를 통해 접근**해야 한다.

예:

```kotlin
val result = transactionManager.run {
    userRepository.save(user)
}
```

***

#### 11. Tip 48 — “전역적이어야 할 만큼 중요하다면 API로 감싸라”

정말 모든 곳에서 접근해야 하는 전역 리소스라면,\
직접 노출하지 말고 **명시적인 API 계층으로 감싸라.**

> ✅ “중요한 전역은 감추고, 공개할 땐 제어하라.”

***

#### 12. 상속은 결합도를 높인다

상속은 **부모의 변경이 자식에게 즉시 전파되는 구조적 결합**이다.\
상속 대신 **조합(Composition)** 을 우선 고려하라.

> 💬 “상속은 재사용의 도구가 아니라 결합의 덫이다.”

***

#### 13. 결합도의 최종 결론 — “결합은 시스템의 부패를 부른다”

결합도가 높을수록

* 테스트가 어렵고,
* 리팩터링이 불가능하며,
* 변경 비용이 폭발한다.

결국 유지보수가 불가능한 시스템으로 이어진다.

***

> **“결합을 줄여라. 묻지 말고 말하라.”**\
> 느슨한 연결이 유연한 시스템을 만든다.\
> &#xNAN;_&#x4C;ow coupling, high cohesion._







### 💻 실용주의 프로그래머: Topic 29

#### 실세계를 갖고 저글링하기 (Program Close to the Real World)

***

#### 1. 핵심 개념 — “세상은 동적이다. 코드도 그래야 한다”

> “그냥 일어나는 일은 없다. 일어나도록 만들어진 것이다.”\
> — 존 F. 케네디

과거에는 사람이 컴퓨터의 한계에 맞춰야 했다.\
하지만 이제는 **컴퓨터가 현실의 세계에 맞춰야 한다.**\
현실은 사건이 일어나고, 상태가 바뀌며, 사람들이 반응한다.\
따라서 우리의 프로그램도 이 **“동적인 세계의 리듬”** 을 반영해야 한다.

> 💡 “반응형 시스템(Responsive System)”이란,\
> 현실의 이벤트 흐름을 모델링하여 유연하게 반응하는 시스템이다.

***

#### 2. 이벤트(Event)의 개념

이벤트란 **“무언가가 일어났음”을 나타내는 신호**이다.

* 버튼 클릭
* 데이터 갱신
* 네트워크 수신
* 센서 신호
* 계산 완료

즉, 프로그램은 입력값만 다루는 것이 아니라 **“시간상 발생하는 사건”** 에 반응해야 한다.

***

#### 3. 이벤트 기반 설계의 필요성

> “이벤트를 중심으로 코드를 설계하면,\
> 프로그램은 현실의 시간 축과 나란히 흐를 수 있다.”

이벤트 기반 구조는 시스템을 더 유연하게 만들고,\
비동기 환경에서도 직관적인 동작을 유지시킨다.

***

#### 4. 이벤트 처리 전략 — 4가지 핵심 기법

1. **유한 상태 기계 (Finite State Machine, FSM)**
2. **감시자 패턴 (Observer Pattern)**
3. **게시-구독 (Publish–Subscribe)**
4. **반응형 프로그래밍과 스트림 (Reactive Streams)**

***

#### 5. 유한 상태 기계 (FSM)

FSM은 “현재 상태(State)”와 “이벤트(Event)”에 따라 다음 상태로 전이(Transition)되는 구조다.

예시 👇\
웹소켓 메시지 수신을 FSM으로 표현:

* 초기 상태: 대기 중
* 헤더 메시지 수신 → “메시지 수신 중”으로 전이
* 데이터 수신 → 계속 “수신 중” 유지
* 트레일러 수신 → “완료”로 전이
* 오류 발생 → “에러”로 전이

이 과정을 코드로 나타내면 다음과 같다:

```ruby
TRANSITIONS = {
  initial:  { header: :reading },
  reading:  { data: :reading, trailer: :done },
}

state = :initial
while state != :done && state != :error
  msg = get_next_message()
  state = TRANSITIONS[state][msg.type] || :error
end
```

FSM은 복잡한 분기문을 단순한 **전이표(transition table)** 로 표현한다.\
상태별 로직을 독립적으로 유지할 수 있어 확장성이 좋다.

***

#### 6. 감시자 패턴 (Observer Pattern)

> “누군가가 무언가를 보고 있다.”

이벤트를 발생시키는 쪽은 **감시 대상(Observable)**,\
이벤트를 처리하는 쪽은 **감시자(Observer)** 다.

Ruby 예시:

```ruby
module Terminator
  CALLBACKS = []

  def self.register(callback)
    CALLBACKS << callback
  end

  def self.exit(status)
    CALLBACKS.each { |cb| cb.call(status) }
  end
end

Terminator.register(->(status) { puts "callback: 종료 상태 #{status}" })
Terminator.exit(99)
```

👉 여러 콜백을 등록하고, 한 이벤트 발생 시 **모두 호출**된다.\
하지만 **모든 감시자가 등록되어 있어야 한다는 제약** 때문에 결합도가 높아질 수 있다.

***

#### 7. 게시-구독 (Publish–Subscribe)

감시자 패턴의 결합도를 낮춘 형태.\
**발행자(Publisher)** 와 **구독자(Subscriber)** 가 직접 연결되지 않고,\
**채널(Channel)** 을 통해 간접적으로 통신한다.

* 발행자: 특정 채널에 이벤트를 보냄
* 구독자: 관심 있는 채널을 구독함
* 메시지는 채널을 통해 비동기적으로 전달됨

이 구조는

* 느슨한 결합(loose coupling),
* 병렬 처리,
* 분산 시스템 이벤트 전달\
  에 모두 적합하다.

***

#### 8. 반응형 프로그래밍과 스트림 (Reactive Programming & Streams)

> “이벤트를 데이터처럼 다뤄라.”

이벤트가 연속적으로 발생한다면,\
그것은 **시간을 따라 흐르는 데이터 스트림(stream)** 이 된다.

대표적인 구현이 RxJS.

```javascript
import { Observable } from "rxjs";
import { logValues } from "./logger.js";

let animals = Observable.of("ant", "bee", "cat", "dog", "elk");
let ticker = Observable.interval(500);

Observable.zip(animals, ticker)
  .subscribe(next => logValues(JSON.stringify(next)));
```

📈 0.5초마다 동물 이름을 하나씩 출력하는 스트림이 만들어진다.\
이벤트와 시간의 흐름이 “데이터 파이프라인”으로 결합된 형태다.

***

#### 9. 비동기 스트림과 REST 결합

RxJS는 REST 호출도 스트림으로 연결할 수 있다:

```javascript
import { ajax } from "rxjs/ajax";
import { mergeMap } from "rxjs/operators";

let users = Observable.of(3, 2, 1);

let result = users.pipe(
  mergeMap(user => ajax.getJSON(`https://reqres.in/api/users/${user}`))
);

result.subscribe(resp => console.log(resp));
```

여러 요청이 병렬로 실행되어,\
각 응답이 **독립적인 이벤트로 처리**된다.

> 💡 동기/비동기를 하나의 추상화로 통합하는 것이\
> 반응형 프로그래밍의 가장 큰 강점이다.

***

#### 10. 어디에나 이벤트가 있다

이벤트는 단순히 UI 클릭만이 아니다.

* 로그인이 성공했다
* 파일이 수정되었다
* 센서가 감지했다
* 트랜잭션이 커밋되었다

모든 시스템은 “이벤트의 연쇄”로 움직인다.\
이 이벤트를 중심으로 코드를 구성하면,\
시스템은 **실세계를 반영하는 유연한 흐름**을 갖게 된다.

***

#### 11. 관련 항목

* **Topic 28:** 결합도 줄이기
* **Topic 36:** 철판 (Concurrency & Synchronization)

***

> **“코드는 현실의 시간축 위에서 반응해야 한다.”**\
> 이벤트, 상태, 스트림을 저글링하라.\
> &#xNAN;_&#x44;on’t fight the world. Model it._







## 🧩 변환 프로그래밍 — “프로그램은 결국 변환이다”

> _“자신이 하고 있는 일 하나의 과정으로 서술할 수 없다면,_\
> &#xNAN;_&#xC790;기가 뭘 하고 있는지 모르는 것이다.” – W. 에드워즈 데밍_

### ✔️ 프로그래밍은 곧 변환 과정이다

우리는 코드를 짤 때 클래스 구조, 언어 문법, 알고리즘, 프레임워크 고민에 대부분의 시간을 쓴다.\
하지만 정작 **프로그램의 본질이 무엇인지**는 생각하지 않는다.

📌 **프로그램의 본질: 입력을 받아 출력으로 ‘변환’하는 것**

* 파일을 읽고 →
* 특정 규칙을 적용하며 →
* 결과를 출력한다

이 단순한 흐름이 모든 프로그램의 근본이다.

하지만 이 본질을 잊으면, 코드는 불필요하게 복잡해지고\
“왜 만든 기능인지”조차 불명확한 구조가 된다.

***

## 🔍 1970년대로 돌아가 파이프라인을 보자

옛 유닉스 환경에서는 에디터도 부실했고, C 컴파일러조차 불편했다.\
그래서 \*“잘게 나뉜 프로그램들을 파이프로 연결해 복잡한 작업을 완성”\*하는 방식이 자연스럽게 발달했다.

대표적인 예:

```bash
$ find . -type f | xargs wc -l | sort -n | tail -5
```

이 명령은 여러 변환 단계를 연결해 원하는 결과를 만들어낸다.

### 단계별로 쪼개보면?

#### 1) `find . -type f`

→ 디렉터리 내 모든 일반 파일 목록을 출력

#### 2) `xargs wc -l`

→ 각 파일의 줄 수를 구한다

#### 3) `sort -n`

→ 줄 수 기준 오름차순 정렬

#### 4) `tail -5`

→ 가장 줄 수가 많은 다섯 개만 출력

이렇게 보면, 전체는 단순히 \*\*연결된 변환들의 연속(line)\*\*일 뿐이다.

***

## 📦 데이터 공장처럼 생각하기

책에서는 이 구조를 \*\*“공장 조립 라인”\*\*에 비유한다.

입력(원자재)을 넣으면\
→ 변환 단계를 거쳐\
→ 완성된 결과물이 나온다.

이 사고방식은 코드를 작성할 때도 그대로 적용된다.

***

## 🧠 변환 모델로 생각하기

책에서는 이를 **파이프라인(pipeline)** 철학으로 정리한다.\
각 단계는 다음 작업의 입력이 되어 흐른다.

예시 그림 구성:

```
find → wc → sort → tail → head
```

이 단계를 통해\
\*\*“가장 줄 수가 긴 파일 5개”\*\*라는 결과가 나온다.

***

## 🧩 예제: 애너그램 탐색기 만들기

이제 실전 예제로 ‘애너그램(anagram)’ 프로그램을 만든다고 해보자.\
입력된 단어로 만들 수 있는 조합, 사전 검색, 분류 작업을 변환 단계로 나눠본다.

### 변환 단계 요약

| 단계   | 변환 내용                    | 예시                     |
| ---- | ------------------------ | ---------------------- |
| 단계 0 | 초기 입력                    | "lyvin"                |
| 단계 1 | 모든 글자 조합 구하기             | lv, ly, lvy, …         |
| 단계 2 | 각 조합을 정렬한 signature 만들기  | ilv, ilny, …           |
| 단계 3 | 사전에서 signature에 맞는 단어 찾기 | ivy, yin, vinyl …      |
| 단계 4 | 길이 기준으로 묶기               | 3→ivy, 4→liny, 5→vinyl |

이 전체 흐름이 하나의 **변환 파이프라인**이다.

***

## 🧪 실제 코드 예시(엘릭서)

책에서는 엘릭서로 파이프라인을 구현한다.

#### 예: 가능한 조합 생성

```elixir
def all_subsets_longer_than_three_characters(word) do
  word
  |> String.codepoints()
  |> Comb.subsets()
  |> Stream.filter(fn subset -> length(subset) >= 3 end)
  |> Stream.map(&List.to_string(&1))
end
```

이런 식으로 **각 단계를 하나의 변환 함수로** 만들고\
마지막에 연결해 하나의 파이프라인을 완성한다:

```elixir
def anagrams_in(word) do
  word
  |> all_subsets_longer_than_three_characters()
  |> as_unique_signatures()
  |> find_in_dictionary()
  |> group_by_length()
end
```

***

## 🛠 오류 처리는 어떻게?

세상은 완벽하지 않기 때문에 **변환 도중 오류가 날 수 있다.**

책은 이를 간단한 wrapper(튜플)로 처리한다:

* 성공: `{:ok, 결과}`
* 실패: `{:error, 이유}`

변환 단계 중 어느 단계에서든 에러가 나면\
그 이후 단계는 실행되지 않고 `{:error, 이유}`가 그대로 파이프라인을 지나간다.

이 방식은 명확하고, 테스트하기 쉽고, 확장이 용이하다.

***

## 🧭 왜 이것이 중요한가?

프로그램이 복잡해지는 이유는 대부분\
“데이터 흐름”이 아니라\
“구조, 클래스, 메서드 호출 관계”에 집착하기 때문이다.

하지만 책이 강조하는 핵심은 단 하나:

> **프로그램은 입력을 받아 출력으로 변환하는 과정이다.**

이 관점으로 전환하면,

* 기능 설계 명확해지고
* 테스트가 쉬워지고
* 오류 처리 구조가 간단해지고
* 대규모 리팩토링이 쉬워진다

***

## 📝 실용 정리

#### ✔️ 변환 중심으로 사고하라

입력 → 단계별 변환 → 출력\
이 흐름만 명확하면 코드는 단순해진다.

#### ✔️ 복잡도는 구조가 아니라 흐름에서 줄인다

결국 변환들이 잘 연결되어 있으면 프로그램은 자연스럽게 이해된다.

#### ✔️ 파이프라인은 언어에 상관 없다

어떤 언어든 변환 철학만 있으면 적용 가능하다.







## 🧱 Topic 31 — 상속세: 상속은 왜 이렇게 위험한가

> “당신이 원한 것은 바나나였지만, 당신이 받은 것은 바나나를 들고 있는 고릴라와 정글 전체다.”\
> — 조 암스트롱

“객체지향 언어를 쓰면 당연히 상속을 써야 하지 않나?”\
많은 개발자가 이렇게 생각한다.\
하지만 현실에서 상속은 **막강한 만큼 위험하고, 대부분의 경우 적합하지 않다.**

이 글에서는 상속이 왜 문제를 일으키는지,\
그리고 상속 없이도 더 깔끔하게 구조를 설계하는 방법을 살펴본다.\
코드는 모두 10년차 코틀린 개발자 관점으로 재구성했다.

***

## 🕰️ 상속의 역사: Simula67이 의도한 것

처음 상속이 등장한 시절, 목적은 지금과 달랐다.\
Simula67에서 상속은 단순히 “여러 타입을 하나의 리스트에 담기 위한 편의 기능”이었다.

초기 개념은 대략 이런 느낌에 가까웠다:

* 여러 객체를 한 리스트에 넣고 싶다
* 공통 필드를 prefix로 갖고 있으면 편하겠다
* 그래서 “상속 비슷한 것”이 등장

지금 우리가 사용하는 “부모 클래스—자식 클래스” 구조는 그 이후 스몰토크와 C++에 의해 굳어진 형태다.

***

## ⚠️ 상속이 위험한 이유 1: 결합 폭발

상속을 쓰면 자식 클래스는 부모 클래스의 구현에 **강하게 결합**된다.\
부모의 필드, 메서드, 내부 로직, 심지어 변경 이력까지 전부 자식에게 흘러들어온다.

아래는 책의 Ruby 예제를 **코틀린 버전으로 재구성한 것**이다.

### ❗ 잘못된 상속 예 (코틀린)

```kotlin
open class Vehicle(
    protected var speed: Int = 0
) {
    fun stop() {
        speed = 0
    }

    fun moveAt(speed: Int) {
        this.speed = speed
    }
}

class Car : Vehicle() {
    fun info(): String {
        return "현재 속도: $speed km/h 로 주행 중"
    }
}

fun main() {
    val car = Car()
    car.moveAt(30)   // ← Vehicle의 구현을 그대로 사용
    println(car.info())
}
```

이 코드의 문제를 보자.

* Vehicle의 moveAt() 구현이 변경되면 Car는 그대로 영향받는다
* Car가 Vehicle의 내부 구조를 “상식적으로” 안다고 착각한다
* Vehicle이 API를 바꾸면 Car는 바로 고장난다
* Car는 Vehicle이라는 설계 결정에서 벗어날 수 없다

**코드 재사용처럼 보이지만, 사실 결합도 급증이다.**

***

## ⚠️ 상속이 위험한 이유 2: 타입 분류를 위해 상속을 쓰는 것은 부적절

많은 사람이 상속을 “타입 분류”라고 착각하지만, 현실의 도메인은 훨씬 복잡하다.

예를 들어 Car는 Vehicle이지만 동시에:

* 보험 대상(InsuredItem)
* 대출 담보물(Collateral)
* 회계상의 자산(Asset)

이 될 수 있다.

즉 **단일 상속으로는 현실의 다중 정체성을 반영할 수 없다.**\
그래서 옛날 C++이 다중 상속을 만들었지만 그것은 더 큰 혼란만 만들었다.

***

## 🧩 상속의 대안 1 — 인터페이스 / 프로토콜

상속 없이도 타입을 통합할 수 있다.\
코틀린에서는 \*\*인터페이스 + 기본 구현(default method)\*\*로 충분하다.

### ✔️ 다형성 목적이라면 인터페이스가 정답

```kotlin
interface Drivable {
    fun currentSpeed(): Int
    fun stop()
}

interface Locatable {
    fun location(): Coordinate
    fun isLocationValid(): Boolean
}

data class Coordinate(val lat: Double, val lng: Double)

class Car(
    private var speed: Int = 0,
    private var coord: Coordinate = Coordinate(0.0, 0.0)
) : Drivable, Locatable {

    override fun currentSpeed() = speed
    override fun stop() { speed = 0 }

    override fun location() = coord
    override fun isLocationValid() = coord.lat != 0.0 || coord.lng != 0.0
}

fun printLocation(item: Locatable) {
    if (item.isLocationValid()) println(item.location())
}
```

#### 장점

* 다중 구현 가능
* 필요 함수만 강제
* 상속 트리 없음
* 도메인 정체성 충돌 없음

***

## 🧩 상속의 대안 2 — 위임(Delegation)

코틀린은 **위임을 언어 차원에서 지원**한다 (`by` 키워드 제공).

### ✔️ Repository 기능을 상속하지 않고 위임으로 해결

```kotlin
interface Persistence {
    fun save()
}

class DefaultPersistence(private val owner: Any) : Persistence {
    override fun save() {
        println("Saving $owner ...")
    }
}

class Account(
    private val id: Long,
    private val persistence: Persistence = DefaultPersistence(owner = "Account:$id")
) : Persistence by persistence // ← 위임!
```

#### 장점

* Account는 Persistence API 전체를 상속받지 않는다
* 필요한 메서드만 노출
* Persistence 구현 변경시 Account 영향 없음
* 결합도 ↓, 유연성 ↑

***

## 🧩 상속의 대안 3 — 믹스인 / 트레이트 / extension

코틀린은 mixin은 없지만,\
**interface + default method + 확장 함수** 조합으로 믹스인을 충분히 구현할 수 있다.

### 공통 finder 기능 믹스인 예시 (코틀린 스타일)

```kotlin
interface CommonFinders<T> {
    fun find(id: Long): T?
    fun findAll(): List<T>
}

class BasicRecord

class AccountRecord : BasicRecord(), CommonFinders<AccountRecord> {
    override fun find(id: Long): AccountRecord? = TODO()
    override fun findAll(): List<AccountRecord> = TODO()
}

class OrderRecord : BasicRecord(), CommonFinders<OrderRecord> {
    override fun find(id: Long): OrderRecord? = TODO()
    override fun findAll(): List<OrderRecord> = TODO()
}
```

mixin의 핵심 기능을 코틀린에서는 **인터페이스 + 디폴트 메서드**로 대체한다.

***

## 🧪 계층이 복잡해지면 위험은 기하급수적으로 증가한다

현실 시스템은 금방 복잡해진다.

* 검증 로직이 많아짐
* 정책마다 조건이 다름
* API 규칙 다름
* UI마다 제약 다름

상속 트리로 이 모든 것을 해결하려 하면 **트리가 기괴하게 커진다.**

책에서 나온 복잡한 상속 트리 그림은\
코틀린 백엔드 개발자가 Spring에서 계층 구조를 잘못 설계했을 때 종종 보는 모습 그대로다.

***

## 🧩 도메인 검증의 현실: 상속보다 mixin/조합이 더 적합하다

예를 들어 "계정 검증"이라는 공통 기능이 있다고 하자.

코틀린에서는 이렇게 구현하는 것이 훨씬 자연스럽다:

```kotlin
interface AccountValidations {
    fun validatePassword(): Boolean
    fun validateEmail(): Boolean
    fun validateBeforeSave(): Boolean
}

class AccountForCustomer(
    private val email: String,
    private val password: String
) : AccountValidations {
    override fun validatePassword() = password.length >= 8
    override fun validateEmail() = email.contains("@")
    override fun validateBeforeSave() = validatePassword() && validateEmail()
}

class AccountForAdmin(
    private val email: String,
    private val password: String
) : AccountValidations {
    override fun validatePassword() = password.length >= 10
    override fun validateEmail() = email.endsWith("@company.com")
    override fun validateBeforeSave() = validatePassword() && validateEmail()
}
```

➡️ 상속 트리가 늘어나지 않고,\
➡️ 필요한 기능만 조립하며,\
➡️ 검증 로직이 분리되며,\
➡️ 변경의 충격이 국소적이다.

***

## 🎯 결론 — 상속이 답인 경우는 거의 없다

Topic 31이 전달하는 핵심 메시지는 매우 명확하다.

#### ✔ 상속을 피하고 인터페이스·위임·믹스인을 먼저 고려하라

#### ✔ 상속은 결합도를 급격히 증가시킨다

#### ✔ 타입 정리는 상속 계층보다 조합(composition)이 맞다

#### ✔ 공통 기능은 mixin/trait/interface default method로 해결하라

#### ✔ 프레임워크가 강제하지 않는 이상 상속을 쓰지 마라







## 🧩 Topic 32 — 설정(Configuration): 코드 밖으로 빼라

> “모든 물건은 제자리에 두고, 일은 모두 때를 정해서 하라.”\
> — 벤저민 프랭클린

애플리케이션이 출시된 후, 바뀔 가능성이 있는 값에 **코드가 직접 의존하고 있다면**\
그 값은 “코드 내부”가 아니라 \*\*외부 설정(external configuration)\*\*으로 관리하는 것이 올바르다.

* 환경이 다르면 값이 달라진다
* 고객마다 정책이 다를 수 있다
* 인프라 환경이 변경될 수 있다
* 서버 위치/로깅/포트/권한 같은 값은 실행 환경마다 다르다

따라서 설정을 외부로 빼두면 애플리케이션은 더 유연해지고,\
"코드가 환경에 종속되지 않는" 구조를 만들 수 있다.

***

## 🧭 Tip 55 — 외부 설정으로 애플리케이션을 조정할 수 있게 하라

일반적으로 설정 데이터에 들어가는 값들:

* 외부 API/DB 인증 정보
* 로그 레벨, 로그 저장 위치
* 포트 번호, IP 주소, 기계/클러스터 이름
* 특정 환경에서만 적용되는 검증 값
* 배송비 같은 정책성 매개변수
* 지역(국가/언어)에 따른 서식
* 라이선스 키

➡️ **이 값들이 코드 안에 박혀 있다면 애플리케이션은 환경 변화에 매우 취약해진다.**

***

## 🏗️ 정적(static) 설정: 파일이나 DB로 관리

현대 애플리케이션 대부분은 다음 방식 중 하나로 정적 설정을 관리한다:

* YAML (`application.yml`)
* JSON
* Properties 파일
* 데이터베이스 테이블

코틀린 + Spring이라면 자연스럽게 이러한 형태가 된다.

### ✔️ 실전 예 — application.yml

```yaml
app:
  payment:
    base-fee: 2300
    region-fee:
      seoul: 500
      busan: 800

external:
  s3:
    bucket: my-app-bucket
    region: ap-northeast-2
```

### ✔️ 실전 예 — Kotlin 구성 객체로 바인딩

```kotlin
@ConfigurationProperties("app.payment")
data class PaymentProperties(
    val baseFee: Int,
    val regionFee: Map<String, Int>
)
```

이제 서비스 코드는 설정에 종속되지만 **환경 파일만 바꾸면 애플리케이션 전체가 다른 정책으로 작동한다.**

***

## 🚚 서비스형 설정(Configuration-as-a-Service)

기업 규모가 커지면, 설정을 단순 파일/DB로 관리하는 것만으로는 부족해진다.\
이때 **설정 서버(Config Server)** 같은 별도 서비스가 등장한다.

예:

* Spring Cloud Config
* AWS AppConfig
* Consul KV
* etcd
* Git 기반 설정 저장소

#### 장점

* 여러 마이크로서비스가 설정을 공유
* 인증/권한 컨트롤 가능
* 인스턴스 전체 설정을 동시에 변경 가능
* 설정 변경을 실시간으로 동기화

#### 실전 예 — 코틀린에서 설정 서버로부터 값 받기

```kotlin
@Component
class RegionFeeResolver(
    private val paymentProperties: PaymentProperties,
    private val dynamicConfigClient: DynamicConfigClient // 예: AWS AppConfig Wrapper
) {
    fun deliveryFee(region: String): Int {
        val dynamicFee = dynamicConfigClient.getInt("delivery.fee.$region")
        return dynamicFee ?: paymentProperties.regionFee[region] ?: paymentProperties.baseFee
    }
}
```

이제 **코드를 바꾸지 않고도**\
운영자가 AppConfig 값만 바꾸면 배송비 정책 전체가 바뀐다.

***

## 🦤 도도 코드를 작성하지 말라 (Dodo Code)

책에서는 “환경에 적응하지 못해 멸종된 도도새(Dodo)”를 비유로 사용한다.

➡️ 즉, **환경 변화에 반응하지 못하는 코드는 곧 죽은 코드다.**

아래가 도도 코드:

* 설정을 하드코딩한다
* 값이 바뀔 때마다 코드 릴리즈가 필요
* 빌드 → 배포 → 재시작을 반복해야만 한다
* 여러 서비스가 다른 정책을 가져야 하는데 모두 코드로 박혀 있다

현대 백엔드 시스템에서 이런 코드는 생존할 수 없다.

***

## 🧾 실제 코틀린 서비스에서 자주 등장하는 설정 Anti-Pattern

#### ❌ 1. 하드코딩된 URL, 포트, 토큰

```kotlin
val apiUrl = "http://prod-api.myservice.com"
val token = "sk_live_1234..."
```

#### ❌ 2. 환경 별 if/else 분기

```kotlin
if (env == "prod") {
    fee = 5000
} else {
    fee = 1000
}
```

#### ❌ 3. 운영 중 정책 변경을 배포로 해결

→ “배송비 좀 바꿔주세요” → 배포 필요 → 장애 위험 증가

이 모든 문제는 **외부 설정화**로 즉시 해결된다.

***

## 🛡️ 제대로 된 설정 구조를 설계하는 법

### ✔️ 1. 소스코드와 설정을 철저히 분리

설정은 YAML/DB/설정 서버에 두고\
코드에서는 오직 “참조”만 한다.

### ✔️ 2. 모든 실행 환경에서 safe fallback 제공

* 설정 서버가 죽으면 local YAML 값 사용
* 존재하지 않는 키면 default 값 사용

### ✔️ 3. 설정 변경은 코드 배포 없이 반영

* 설정 서버
* 런타임 refresh
* 메시지 기반 변경 알림 (Kafka/SNS)

### ✔️ 4. 설정도 테스트해야 한다

설정을 잘못 넣으면 서비스 전체가 망가질 수 있다.

***

## 🧪 실전 예: 정책을 외부 설정으로 빼기 (코틀린)

#### 기존 나쁜 코드

```kotlin
class DeliveryFeeService {
    fun feeFor(region: String): Int {
        return when (region) {
            "seoul" -> 5000
            "busan" -> 8000
            else -> 3000
        }
    }
}
```

➡️ 정책이 바뀔 때마다 반드시 배포가 필요하다.

#### 개선된 코드

```kotlin
@Service
class DeliveryFeeService(
    private val paymentProperties: PaymentProperties
) {
    fun feeFor(region: String): Int =
        paymentProperties.regionFee[region] ?: paymentProperties.baseFee
}
```

**환경 파일만 바꾸면 끝.**

***

## 📌 결론: 설정을 코드 밖에 두어라

Topic 32 전체의 핵심 메시지는 단 하나다.

> **코드는 로직을 담고, 설정은 외부에서 조정된다.**\
> **두 역할을 섞지 마라.**

#### ✔ 설정을 외부로 빼면…

* 배포 없이 정책 변경 가능
* 고객/환경별 동작 분리
* 유지보수 비용 절감
* 개발과 운영 분리
* 재사용성, 이식성 증가

#### ✔ 상시 변화하는 현대 서비스에는 필수다

* 마이크로서비스
* 서버리스
* 글로벌 서비스
* 고객별 정책
* A/B 테스트
* 환경별 다양한 런타임

***
