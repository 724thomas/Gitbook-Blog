---
description: 카카오톡 메시징 지표 이상감지 시스템의 개선사례 / if(kakao) 2022
---

# Improving the Anomaly Detection System for KakaoTalk Messaging Metrics

https://www.youtube.com/watch?v=j5xsDv35Nd4\&ab\_channel=kakaotech

![Image](https://github.com/user-attachments/assets/4e64a7d2-11a5-4e1d-b8e5-dc913adaae1d)

“이상이 있으면 더 빨리 알려주면 좋지 않을까?” 이렇게 문제가 발생했을때, 빠르게 인지하고, 원인을 파악하고, 재발방지할 수 있도록 구조를 개선려고함.

케이스:

문제: 말풍선 좋아요가 안됌!

예상 원인 파악:

* HBase문제?
* Grafana에서는?
* API 호출량은?
* K8s ingress?
* 타임아웃이 늘었다
* 최근 배포 확인 (커밋 로그, 릴리즈)
* 네트워크 장비, 작업 여부
* DB인가?

원인: 비즈니스 변경으로 인해 api호출이 많아졌고, mongoDB 조회 호출이 증가해, 응답이 느려져서 발생

해결: 쿼리가 몰리지 않도록 수정

재발방지: 방어 코드, 테스트 코드, 인프라 구축(지표 알림, 관리 기능)

일평균 수발신량: 백억건

트래픽: 5만 TPS

일평균 로그 용량: 6TB

* 로그 정보
  * 애플리케이션 API 호출 관련 내용: API 호출 수, 에러 응답 수, 응답시간
  * 애플리케이션 에러 관련 내용: 종류별 에러 수
  * Memchached, Redis, DB, kafka, JVM: Status 지표
  * k8s: ingress, worker node 시스템 지표
  * System: CPU, 메모리, 디스크, 네트워크 등 지표

`효율적으로 관리하는 방법 (ELK 스택 구축)`

![Image](https://github.com/user-attachments/assets/ad8faf8b-84ac-4d8c-9a95-22ef770dfdaa)

* Kafka: 안정적 메세지 수집
* Grafana: 지표 시각화
* 이상 감지 알림 시스템은 자체 구축
* 지표의 수: `API 종류 * 호출양 * 서버 수`

수천 개의 지표 중에서 비정상정 패턴 탐지 방법 **(전통적인 방법)**

* 임계치 설정 방법
  * 서비스 이해도와 경험적인 임계치 선정 필요
  * 기능 변경으로 임계치 변경이 필요한 경우 개입 필요
  * 지표가 줄어든 경우 탐지가 어려움
* 과거 비교 방법
  * 트래픽이 비슷한 지난날의 지표와 비교하여 문제 탐색
  * 비교하는 날에 장애가 발생했다면, 지표가 장애로 오염되어 판별하기 어려움

“지표가 너무 많다” → “수 천 개의 지표에서 문제 발생시 알림을 받아보자 (aka 찾아오는 알림): 문제를 찾아가는 것이 아닌, 문제가 알림으로 찾아오도록

개선 방안

1. 시계열 모델로 이상 탐지

* 이상 탐지 모델 사용
* 텍스트나 이미지 학습에 비해 쉬움 → GPU 없이 가능
* 하둡과 같은 별도의 인프라 구축 불필요
* “시간:값” 데이터이기 때문에 전처리 작업도 시계열 데이터라 단순
* 이상탐지: 데이터 세트의 예상치 못한 변경 또는 예상 패턴에서 편차를 찾아냄
  * Point Anomlay: 정상 데이터 분포로부터 벗어난 데이터
  * Contextual Anomaly: 데이터 흐름이 정상이 아닌 데이터
  * Group Anomaly: 비정상 구간

시계열 데이터 메시징 지표 특징

* 추세성 (데이터가 우상향, 하향)
* 계절성, 순환성, 불규칙성
* 메시징 지표는 평일과 주말에 변화가 있는 계절성을 갖는 지표

시계열 데이터 분석 학습 모델 && 알고리즘

* ARMIA: 방식이 단순하고, 이상치를 찾는데 효과적
* **Prophet: 직관적이며 사용이 쉽고, 속도가 빠름. Seasonality, holiday 지원**
* RNNModel: 딥러닝 LSTM, GRU 사용 예측 기반 모델
* Informer: transformer 기반으로 단점을 해결한 모델, AAAI에서 best paper 선정
* AutoEncoder: 재구성 기반, 이미지간 차이로 이상 탐지

Prophet을 먼저 구축하고, 추후에 다른 모델들도 함께 사용할 수 있도록 구축

**`자동 학습 시스템`**

![Image](https://github.com/user-attachments/assets/c80b36c5-cf22-42ae-b6b3-b90c22be9a4d)

* grafana에서 데이터를 접근할때 가로채서, 데이터를 자동 수집, 학습하고 결과를 출력하도록 구성.
* grafana에서 그래프의 데이터를 선택할때, 어떤 모델로, 어떻게 학습시킬지, 커스텀 쿼리를 놓음.
* grafana 그래프에서 outlier가 보여지도록 하였고, 이 구간을 넘어가면 알려주도록 함.
* outlier 모델이 최적화가 되지 않아서 정상 지표들이 outlier 범위를 넘어갔고, 모델을 개선하기 위한 평가 방법 설정.

**`학습 모델 평가 방법`**

![Image](https://github.com/user-attachments/assets/e6b0a444-48aa-48f5-8ffc-e56bfad1bb59)

* 오차를 기반으로 오차 수 자체가 많은지, 정상 값에서 오차가 얼마나 벌어졌는지를 평가하고 서비스에 맞게 적절하게 평가 방법에 우선순위를 두고 사용하기

**`학습 모델 교차 검증`**

![Image](https://github.com/user-attachments/assets/fa6f0200-af9c-4cb2-84a1-9f0d32ffaaee)

* 시계열 데이터에서 initial 기간을 학습 → horizon 기간을 예측 평가
* period / 2 만큼 순차적으로 이동하면서 평가 반복

왜 순차적으로 이동하면서 평가를 반복하는가?

* 테스트 데이터를 구간별로 여러개 두어서, 학습 모델을 점차적으로 수정.
* 학습 모델은 새로 만드는 것보단 기존 학습 데이터를 수정하는데 더 효율적.(기존 학습데이터는 더 많은 학습데이터로 만들어졌기때문)

**`Prophet Parameters`**

![Image](https://github.com/user-attachments/assets/9ee6766a-6188-42a8-8e1f-8e999de283d1)

* 파라미터의 조합을 통해 교차 검증을 하면서 모델 성능을 향상 시킬 수 있음

**`구현 예정`**

* 최적화된 모델을 찾아서 교체: 모델 파라미터들을 자동으로 조합하여 교차 검증을 지속적으로 함
* 어노테이션 사용: grafana에서 학습에 제외시킬 부분 / 데이터 라벨링이 필요한 경우 바로 피드백을 전달

**`결과: 기존 방식과 비교`**

![Image](https://github.com/user-attachments/assets/239073d1-cfa9-4daf-b137-0c67ad4fe03f)

* 모든 지표에서 이상 탐지를 미리 알 수 있는것을 기대
* 지표에 대한 추세를 확인해 임계치를 넘지 않는, 이상 지표라도 미리 대응

느낀점

* 기발하게 접근했다.
* 다만, “예측 모델”이기때문에, 정확하지 않으면, 다시 알림을 참고해서 문제를 식별해야할 수도 있을 것 같다.
* 대체하기에는 위험이 좀 있을 수 있을 것 같아서, 기존 알림이랑 같이 사용하면 좋을 것 같다.
