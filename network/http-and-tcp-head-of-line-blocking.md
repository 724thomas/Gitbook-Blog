# HTTP & TCP head of line blocking

## Head of Line Blocking(Holb)이란?

큐 또는 채널에서 먼저 처리되어야하는 데이터가 지연되면서, 뒤에 있는 데이터까지 모두 지연되는 현상. 네트워크에서는 전송 계층(TCP), 애플리케이션 계층(Http)에서 발생하며, 성능 저하의 주된 원인이 됩니다.

***

## HTTP Holb 발생 요인

HTTP/1.1에서 등장한 Keep-alive와 pipelining으로 인해, 요청들이 직렬적으로 처리되는 상황에서 발생합니다.

### Keep-alive에서 발생하는 상황

<figure><img src="../.gitbook/assets/image (417).png" alt=""><figcaption></figcaption></figure>

* 총 4개의 요청과 응답을 주고 받고 있습니다.
* 각 요청은 응답을 받은 후에 다음 요청을 보내게 됩니다.
* 요청 3에서 요청이 손실되게 되면, timeout을 기다린 후 클라이언트가 요청3을 재전송하게 됩니다.
* 요청 4이후 요청들은, 요청3의 timeout만큼 추가로 기다려야합니다.



### Pipelining에서 발생하는 상황

<figure><img src="../.gitbook/assets/image (419).png" alt=""><figcaption></figcaption></figure>

* pipelining은 응답을 받지 않고도 요청들을 계속 보낼 수 있게 해줍니다.
* pipelining에는 streamID가 존재하지 않아서, 클라이언트는 순서에 의존해서 요청/응답을 매핑합니다.
* 만약 요청 3이 손실이 나게 되면, 요청 4는 처리되더라도, 기다리게됩니다.
* 요청 1, 2는 순차적으로 처리되는대로 응답 1,2를 보내게 됩니다.
* 요청 4는, 요청3이 도착하여 처리된 후에, 전송됩니다.
* 응답 3, 4를 순차적으로 보내면 클라이언트는 무조건 순차적으로 받게됩니다. (TCP는 바이트 스트림의 순서를 보장하기 때문)

***

## HTTP Holb 해결책

### HTTP/2 멀티플랙싱 도입

HTTP/2는 하나의 TCP 커넥션 내에서 여러개의 스트림(논리적 연결 단위)을 생성하고, 각각의 스트림을 프레임 단위로 쪼개어 병렬로 전송할 수 있습니다.

<figure><img src="../.gitbook/assets/image (420).png" alt=""><figcaption></figcaption></figure>

* 각 요청/응답은 독립된 StreamID를 가짐
* 요청과 응답은 Frame 단위로 쪼개져 전송되며, 중간에 교차도 가능
* 서버는 순서와 상관없이 응답 가능
* 응답 순서에 제약이 없어지고, 하나의 요청이 느려도, 다른 요청은 독립적으로 응답이 가능해집니다.

***

## TCP Holb 발생 요인

TCP는 전송 계층에서의 연결 지향적 프로코롤로, 데이터 순서를 보장하며, 이 부분으로 인해 Holb가 발생합니다.

<figure><img src="../.gitbook/assets/image (424).png" alt=""><figcaption></figcaption></figure>

* TCP 수신자는 순서를 지키기 위해 중간 패킷이 오기를 기다립니다.
* 전송 계층에서 발생하며, 패킷 단위입니다.
* 손실률이 높은 네트워크에서 자주 발생하게 됩니다.
* 각 응답들은 4개의 패킷을 갖고 있습니다.
* 응답1의 2번째 패킷이 손실이 발생하면, 3, 4패킷은 애플리케이션에 전달되지 않고 버퍼에서 대기하게 됩니다.
* &#x20;응답1의 2번째 패킷이 도착하면, 2,3,4 패킷을 애플리케이션에 전달합니다.

***

## TCP Holb 해결책

### HTTP/3.0 QUIC 도입

QUIC는 구글이 개발한 프로토콜로, UDP 기반으로 TCP의 단점을 극복하기 위해 만들어졌습니다. 스트림 단위의 독립 전송을 지원하며, TCP의 순서 보장 강제성과 Holb를 회피할 수 있습니다.

<figure><img src="../.gitbook/assets/image (425).png" alt=""><figcaption></figcaption></figure>

* 하나의 연결 안에 여러 스트림 존재(논리적)
* 각 스트림은 독립적인 순서로 데이터 전송 가능
* 스트림 A의 데이터 손실이 스트림 B에 영향을 주지 않음
* TCP의 순서 보장으로 인한 Holb 제거
* 동시에 여러 요청을 처리하는 구조에서 병목 최소화



## 결론

<table data-full-width="true"><thead><tr><th>항목</th><th>HTTP Holb</th><th>TCP Holb</th></tr></thead><tbody><tr><td>단위</td><td>요청</td><td>패킷</td></tr><tr><td>발생 계층</td><td>애플리케이션 계층</td><td>전송 계층</td></tr><tr><td>원인</td><td>직렬 응답 처리, 응답 순서 보장</td><td>순서 보장 중간 패킷 손실</td></tr><tr><td>기다리는 쪽</td><td>클라이언트 (응답 대기)</td><td>수신자 (패킷 재전송 대기)</td></tr><tr><td>해결 방법</td><td>HTTP/2 멀티플랙싱</td><td>HTTP/3 QUIC</td></tr></tbody></table>

