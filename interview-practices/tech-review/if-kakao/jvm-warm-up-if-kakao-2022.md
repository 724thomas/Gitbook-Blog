---
description: JVM warm up / if(kakao)2022
---

# JVM warm up

https://www.youtube.com/watch?v=CQi3SS2YspY\&list=PLwe9WEhzDhwHLNl0f39E1lAyT8dxdTUGY\&index=6\&ab\_channel=kakaotech

응답 레이턴시 문제, 해결 과정에서 알게된 jvm 디스컴파일러와 warm up

서비스: 카카오T 계정 서비스: 가입, 휴면, 탈퇴\
외부 게이트웨이 뿐 아니라 다양한 내부 서비스들이 계정서버로 API요청을 많이하기에, 운영되는 micro 서비스중에 TPS가 높은 서버

→ 빠른 응답속도를 보장해야하는 것이 중요.

기술 스택 : Java 11(LTS), Spring Boot 2

#### JAVA 언어의 특성

1. 자바 코드를 중간언어인 Byte Code로 컴파일
   1. 주로 Byte Code는 Jar 또는 War 파일로 아카이브하여 활용
   2. 빌드된 파일을 실행하면 JVM에서는 Byte Code를 번역하여 기계어로 만들고 CPU에서 처리
   3. 이렇게 빌드된 바이트 코드는 별도의 추가적인 빌드없이 자바가 실행가능한 CPU아키텍처 그리고 OS에서 실행할 수 있는 장점
   4. 컴파일과 인터프리트라는 두가지 동작으로 실행되는 언어
2. JAVA는 컴파일 과정에서 바로 기계어로 만드는 컴파일 개발언어보다 성능이 뒤쳐짐
   1. 런타임환경에서 준비된 기계어를 즉시 실행 가능하기 때문
   2. 또한 컴파일 단계에서 코드 최적화를 하여 기계어를 만들기에 인터프리터 언어보다 더 빠름
   3. 컴파일을 통해 생성된 기계어는 빌드환경의 CPU 아키텍처에 종속적이라 다른 아키텍처에서 실행하고 싶으면 해당 빌드환경에서 빌드를 다시 해야하는 단점이 있음.

#### JIT 컴파일러 (Hotspot) (JDK 1.3\~)

![Image](https://github.com/user-attachments/assets/9cd72a5b-16c8-49d7-a49a-a593ae585ab4)

* 인터프리터
  * 컴파일된 바이트코드를 JVM이 인터프리터로 한 줄씩 읽으며 실행.
  * 초기 실행은 빠르게 시작하지만 느림.
  * 성능을 위해 JIT 컴파일러 등장.
* JIT compiler
  * 자주 실행되는 메서드를 감지하면 머신코드로 컴파일. 런타임에 최적화된 캐싱.

하지만 JIT 컴파일러가 실행되려면, “자주 실행되는” 기준을 만족해야함.

#### 계정 서비스 배포시 발생한 Latency 이슈

계정 서버는 쿠버네티스 운영

* 롤링 업데이트 방식 배포
* pod는 순차적으로 배포

문제:

* 10초 이상의 응답 지연이 발생해, 계정 서버에 요청을 보내는 많은 내부 서버에 영향을 줌
* TPS 3800 \~ 4000
* 지연이 유지되는 것이 아니라, 시간이 지나면 지연이 해소됨

#### 응답 지연 분석 과정

1. 리소스 먼저 분석

* cpu
  * 평균 10% 이하
* memory
  * 60% 이하
* network bandwidth
  * 워커 노드 당 10\~20MB → 여유 있음
* TPS
  * 배포 전과 비슷, 트래픽이 몰리는 상황은 X

1. 애플리케이션 모니터링 (APM)

* 외부 DB의 지연에서는 확인되지 않음
* 대부분의 지연은 애플리케이션 영역
  * 도메인 로직에 대한 지연은 없음
* 톰캣의 애플리케이션 스레드 개수
  * 10\~8000개 설정
  * 서비스 시작 후 스레드가 200개까지 늘어남. → **`스레드 시작 개수에 대한 조정의 필요성을 느낌`**
* 데이터베이스 Connection Pool
  * RDB
    * 20-50개로 설정
    * 20개로 시작하였고, 서버 시작후에도 큰 변화가 없음
  * Redis
    * 단일 노드

1. JVM Warm up 확인

서버 시작 과정

* k8s의 readiness probes 요청에 대해 400 응답 코드
* 애플리케이션 Ready Event가 발생하고 3초의 딜레이 후, Readiness 요청에 200 응답을 주도록 하여 트래픽이 인입되도록 되어 있음

Warm up 동작 과정

* liveness/readiness probes 요청 발생 시, DB에 정해진 정보를 조회
* liveness/readiness 요청을 처리하는 컨트롤러에서, **`warmer()`** 호출
  * wamer()는 실제 서비스에 사용되는 api가 아니고, warmer를 호출하더라도 특정 데이터만 호출이 되는 문제.
  * 충분한 warm up이 일어나지 않다고 판단.

개선

* 각 Pod의 톰캣 스레드를 200여개로 시작하는 작업 필요
* JVM warm up 실제 사용하는 api를 localhost로 호출하되, real 트래픽과 같은 유사한 요청이 되도록 개선

#### 대응 - API 시작 과정

![Image](https://github.com/user-attachments/assets/3ff800a0-e07a-412a-bb65-46ed4786e955)

* 애플리케이션 실행은 기존과 동일하게 liveness/readiness 과정에서 처리
  * readiness probes 요청에 대해 400 응답 코드를 통해 외부 트래픽을 차단
* Warm up 절차가 완료되면 readiness probes 응답을 200으로 변경
  * 예전에는 정해진 시간만큼만 지연시키고 트래픽을 인입
  * 이제는 warm up이 완료되어야 트래픽 인입하도록 개선
* Warm up은 각 pod마다 LocalHost로 Api요청을 하도록 변경.
  * 실제 api 요청과 동일한 응답 코드로 처리되도록 호출을 진행.
  * 서비스에서 자주 사용되는 GET api, 배포 과정에서 apm에서 확인된 지연 대상들을 포함
* 스레드 시작 개수, \[[Localhost](http://localhost/)]\(http://Localhost) 기반 워머를 포함하여 프로덕션 배포를 진행.
  * 초기 응답 지연이 사라짐

#### 그러나 2달 후, 같은 문제 발생

* TPS 5000, 이전 트래픽 대비 20퍼센트 높아진 상황에서 계정서버를 배포시
  * 초기 응답 지연 상황 또 발생
  * 보여지는 양상은 비슷하지만 JVM warm up은 조치하여 다른 개선 방법을 고민

#### 아이데이션

1. Graal JIT

* 코드 최적화에 좋은 성능을 보이지 않을까하는 기대 → 별다른 소득 X

1. AOT(Ahead of time) Complie

* 그랄 vm을 활용해 네이티브 바이너리까지 만들 수 있는 AOT 컴파일러임
* 스프링부트에서는 아직 시험적 도입 단계라 불확실함

1. Redis Connection Pool 도입

* 레디스를 사용하는 get api 지연이 상당 수 발견되어서 고려함
* 지연 이슈 해결 안됨

1. Warm up 카운트

* warm up 카운트를 늘리고 나서야 내부재열 환경에서 응답지원 문제 해결
* 어떻게 해결됐는지 확인하기 위해서 JIT 내부 동작을 알아야 함

#### JIT Internals (JIT 내부 동작)

* Method 전체 단위 컴파일
  * JIT는 메서드 전체 단위로 컴파일 함
  * 메서드 내의 모든 바이트코드는 한꺼번에 네이티브 코드로 컴파일 됨
* 프로파일링
  * 네이티브 코드 전환 후 최적화 작업을 위해 프로파일링 정보 수집
* Tiered compilation (단계별 컴파일)
  * 단계별 컴파일을 통해 코드 최적화 진행. c1과 c2로 이루어짐
  * C1: optimization (간략 최적화)
  * C2: fully optimization (최대 최적화) → 코드 캐시에 저장하고 활용하여 코드의 신속도 향상

Compilation Level (Tiered compilation)

* Level 0: Interpreted Code:
  * 최적화 없이 기계어로 변경하는 해석 단계
* Level 1: Simple C1 compiled code:
  * 최적화가 불필요하다고 판단된 간단한 코드를 컴파일 하는 단계
  * 프로파일링 정보 수집되지 않음
* Level 2: Limited C1 compiled code:
  * 제한된 최적화를 진행.
* Level 3: Full C1 compiled code
  * 프로파일링 모드로서 정보를 수집하고 최적화를 진행
* Level 4: C2 compiled code
  * 최대 최적화를 진행함으로써 성능을 보장.

![Image](https://github.com/user-attachments/assets/87e9db9e-88cd-42af-89e7-609c847ad657)

절차:

1. Level0에서 기계어로 변환됩니다
2. C1 임계치보다 더 자주 실행되면 Level3의 최적화가 발생하고, Code Cache에 저장합니다.
3. C2 임계치보다 더 자주 실행되면 Level4의 최적화가 발생하고, Code Cache에 다시 저장합니다.

#### JIT 로그

VM options

* -XX:+UnlockDiagnosticVMOptions
  * 진단 모드 활성화
* -XX:+LogCompilation

**250회 findMax() 호출 결과**

![Image](https://github.com/user-attachments/assets/0139c3a8-db3b-4a35-b028-9fccb0645388)

* findMax()를 1회 동작시키면 c1 컴파일 임계치에 도달하지 않기 때문에 관련 로그 확인 불가능
* 레벨 3 컴파일을 위해 c1 큐에 메서드가 적재됨
* nmethod는 프로파일링 정보로서, count와 backedge\_count를 통해 임계치를 넘었는지 확인

**1000회 findMax() 호출 결과**

![Image](https://github.com/user-attachments/assets/612e1d39-9eda-4f8f-b0bc-0e0cd5c9fad6)

* findMax() 메서드가 c2 컴파일러, level 4로 최적화 되어있음

#### 결과

![Image](https://github.com/user-attachments/assets/e7280f34-a090-49df-a9d2-739132a049ee)

* warm up count가 증가할 때마다 최적화가 더 많이 진행됨
  * 그만큼 warm up 시간도 오래 걸림
* 적절한 횟수 선택
  * 시간 대비 효율적이라 판단한 250회 선택
  * 개발환경에서 지연테스트를 하였을 때, 지연문제가 발생하지 않음.





## 궁금한점.

1. 보통 DB 통신 지연이 훨씬 더 큰데도 JVM을 봤다는거면, DB통신을 극한으로 이미 최적화를 했는데도 문제가 발생해서 JVM을 건드린건가?

일반적으로 다음 순서대로 문제를 추적합니다:

1. **DB 통신 지연 (외부 I/O)**
2. **네트워크 병목**
3. **서버 리소스 부족 (CPU, Memory)**
4. **Application 내부 로직 병목**
5. **JVM 내부 최적화 상태 (JIT, GC, 코드 캐시 등)**



### 그런데 이 사례에서는?

#### 1. **DB나 네트워크 병목이 아니었음.**

* APM 로그로 확인해 보니,
  * **RDB / Redis 쿼리 지연 없음**
  * **네트워크 대역폭 여유 있음**
  * **CPU 사용률도 10% 이하로 낮음**

즉, **외부 I/O 병목 원천 배제**

***

#### 2. **Application 로직 병목도 없었음.**

* 도메인 로직 처리도 지연 시간 없음

남은 건 **"자바 코드 실행 그 자체"**, 즉 **JVM 내부 동작**밖에 없던 거예요.

***

### 그래서 JVM을 본 이유는?

바로 아래와 같은 시나리오 때문입니다:

> "실제 트래픽은 별로 안 왔는데, 서버 배포 직후 일부 요청이 느린다."\
> → 이는 CPU, DB, 네트워크 이슈가 아니라\
> **JIT 컴파일이 아직 작동하지 않아서 인터프리터로 느리게 실행되는 문제**일 가능성이 큼.

그리고 실제로:

* **JIT의 Tiered Compilation 임계치를 넘기지 못함**
* → **C2 최적화 미적용**
* → **실제 트래픽이 들어오기 전까지 느림**
* → **그래서 Warm-up을 직접 유도해야 함**



**“우리는 이미 DB, 네트워크, 로직 모두 최적화했기 때문에, 이제 JVM 자체를 Warm-up 해서 응답속도를 높이는 수밖에 없었다.” 라는 결론이 나오는 것 같습니다.**
