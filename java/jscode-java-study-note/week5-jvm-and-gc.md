---
description: JVM & GC
---

# Week5 JVM & GC

## &#x20;JVM 구조에 대해 설명해주세요.

크게 클래스 로더, 메모리영역, 실행엔진이 있습니다.

클래스 로더

* .class 파일을 읽어와 메서드 영역에 로드. (프로그램 실행중에 필요한 클래스를 클래스 로더가 동적으로 로딩 가능)
* 클래스 로딩은 로딩, 링크, 초기화 단계로 나뉩니다.

메모리 영역

* 메서드: 클래스와 메서드, 정적 변수, 상수 풀 등의 정보를 저장
* 힙: 동적으로 생성된 객체와 배열을 저장
* 스택: 각 스레드마다 할당. 메서드 호출 시 스택 프레임과 지역 변수를 저장
* PC 레지스터: 각 스레드마다 할당. 현재 실행 중인 명령어의 주소를 저장
* 네이티브 메서드 스택: 자바 외부의 네이티브 메서드 호출 시 사용

실행 엔진 (자바 바이트 코드를 실제 머신 코드로 변환)

* 인터프리터
* JIT 컴파일러
* 카비지컬렉터

JNI (C, C++ 등의 네이티브 코드 호출이 가능한 인터페이스)

Native Method Library (JNI를 통해 호출되는 네이티브 메서드를 포함하는 라이브러리)





## 클래스 로더가 뭔가요? (인스턴스와 관련X)

* JVM의 구성요소중 하나.
* 자바 애플리케이션이 실행되는 동안 필요한 클래스 파일(.class)을 런타임에 동적으로 로드하는 역할을 담당. (실제로 사용될 때만 클래스를 런타임에 메모리로 올림)
* 클래스 로더는 크게 세 단계로 구성되며 각각 로딩, 링크, 초기화입니다.
  * 로딩: .class파일을 찾아서 메모리에 올립니다
  * 링킹: 클래스의 바이트코드가 유효한지 확인 및 JVM이 이해할 수 있는 구조로 변환
  * 정적 초기화 블록 및 정적 필드를 초기화.
* 여러 종류의 로더가 존재합니다. (부트스트랩 클래스 로더, 확장 클래스 로더, 애플리케이션 클래스 로더)



## JVM 메모리 구조를 자세히 설명해주세요.

JVM의 메모리 구조는 크게 5가지 영역으로 나눌 수 있습니다:

1. **메서드 영역 (Method Area)**: 클래스 로더가 로드한 **클래스 메타데이터(클래스 이름, 메서드, 필드 정보)**, **정적 변수(static field)**, **상수 풀** 및 **메서드의 바이트코드**가 저장됩니다. 이 영역은 모든 스레드가 공유하며, 클래스와 관련된 정보가 여기에 저장됩니다.
2. **힙 영역 (Heap Area)**: **객체와 배열**이 동적으로 생성되는 메모리 공간입니다. JVM의 \*\*가비지 컬렉터(GC)\*\*가 관리하는 영역으로, 더 이상 참조되지 않는 객체는 가비지 컬렉션을 통해 제거됩니다. 모든 스레드가 공유하는 공간입니다.
3. **스택 영역 (Stack Area)**: **각 스레드마다 독립적으로 할당**되며, 메서드가 호출될 때마다 \*\*스택 프레임(Stack Frame)\*\*이 생성됩니다. 각 스택 프레임에는 **메서드의 지역 변수, 매개변수, 리턴 값** 및 **임시 계산 결과**가 저장됩니다. 메서드가 종료되면 해당 스택 프레임은 제거됩니다.
4. **PC 레지스터 (Program Counter Register)**: 각 스레드마다 할당되며, 현재 **실행 중인 JVM 명령어의 주소**를 저장합니다. PC 레지스터는 스레드가 실행하는 바이트코드 명령어의 위치를 추적합니다.
5. **네이티브 메서드 스택 (Native Method Stack)**: 자바 외부의 \*\*네이티브 코드(C, C++ 등)\*\*를 호출할 때 사용하는 스택입니다. \*\*JNI(Java Native Interface)\*\*를 통해 호출된 네이티브 메서드의 실행을 지원하는 메모리 영역입니다."



## GC란 무엇인가요?

* 힙 메모리에서 더 이상 사용되지 않는 객체를 자동으로 제거하는 자동 메모리 관리 기법
* GC는 Minor, Major GC가 각각 Young Generation, Old Generation 메모리 영역에서 발생합니다.
* GC가  Young에서실행되고 살아남은 객체들은 survivor영역으로 이동하며, 일정 횟수 이상 GC를 통과하면 Old로 이동합니다.



## GC의 장단점

장점

* 메모리 관리 자동화

단점

* 성능 오버헤드: 주기적 GC 발생. 특히 Major GC는 애플리케이션을 일시중지
* 메모리 해제 예측 시간: 언제 메모리가 해제될지 모릅니다.



## GC에서 사용하는 알고리즘과, Java에서 사용하는 알고리즘

GC에서 사용하는 주요 알고리즘

* Reference Counting(참조 카운팅)
* Mark-and-Sweep(마크 앤 스윕)
* Mark-and-Compact(마크 앤 컴팩트)
* Generational GC(세대별 GC)

자바에서는 주로 Generational GC를 사용하는걸로 알고 있습니다.



## Java 8 기준으로, GC는 어떤 방식으로 수행되나요?

GC는 세대별로 수행됩니다.

* Young(Eden, Survivor 1,2), Old Gen
* Minor GC, Major GC
* 각 영역이 꽉차면 GC 실행



## 왜 Heap 영역은 Young, Old Generation으로 나뉘나요?

객체의 생명주기에 따라 GC의 효율성을 높이기 위해.

* 수명이 짧은 객체는 빠르게 수거
* 오래된 객체는 적은 빈도로 수거



## GC의 실행 방식을 아는 만큼 설명해주세요

* Eden, Survivor1, 2, Old
* Minor(짧고 빈번), Major(적은 빈도, stop the world)



## Java8, 11의 디폴트 GC 실행 방식

Java 8:

* Parallel GC: 여러 스레드 사용

Java 11:

* G1 GC: 고정된 크기의 Region으로 힙을 분리. 가비지가 많은 Region부터 병렬로청소.



## G1 GC에 대해 설명해주세요.

* G1은 Java7에서 처음 도입되었고, 9에서 기본 GC 실행방식이 되었습니다
* 힙 메모리를 고정된 크기의 작은 Region 단위로 나누어 관리
* 가비지 비율이 높은 Region부터 우선적으로 청소
* Mixed GC를 통해 일부 Old Gen도 함께 청소



## GC를 모니터링해야 하는 이유.

* GC가 자주 발생하거나 오래 걸리는 상황에 성능 저하
* 메모리 누수가 발생하면 더 이상 사용되지 않는 객체들이 GC들에 의해 해제되지 않음\
  (객체 참조해제가 안되는 문제. 잘못된 객체 참조, static 변수, 내부 클래스 참조 등)
* OOM을 예방



## OOM 발생시 대처 방법

* Heap Dump 분석 (JVM 옵션에 -XX:+HeapDumpOnOutOfMemoryError)\
  어떤 객체가 메모리를 많이 차지하고 있는지 보여주는 스냅샷
* 모니터링 도구 사용
* 메모리 누수 여부 확인
* JVM 힙 크기 조정
* GC 튜닝



## 메모리 누수 확인 방법

* 모니터링 툴 (JProfiler)
* Heap Dump 스냅샷 생성
* WeakReference사용

<details>

<summary>WeakReference</summary>

```java
MyObject obj = new MyObject();  // 강한 참조
WeakReference<MyObject> weakRef = new WeakReference<>(obj);  // 약한 참조
```

</details>