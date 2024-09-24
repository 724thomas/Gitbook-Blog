---
description: Domain Name System
---

# Week2(3/3) Application Layer Protocol - DNS

## DNS가 뭔가요?

* 도메인 이름을 IP 주소로 변환해주는 분산형 데이터베이스 시스템
* 도메인 이름을 IP주소로 변환해줍니다.
* 로컬 호스트 파일과 브라우저 캐시를 확인하여 해당 도메인의 IP 주소가 있는지 확인
* 재귀적(recursive, dns resolver) 또는 반복적(iterative) 방식으로 이루어집니다.
* 재귀적 방식은 하나의 DNS 서버가 다른 DNS 서버로 계속 질의를 넘기면서 최종 IP 주소를 반환하는 방식이고, 반복적 방식은 클라이언트가 직접 각 DNS 서버에 차례대로 질의하는 방식
* A: IPv4, AAAA: IPv6, CNAME: 다음 DNS주소



## DNS 작동 방식

* 사용자가 도메인 이름을 입력합니다
* 로컬 호스트 파일과 운영체제의 DNS 캐시를 확인합니다.
* DNS 리졸버에 질의를 보냅니다.
* DNS리졸버는 각각 Recursive, Iterative 방식으로 질의가 되며, 결과 IP를 반환합니다.
* (브라우저 캐시는 사용X. 단순히 웹페이지의 리소스를 저장하는 공간)
* 둘다 혼합해서 사용. 브라우저 입장에서는 Recursive가 되지만, DNS리졸버 입장에서는 Iterative가 된다.

## DNS 질의의 종류에 대해 설명해주세요

Iterative

* 루트 DNS 서버에 질의 후, Top level Domain 서버의 주소를 제공
* 리졸버는 **TLD 서버**에 질의하여 해당 도메인의 **권한 있는 DNS 서버**의 주소를 받습니다.
* 마지막으로 **권한 있는 DNS 서버**에 질의하여 최종 **IP 주소**를 받습니다.

Recursive

* **클라이언트가 로컬 DNS 리졸버**에게 질의를 보내면, 로컬 리졸버가 대신 **재귀적으로 질의를 진행**하는 방식
* 클라이언트는 한 번의 요청만 보내고, **로컬 리졸버가 루트 DNS → TLD DNS → 권한 있는 DNS 서버**에 재귀적으로 질의를 보내고, **최종 IP 주소를 받아 클라이언트에게 반환**



## DNS 계층의 이름들이 뭔가요?

* Root - 가장 상단
* TLD(Top Level Domain) - Root 다음
* SLD(Second Level Domain)
* Subdomain
* Host순.
* Authoritative Name Server - IP를 반환할 수 있어서 더 이상 Iterative 안해도되는 서버.

## DNS 서버에서 IP 주소를 요청할 때, 왜 UDP를 사용하나요?

* 간단한 요청-응답 패턴이고 속도와 효율성 떄문 (핸드쉐이크X)
* 응답을 못받거나, 응답데이터가 512바이트를 초과할때 TCP로 전환



## DNS 레코드가 뭔가요?

* DNS 서버에 저장되는 도메인 이름과 관련된 정보를 저장하는 데이터 항목
* 특정 도메인 이름에 어떤 IP 주소가 할당되어 있는지(A Record), 또는 다른 도메인으로 리다이렉트할지(CNAME Record) 등 여러 가지 정보를 관리하는 것이 **DNS 레코드**



