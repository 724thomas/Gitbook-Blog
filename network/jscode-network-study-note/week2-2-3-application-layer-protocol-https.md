---
description: HTTPS
---

# Week2(2/3) Application Layer Protocol - HTTPS

## HTTPS에 대해서 설명해주세요

* HTTP에 \*\*TLS(Transport Layer Security)\*\*를 추가한 프로토콜로, **데이터의 기밀성, 무결성, 인증**을 보장
* **TLS Handshake**를 통해 클라이언트와 서버 간에 안전한 암호화 통신을 위한 세션 키를 협상



## TLS가 뭔가요? (Transport Layer Security)

* 클라이언트와 서버 간의 안전한 통신을 보장하기 위해 사용되는 프로토콜
* **기밀성, 무결성, 인증**을 보장
* ClientHello, ServerHello, 서버의 공개키 -> Pre-Master-Key -> Master Key -> Session Key



## 대칭키 암호화 방식에 대해 설명해주세요.

* 클라이언트와 서버가 동일한 암호화 키를 사용하여 데이터를 암호화하고 복호화하는 방식
* 매우 빠르고 효율적이기 때문에, 대량의 데이터를 처리할 때 자주 사용
* **단점**은 키를 안전하게 전달하는 것이 어렵다는 점



## 비대칭키(공개키) 암호화 방식에 대해서 설명해주세요

* \*\*공개키(Public Key)\*\*와 \*\*개인키(Private Key)\*\*를 사용하는 암호화 방식
* 서버는 자신의 **공개키**를 클라이언트에게 전달하고, 클라이언트는 이 공개키를 사용하여 데이터를 암호화합니다. 암호화된 데이터는 서버가 자신의 **개인키**로만 복호화할 수 있습니다.



## 전자 서명에 대해 설명해주세요

* 비대칭 암호화 방식을 사용하여 **데이터의 무결성과 신원 인증**을 보장하는 기술
* **서명 과정**: 서명자가 데이터를 해시하여 \*\*해시 값(Hash Value)\*\*을 생성한 후, 이를 자신의 \*\*개인키(Private Key)\*\*로 암호화하여 전자 서명을 만듭니다. 이 전자 서명은 데이터를 서명한 사람의 신원을 증명하는 역할을 합니다.
* **검증 과정**: 수신자는 서명자의 \*\*공개키(Public Key)\*\*로 전자 서명을 복호화하여 해시 값을 복원한 후, 원본 데이터로 새로운 해시 값을 계산합니다. 두 해시 값이 동일하면 데이터가 전송 중에 변조되지 않았으며, 서명자가 데이터를 생성했음을 확인할 수 있습니다.



## SSL 핸드쉐이크에 대해 설명해주세요

* 클라이언트와 서버는 먼저 **TCP 연결**을 설정합니다. 이 후 TLS Handshake가 시작
* 클라이언트는 서버에게 **ClientHello** 메시지를 보냅니다.
* 서버는 **ServerHello** 메시지로 응답하고, 서버의 \*\*인증서(Certificate)\*\*와 서버 측 난수를 함께 클라이언트에게 전송
* 클라이언트는 받은 인증서의 **유효성**을 확인합니다. 클라이언트는 **CA의 공개키**로 인증서의 서명을 검증
* 클라이언트는 **Pre-Master Secret**을 생성하고, 이를 서버의 **공개키**로 암호화하여 서버로 전송. 서버는 자신의 **개인키**로 이를 복호화
* 클라이언트와 서버는 각각 자신이 가진 **Pre-Master Secret**과, 서로 교환한 \*\*난수(ClientHello 및 ServerHello의 랜덤 값)\*\*를 사용해 **Master Secret**을 생성
* 클라이언트와 서버는 Master Secret을 바탕으로 \*\*세션 키(대칭키)\*\*를 생성


