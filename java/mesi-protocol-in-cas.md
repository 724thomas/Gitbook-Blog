# MESI protocol in CAS

CAS연산은 스냅샷과 연산된 결과값을 비교한 뒤에 메모리에 업데이트 하는 작업을 원자적으로 수행합니다. 이게 가능한 이유는, lock prefix(lock cmpxchg, lock xadd, lock add 등) 명령어 때문에 가능합니다.

MESI는 캐시라인(64B block)에 현재 데이터가 어떤 상태인지 나타내게됩니다.

```
┌────────────────────────────┐
│  Cache Line (64B block)    │
├────────────────────────────┤
│  주소 태그 (Tag)             │ ← 어느 메모리 주소인지
│  상태 (MESI)                │ ← Modified / Exclusive / Shared / Invalid
│  데이터 (Data: 64B)          │
└────────────────────────────┘

```



lock prefix가 붙으면 발생하는 상황:

### 1단계: CAS 시작 전 캐시 로딩

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 스레드1과 스레드2는 CAS 시작 전 캐시 로딩.
  * 캐시 메모리에 값을 올리기 전에, Bus Request라는게 발생(이는 아래서 설명)
* 이때 메모리에서 값을 읽어서 스냅샷을 각 L1캐시에 저장합니다.
* 상태는 Exclusive 또는 Shared 상태. (예시에서는 shared)
  * 처음에는 Exclusive(E)일 수 있음 (다른 코어가 안 갖고 있으면)
  * 둘 다 갖고 있으면 **Shared(S)**

***

### 2단계: 스레드1이 CAS 수행

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 스레드1이 연산을 끝내고, lock cmpxchng \[0x1000], eax를 실행함.
* Bus Snooping을 통해, 다른 모든 L1캐시들의 해당 값의 상태를 Invalid로 변경.
* CPU1-L1캐시의 해당 값의 상태를 Modified로 변경됨.
* 이로써, CPU-L1캐시가 해당 캐시라인을 독점하게 되며, 안전하게 값을 바꿀 수 있음. (연산의 원자성 보장)

***

### 3단계: 스레드2가 CAS 시도

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* CPU2는 해당 캐시라인이 없음. (쓰레드2는 CAS를 하기 위해 다시 CPU2-L1캐시에 값을 올리려고 함)
  * 캐시에 해당 주소가 없으므로, 이때, Bus Request가 발생.
  * "0x1000 주소 가져오고 싶은데 캐시에 없으니, 메모리나 다른 CPU에서 줘!"
* 시스템은, 0x1000 주소가 현재 CPU1에서 Modified 상태로 들고 있다고 알려줌.
  * CPU1이 Write-back 수행(메모리에 반영)
  * 이후 상태를 Modified에서 Shared로 변경.
* 쓰레드2는 대기 상태로됨.

***

### 4단계: CPU2가 캐시라인 획득

<figure><img src="../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

CPU1 CAS가 완료되고 쓰레드1은 메모리에 연산 결과를 업데이트한 뒤, (CPU1이 Write-back 수행)

* 쓰레드2는 0x1000주소의 상태가 Modified가 없는 것을 확인.
* CPU1-L1 또는 메모리에서 0x1000을 가져옴. 보통 캐시에서 가져옴 (캐시-투-캐시 전송. 더 빠르기 때문)

***

### 5단계: CPU2가 CAS 시도

<figure><img src="../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* CPU2에서 lock cmpxchg를 수행합니다.
* Bus snooping 발생.
  * 다른 CPU L1 캐시들의 상태를 Invalid로 변경
  * CPU2-L1캐시 값 상태를 Modified로 변경.
  * 이후 CAS 원자적 수행

***

### **Bus Snopping 방식**

1. CPU2가 어떤 주소 0x1000을 캐시에 로드하려고 함
2. CPU2는 메모리 컨트롤러/버스를 통해 "이 주소 누가 갖고 있어?"라고 물어봄
3. CPU1이 이 주소를 M 상태로 갖고 있으면
   1. "나 갖고 있고 M 상태야!" 라고 snoop 응답
   2. CPU1은 Write-back 수행 (RAM에 반영. 조회 후 "변경"한 것)
   3. 자신의 캐시 라인 상태를 M->S 또는 E로 다운그래이드.



만약 CPU가 수십\~수백 개로 많아지면 **Bus Snooping 방식은 한계**가 있습니다.

이 경우엔:

* **Directory-based coherency**를 사용.
* RAM 컨트롤러나 중앙 디렉토리에서 **“어떤 캐시가 이 주소 갖고 있는지”** 관리함
* CPU2가 요청하면, 디렉토리에서 CPU1이 Modified 상태임을 확인하고 조율합니다.



### CAS연산의 스핀락때문에 발생하는 성능 저하

여기서 발생하는 성능 저하는 크게 2가지입니다.

1. Invalid된 스레드들이 대기하면서 계속해서 Bus snooping을 하면서 발생하는 성능저하.
2. Shared로 변경되고, 대기하던 모든 스레드들이 연산을 시작하지만, 결국 하나만 반영되기떄문에, 1개를 제외한 나머지 스레드의 연산은 무의미한 것.

결국 Modified가 된 상태일때는 비즈니스 로직 연산이 최대 1번만 수행되고, bus snooping 스핀이 계속 발생



참고:&#x20;

* [https://stackoverflow.com/questions/39393850/is-incrementing-an-int-effectively-atomic-in-specific-cases](https://stackoverflow.com/questions/39393850/is-incrementing-an-int-effectively-atomic-in-specific-cases)
* [https://stackoverflow.com/questions/59020823/how-does-lock-cmpxchg-work-in-assembly](https://stackoverflow.com/questions/59020823/how-does-lock-cmpxchg-work-in-assembly)
* [https://www.felixcloutier.com/x86/lock](https://www.felixcloutier.com/x86/lock)
