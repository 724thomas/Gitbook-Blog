---
description: 전략 패턴
---

# Strategy Pattern

## 정의

실행 중에 알고리즘 전략을 선택하여 객체 동작을 실시간으로 바뀌도록 할 수 있게 하는 행위 디자인 패턴입니다. 어떤 일을 수행하는 알고리즘이 여러개 일때, 동작들을 미리 전략으로 정의함으로써 손쉽게 전략을 교체할 수 있는, 알고리즘 변형이 빈번하게 필요한 경우에 적합한 패턴입니다.

<figure><img src="../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

## 전략 패턴 구조

<figure><img src="../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

* 전략 알고리즘 객체들: 알고리즘, 행위, 동작을 객체로 정의한 구현체
* 전략 인터페이스: 모든 전략 구현제에 대한 공용 인터페이스
* 컨텍스트: 알고리즘을 실행해야 할 때마다 해당 알고리즘과 연결된 전략 객체의 메소드를 호출
* 클라이언트: 특정 전략 객체를 컨텍스트에 전달 함으로써 전략을 등록하거나 변경하여 전략 알고리즘을 실행한 결과를 가져옵니다.



## 전략 패턴 흐름

<figure><img src="../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

```java
// 전략(추상화된 알고리즘)
interface IStrategy {
    void doSomething();
}

// 전략 알고리즘 A
class ConcreteStrateyA implements IStrategy {
    public void doSomething() {}
}

// 전략 알고리즘 B
class ConcreteStrateyB implements IStrategy {
    public void doSomething() {}
}
    
class Context {
    IStrategy Strategy; // 전략 인터페이스를 합성(composition)
	
    // 전략 교체 메소드
    void setStrategy(IStrategy Strategy) {
        this.Strategy = Strategy;
    }
	
    // 전략 실행 메소드
    void doSomething() {
        this.Strategy.doSomething();
    }
}
```



## 전략 패턴과 OOP

전략 패턴은:

* 동일 계열의 알고리즘군을 정의합니다 (전략 구현체로 정의)
* 각각의 알고리즘을 캡슐화 합니다 (인터페이스로 추상화)
* 이들을 상호 교환이 가능하게 합니다 (Composition으로 구성)
* 알고리즘을 사용하는 클라이언트와 상관없이 독립적으로 동작한다. (컨텍스트 객체 수정없이)
* 알고리즘을 다양하게 변경할 수 있게 한다. (메소드를 통해 전략 객체를 실시간으로 변경)
