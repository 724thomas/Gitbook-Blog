---
description: TCP 3, 4 way handshake
---

# 2.2.1-1 TCP 3, 4 way handshake

#### TCP 연결 설정 및 종료: 3-Way Handshake와 4-Way Handshake

TCP(Transmission Control Protocol)는 신뢰성 있는 데이터 전송을 위해 연결 지향 방식을 사용합니다. 이를 위해 TCP는 연결을 설정하는 과정에서 3-Way Handshake, 연결을 종료하는 과정에서 4-Way Handshake를 사용합니다.

**TCP 연결 설정: 3-Way Handshake**

3-Way Handshake는 TCP 연결을 설정하기 위해 세 단계로 이루어진 절차입니다. 이 과정은 클라이언트와 서버 간의 연결을 안전하게 설정하고 데이터 전송을 시작하는 데 사용됩니다.

1. **SYN (Synchronize)**:
   * 클라이언트는 서버에 연결 요청을 보냅니다. 이 요청은 SYN 플래그가 설정된 세그먼트를 포함합니다.
   * 클라이언트는 초기 순서 번호(Initial Sequence Number, ISN)를 설정합니다.
2. **SYN-ACK (Synchronize-Acknowledge)**:
   * 서버는 클라이언트의 요청을 수신하고 이를 수락합니다. 서버는 SYN과 ACK 플래그가 모두 설정된 세그먼트를 클라이언트에게 보냅니다.
   * 서버는 자신의 ISN을 설정하고 클라이언트의 ISN에 대한 ACK를 보냅니다.
3. **ACK (Acknowledge)**:
   * 클라이언트는 서버의 응답을 수신하고, 서버의 ISN에 대한 ACK를 보냅니다.
   * 이로써 TCP 연결이 설정되고, 클라이언트와 서버는 데이터를 주고받을 준비가 완료됩니다.

**예시: 3-Way Handshake**

1. 클라이언트 -> 서버: SYN, Seq=X
2. 서버 -> 클라이언트: SYN, ACK, Seq=Y, Ack=X+1
3. 클라이언트 -> 서버: ACK, Seq=X+1, Ack=Y+1

**TCP 연결 종료: 4-Way Handshake**

4-Way Handshake는 TCP 연결을 종료하기 위해 네 단계로 이루어진 절차입니다. 이 과정은 클라이언트와 서버 간의 연결을 안전하게 종료하고, 모든 데이터 전송이 완료되었음을 보장합니다.

1. **FIN (Finish)**:
   * 클라이언트는 서버에게 연결을 종료하겠다는 요청을 보냅니다. 이 요청은 FIN 플래그가 설정된 세그먼트를 포함합니다.
2. **ACK (Acknowledge)**:
   * 서버는 클라이언트의 요청을 수신하고 이를 수락합니다. 서버는 FIN 세그먼트를 수신했다는 ACK를 클라이언트에게 보냅니다.
   * 이로써 클라이언트는 더 이상 데이터를 보내지 않지만, 서버는 여전히 데이터를 보낼 수 있습니다.
3. **FIN (Finish)**:
   * 서버는 클라이언트에게 연결을 종료하겠다는 요청을 보냅니다. 이 요청은 FIN 플래그가 설정된 세그먼트를 포함합니다.
4. **ACK (Acknowledge)**:
   * 클라이언트는 서버의 요청을 수신하고 이를 수락합니다. 클라이언트는 FIN 세그먼트를 수신했다는 ACK를 서버에게 보냅니다.
   * 이로써 TCP 연결이 완전히 종료됩니다.

**예시: 4-Way Handshake**

1. 클라이언트 -> 서버: FIN, Seq=X
2. 서버 -> 클라이언트: ACK, Seq=Y, Ack=X+1
3. 서버 -> 클라이언트: FIN, Seq=Y+1
4. 클라이언트 -> 서버: ACK, Seq=X+1, Ack=Y+1
