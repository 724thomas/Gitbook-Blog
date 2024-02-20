# Implementation of Kafka

## 서론

메시징 플랫폼은 현대 웹 서비스의 핵심 구성 요소 중 하나입니다. CodeMentor 서비스에서도 이러한 시스템의 중요성을 인식하고, Apache Kafka를 선택하게 된 배경에는 여러 이유가 있습니다. 본 글에서는 Kafka의 도입 이유를 설명하기에 앞서, CodeMentor 서비스의 기본 구조와 데이터 파이프라인에 대해 간략히 소개하고자 합니다.



## 배경 설명:

CodeMentor 서비스에서는 메세징 플랫폼이 필요했습니다.

앞서 카프카를 왜 사용하게 됬는지 이유를 얘기하기전에, 현재 서비스에 대한 간단한 배경지식을 설명해드리겠습니다.



<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>



해당 사진은 유저의 코드를 제출했을때의 데이터 파이프라인입니다.

**CodeMentor 서비스의 데이터 파이프라인:**

1. **유저의 상호작용**: \
   유저는 API Gateway를 통해 Server-Sent Events(SSE) 연결을 맺습니다. 이 연결을 통해 유저는 정답 코드를 제출하게 됩니다.
2. **인증 및 라우팅**: \
   제출된 코드는 먼저 인증 과정을 거친 후, URI prefix에 기반하여 적절한 서버(이 경우 Execute 서버)로 라우팅됩니다.
3. **Question 서버와의 상호작용**:\
   Execute 서버는 Question 서버에 문제 정보, 테스트케이스 및 언어별 실행 코드를 요청합니다.
4. **Kafka를 통한 데이터 처리**: \
   Usercode에 정보를 저장한 후, 이를 Kafka로 전송합니다. 여기서 Kafka는 효율적인 메시지 처리를 위한 프로듀서 역할을 합니다.
5. **Evaluation 서버의 역할**: \
   Evaluation 서버는 Kafka에서 이벤트를 리스닝하며, 코드 실행 서버와 GPT API를 비동기 방식으로 호출합니다. 이 과정에서 Kafka는 메시지 큐로서의 역할을 수행합니다.
6. **결과의 전달**: \
   처리된 결과는 다시 Kafka를 통해 Execute 서버로 전송되며, Execute 서버는 이를 리스닝하여 초기에 맺은 SSE 연결을 통해 유저에게 결과를 전달합니다.



## **산업 내 다른 사례들과의 비교**

아래는 _**쿠팡이츠의**_ 데이터 파이프라인입니다.

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

쿠팡이츠의 데이터 파이프라인은 Non real-time, Near real-time, Pure real-time 세 부분으로 나뉘어져 있으며, 데이터의 전처리, 후처리, 적재 단계에서 Kafka를 활용합니다. 특히, Pure real-time 부분에서 Kafka의 역할이 중요합니다.



아래는 _**우버**_의 데이터 파이프라인입니다.

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

우버 또한 비실시간, 실시간을 나눠서 처리하고 있습니다.



마지막으로 _**넷플릭스**_입니다.

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

1차적으로 수신의 접점으로 카프카에 적재되고 있습니다.



위 3가지 사레의 공통점은, 이벤트가 발생하는 모든곳에 카프카가 사용되고 있다는 것입니다. 이를 통해, 실시간성이 중요한 유저의 코드제출쪽에 적용할 수 있을거라 생각하였습니다.



## 카프카 도입 배경

이제 카프카의 도입에 대한 핵심 이유는 다음과 같습니다.

* **Code Execute 서버**: 이 서버는 사용자의 코드를 실행하고 정답을 확인합니다. 하지만, 언어에 따라 실행 시간이 길어지는 문제가 있었습니다. 예를 들어, Python 코드는 평균 1.53초, Java 코드는 2.98초가 소요되었습니다.
* **ChatGPT Turbo 3.5 API**: 사용자의 코드에 대한 평가를 담당합니다.

이러한 서버들의 특성상, 메시징 큐가 없는 경우 동기식 요청으로 인해 서버 과부하 및 안정성 문제가 발생했습니다.

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

##

## 효과

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

Kafka의 도입으로 안전하게 메세지를 차곡차곡 쌓아둘 수 있었고, 아래와 같은 효과를 얻었습니다:

1. **메시지 관리의 효율성**: Kafka를 사용하여 메시지를 안전하게 관리하고 순차적으로 처리할 수 있게 되었습니다.
2. **Evaluation 서버의 비동기 처리**: Evaluation 서버는 Kafka에서 메시지를 하나씩 꺼내어 나머지 작업들을 비동기 방식으로 안전하게 처리합니다.
3. **시스템의 동적 확장성**: 15초마다 실행되는 Probe를 통한 헬스체크를 통해, Code Execution 파드는 동적으로 확장되어 많은 요청을 유연하게 처리할 수 있게 되었습니다.\
   _(Probe에 대한 설명은 다른 문서에서 다루겠습니다.)_



## 결론

Kafka의 도입은 동시에 발생하는 요청들에 대해 신속한 처리를 가능하게 했으며, 요청 누락 문제를 해결했습니다.

### 특히

특히, 코딩 테스트와 같은 고부하 상황에서도 Kafka의 병렬성과 확장성을 활용하여 더 많은 토픽들을 효과적으로 처리할 수 있도록 시스템을 설계하면, Kafka 관련 문제를 최소화할 수 있을 것으로 기대됩니다.

