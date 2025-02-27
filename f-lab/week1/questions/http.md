# HTTP

**서버와 클라이언트의 통신 과정을 HTTP 프로토콜을 기반으로 설명하세요.**

클라이언트는 IP를 얻기 위해 DNS에 조회를 합니다.

* Hosts 파일, 브라우저 캐시, OS 캐시 확인
* 로컬 DNS 리졸버에 재귀적 질의를 요청(로컬 DNS 리졸버는 반복적 질의를 진행)
* 로컬 DNS 리졸버는 루트 DNS 서버로 부터 TLS 주소를 받음
* 로컬 DNS 리졸버는 TLS DNS 서버로 부터 Authoritative DNS 서버의 주소를 받음
* 로컬 DNS 리졸버는 Authoritative DNS 서버에서 Ip 주소를 받음.
* 로컬 DNS 리졸버는 IP를 브라우저에 반환.

서버와 연결을 합니다 (3 way handshake, SSL/TLS handshake)

* 클라이언트 -> 서버: syn
* 서버 -> 클라이언트: syn, ack
* 클라이언트 -> 서버: ack
* 클라이언트 -> 서버: Client Hello
* 서버 -> 클라이언트: 서버의 인증서, 서버의 공개 키, Server Hello
* 클라이언트 -> 서버: 인증서 유효 확인, Client Hello + Server Hello + 서버의 공개키 = pre-master-secert 전송
* 서버, 클라이언트: pre-master-secret을 통해 세션키를 생성.

HTTP 요청을 서버에 전송하고 응답을 받습니다.

* 클라이언트 -> 서버: 요청 메서드, URL, 헤더 정보를 포함한 요청을 보냄
* 서버 -> 클라이언트: 요청에 대한 응답을 생성 후, 상태 코드와 처리 결과를 전송

클라이언트가 응답을 해석하여 사용자에게 표시.



1. HTTP/1.1과 HTTP/2의 차이점은 무엇인가요?

* HTTP/1.1: Keep-alive, 파이프라이닝, chunk transfer, caching(etag)
* HTTP/2.0: Binary Framing Layer, 멀티플랙싱, Stream Prioritization, Server Push, HPack compression

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

2. HTTP 요청과 응답의 헤더 필드 중 중요한 항목을 설명하세요.

* **요청(Request) 헤더**:
  1. **Host**: 서버의 호스트 이름 및 포트 (예: `Host: www.example.com`).
  2. **User-Agent**: 클라이언트의 정보 (브라우저, OS 등).
  3. **Accept**: 클라이언트가 처리 가능한 MIME 타입 (예: `application/json`).
  4. **Authorization**: 인증 정보 전달 (예: Bearer 토큰).
  5. **Content-Type**: 요청 바디의 데이터 타입 (예: `application/json`).
* **응답(Response) 헤더**:
  1. **Content-Type**: 응답 데이터의 타입 (예: `text/html`).
  2. **Content-Length**: 응답 바디의 크기.
  3. **Cache-Control**: 캐싱 정책 설정 (예: `no-cache`).
  4. **Set-Cookie**: 클라이언트에 설정할 쿠키 정보.
  5. **Access-Control-Allow-Origin**: CORS 관련 헤더.



3. HTTP 상태 코드 중 4xx와 5xx의 차이를 설명하세요.

4xx: 클라이언트의 오류

* 400(Bad request) : 잘못된 요청 문법
* 401 (Unauthroized): 인증이 필요
* 403 (Forbidden): 권한이 없음
* 404 (Not Found): 리소스가 존재하지 않음

5xx: 서버 오류

* 500 (Internal Server Error): 서버 내부 오류
* 502 (Bad Gateway): 게이트웨이 또는 프록시 서버의 오류
* 503 (Service Unavailable): 서버가 일시적으로 사용 불가능



4. HTTPS가 HTTP보다 안전한 이유는 무엇인가요?

SSL/TLS 암호화 계층이 추가된 프로토콜입니다.

* 데이터 암호화: 네트워크 상에서 도청 방지
* 데이터 무결성: 데이터가 변경되거나 손상되지 않음을 보장
* 서버 인증: 신뢰할 수 있는 서버와 통신중인 것을 증명
* Man-in-the-middle 공격 방지: 데이터를 중간에서 가로채거나 변조하는 것을 차단.



5. HTTP/3에서 QUIC 프로토콜이 도입된 이유는 무엇인가요?

* TCP의 Head-of-line blocking

HTTP/2.0에서 멀티플랙싱이 도입되었지만 TCP기반의 프로토콜에서 순서 보장 메커니즘으로 인해 Head-of-line blocking이 발생. 단일 스트림에서 여러 패킷들을 받는 과정에서, 패킷이 손상되면 다른 요청들의 처리가 차단됩니다. 여러 스트림을 열어서 독립적으로 요청이 처리되어 Head-of-line blocking을 해결

* 0-RTT

클라이언트는 한번 연결한 서버의 세션 티켓, 사전 공유키를 저장합니다. 두번째 연결시, 이를 재활용하여 인증 정보와 데이터를 같이 보내게 됩니다

* 패킷 손실 복구

손상된 패킷에 대해서만Selective Ack를 보냅니다.

* 헤더 압축

Hpack보다 더 효율적인 QPack을 사용

