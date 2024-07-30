# https://www.google.com

## https://www.google.com을 브라우저에 입력하면 무슨 일이 발생할까?

**!!! GSLB Glover Server Load Balancer**

## 1. IP 가져오기

브라우저는 도메인 이름을 IP 주소로 변환하기 위해 다음 과정을 거칩니다.

### 1.1 애플리케이션 계층

1. 브라우저 캐시 확인: 브라우저 캐시에 최근 방문한 웹사이트의 DNS 정보가 있는지 확인합니다.
2. 운영체제의 로컬호스트 파일 확인: 시스템콜을 호출 하여 운영체제의 호스트 파일을 확인합니다.
3. 운영체제의  DNS 캐시 확인: 시스템 콜을 호출 하여 브라우저는 OS 캐시(OS의 네트워크에스택)에 도메인 IP 주소를 요청합니다.
4. ISP의 DNS 리졸버(또는 루트DNS) ip주소를 요청합니다.
5. 해당 DNS 리졸버(또는 루트DNS)에 캐시가 존재하지 않을 경우, 아래 두 방법중 하나로 IP를 가져옵니다.

<details>

<summary>Iterative Query</summary>

1. 브라우저가 직접 루트 DNS 서버에 www.google.com의 ip 주소를 묻습니다.
2. 루트 DNS 서버는 .com TLD(Top-level-domain) 서버의 주소를 브라우저에게 반환합니다.
3. 브라우저가 .com TLD 서버에 www.google.com의 ip주소를 묻습니다.
4. .com TLD 서버는 www.google.com의 권한이 있는 DNS 서버 주소를 브라우저에게 반환합니다.
5. 브라우저가 해당 DNS 서버에 www.google.com의 ip주소를 묻습니다.
6. 해당 DNS 서버가 www.google.com의 ip 주소를 브라우저에게 반환합니다.

</details>

<details>

<summary>Recursive Query</summary>

여기에 숨기고 싶은 내용을 입력하세요. 마크다운 문법도 사용할 수 있습니다.

* 브라우저가 www.gooogle.com에 대한 리커시브 쿼리를 DNS 리졸버에 보냅니다.
* DNS 리졸버는 캐시에 없다면 루트 DNS 서버에 www.google.com 의 ip주소를 묻습니다.
* 루트 DNS 서버는 .com TLD 서버의 주소를 리졸버에게 반환합니다.
* 리졸버는 .com TLD 서버에 www.google.com의 ip주소를 묻습니다.
* .com TLD 서버는 www.google.com의 권한이 있는 DNS 서버 주소를 리졸버에게 반환합니다.
* 리졸버가 해당 DNS 서버에 www.google.com의 ip주소를 묻습니다.
* 해당 DNS 서버가 www.google.com의 ip 주소를 리졸버에게 반환합니다.
* 리졸버는 최종 ip 주소를 브라우저에게 반환합니다.

</details>



## 2. TCP/IP 연결.

### 2.1 3way handshake

1. 클라이언트에서 ISN(Initial Sequence Number)가 포함된  SYN 패킷(예: <mark style="color:red;">2341</mark>)을 서버에 보냅니다.
2. 서버는 클라이언트의 SYN 패킷(<mark style="color:red;">2341</mark>)을 수신하고, 요청을 수락한다는 의미로 자신의 ISN이 포함된 SYN(예:<mark style="color:blue;">9853</mark>)과,  ACK 패킷(예: <mark style="color:red;">2342</mark>)을 클라이언트에게 전송합니다.
3. 클라이언트는 서버의 SYN-ACK를 수신하고, 서버에게 ACK(예:<mark style="color:blue;">9854</mark>) 패킷을 전송합니다.

### 2.2 TLS 핸드쉐이크

1. 클라이언트가 서버와의 연결을 시작하기 위해 'ClientHello(Client Random값포함)' 메세지를 보냅니다.
2. 서버는 'ClientHello'를 수신하고, 암호화 방법을 선택하여 'ServerHello(Server Random값 포함)'를 보냅니다.
3. 서버는 'ServerHello' 메세지 이후에 'Certificate' 메세지를 전송합니다. 서버의 인증서(<mark style="color:blue;">**공개키, 서버 정보, 인증기관(CA)서명)**</mark> 가 포함되어있습니다.
4. 브라우저는 서버에서 받은 인증서를 검증하는 과정을 거칩니다.
5. 유효하다고 판단되면 브라우저는 Pre-Master-Secret 값을 생성하고, 서버에서 받은 공개 키로 암호화하여 서버로 보냅니다.
6. 서버는 클라이언트에게 받은 Pre-Master-Secret 값을 비밀키로 복호화합니다.
7. 클라이언트와 서버는 'Client Random', 'Server Random', 'Pre-Master-Secret'을 통하여 'Master Secret'을 생성합니다.
8. 클라이언트와 서버는 'Master Secret'값을 기반으로 세션키를 생성합니다.



## 3. 네트워크를 통한 데이터 전송

### 3.1 HTTP 요청 생성

1. 애플리케이션 계층에서 HTTP 요청이 생성됩니다.
2. 전송 계층에서 HTTP 요청이 TCP 세그먼트로 캡슐화 됩니다.
3. 네트워크 계층에서 TCP 세그먼트가 IP 패킷으로 캡슐화 됩니다.
4. 데이터 링크 계층에서 IP 패킷이 이더넷 프레임으로 캡슐화 됩니다.
5. 물리 계층에서 클라이언트의 네트워크 인터페이스 카드(NIC)에서 이더넷 프레임을 물리적 신호로 변환하여 네트워크(스위치)로 전송합니다.

### 3.2 네트워크 전송

1. 클라이언트의 NIC로 부터 전송된 바이트코드를 스위치가 수신하고 액세스 스위치로 전달됩니다.
2. 스위치가 데이터 링크 계층에서 이더넷 프레임을 확인하고, 프레임의 목적지 MAC주소를 기반으로 적절한 포트로 전달합니다.
3. 스위치는 MAC 주소 테이블을 사용하여 목적지 MAC 주소가 연결된 포트를 찾습니다.
4. 목적지 MAC주소가 MAC주소 테이블에 없으면 스위치는 프레임을 네트워크의 모든 포트로 브로드캐스트합니다.
5. 스위치가 프레임을 라우터로 전달합니다.
6. 라우터에서 비캡슐화를 거쳐서 네트워크 계층에서 헤더를 열어보고 IP 주소를 확인합니다.
7. 라우팅 테이블을 확인하여 목적지 네트워크와 다음 홉 정보를 확인합니다.
8. 다시 캡슐화를 거쳐서 바이트 코드를 다음 홉으로 전달합니다.
9. 최종 목적지 네트워크의 라우터로부터 프레임을 수신하고 서버로 전달합니다.

### 3.3 서버측로드 밸런서

1. 로드밸런서는 스위치로부터 수신한 이더넷 프레임을 비캡슐화하여 IP를 추출하게 됩니다.
2. 로드밸런서는 로드밸런싱 알고리즘(라운드로빈, 최소연결,  최소응답시간 등)을 통해 요청을 보낼 서버를 선택합니다.
3. 선택된 서버의 IP주소와 MAC주소를 사용하여 IP 패킷을 새로운 이더넷 프레임으로 캡슐화하고 서버로 전달합니다.
4. HTTP 요청 생성 과정과 반대의 절차를 거쳐서 HTTP 요청을 받습니다.

## 4. 서버 요청 처리

SpringBoot에서 요청이 들어온 경우를 가정합니다.

1. HTTP 요청을 수신하게 되면, WAS(내장 웹 애플리케이션 서버)에서 요청을 받습니다.
2. 요청을 서블릿 컨테이너로 전달하게 되고, 서블릿 컨테이너는 요청 URL에 따라 해당 서블릿으로 요청을 라우팅합니다. (스프링 부트는 기본 DispatcherServlet)
3. DispatcherServlet에 도착하기 전에, 요청은 필터 체인을 거치게되는데, 요청과 응답을 변형하거나 요청을 차단하는 등의 역할(예: 인증, 로깅)을 수행하게 됩니다.
4. 필터를 거치고 Dispatcher Servlet이 요청을 처리하는 과정에서 인터셉터(예: 인증, 인가, 로깅 등)가 호출됩니다.
5. 인터셉터를 거치고 Dispatcher Servlet은 HandlerMapping을 통해 요청 URL에 맞는 핸들러(컨트롤러)를 찾고, HandlerAdapter를 사용하여 핸들러를 실행합니다.
6. 핸들러가 실행되기 전 인터셉터(preHandle)이 실행됩니다.
7. 컨트롤러는 요청을 처리하고 서비스 레이어에 로직 수행을 위임합니다.
8. 서비스 레이어에서 받은 결과값을 받고 인터셉터(postHandle)이 실행됩니다.
9. ViewResolver를 통해 뷰로 반환이되고 인터셉터(afterCompletion)이 실행됩니다.
10. 클라이언트로 전송됩니다.

