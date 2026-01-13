---
description: InstanceOf의 단점
---

# problems of InstanceOf

## 타입 종속성 증가

코드가 상위 타입이 아니라 구체 타입에 의존하게 되는 문제\
원래 객체지향의 목표는 "구현이 아닌 추상화에 의존".

```java
void process(Shape shape) {
    if (shape instanceof Circle) {
        Circle c = (Circle) shape;
        c.drawCircle();
    } else if (shape instanceof Rectangle) {
        Rectangle r = (Rectangle) shape;
        r.drawRectangle();
    }
}
```

여기서 종속되는 것들:

* Circle
* Rectangle

겉으로보면 process(Shape shake)는  Shape에만 의존하는 것처럼 보이지만, 실제로는 그렇지 않다.

process()가 실제로 알고 있는 것들은:

* 구체 타입 목록
  * Circle
  * Rectangle
* 각 타입의 전용 메서드
  * drawCircle()
  * drawRectangle()
* 타입 <-> 행동 매핑 규칙
  * Circle -> drawCircle()
  * Rectangle -> drawRectangle()

이뜻은, process()는 단순히 Shape를 아는것 뿐만 아니라,\
'이 객체가 어떤 구체 타입인지에 따라 어떤 메서드를 호출해야하는지' 분기 규칙을 두고 있습니다.

Shape에 제대로 의존하려면,

* Shape가 가진 추상적인 행동만 알고
* 구체 구현은 전혀 몰라도 되는 상태를 의미.



실제로 Shape에만 의존하는 코드:

```java
interface Shape {
    void draw();
}

class Circle implements Shape {
    public void draw() {
        drawCircle();
    }
}

class process(Shape shape) {
    shape.draw();
}
```

이 경우 process()가 아는 것

* shape
* draw()

process가 모르는 것

* circle? rectangle? 어떤거지?
* drawCircle(), drawRectangle()같은 세부 구현



타입 종속성이 증가하게되면, 새로운 타입 추가시, else if (shape instanceof Triangle) 같은 코드가 추가되고, 확장에 닫혀있지 않습니다.\
또한 Circle 메서드 시그니처 변경이나, Rectangle을 제거했을때 연쇄적으로 수정이 발생합니다.



## 다형성의 저해

객체에게 행동을 위임하지 않고, 호출자가 타입에 따라 행동을 결정하는 문제.

```java
shape.draw()
```

"어떻게" 그릴지는 객체가 결정하고 호출자는 shape만 알고 있어야함.

```java
void draw(Shape shape) {
    if (shape instanceof Circle) {
        ((Circle) shape).drawCircle();
    }
}
```

제어의 역전 실패:호출자가 행동을 결정하게 됨.



#### 왜 다형성을 저해할까?

#### 1. 메시지 → 타입 분기

| 정상적인 다형성 | instanceof |
| -------- | ---------- |
| 메시지 전달   | 타입 검사      |
| 객체 책임    | 호출자 책임     |
| 확장 안전    | 수정 필요      |

***

#### 2. 리스코프 치환 원칙(LSP) 위반 신호

```java
void move(Bird bird)
```

* 이 함수는 Bird면 충분해야 함

```java
if (bird instanceof Penguin) ...
```

➡ 사실은 **Bird가 충분하지 않다는 뜻**

***

#### ❌ 대표 예시

```java
class Bird {
    void fly() {}
}
```

```java
class Penguin extends Bird {
    @Override
    void fly() {
        throw new UnsupportedOperationException();
    }
}
```

➡ 그래서 instanceof 필요해짐\
➡ 설계가 이미 잘못됨

***

#### ✅ 대안: 역할 분리

```java
interface Flyable {
    void fly();
}
```

```java
interface Swimmable {
    void swim();
}
```

➡ 다형성 복원\
➡ instanceof 제거





## 복잡한 논리와 가독성 저하

조건 분기 증가로 인해 코드가 읽기 어렵고, 수정 시 실수하기 쉬워지는 문제

#### ❌ instanceof 체인의 전형

```java
void handle(Event event) {
    if (event instanceof UserCreated) {
        ...
    } else if (event instanceof UserDeleted) {
        ...
    } else if (event instanceof UserUpdated) {
        ...
    } else if (event instanceof UserBlocked) {
        ...
    }
}
```

#### 문제점

* 분기 길어짐
* 흐름 파악 어려움
* 변경 시 누락 가능

***

#### 실무에서 자주 생기는 문제

#### ❌ 순서 의존 버그

```java
if (event instanceof UserEvent) {
    ...
} else if (event instanceof UserCreated) {
    ...
}
```

➡ `UserCreated`는 절대 도달하지 않음

***

#### ❌ 조건 + 캐스팅 반복

```java
if (x instanceof A) {
    A a = (A) x;
    ...
}
```

* boilerplate 증가
* 실수 여지 증가

***

#### ✅ 대안 1: 다형성으로 분기 제거

```java
interface Event {
    void handle();
}
```

```java
event.handle();
```

***

#### ✅ 대안 2: Kotlin sealed class (의도된 분기)

```kotlin
sealed class Event
```

```kotlin
when (event) {
    is UserCreated -> ...
    is UserDeleted -> ...
}
```

* 컴파일러가 모든 분기 강제
* 가독성 ⬆️
* 안정성 ⬆️

***

#### 4️⃣ 세 단점의 관계 한눈에 보기

```
instanceof 사용
   ↓
구체 타입 인식 필요
   ↓
타입 종속성 증가
   ↓
호출자가 행동 결정
   ↓
다형성 붕괴
   ↓
if-else 증가
   ↓
복잡도 & 가독성 저하
```

***



## 면접용 요약 문장

#### 한 줄 요약

> **instanceof는 호출자를 구체 타입에 종속시키고,**\
> **객체가 가져야 할 행동 결정권을 빼앗아**\
> **다형성을 무너뜨리며,**\
> **그 결과 조건 분기가 늘어나 코드가 복잡해진다.**

#### 짧은 버전

* 타입 종속성 증가 → 확장에 취약
* 다형성 저해 → 객체 책임 붕괴
* 복잡성 증가 → 가독성·유지보수 악화
