---
description: 2장. 자바 메모리 영역과 메모리 오버플로
---

# Chapter 2. Java Memory Area & Memory Overflow

## 2-2. 런타임 데이터 영역

<figure><img src="../../.gitbook/assets/image (329).png" alt=""><figcaption></figcaption></figure>

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (330).png" alt=""><figcaption></figcaption></figure></div>

### 프로그램 카운터

JVM이 현재 실행 중인 명령의 바이트코드 주소를 저장하는 작은 메모리 공간. JVM은 어떤 명령을 실행해야 하는지 추적할 수 있음

* 역할
  * JVM의 바이트코드 인터프리터는 프로그램 카운터에 저장된 주소를 읽고 해당 바이트 코드 명령을 실행 ( .java -> javac compiler -> .class -> 인터프리터/JIT compiler )
  * 실행 종료 후, 프로그램 카운터의 값을 갱신하여 다음 명령어로 이동. (동적 실행 구조)
  * 흐름 제어(조건문, 반복문, 예외 처리 등)를 수행할 때도 활용
* 멀티스레딩과 프로그램 카운터
  * JVM은 멀티스레딩을 지원하며, 각 스레드는 독립적인 실행 흐름을 가짐
  * 스레든느 번갈아가며 CPU를 점유하고, 각 스레드는 자신만의 프로그램 카운터를 별도로 유지.
  * 특정 스레드가 실행을 멈췄다가 다시 실행될 때, 프로그램 카운터의 값을 참조하여 중단된 위치에서 실행을 재개.
* 네이티브 메서드 실행 시
  * 네이티브 메서드를 실행할 때는 프로그램 카운터를 유지할 필요X
  * 네이티브 메서드 실행중인 스레드의 프로그램 카운터 값은 Undefined.
* 예외 처리
  * 프로그램 카운터는 JVM 명세에서 OutOfMemoryError가 발생하지 않는 유일한 메모리 영역

<details>

<summary>왜  네이티브 메서드 실행중인 스레드의 프로그램 카운터 값은 Undefined인가?</summary>

* **PC(프로그램 카운터)**: 현재 실행 중인 바이트코드 명령어의 주소를 저장.
* **스택(메서드 호출 프레임 저장)**: 메서드 호출 시 리턴 주소(다음 실행할 위치)를 저장.

</details>



### 자바 가상 머신 스택 (JVM Stack)

JVM 스택은 각 스레드가 실행하는 메서드 호출 정보를 저장하는 공간이고, 스레드마다 독립적으로 생성됨. 해당 공간은 JVM이 스레드의 실행 상태를 관리하는데 사용되고, 스택 프레임 단위로 구성됨.

* 스택 프레임의 구성 요소
  * 지역 변수 테이블(Local Variable Table) - 메서드의 지역 변수들을 저장
  * 연산자 스택(Operand Stack) - 연산 과정에서 사용할 피연산자와 결과값을 임시 저장
  * 동적 링크(Dynamic Linking) - 메서드 실행 중 참조해야 할 메서드나 클래스 정보를 저장
  * 메서드 반환 주소(Return Address) - 메서드가 종료된 후 실행이 재개될 위치를 저장
* 스택 프레임 동작 방식
  * 메서드가 호출될 때 새로운 스택 프레임이 생성되어 JVM 스택에 push 된다.
  * 메서드 실행이 종료되면 해당 스택 프레임은 pop 되어 제거 된다.
* JVM 스택과 메모리 오류
  * StackOverflowError: 너무 깊은 재귀 호출 / 무한 루프
  * OutOfMemoryError: JVM이 스택 확장을 시도할 때 메모리가 부족하면 발생

<details>

<summary> 왜 push, pop인가?</summary>

* 메서드가 실행을 마치면, 해당 **스택 프레임이 pop** 되면서 이전 메서드의 실행 상태로 돌아간다.
* 즉, **마지막으로 호출된 메서드가 가장 먼저 종료되므로 후입선출(LIFO) 원칙을 따르게 됨**.

</details>



### 네이티브 메서드 스택 (Native Method Stack)

자바 코드가 아닌 네이티브 코드를 실행할 때 사용하는 스택. 스레드 별로 별도로 생성되며, JVM 명세에는 "네이티브 메서드 스택의 구조나 구현 방식에 대한 규정을 두지 않는다"

* 역할
  * JNI를 통해 호출된 네이티브 메서드가 사용하는 실행 환경 제공
  * 네이티브 코드 실행을 위한 지역 변수 저장 및 연산 수행
  * 네이티브 코드 실행 중 JVM과의 인터페이스 역할 수행
* 메모리 오류
  * 네이티브 메서드 스택 크기가 초과되면 StackOverflowError
  * 확장시 메모리가 부족하면 OutOfMemoryError



<details>

<summary>JVM에서의 실행 환경 vs 네이티브 코드 실행 환경</summary>

* JVM에서 실행되는 자바 메서드는 **자바 가상 머신 스택**을 사용하며, 바이트코드를 실행하는 구조
* 하지만 JNI (Java Native Interface)를 통해 **C, C++ 같은 네이티브 코드**를 실행하려면 **자바가 아닌 시스템의 실행 환경**이 필요
* 네이티브 코드가 실행될 때 **해당 스레드의 JVM 스택이 아니라 네이티브 메서드 스택을 사용**하여 실행 환경을 제공

</details>

<details>

<summary>실행 환경이 필요한 이유</summary>

* **네이티브 코드는 CPU의 기계어 명령을 실행하는 것이므로 실행 자체는 OS에서 이루어지고**, **네이티브 메서드 스택은 실행에 필요한 부가적인 데이터를 저장하는 역할을 한다**
* JVM 내부에서 실행되는 코드는 바이트코드를 실행하지만, 네이티브 코드는 CPU의 기계어 명령을 직접 실행
* JVM이 직접 관리하지 않는 **OS의 메모리, CPU 레지스터, 스레드 스케줄링** 등을 다룰 필요
* 그래서 **JVM이 직접 다루지 않는 네이티브 코드를 실행할 별도의 공간(네이티브 메서드 스택)이 필요**
* 연산 결과를 **변수에 저장해야 하는 경우**, 네이티브 스택에 저장

</details>

<details>

<summary>네이티브 코드 실행을 위한 지역 변수란?</summary>

네이티브 메서드 스택도 자바 가상 머신 스택과 마찬가지로 **메서드 실행 중 필요한 지역 변수(스택 프레임 내 변수)를 저장하는 공간**을 제공

지역 변수 저장이 필요한 이유

* **스레드별 독립적인 데이터 저장**: 네이티브 메서드는 다중 스레드 환경에서도 독립적으로 실행될 수 있어야 함.
* **메서드 실행이 끝나면 자동 해제**: 네이티브 메서드가 끝나면 지역 변수도 함께 제거됨.

</details>

<details>

<summary>네이티브 코드 실행 중 JVM과의 인터페이스 역할</summary>

네이티브 코드가 JVM과 상호작용하는 기능을 제공한다는 의미 (네이티브 코드에서 JVM 내부의 기능(예: 객체 생성, 메서드 호출, 필드 접근)을 사용할 수 있도록 하는 역할)

* JNI를 이용한 자바 객체 조작
  * 네이티브 코드에서 **자바 객체를 생성하고 조작**할 수 있음.
* 네이티브 코드에서 자바 메서드 호출
  * 네이티브 코드에서 자바 클래스의 메서드를 호출할 수 있음
* 예외 발생 및 처리
  * 네이티브 코드에서 자바 예외를 던질 수 있음(예: NullPointerException)
* GC와의 상호작용
  * JNI를 통해 자바 객체를 네이티브 코드에서 사용하면, JVM GC가 이를 **알아서 수집하지 않도록** 관리해야 함

</details>



### 자바 힙 (Heap)

JVM에서 가장 큰 메모리 영역으로, 모든 객체 인스턴스가 저장되는 공간이며, JVM 시작과 함께 생성되고 모든 스레드가 공유함.

* 역할
  * 객체 인스턴스 및 배열 저장
  * GC에 의해 관리됨
  * 메모리 회수 및 재사용을 통해 효율적 관리
* 힙 영역 세분화 (Generatinal Garbage Collection)
  * Young(Eden, Survivor)
  * Old
  * Permanent Generation(JDK7까지): 클래스 메타데이터 저장(JDK8 이후 메타스페이스로 변경됨)
* 메모리 오류
  * OutOfMemoryError: 힙 메모리가 부족하여 새로운 객체를 할당 할 수 없을 때 발생

<details>

<summary>JDK 8부터는 PermGen이 제거되고, 대신 메타스페이스가 도입</summary>

**메타스페이스는 JVM 힙 영역이 아니라 네이티브 메모리를 사용**해.

* 즉, **OS에서 직접 메모리를 할당받는 방식.**
* 반면, 힙은 JVM이 직접 관리하는 메모리 공간.

메타스페이스의 장점

* 더 이상 힙 크기에 제한되지 않음
* 클래스 로딩이 많은 애플리케이션에서 OutOfMemoryError 감소
  * PermGen은 제한된 크기로 인해 **클래스 로딩이 많으면 OOM이 발생**했지만, 메타스페이스는 OS 메모리를 활용하므로 상대적으로 유연

</details>

<details>

<summary>JDK 버전별 GC의 변화</summary>

**Stop-the-World(STW) 시간을 줄이고, 성능을 최적화하는 방향**으로 발전

* 1.3 - **Parallel GC 도입** (Serial GC만 있던 시절에서 멀티스레드 지원)
* 1.4 - **CMS(Concurrent Mark-Sweep) GC 도입** (STW 시간을 줄이는 첫 GC)
* 5 - **Parallel GC 개선** (병렬 GC 성능 향상)
* 6 - **CMS GC 개선** (Concurrent 모드 실패 문제 해결)
* 7 - **G1(Garbage First) GC 도입** (Old GC보다 효율적)
* <mark style="color:blue;">**8 - G1 GC가 기본 GC로 변경 (PermGen 제거 → Metaspace 도입)**</mark>
* 9 - **G1 GC 개선** (Full GC 발생 감소)
* 10 - **Parallel Full GC 지원** (G1 GC 성능 향상)
* <mark style="color:blue;">11 -</mark> <mark style="color:blue;"></mark><mark style="color:blue;">**ZGC 도입**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">(초저지연 GC)</mark>
* 12 - **Shenandoah GC 도입** (낮은 STW 지연 목표)
* 14 - G1 GC 성능 최적화, NUMA 지원
* 15 - ZGC가 Production Ready (JDK 11에서는 Experimental)
* 16 - ZGC가 Thread Stack 처리 지원, Heap 영역 확장
* <mark style="color:blue;">17 - ZGC 개선, G1 GC의 STW 시간 감소</mark>
* 18 - G1 GC와 Parallel GC의 강제 Full GC 튜닝 가능
* <mark style="color:blue;">19 - Generational ZGC 도입 (ZGC를 Young/Old 구조로 분리하여 효율 증가)</mark>
* 20 - Shenandoah GC 최적화, G1 GC 개선
* <mark style="color:blue;">21 - Generational ZGC 공식 도입 (최신 GC 개선)</mark>



### **🔹 JDK별 주요 GC의 변화 상세 분석**

#### **🔹 JDK 1.3 이전 → 기본 GC: Serial GC**

* 단일 스레드로 실행되는 GC로, **멀티스레드 환경에서는 비효율적**.
* 작은 애플리케이션에서는 쓸 수 있지만, **대규모 애플리케이션에서는 성능 문제** 발생.

#### **🔹 JDK 1.4 → CMS(Concurrent Mark-Sweep) GC 도입**

* **STW 시간을 줄이기 위해 처음으로 "동시(Concurrent) GC"가 도입**됨.
* CMS GC는 **백그라운드에서 가비지 수집을 수행하여 애플리케이션의 중단 시간을 줄임**.
* 하지만, **Concurrent Mode Failure(동시 모드 실패)** 문제가 있었고, **프래그멘테이션(Fragmentation) 문제**가 심각했음.

#### **🔹 JDK 7 → G1(Garbage First) GC 도입**

* **Old Generation GC의 단점을 보완하기 위해 등장**.
* 기존 CMS GC의 **프래그멘테이션 문제를 해결**하고 **Region 기반의 메모리 관리 방식**을 도입.
* **JDK 8에서 기본 GC로 변경**됨.

#### **🔹 JDK 8 → G1 GC 기본 적용 & Metaspace 도입**

* **G1 GC가 기본 GC가 됨.** (기존 Parallel GC 대신)
* **PermGen(Permanent Generation)이 제거되고, Metaspace로 변경됨.**
  * Metaspace는 **네이티브 메모리를 활용하여 크기 제한이 사라짐.**

#### **🔹 JDK 11 → ZGC 도입 (초저지연 GC)**

* **ZGC는 10ms 미만의 초저지연(ultra-low latency) GC**를 목표로 개발됨.
* **1TB 이상의 대용량 힙에서도 성능 유지**.
* STW 시간이 **10ms 이하로 유지되므로 실시간 서비스에 적합**.

#### **🔹 JDK 12 → Shenandoah GC 도입**

* Red Hat이 개발한 **Shenandoah GC**가 공식 포함됨.
* **GC Pause(정지 시간)가 매우 짧아지고, GC의 응답성을 높이는 것이 목표**.

#### **🔹 JDK 15 → ZGC가 Production Ready**

* **JDK 11에서는 실험적(Experimental)이었지만, JDK 15부터는 정식으로 지원**.
* ZGC는 **4TB 이상의 대형 힙 메모리를 다룰 수 있는 GC**.

#### **🔹 JDK 19 → Generational ZGC (세대별 ZGC) 도입**

* 기존 ZGC는 **세대 구분 없이 모든 객체를 동일하게 관리**했지만,\
  **Generational ZGC는 Young / Old 구조를 도입하여 GC 효율을 높임**.
* 결과적으로 **Full GC를 최소화하고 성능을 더욱 향상**.

#### **🔹 JDK 21 → Generational ZGC 공식 적용**

* JDK 19에서 실험적으로 도입되었던 **Generational ZGC가 정식으로 포함됨**.
* **ZGC의 단점이었던 "모든 객체를 동일하게 관리"하던 방식에서 벗어나 성능 개선**.

</details>



### 메서드 영역 (jdk 7힙 -> jdk 8 메타스페이스)

메서드 영역은 모든 스레드가 공유하는 메모리 공간. 클래스 관련 정보와 JIT 컴파일된 코드 캐시를 저장

* 저장되는 정보
  * 로드된 클래스 정보 (클래스, 인터페이스, 필드, 메서드 정보)
  * 런타임 상수 풀
  * 정적 변수
  * JIT 컴파일된 코드
* 변화된 JDK 8 이후
  * OS가 네이티브 메모리 할당.

<details>

<summary>메서드 영역은 OS가 직접 관리하는 메모리인가?</summary>

* 메타스페이스(Metaspace)는 **OS가 직접 메모리를 할당하지만, JVM이 내부적으로 관리하는 영역**
* 즉, **JVM이 요청하면 OS가 네이티브 메모리를 할당**하고, JVM이 이 메모리를 메서드 영역으로 사용
* OS의 네이티브 메모리를 사용하지만, JVM이 메타데이터를 어떻게 저장하고 해제할지는 JVM이 결정하는 구조

👉 **"OS가 관리하는 공간"이라기보다는, "OS가 할당한 네이티브 메모리를 JVM이 메서드 영역으로 활용하는 구조"라고 보는 것이 정확.** JVM이 메서드 영역(Metaspace)을 과도하게 사용하면, 애플리케이션(JVM)뿐만 아니라 OS의 성능에도 영향을 줄 수 있다.

</details>



### 런타임 상수 풀 ( Runtime constant pool: 메서드 영역에 포함)

메서드 영역의 일부로, 클래스 로딩 시 생성되는 상수 및 심볼 정보를 저장하는 공간

* 포함되는 정보
  * 클래스, 메서드, 필드의 심볼 정보
  * String Poll (interned 문자열)
  * 상수 값 저장
* 메모리 오류
  * 상수 풀 공간이 부족하면 OutOfMemoryError 발생

<details>

<summary>심볼 정보란?</summary>

클래스의 구조를 식별하는 데 필요한 모든 정보(클래스명, 메서드명, 필드명, 타입 등)를 포함하는 **문자열 및 참조 정보의 집합**

**클래스, 메서드, 필드 등의 고유한 식별자**를 저장하는 데이터. JVM이 클래스를 로드할 때, 클래스 파일 내의 \*\*상수 풀(Constant Pool)\*\*에서 이러한 심볼 정보를 읽어 **클래스 메타데이터를 구성**한다.

**심볼 정보의 예시**

1. **클래스 이름**
   * 예: `com.example.MyClass`
2. **메서드 이름 및 시그니처**
   * 예: `void printMessage(String message)`
   * 여기서 `printMessage`는 메서드 이름, `(Ljava/lang/String;)V`는 JVM 내부 시그니처
3. **필드 이름 및 타입**
   * 예: `private int age;`
   * `age`라는 필드명이 심볼 정보로 저장됨
4. **인터페이스 이름**
   * 어떤 클래스가 구현하는 인터페이스의 이름 정보



</details>

<details>

<summary>Interned 문자열이란?</summary>

JVM이 동일한 문자열 리터럴을 **중복 저장하지 않고 공유**하기 위해 사용하는 메커니즘



String s1 = "hello"; // 상수 풀에 "hello" 저장\
String s2 = "hello"; // 같은 "hello"를 참조\
String s3 = new String("hello"); // 새로운 객체 생성\
String s4 = s3.intern(); // s3의 문자열을 상수 풀에서 찾음

System.out.println(s1 == s2); // true (동일한 상수 풀 객체)\
System.out.println(s1 == s3); // false (다른 객체)\
System.out.println(s1 == s4); // true (s4가 상수 풀의 "hello" 참조)

</details>

<details>

<summary>JDK 7까지는 PermGen, JDK 8부터는 네이티브 메모리?</summary>

JDK 7까지는 PermGen(Permanent Generation)에 런타임 상수 풀이 저장되었고, JDK 8부터는 네이티브 메모리 기반의 Metaspace로 이동

* JDK 7에서는 `"hello".intern()` 하면 PermGen에 저장.
* JDK 8에서는 `"hello".intern()` 하면 **Java Heap**에 저장됨.

</details>



### 다이렉트 메모리 (Direct Memory: JVM 관리 영역X)

JVM의 힙 영역이 아니라 네이티브 메모리를 직접 할당하는 방식으로 사용

* 사용 목적
  * DirectByteBuffer를 활용한 고속 I/O 처리
  * 데이터 복사를 최소화하여 성능 향상
* 메모리 할당 과정
  * ByteBuffer.allocateDirect()를 호출 -> OS에서 직접 네이티브 메모리를 할당
  * JVM 힙이 아닌 네이티브 메모리에 저장
  * JVM 내부적으로 DirectByteBuffer 객체를 생성하여 참조
  * 사용 후 명시적으로 해제해야함
* 장점
  * 빠른 성능
    * 힙 메모리의 객체를 사용할 경우, 네이티브 코드로 데이터를 복사하는 과정이 필요.
    * 다이렉트 메모리는 OS의 네이티브 메모리를 직접 사용하므로 즉시 접근 가능(I/O 최적화)
  * GC 영향없음
    * GC로 인한 성능 저하 없음
  * 대용량 데이터 처리 최적화
    * 네트워크 통신, 파일 입출력, 데이터베이스 연동 등 대용량 데이터를 처리할 때 효율적.
* 제한 사항
  * `-Xmx` 등의 설정으로 JVM의 힙 크기를 조절할 수 있지만, 다이렉트 메모리는 JVM의 관리 대상이 아니므로 메모리 누수 방지를 위해 개발자가 명시적으로 해제해야함.
  * 운영 체제의 물리 메모리 한계를 초과할 경우 `OutOfMemoryError` 발생 가능

<details>

<summary>DirectByteBuffer란?</summary>

`DirectByteBuffer`는 **JVM 힙이 아닌 OS의 네이티브 메모리를 직접 사용하는 버퍼**를 관리하는 **Java NIO(ByteBuffer) 구현 클래스**



</details>

```java
// DirectByteBuffer를 이용한 파일 I/O 성능 최적화
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class DirectByteBufferFileIO {
    public static void main(String[] args) throws Exception {
        // 파일 채널 생성
        FileInputStream fis = new FileInputStream("input.txt");
        FileOutputStream fos = new FileOutputStream("output.txt");
        FileChannel inputChannel = fis.getChannel();
        FileChannel outputChannel = fos.getChannel();

        // DirectByteBuffer 할당 (네이티브 메모리 사용)
        ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

        while (inputChannel.read(buffer) != -1) {
            buffer.flip();
            outputChannel.write(buffer);
            buffer.clear();
        }

        inputChannel.close();
        outputChannel.close();
        fis.close();
        fos.close();
    }
}

JVM 힙을 거치지 않고 다이렉트 메모리를 사용하여 파일 데이터를 빠르게 복사
네이티브 메모리를 활용하므로 I/O 성능이 향상됨
```







## 2-3. 핫스팟 가상 머신에서의 객체 들여다보기

new 키워드를 사용하여 객체를 생성하지만, 내부에서는 추가 과정들이 존재.

### 1. 객체 생성 과정

1. 클래스 로드 확인
   1. 해당 객체의 클래스가 이미 메모리에 로드되었는지 확인
   2. 로드되지 않았다면 클래스 로딩 -> 해석 -> 초기화 단계를 거침
2. 힙 메모리 할당
   1. 객체의 크기를 결정하고 적절한 힙 공간을 찾아 메모리를 할당
   2. 메모리 할당 방식에는 포인터 증가(Bump-the-pointer)와 Free List 방식이 있음
3. 객체 헤더 설정
   1. 객체의 기본 정보를 설정 (해시값, GC 연령 정보 등)
4. 필드 초기화
   1. 객체의 모든 필드를 기본값으로 설정
5. 생성자 호출
   1. init 메서드를 실행하여 객체를 최종적으로 초기화



### 힙 메모리 할당 방식

객체용 메모리 공간 할당은 자바 힙에서 특정 크기의 메모리 블록을 잘라 주는 일. 객체에 필요한 메모리 크기는 클래스를 로딩하고 나면 완벽히 알 수 있음.

* 포인터 증가 방식 (Bump-the-pointer)
  * 자바 힙이 연속적인 메모리 블록으로 관리될 때 사용
  * 사용중인 메모리와 여유 메모리 사이의 경계를 포인터로 추적
  * 새로운 객체를 생성하면 포인터를 해당 크기만큼 이동하여 메모리를 할당
  * GC가 한 번 실행될 때마다 메모리를 재정렬 해야함(Compact 필요)
  * 단편화되지 않을 시에 매우 빠른 할당 속도.
* Free List 방식
  * 조각난 메모리 공간이 존재할 떄 사용(Compact 불필요)
  * 사용 가능한 빈 메모리 블록 목록을 유지하고 거기에서 적절한 크기의 공간을 찾아 객체를 할당
  * 메모리 관리가 유연하지만, 할당 속도가 상대적으로 느림

<details>

<summary>Compact를 수행하는 GC, 수행하지 않는 GC</summary>

#### **✅ Serial GC**

* **단일 스레드 기반의 GC**이며, **Stop-the-World(전체 애플리케이션 일시 중지) 동안 Compact를 수행**
* Young GC(Minor GC)에서 **Copying GC** 방식을 사용하여 Compact 불필요
* **Old GC(Major/Full GC)에서 Mark-Sweep-Compact 방식으로 압축 수행**

#### **✅ Parallel GC (Throughput GC)**

* 여러 개의 스레드를 사용하여 **GC 성능을 향상**
* Old GC에서 **Mark-Sweep-Compact 방식**을 사용하여 **단편화 해소**
* **Young GC는 Eden → Survivor로 이동하는 과정에서 Compact 불필요**

#### **✅ G1 GC (Garbage-First GC)**

* G1 GC는 **Region 기반 GC**로 동작하며, **Mixed GC 단계에서 Compact 수행 가능**
* 사용되지 않는 Region을 정리하고, **유효한 객체를 새로운 Region으로 이동하여 단편화 해소**
* **Full GC 발생 시에도 압축을 수행하지만, Full GC 자체는 가급적 피하도록 설계됨**



#### **❌ CMS GC (Concurrent Mark-Sweep)**

* **Stop-the-World 시간을 줄이기 위해 압축을 수행하지 않음**
* 단점: **메모리 단편화(Fragmentation)가 발생할 수 있음**
* 단편화를 해결하기 위해 **Full GC가 강제 발생**할 수도 있음
* **JDK 9부터 G1 GC가 CMS GC를 대체**

#### **❌ ZGC**

* **Heap을 조각난 상태로 유지한 채 메모리를 관리**하여 Compact 불필요
* **Large Page 메모리 모델과 색인 기반 메모리 매핑을 사용**
* **즉, ZGC는 단편화 발생을 방지하는 방식으로 설계되어 Compact 자체가 필요 없음**

#### **❌ Shenandoah GC**

* **Concurrent Compaction을 수행**하여 **Stop-the-World 없이 단편화를 해결**
* 기존 GC처럼 **모든 객체를 이동하는 압축 방식이 아니라, 필요할 때만 일부 객체를 이동**
* **기본적으로는 Compact가 필요 없는 구조**

</details>



### 멀티스레드 환경에서의 객체 생성 문제

여유 메모리의 시작 포인터 위치를 수정하는 작업도 thread-safe 하지 않기 떄문에 race condition 발생 가능.

해결 방법은 두 가지 방법이 있다.

* CAS
  * 객체를 할당하는 과정에서 CAS 연산을 활용하여 포인터 갱신을 원자적으로 처리
  * 포인터 값을 비교 후 안전하게 변경
* TLAB (Thread-local allocation buffer)
  * 각 스레드가 자신만의 작은 힙 영역을 별도로 할당하여 객체를 생성
  * 스레드 간 경쟁 없이 독립적인 공간에서 객체를 빠르게 생성 가능
  * TLAB이 가득 차면 새로우 ㄴ힙 영역을 요청하여 사용
  * JVM 옵션으로 -XX;+UserTLAB을 설정하면 활성화 가능
  * JVM은 TLAB을 기본적으로 활성하여 사용

<details>

<summary>TLab에서 스레드에게 충분한 영역을 줄 수 있는지 판단 방법</summary>

결론:

TLAB이 파편화된 힙에서 충분한 공간을 제공할 수 있는지는 **다음과 같은 기준을 통해 판단**된다.

1. **TLAB이 최소 크기(`-XX:MinTLABSize`) 이상인지 확인**
2. **남은 TLAB 공간이 설정된 비율(`-XX:TLABWasteTargetPercent`) 이하인지 확인**
3. **힙에 충분한 연속적인 공간이 존재하는지 확인**
4. **필요한 경우, 기존 TLAB 크기를 축소하거나 글로벌 힙을 활용**

즉, **TLAB이 충분한 공간을 찾지 못하면 글로벌 힙을 활용**하여 객체를 할당하게 되고, JVM은 GC를 통해 단편화를 해결하며 TLAB 크기를 동적으로 조정한다.

👍 **핵심 요약**

* TLAB을 할당할 수 없는 경우, **최소 크기 이하인지 확인 후 글로벌 힙에서 객체를 할당**
* **GC가 실행될 때 단편화를 줄이고 TLAB 크기를 조정**
* JVM은 **TLAB 크기를 동적으로 조절하여 최적화**
* **TLAB 크기 조정은 `-XX:TLABSize`, `-XX:TLABWasteTargetPercent`, `-XX:MinTLABSize` 등의 옵션으로 튜닝 가능**



자세히:

핫스팟 JVM에서는 **TLAB 크기 결정 및 확장 정책**을 사용

1. TLAB 초기 크기 설정 - TLAB 크기는 기본적으로 `-XX:TLABSize` 옵션을 통해 설정할 수 있지만, JVM은 다음을 고려하여 자동으로 TLAB 크기를 결정
   * 현재 힙 메모리 상태
   * 스레드 수
   * Young Generation의 크기 (TLAB은 Young Generation에 할당됨)
   * 스레드별 객체 생성 빈도

스레드마다 동일한 크기의 TLAB을 할당하는 것이 아니라, JVM이 동적으로 크기를 조정



2. TLAB 할당 시 충분한 공간이 있는지 판단하는 기준
   1. 기본적인 최소 크기 조건
      * TLAB이 할당될 최소 크기는 **객체 크기보다 커야 함**
      * `-XX:MinTLABSize` 옵션으로 최소 크기를 조정 가능
      * 만약 현재 힙에 **TLAB 최소 크기보다 작은 연속적인 공간만 남아 있다면**, 해당 스레드는 **TLAB을 할당받지 않고 글로벌 힙에서 직접 할당**하게 됨
   2. TLAB 잔여 공간이 설정된 임계치보다 작다면 재할당
      * JVM은 TLAB 내에서 남아 있는 공간이 **일정 비율 이하**로 줄어들면, 새로운 TLAB을 요청하게 됨
      * `-XX:TLABWasteTargetPercent` 옵션을 통해 이 임계치를 조정할 수 있음
      * 기본적으로 **TLAB의 1% 미만의 공간만 남아 있으면 새로운 TLAB을 할당**받음
   3. 힙 단편화 고려
      * JVM은 Young Generation 내의 **가용 공간이 충분하지 않거나 단편화가 심한 경우**, 기존의 TLAB을 축소하여 작은 크기로 재할당할 수도 있음
      * **만약 연속된 여유 공간이 없다면**, 기존 TLAB을 해제하고 **글로벌 힙에서 객체를 직접 할당**하는 방식으로 전환



3. TLAB이 부족할 경우의 동작 방식
   * **새로운 TLAB을 요청**
     * 연속된 여유 공간이 충분하다면 새로운 TLAB을 할당
   * **TLAB 크기 축소 후 재할당**
     * Young Generation의 단편화가 심하면, TLAB 크기를 축소한 후 재할당 시도
   * **TLAB을 포기하고 글로벌 힙 사용**
     * 단편화가 너무 심해서 TLAB을 할당할 수 없는 경우, 스레드는 글로벌 힙에서 직접 객체를 할당

</details>

\*\*TLAB 최적화 관련 JVM 옵션

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (331).png" alt=""><figcaption></figcaption></figure></div>



### 생성자 호출과 객체의 완전한 초기화

객체는 `new` 키워드와 함께 생성되지만, **실제로 객체가 초기화되는 것은 생성자(`<init>` 메서드)가 실행된 이후**이다.

* `new` 키워드 → **힙 메모리 할당**
* `invokespecial` → **생성자 호출 (초기화 수행)**
* `init` 메서드 실행 → **완전한 객체 생성 완료**





### 2. 객체의 메모리 레이아웃

<div data-full-width="true"><figure><img src="../../.gitbook/assets/image (333).png" alt=""><figcaption></figcaption></figure></div>

핫스팟 가상 머신은 객체를 세 부분으로 나누어 힙에 저장한다.

1. 객체 헤더 - 객체의 메타데이터를 포함
2. 인스턴스 데이터 - 실제 객체 필드 값들이 저장됨
3. 정렬 패딩 - 메모리 정렬을 위해 추가되는 공간



### 객체 헤더 (Object Header)

객체 헤더는 JVM이 객체를 관리하는 데 필요한 정보를 저장하는 영역

* 마크 워드 (Mark Word)
  * 객체의 해시 코드 (hashCode)
  * GC 연령 (Age)
  * 락 플래그 (Lock Flag, 동기화 정보)
  * 스레드 경합 상태 (Thread Contention State)
  * 평활화 스레드 타임스탬프 (Bias Locking Timestamp)

<figure><img src="../../.gitbook/assets/image (332).png" alt=""><figcaption></figcaption></figure>

<details>

<summary><strong>객체 헤더 크기</strong></summary>

* 32비트 JVM에서는 **8바이트**

- 64비트 JVM에서는 **16바이트**

* 배열의 경우 추가적인 4바이트 필요

</details>

<details>

<summary>왜 객체 헤더에 동기화 정보가 필요한가?</summary>

객체 헤더에 동기화 정보가 들어가는 건 **모니터 락(Monitor Lock)** 때문.

VM은 `synchronized` 키워드나 `wait()`, `notify()` 같은 동기화 기능을 사용할 때 **객체 단위로 락을 관리**하는데,\
이때 **락 정보가 객체의 헤더(Mark Word)에 저장**



### **🔹 객체 락의 종류 (3가지 락 단계)**

자바의 동기화 방식은 최적화를 위해 **세 가지 락 단계**를 가지고 있어.

1️⃣ **Bias Lock (편향 락)**

* **한 개의 스레드가 계속 객체를 사용할 때 최적화**
* Mark Word에 **스레드 ID를 저장**하여 불필요한 동기화 제거
* **락 해제 과정 없이 같은 스레드가 계속 사용 가능**
* 단, **다른 스레드가 접근하면 편향 락이 해제됨 (Revoke)**
* 🔹 **성능 최적화 목적**

2️⃣ **Lightweight Lock (경량 락)**

* **여러 스레드가 경쟁하지만 락 충돌이 심하지 않을 때 사용**
* CAS(Compare-And-Swap) 연산을 활용하여 **빠른 락 획득 및 해제**
* Mark Word에 **Lock Record의 포인터 저장**
* 🔹 **빠른 락 전환 및 충돌 최소화**

3️⃣ **Heavyweight Lock (무거운 락)**

* **경합이 심해서 경량 락을 사용할 수 없을 때**
* OS 모니터(커널 수준의 락) 사용 → **성능 저하 발생**
* Mark Word에 **Monitor Object의 주소 저장**
* 🔹 **성능이 낮지만, 확실한 동기화 보장**



</details>

<details>

<summary>객체 헤더의 마크워드에는 락 플래그가 있는데, 그 외에, 현재 락을 획득한 스레드는 누구고, 다음 깨울 스레드는 누구고, 어떤 스레드들이 기다리고 있는지에 대한 정보는?</summary>

자바 객체의 헤더(Mark Word)에는 동기화 관련 정보가 저장되지만,\
&#xNAN;**"어떤 스레드가 현재 락을 가지고 있는지", "어떤 스레드들이 기다리고 있는지", "다음에 락을 얻을 스레드는 누구인지"** 같은 상세한 정보는 Mark Word 자체에는 저장되지 않음

**이 정보들은 객체의 "Monitor Record" 또는 "ObjectMonitor" 구조체에 저장**

</details>

<details>

<summary>락 플래그 vs 스레드 경합 상태</summary>

락 플래그

**객체가 현재 어떤 락 상태인지**를 나타내는 **객체 헤더(Object Header)의 일부**이다.\
이 정보는 \*\*마크 워드(Mark Word)\*\*에 저장되며, **객체 동기화(synchronized) 여부에 따라 값이 달라진다.**



스레드 경합 상태 (Thread Contention State)

**스레드 경합 상태는 JVM이 현재 실행 중인 여러 스레드 간에 어떤 경합이 발생하는지 나타내는 상태 정보**이다.\
즉, **락이 필요할 때 스레드 간 충돌이 발생했는지 여부를 추적하는 정보**라고 볼 수 있다.

#### **스레드 경합 상태의 역할**

* 여러 스레드가 같은 객체를 동시에 접근하려고 할 때 **동기화 충돌이 발생하는지 확인**
* 특정 스레드가 **락을 얼마나 오래 기다렸는지 추적**하여 최적화 가능
* 필요하면 **Bias Lock을 해제하고 Lightweight 또는 Heavyweight Lock으로 전환**

</details>

<details>

<summary>평활화 스레드 타임스탬프 (Bias Locking Timestamp)</summary>

JVM은 Bias Lock을 유지할지, 해제할지를 결정해야 하는데,\
이를 위해 객체가 **언제 Bias Lock을 가졌는지**를 **Bias Locking Timestamp** 값으로 저장한다.

**Bias Locking Timestamp가 하는 일**

1. **Bias Lock이 활성화될 때, 객체의 Mark Word에 해당 스레드 ID와 함께 "타임스탬프"를 저장**
2. 이후 JVM은 **주기적으로 해당 객체가 여전히 같은 스레드에서만 사용되고 있는지 검사**
3. **객체가 일정 시간 동안 다른 스레드에서 사용되지 않았다면 Bias Lock을 계속 유지**
4. **다른 스레드가 해당 객체를 사용하려고 하면 Bias Lock을 해제하고 Lightweight Lock(경량 락) 또는 Heavyweight Lock(중량 락)으로 전환**
5. 이때, Bias Locking Timestamp를 참고하여 얼마나 오랫동안 편향되어 있었는지를 기준으로 결정

즉, **Bias Locking Timestamp는 JVM이 Bias Lock을 유지할지, 해제할지를 판단하는 기준이 되는 값**이다.

</details>



### 인스턴스 데이터

객체의 실제 데이터(필드 값들)가 저장되는 공간.

* **객체의 모든 필드(primitive + reference types) 저장**
* **부모 클래스의 필드도 함께 저장**
* **저장 순서는 JVM의 필드 할당 전략에 따라 달라질 수 있음**
* JVM 옵션 `-XX:FieldsAllocationStyle`을 통해 필드 정렬 방식 변경 가능

핫스팟 JVM은 기본적으로 같은 크기의 필드를 묶어 정렬(padding을 최소화)하는 전략을 사용한다. \
`-XX:CompactFields=true` 옵션을 설정하면 작은 필드를 빈 공간 없이 압축하여 저장





### 정렬 패딩

정렬 패딩은 메모리 정렬(alignment)을 맞추기 위해 추가되는 공간이다.

* **JVM은 8바이트 단위로 메모리를 정렬**하는 경우가 많다.
* 인스턴스 데이터 크기가 **8바이트 단위로 정렬되지 않는 경우**, 패딩을 추가하여 정렬을 맞춘다.
* CPU가 메모리에 **빠르게 접근하도록 최적화**하기 위한 기술.





### 3. 객체에 접근하기

자바 객체는 \*\*참조(reference)\*\*를 통해 접근한다. 참조 방식은 JVM 구현에 따라 다르지만, 핫스팟 JVM은 **두 가지 방식**을 제공한다.



<figure><img src="../../.gitbook/assets/image (334).png" alt=""><figcaption></figcaption></figure>

* 핸들(Handle) 방식
  * 핸들 방식에서는 힙(heap) 외부에 별도의 핸들 풀(handle pool)을 유지
  * 핸들은 **객체의 실제 주소가 아닌 핸들 풀의 주소를 저장**하며, 핸들 풀은 **객체의 실제 데이터 위치를 가리키는 포인터**를 가진다.
  * 장점
    * 객체가 이동할 경우 **핸들만 업데이트하면 되므로 참조를 갱신할 필요 없음**
    * GC가 객체를 이동할 때도 **핸들 주소는 그대로 유지됨**
  * 단점
    * **객체에 접근할 때 항상 한 단계 추가적인 간접 참조가 필요**
    * 성능이 다이렉트 포인터 방식보다 약간 느릴 수 있음



<figure><img src="../../.gitbook/assets/image (335).png" alt=""><figcaption></figcaption></figure>

* 다이렉트 포인터 (Direct Pointer) 방식. (핫스팟 JVM의 기본값)
  * 다이렉트 포인터 방식에서는 **객체 참조가 직접 힙 메모리의 객체 주소를 가리킨다**.
  * 장점
    * 객체에 **빠르게 접근 가능** (한 단계 간접 참조가 없음)
    * 참조를 직접 가리키므로 **CPU 캐시 최적화 가능**
  * 단점
    * **GC가 객체를 이동하면 참조를 업데이트해야 함**
    * 메모리 주소가 직접 바뀌므로 **GC의 복잡도가 증가**



**핸들과 다이렉트 포인터의 차이**

**핸들 방식 구조**

* 객체 참조 → 핸들 풀 → 객체 실제 메모리
* **객체 이동 시 핸들만 업데이트하면 됨**

**다이렉트 포인터 방식 구조**

* 객체 참조 → 객체 실제 메모리
* **객체 이동 시 참조 주소를 직접 변경해야 함**
