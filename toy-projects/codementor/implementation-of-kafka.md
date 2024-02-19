# Implementation of Kafka

## 배경 설명:

CodeMentor 서비스에서는 메세징 플랫폼이 필요했습니다.

앞서 카프카를 왜 사용하게 됬는지 이유를 얘기하기전에, 현재 서비스에 대한 간단한 배경지식을 설명해드리겠습니다.



<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>



해당 사진은 유저의 코드를 제출했을때의 데이터 파이프라인입니다.

1. 유저는 Api Gateway로 SSE 연결을 맺습니다.
2. 유저는 Api Gateway로 질문에 대한 정답 코드를 제출합니다.
3. Auth를 진행하고, URI prefix에 맞는 서버로 전달합니다(이 경우 Execute).
4. Question 서버에게 해당 문제에 대한 정보, 테스트케이스, 언어별 코드 실행을 위한 추가 코드를 요청합니다.
5. 받아온 정보를 Usercode에 저장을 하고 카프카로 전달(프로듀서  역할)  합니다.
6. Evaluation 서버는 이벤트를 리스닝(컨슈머 역할) 하면서 비동기 통신으로 각각 코드 실행 서버와 gpt api를 호출합니다.
7. 결과를 받고, 다시 카프카(프로듀서  역할)로 넣습니다.
8. &#x20;Execute 서버는 이벤트를 리스닝(컨슈머 역할)하면서 결과값을 미리 맺은 SSE 연결을 통해 전달합니다.



## 사례

아래는 _**쿠팡이츠의**_ 데이터 파이프라인입니다.

<figure><img src="../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

보시는것과 같이, Non real-time, Near real-time, Pure real-time 으로, 그리고 데이터 전처리, 데이터 후처리, 데이터 적재로 나누어져있습니다. pure real에서는 전처리가 끝나고 후처리를 하기전 카프카가 사용되는걸 볼 수 있습니다.



아래는 _**우버**_의 데이터 파이프라인입니다.

<figure><img src="../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

우버 또한 비실시간, 실시간을 나눠서 처리하고 있습니다.



마지막으로 _**넷플릭스**_입니다.

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

1차적으로 수신의 접점으로 카프카에 적재되고 있습니다.



위 3가지 사레의 공통점은, 이벤트가 발생하는 모든곳에 카프카가 사용되고 있다는 것입니다. 이를 통해, 실시간성이 중요한 유저의 코드제출쪽에 적용할 수 있을거라 생각하였습니다.



## 카프카가 왜 필요했을까?

이제 카프카를 왜 적용하게 됬는지 말씀드리겠습니다.

제일 오른쪽에 \
code Execute 서버(실제 유저의 코드를 실행시키고 정답을 확인하는 서버),\
chatGpt Turbo 3.5 Api (유저의 코드에 대해 평가받는 Api)\
가 있습니다.&#x20;

code Execute 서버에서는 언어마다 다르지만, 실행시간이 상당히 길었습니다.

제출당 파이썬은 1.53초 자바는2.98초가 나왔습니다.

메세징 큐가 없을때, 동기화식 요청을 하게 되면, 여러 요청이 한번에 들어오게 됐을때, 많은 요청들이 밀리게 되며,  codeExecute 서버가 터지거나, 요청이 누락되는 _**안정성문제가**_ 발생하기도 했습니다.



## 효과

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

이를 해결하기 위해 카프카를 사용함으로써 안전하게 메세지를 차곡차곡 쌓아둘 수 있게 되었습니다.

<figure><img src="../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

Evaluation 서버에서 카프카에 있는 메세지를 하나씩 꺼내면서 비동기 통신으로 나머지 작업들을 안전하게 처리하였습니다.

그 뒤에 15초마다 Probe를 통한 헬스체크로 인해 code Execution 파드들은 동적으로 확장되며, 많은 요청에 대해 유연하게 대처할 수 있도록 하였습니다.\
_(Probe에 대한 설명은 다른 문서에서 다루겠습니다.)_



## 결론

카프카를 적용하고 난 후에, 적어도 동시에 발생하는 요청들에 대해 신속하게 답변을 받을 수 있었고, 요청이 누락되는 상황도 없었습니다.

### 하지만...

코딩 테스트같은 특별한 상황에는 많은 요청들이 발생할텐데, 카프카에서 문제가 발생한다면, 카프카의 병렬성과 확장성을 활용해서 더 많은 토픽들을 처리할 수 있도록 설계를 하면, 적어도 카프카에서의 걱정은 하지 않아도 될것 같습니다.

