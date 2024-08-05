---
description: TLS핸드쉐이크
---

# TLS Handshake

<figure><img src="../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

## 1. ClientHello

클라이언트가 서버와의 연결을 시작하기 위해 'ClientHello' 메시지를 보냅니다. Client Random(무작위 값)이 포함됩니다.

## 2. ServerHello

서버는 'ClientHello' 메시지를 수신한 후, 다음 단계로 'ServerHello' 메시지를 클라이언트에게 보냅니다. 이 메시지에는 서버가 선택한 암호화 방식과 무작위 바이트 문자열(Server Random)이 포함됩니다.

## 3. Server Certificate

서버는 'ServerHello' 메시지 이후에 'Certificate' 메시지를 전송합니다. 이 메시지에는 서버의 인증서가 포함되어 있습니다.

* 서버 인증서: 공개 키, 서버 정보, 인증 기관(CA)의 서명

## 4. ServerHelloDone

서버는 'ServerHelloDone' 메시지를 전송하여 초기 핸드셰이크 메시지를 마무리합니다.

## 5. ClientKeyExchange

클라이언트는 Pre-Master-Secret 값을 생성하고, 서버에서 받은 공개 키로 암호화하여 'ClientKeyExchange' 메시지로 서버에 전송합니다.

* **Pre-Master-Secret**: 클라이언트와 서버가 공유하는 비밀 값으로, 이를 통해 최종 암호화 키(Master Secret)를 생성합니다.

## 6. Master Secret 생성

클라이언트와 서버는 'Client Random', 'Server Random', 'Pre-Master-Secret' 값을 사용하여 'Master Secret'을 생성합니다. 이 값은 이후의 통신을 암호화하는 데 사용되는 세션 키를 생성하는 데 사용됩니다.

## 7. Session Key 생성

클라이언트와 서버는 'Master Secret' 값을 기반으로 세션 키를 생성합니다. 이 세션 키는 대칭 암호화 알고리즘을 사용하여 데이터를 암호화하고 복호화하는 데 사용됩니다.



## TLS 핸드쉐이크 완료 후의 통신

핸드쉐이크가 완료된 후 클라이언트와 서버 간의 통신은 다음과 같은 방식으로 진행됩니다:

* **암호화된 데이터 전송**: 클라이언트와 서버는 세션 키를 사용하여 데이터를 암호화하고 전송합니다. 이로 인해 데이터가 도청되거나 변조되는 것을 방지할 수 있습니다.
* **데이터 무결성 확인**: 전송된 데이터는 메시지 인증 코드(MAC)를 사용하여 무결성을 확인할 수 있습니다. 이를 통해 데이터가 전송 중에 변경되지 않았음을 보장할 수 있습니다.

클라이언트와 서버가 세션 키를 사용하여 안전하게 통신하는 예를 간단히 설명하면 다음과 같습니다:

1. 클라이언트가 서버에 데이터를 전송할 때:
   * 데이터를 세션 키로 암호화합니다.
   * 암호화된 데이터를 서버로 전송합니다.
2. 서버가 클라이언트로부터 데이터를 수신할 때:
   * 수신된 데이터를 세션 키로 복호화합니다.
   * 복호화된 데이터를 처리합니다.

이 과정을 통해 클라이언트와 서버는 안전하게 데이터를 주고받을 수 있습니다.



