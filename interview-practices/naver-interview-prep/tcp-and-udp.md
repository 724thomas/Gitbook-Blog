# TCP & UDP

TCP와 UDP의 차이점을 설명해주세요.

| 항목   | TCP                  | UDP               |
| ---- | -------------------- | ----------------- |
| 연결   | 연결형(3-way handshake) | 비연결형              |
| 신뢰성  | 보장 (재전송, 순서 정렬)      | 비보장 (손실 시 무시)     |
| 속도   | 상대적으로 느림             | 빠름                |
| 제어   | 흐름/혼잡 제어 있음          | 없음                |
| 사용 예 | HTTP, FTP, SMTP      | DNS, VoIP, 스트리밍 등 |



TCP에서 흐름 제어(flow control)와 혼잡 제어(congestion control)는 각각 어떤 문제를 해결하기 위한 것이며, 어떤 방식으로 동작하나요?

* **흐름 제어 (Flow Control)**:
  * 목적: **수신 측의 처리 능력 초과 방지**
  * 방법: **Receiver Window** 를 이용해 수신 가능한 byte 수를 송신 측에 전달
* **혼잡 제어 (Congestion Control)**:
  * 목적: **네트워크 혼잡 방지**
  * 알고리즘: Slow Start, Congestion Avoidance, Fast Retransmit, Fast Recovery



TCP 혼잡 제어에서 사용하는 알고리즘 중, Fast Retransmit와 Fast Recovery는 어떤 상황에서 동작하며, 어떤 차이점이 있나요?

* **Fast Retransmit**:
  * 3중 중복 ACK 수신 시, **timeout을 기다리지 않고** 손실된 패킷을 즉시 재전송
  * 윈도우 조절은 하지 않음
* **Fast Recovery**:
  * Fast Retransmit 직후, 혼잡 윈도우를 **절반으로 줄이고**, 선형적으로 증가
  * 네트워크 전체 속도를 다시 올리기 위한 중간 단계



Slow Start, Congestion Avoidance, Fast Retransmit, Fast Recovery는 서로 어떤 관계를 갖고 있고, 어떤 흐름으로 전이되나요? 각각의 역할도 설명해주세요.

1. **Slow Start**: 윈도우를 1 → 2 → 4 → 8... 두 배씩 증가 (지수적)
2. **Congestion Avoidance**: `ssthresh` 도달 시, 1씩 선형 증가
3. **패킷 손실**:
   * **Timeout 발생**: Slow Start로 복귀 (`ssthresh` 절반 설정, cwnd=1)
   * **중복 ACK 3개**: Fast Retransmit → Fast Recovery
4. **Fast Recovery**: cwnd 절반으로 줄이고 선형 증가 → ACK 정상 도착 시 Congestion Avoidance로 전이



