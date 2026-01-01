# Chapter 8. Bytecode Executor Engine (1/2)

## 런타임 스택 프레임 구조

자바 가상 머신은 메서드를 가장 기본적인 실행 단위로 사용. 메서드 호출과 실행을 뒷받침하는 내부 데이터 구조로 스택 프레임을 이용합니다. 스택 프레임(Stack Frame)은 **메서드 호출 시점에 생성**되는 단위입니다. 스레드마다 하나의 JVM 스택이 있고, 그 안에 여러 개의 스택 프레임이 생성됩니다.

스텍 프레임은 가상 머신 런타임 데이터 영역에 있는 가상 머신 스택의 요소. 여기에는,

* 메서드의 지역 변수 테이블: 메서드의 매개변수와 지역 변수를 저장하는 공간
* 피연산자 스택: 바이트코드 명령들이 계산을 수행할 때 사용하는 스택
* 동적 링크: 현재 메서드가 속한 클래스의 상수 풀(constant pool)에서 **현재 실행 중인 메서드**의 참조 정보를 담고 있음
* 반환 주소: 현재 메서드 실행이 끝난 후 **어디로 돌아가야 할지**에 대한 정보
* 및 몇가지 추가 정보: 예외 처리 정보, 디버깅용 테이블 등 JVM 구현체에 따라 부가적으로 포함될 수 있는 정보들

가 담깁니다.



#### Code 속성

`Code` 속성은 JVM 바이트코드에서 메서드마다 하나씩 존재하는 속성(Attribute)입니다. 이 속성은 다음과 같은 중요한 정보를 담고 있습니다.

* `max_locals`: 지역 변수 테이블의 크기 (int, float, 참조 등 포함)
* `max_stack`: 피연산자 스택의 최대 깊이
* `code`: 실제 바이트코드 (명령어 시퀀스)
* `exception_table`, `LineNumberTable`, `LocalVariableTable` 등의 부가 정보

**`Code` 속성은 메서드마다 하나씩 생성되고, 해당 메서드의 실행에 필요한 메모리 정보 등을 담고 있는 바이트코드의 메타데이터**입니다.

스택 프레임에 할당해야하는 메모리 크기는, **컴파일 시점에 생성되는 `Code` 속성의 `max_stack(피연산자 스택에 필요한 깊이)`과 `max_locals(지역 변수 테이블의 크기)` 값에 따라 결정**됩니다. 그리고 런타임에는 이 값에 따라 스택 프레임을 생성하게 됩니다.

***

스레드의 호출 체인은 매우 길 수 있습니다. **자바 프로그램 관점**에서는 특정 시점에 한 스레드의 호출 스택에 쌓여 있는 메서드는 모두 실행 중인 상태입니다. 다만, **실행 엔진 관점**에서는, 맨 위에 있는 메서드만 실행 중이며, 스택 맨 위에 있는 스택 프레임만 유효합니다. (이를 **스택 프레임**이라고 하고, 대변하는 메서드를 **현재 메서드**라고 합니다)

<figure><img src="../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

### 지역 변수 테이블

메서드 매개 변수와 메서드 안에서 정의된 지역 변수를 저장하는 공간. 지역 변수 테이블의 최대 용량은 Code 속성 중 max\_locals 항목에 기록합니다.

지역 변수 테이블의 용량 기준은 가장 작은 단위인 변수 슬롯이고, 슬롯 하나가 boolean, byte, char, short, int, float, 참조 타입, returnAddress를 저장할 수 있어야합니다. 따라서 변수 슬롯 하나의 크기는 32비트 또는 64비트로 구현됩니다.

* 참조 타입: 객체 인스턴스를 가리키는 참조. (JVM은 자바 객체 그 자체를 스택에 저장하지 않고, **힙 메모리에 있는 객체를 참조하는 주소값만 스택에 저장**합니다.)
* returnAddress: 바이트코드 명령어 jsr, jsr\_w, ret에 다른 바이트코드 명령어의 주소를 알려 주는 용도로 쓰입니다. 지금은 잘 쓰이지 않습니다. (옛날 JVM에서는 **`finally` 블록**을 구현하기 위해 바이트코드에서 `jsr`(jump to subroutine), `ret` 명령어를 사용했습니다. 이 때 **`jsr` 명령어로 분기된 후, 되돌아갈 주소를 지역 변수 테이블의 슬롯에 `returnAddress`로 저장**했습니다.)



자바 가상 머신은 지역 변수 테이블을 인덱스 방식으로 사용합니다. 인덱스 값은 컴파일 시점에 javac 컴파일러가 계산해서, 클래스 파일의 바이트코드 안에 직접 기록됩니다.

인덱스 값의 범위는 0부터 지역 변수 테이블이 담을 수 있는 변수 슬롯의 최대 개수까지입니다.  32비트 변수에 접근할 시 인덱스N은 N번째 변수 슬롯을 뜻하고, 64비트 변수에 접근할떄는 N번쨰와 N+1번째 변수 슬롯을 동시에 사용한다는 뜻입니다. 만약 자바 가상 머신이, 두 슬롯 중, 한 슬롯에만 접근하려고하면, 클래스 로딩 중 검증 단계에서 예외를 던집니다.

<details>

<summary>Q. 앞에 64비트 변수가 존재했으면, 인덱스N은 N+1과 N+2가 아닌가?</summary>

A. **JVM의 지역 변수 테이블은 "슬롯 인덱스를 명확히 지정"하기 때문에**, 앞에 뭐가 있었든 **다음 사용 가능한 인덱스 번호를 기준으로** 정확히 슬롯을 배정합니다. 그래서 "앞에 뭐가 있었는지"는 자동으로 계산되지 않고, **컴파일 타임에 고정된 슬롯 번호로 직접 지정되기 떄문에, 정확합니다.**

</details>



메서드 호출 시 매개 변수들도 지역 변수 테이블을 통해 전달됩니다. 아래는 인스턴스 메서드가 호출될 때 지역 변수 테이블에 추가되는 변수들의 순서를 보여줍니다.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 인덱스 0: 메서드가 속한 객체의 인스턴스의 참조.(this가 가능한 이유도 이때문입니다)
* 인덱스 1-N: 매개 변수 테이블의 갯수만큼 순서대로 할당 됩니다.
* 인덱스 N+1-: 메서드 본문에서 정의한 지역 변수들이 정의 순서와 유효 범위에 따라 할당됩니다.



스택 프레임이 메모리를 절약하기 위해 변수 슬롯을 재사용하기도 합니다. 현재 바이트코드를 가리키는 프로그램 카운터의 값이 변수의 유효 범위를 벗어나면 해당 변수를 담고 있던 변수 슬롯을 다른 변수를 담는 데 재사용할 수 있습니다. 하지만 이러한 방법에는 부작용이 있습니다.

1. 상황에 따라 시스템의 GC 동작에 영향을 줄 수 있습니다.

```java
byte[] placeholder = new byte[64 * 1024 * 1024]; // 64MB
System.gc();
```

* **결과**: `placeholder`는 여전히 **유효한 슬롯에 남아 있어** GC가 회수하지 않음
* **GC 로그**: 67M -> 65M (해제 안 됨)

***

```java
{
    byte[] placeholder = new byte[64 * 1024 * 1024];
}
System.gc();
```

* **결과**: 범위를 벗어났지만, **슬롯은 여전히 점유**하고 있어 GC가 회수하지 않음
* 이유: JVM이 **변수 범위가 끝났다고 해서 슬롯 값을 null로 지우진 않음**

***

```java
{
    byte[] placeholder = new byte[64 * 1024 * 1024];
}
int a = 0;  // 새로운 변수 추가
System.gc();
```

* **결과**: `int a`가 placeholder가 쓰던 **슬롯을 덮어씀**
* → placeholder 참조가 사라졌고 GC 가능해짐
* **GC 로그**: 67M -> 9M (메모리 회수 성공)

***

```java
int a;
System.out.println(a);
```

* **결과**: `a`를 선언만 하고 초기화하지 않으면 **컴파일 에러가 발생함**
* 이유: **지역 변수는 자동 초기화되지 않음**

***

* 지역 변수 범위를 벗어났다고 해서 참조가 바로 사라지지 않습니다.
* 슬롯 재사용을 통해 참조를 덮어써야 GC가 가능합니다.
* null 할당을 강제하는 규칙은 JVM 명세에 없습니다. null처리는 명시적이지만 바람직한 방식은 아닙니다.
* JIT컴파일러가 최적화 시점에 null로 덮어 쓸 수도 있지만, 실행 환경에 따라 다릅니다.

가장 좋은 방법은 범위를 적절히 나눠 슬롯 재사용을 유도하는 것입니다.

#### 실무 예시

#### BEFORE — GC가 회수 못하는 경우

```java
public void handleRequest() {
    // 1. 클라이언트 요청 바디를 대용량으로 역직렬화
    LargeDto largeDto = parseLargeRequest(); // 100MB DTO

    // 2. 필요한 정보만 뽑아서 처리
    String userId = largeDto.getUserId();
    processUser(userId);

    // 3. 이후에는 largeDto는 사용하지 않지만,
    // → 여전히 지역 변수 슬롯에 남아 있어서 GC 회수 불가

    doSomethingElse(); // 이 작업에서 또 메모리 할당 발생 → GC 압박
}
```

#### 문제 요약:

* `largeDto`는 더 이상 사용되지 않지만 여전히 **슬롯을 점유**
* GC 입장에선 아직 살아있는 객체로 판단 → **회수 안됨**
* → 이 메서드가 자주 호출되면 **GC 빈도 증가 / OutOfMemory 가능성 증가**

***

AFTER — DTO 범위를 블록으로 제한해 GC 유도

```java
public void handleRequest() {
    String userId;

    {
        // 1. DTO 사용을 블록 스코프로 제한
        LargeDto largeDto = parseLargeRequest();
        userId = largeDto.getUserId();
        // largeDto는 여기까지만 사용되고
        // → 블록 끝나면서 스코프 밖으로 나감
    } // largeDto는 더 이상 참조되지 않음 → GC 대상

    // 2. 이후 메모리 많이 쓰는 로직 수행
    doSomethingElse(); // GC가 largeDto를 회수하고 부담 줄어듦
}
```

개선 요약:

* `largeDto`의 **유효 범위를 줄여**서 지역 변수 슬롯에서 **덮어쓰거나 해제 가능**
* GC가 해당 객체를 **빠르게 회수할 수 있어** 성능 및 안정성 향상
* 실무에서 특히 **대용량 요청 처리, 이미지/JSON 처리, 파일 업로드 등**에 유효

***

### 피연산자 스택

* 피연산자 스택은 후입선출(LIFO) 스택입니다.&#x20;
* 스택의 최대 깊이도 컴파일시 Code 속성의 max\_stacks 항목에 기록됩니다.
* long과 double을 포함한 모든 자바 데이터 타입을 담을 수 있습니다. (32비트 데이터타입의 스택 용량은 1, 64비트는 2)
* 메서드 실행중 피연산자 스택의 깊이가 max\_stacks에 설정된 값을 절대 초과하지 않도록 합니다.

메서드가 실행될때 해당 메서드의 피연산자 스택은 비어있습니다. 그리고 실행하는 동안 다양한 바이트코드 명령어가 스택에 내용을 쓰거나(push) 읽어옵니다(pop).



피연산자 스택에 있는 데이터 타입은 바이트코드 명령어의 순서와 정확히 일치해야합니다. (이는 컴파일러가 컴파일시 보장하며, 클래스 검증 단계에서 데이터 흐름을 분석해 또 한번 검증합니다.)

* iadd를 실행할때 스택 맨위에 있는 두 원소는 같은 데이터 타입이어야합니다(int).



개념 모델에서 서로 다른 메서드의 가상 머신 스택에 있는 스택 프레임들은 완전히 독립적이지만, 최적화 과정을 통해 스택 프레임들을 아래와 같은 이유들로 부분적으로 겹쳐 사용하기도 합니다.

* 공간 적약
* 메서드 호출 시 매개 변수로 전달할 데이터를 복사할 필요가 없음.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

### 동적 링크

현재 메서드가 속한 클래스의 상수 풀(constant pool)에서 **현재 실행 중인 메서드**의 참조 정보를 담고 있습니다. JVM에서 메서드가 사용하는 외부 메서드나 필드 참조는 런타임 상수 풀을 통해 관리됩니다.

* 메서드에서 사용하는 외부 참조는 런타임 상수 풀에 저장되어 있습니다.
* 각 메서드의 스택 프레임은 실행 중에 이 상수 풀 항목을 참조하는 방식으로 구성됩니다.
* 이 참조 구조를 동적 링크(dynamic linking)이라고 합니다.

실행 시점 참조(resolve): 메서드가 실행되면서 클래스 로딩 / 실행 시점에 처음 사용될 때, 런타임 상수 풀의 항목이 실제 메모리 주소로 변환됩니다.

***

### 반환 주소

현재 메서드 실행이 끝난 후 **어디로 돌아가야 할지**에 대한 정보입니다.

메서드가 종료되는 방식은 크게 2가지입니다.

1. 정상 종료(Normal Method Invocation Completion)
   1. return 바이트코드를 만나서 메서드가 정상적으로 끝나는 경우
   2. 호출자로 값이 있을 경우 반환값도 전달됩니다.
2. 비정상 종료 (Abrupt Method Invocation Completion)
   1. 메서드 실행 도중 예외가 발생했을 때
   2. 예외가 메서드 내부에서 처리되지 않으면 상위 호출 스택으로 전파됩니다.
   3. 이때는 반환값이 전달되지 않습니다.

메서드가 종료된 후, JVM은

* 호출자의 지역 변수 테이블과 피연산자 스택을 복원합니다
* 반환값이 있다면 호출자 쪽 스택 프레임이 push합니다.
* 프로그램 카운터를 참고, 조정하여, 호출 명령 다음 명령을 실행합니다.
* 메서드 종료는 스택 프레임 제거와 같은 작업입니다.

***

## 메서드 호출

메서드 호출은 메서드 본문 코드를 실행하는 일과 **다릅니다.** 메서드 호출 단계에서 수행하는 일은 호출한 메서드의 버전을 선택하는 것.&#x20;

* 예: a클래스와 a클래스를 상속한 클래스 b가 모두 hello() 메서드를 가지고 있을때.
* a.hello()를 실행하면 어떤 hello()를 실행할지 해석해야합니다.

그렇기 때문에, 때에 따라 클래스 로딩 시점 또는 런타임에 대상 메서드의 직접 참조를 알아내야합니다.



### 해석

메서드 호출의 해석은 동적과 정적으로 나뉩니다. 자바의 메서드 호출은 대부분 클래스 파일의 상수 풀에 있는 심볼릭 참조로 이루어집니다. 클래스 로딩 시점 또는 런타임 시점에 이 참조를 실제 메서드로 변환(resolve)해야합니다. 메서드 호출의 해석 방식은 정적과 동적으로 나뉩니다.

#### 정적

* 컴파일 타임에 호출 대상이 결정됩니다
* 런타임에 다른 메서드로 바뀔 수 없습니다.
  * static
  * private
  * 생성자
  * 부모 클래스 메서드
  * final 메서드

#### 동적

* 런타임에 호출 대상이 결정됩니다.
* 주로 오버라이딩, 다형성이 작동하는 상황입니다.
  * 일반 인스턴스 메서드
  * 인터페이스 메서드

***

### 디스패치

* 메서드 호출 시점에 어떤 메서드가 실제 실행될지를 결정하는 과정
* 정적 디스패치(static dispatch): 컴파일 타임에 메서드가 결정됨
* 동적 디스패치(dynamic dispatch): 런타임에 실제 객체 타입을 보고 메서드가 결정됨



#### 오버로딩과 정적 디스패치

자바에서 오버로딩된 메서드는 **컴파일 타임에 호출될 메서드가 정적 타입을 기준으로 선택되며**, 이 과정을 **정적 디스패치**라고 부릅니다.

* 오버로딩은 정적 디스패치 방식으로 동작
* 오버로딩된 메서드는 매개변수의 타입과 개수에 따라 여러 개가 존재
* 이 중 어떤 메서드가 호출될지는 변수의 정적 타입(선언된 타입)을 기준으로 컴파일 타임에 결정됩니다.

```java
public class StaticDispatch {
    static abstract class Human{}
    static class Man extends Human{}
    static class Woman extends Human{}
    
    public void sayHello(Human guy) System.out.println("guy");
    public void sayHello(Man man) System.out.println("man");
    public void sayHello(Woman woman) System.out.println("woman");
    
    static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        StaticDispatch sr = new StaticDispatch();
        sr.sayHello(man) // guy
        sr.sayHello(woman) // woman
```

* 비록 `man`과 `woman`의 실제 타입은 다르지만, 정적 타입이 `Human`이므로 `sayHello(Human)`만 호출됩니다.
* 정적 타입은 컴파일 타임에 결정되며, 오버로딩 메서드는 이 타입을 기준으로 정해집니다.
* 이 메서드는 `invokevirtual`로 컴파일되지만, 어떤 버전이 호출될지는 이미 **javac가 결정한 것**



**정적 타입 vs 실제 타입**

* 정적 타입 (Static type, Apparent type): 선언 시 명시한 타입
* 실제 타입 (Actual type): new 키워드로 생성된 객체의 실제 클래스 타입
* 오버로딩에서는 정적 타입 기준으로 메서드를 결정
* 오버라이딩에서는 실제 타입 기준으로 결정



적합한 오버로딩 메서드 선택 기준은 아래와 같습니다.

* **정확한 타입 매칭** 우선 → `char` 있으면 우선 사용
* **자동 형변환**
  * `char` → `int` → `long` → ...
  * 오토박싱: `char` → `Character`
  * 인터페이스 변환: `Character` → `Serializable`, `Comparable<Character>` 등
* **가장 구체적인 타입 선택 (가까운 상속 관계 우선)**

***

#### 동적 디스패치

자바에서 오버라이딩된 메서드는 객체의 **실제 타입에 따라 런타임에 결정되며**, 이 과정을 동적 디스패치(dynamic dispatch)라고 부릅니다.

* 메서드 호출 대상은 실행 시간(runtime)에 결정됩니다
* JVM의 **`invokevirtual` 명령어**를 통해 이루어집니다.
* 오버라이딩된 메서드를 **객체의 실제 타입**에 따라 선택합니다
* 메서드는 다형성 지원합니다 (필드는 정적 디스패치로 다형성 미지원)

```java
public class DynamicDispatch {
    static abstract class Human {
        protected abstract void sayHello();
    }
    
    static class Man extends Human {
        @Override
        protected void sayHello() {
            System.out.println("Man");
        }
    }    
    static class Woman extends Human {
        @Override
        protected void sayHello() {
            System.out.println("Woman");
        }
    }
    
    public static void main(String[] args) {
        Human man = new Man();
        Human woman = new Woman();
        
        man.sayHello(); // "man"
        woman.sayHello(); // "woman"
        man = new Woman(); // 실제 타입 = woman
        man.sayHello(); // "woman"
```



#### 정적 타입 vs 실제 타입

* 정적 타입 (Static type, Apparent type): 선언 시 명시한 타입
* 실제 타입 (Actual type): new 키워드로 생성된 객체의 실제 클래스 타입
* 동적 디스패치 대상: **실제 타입 기준**으로 메서드 호출



동작 방식: invokevirtual #13 (Method Human.sayHello:()V )

* 호출 당시 스택에서 **this 객체 참조**를 꺼냄
* 객체의 **실제 클래스 타입을 확인**
* 메서드 테이블(vtable)을 따라 **적절한 오버라이딩된 메서드** 탐색 및 실행

이 구조로 인해 자바는 런타임 다형성을 구현할 수 있습니다.



### 바이트코드 흐름 분석

```java
Human man = new Man();
Human woman = new Woman();

man.sayHello();
woman.sayHello();
```

```
바이트코드 흐름:
aload_1
invokevirtual #13  // Human.sayHello() → 실제 타입: Man
aload_2
invokevirtual #13  // Human.sayHello() → 실제 타입: Woman
```

바이트코드 상으로는 동일한 메서드 호출처럼 보이지만, 런타임 시점의 객체 타입이 다르기 때문에 **다른 메서드가 호출됩니다.**



**여기서 필드는 다형성을 미지원합니다 (정적 디스패치)**

```java
public class FieldHasNoPolymorphic {
    static class Father {
        public int money = 1;

        public Father() {
            money = 2;
            showMeTheMoney();
        }

        public void showMeTheMoney() {
            System.out.println("I am a Father, I have $" + money);
        }
    }

    static class Son extends Father {
        public int money = 3;

        public Son() {
            money = 4;
            showMeTheMoney();
        }

        @Override
        public void showMeTheMoney() {
            System.out.println("I am a Son, I have $" + money);
        }
    }

    public static void main(String[] args) {
        Father guy = new Son();
        System.out.println("This guy has $" + guy.money);
    }
}

```

* `Father` 생성자에서 호출한 `showMeTheMoney()`는 **오버라이딩된 Son의 메서드**지만, 이 시점에는 **Son.money는 아직 초기화되지 않았기 때문에 0**이 출력됩니다
* `Son` 생성자에서 다시 한 번 `showMeTheMoney()`를 호출하면 **money = 4**가 출력됩니다
* 마지막 `guy.money`는 `Father` 타입의 필드이기 때문에 **정적 디스패치**로 `Father.money`인 2가 출력됩니다



#### 단일 디스패치와 다중 디스패치

디스패치는 **메서드 호출 시점에 어떤 버전의 메서드를 실행할지 결정하는 과정**입니다. 디스패치를 할 때 고려하는 기준의 개수에 따라, 단일, 다중 디스패치로 나뉩니다.

* **단일 디스패치(Single Dispatch)**
  * 하나의 기준 (보통 수신 객체의 타입)
  * 수신 객체의 실제 타입만 기준으로 메서드 선택
* **다중 디스패치(Multiple Dispatch)**
  * 둘 이상의 기준 (예: 수신 객체 + 인자 타입)
  * 수신 객체뿐 아니라 인자들의 정적 타입도 함께 고려

(자바는 **정적 타입 기반 + 단일 디스패치 언어**로서, 수신 객체는 런타임에 결정하지만, 인자 타입은 컴파일 시점에 결정되기 때문에 **다중 디스패치를 지원하지 않습니다.)**

```java
public class Dispatch {
    static class QQ {}
    static class _360 {}

    public static class Father {
        public void hardChoice(QQ arg) {
            System.out.println("Father chose a qq");
        }

        public void hardChoice(_360 arg) {
            System.out.println("Father chose a 360");
        }
    }

    public static class Son extends Father {
        @Override
        public void hardChoice(QQ arg) {
            System.out.println("Son chose a qq");
        }

        @Override
        public void hardChoice(_360 arg) {
            System.out.println("Son chose a 360");
        }
    }

    public static void main(String[] args) {
        Father father = new Father();
        Father son = new Son();

        father.hardChoice(new _360());  // 출력: Father chose a 360
        son.hardChoice(new QQ());       // 출력: Son chose a qq
    }
}

```

```java
class QQ {}
class _360 {}

class Father {
    public void hardChoice(QQ arg) { ... }
    public void hardChoice(_360 arg) { ... }
}

class Son extends Father {
    public void hardChoice(QQ arg) { ... }
    public void hardChoice(_360 arg) { ... }
}

```

#### 결과

```
Father chose a 360
Son chose a qq
```

* son 변수는 실제 타입은 `Son`이지만, 정적 타입은 `Father`
* 메서드 선택은 **두 기준이 필요함**
  * 수신 객체의 타입 (정적 or 실제)
  * 전달되는 인자의 타입 (정적 타입)
* 하지만 자바에서는 **수신 객체는 런타임에, 인자 타입은 컴파일 타임에** 결정됨
* 즉, **수신 객체는 동적 디스패치, 인자 타입은 정적 디스패치**



이를 바탕으로 자바는 단일 디스패치 언어인 것을 확인할 수 있습니다.

* **Son** 객체의 **실제 타입**에 따라 어떤 클래스의 메서드를 실행할지는 결정됨
* 하지만 어떤 **오버로딩 메서드**를 선택할지는 **정적 타입(컴파일 타임)** 기준
* 따라서 두 조건을 모두 동적으로 판단하지 않으므로 **다중 디스패치가 아님**



#### **가상 머신의 동적 디스패치 구현**

자바 가상 머신은 **vtable 구조**를 이용해 런타임에 빠르고 정확한 **동적 디스패치**를 구현합니다. 이를 통해 자바는 다형성과 오버라이딩을 지원하면서도 **성능 저하 없이 효율적인 메서드 호출**이 가능하게됩니다.

가상 메서드 테이블(vtable)

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

* **vtable**은 객체의 클래스에 대응되는 **메서드 테이블**
* 테이블에는 각 메서드의 \*\*시작 주소(포인터)\*\*가 저장됨
* 실행 시점에 해당 메서드의 실제 주소를 **vtable에서 직접 참조**하여 빠르게 호출



구현 방식:

* 자바 클래스가 로딩될 때, **클래스 계층도 분석**
* 오버라이딩된 메서드는 **하위 클래스에서 덮어씀**
* 오버라이딩되지 않은 메서드는 **상위 클래스의 메서드 주소를 상속**
* 각 클래스는 독립된 vtable을 가짐 (shared 불가)
* 자바 객체는 자신이 속한 클래스의 **vtable 포인터를 가지고 있음**

