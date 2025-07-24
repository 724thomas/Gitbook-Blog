---
description: 보안
---

# Security

**Spring Security**

## Spring Security란 무엇이며, 어떤 기능을 제공하나요?

* 스프링 시큐리티는 자바 애플리케이션에서 인가와 인증을 쉽게 구현할 수 있게 해주는 시큐리티 프레임워크입니다.
* 인증(Authentication): 누구?
* 인가(Authorization): 어떤 리소스에 접근 가능?
* OAuth2, JWT 등의 최신 보안 기법과 쉽게 통합 가능



## Spring Boot에서 Spring Security를 활용하여 로그인을 구현하는 방법을 설명해주세요.

spring-boot-starter-security 의존성을 추가하여 사용





**JWT와 OAuth2**

## JWT(JSON Web Token)란 무엇이며, 어떻게 인증에 활용되나요?

**JSON 포맷**을 사용해 정보를 안전하게 전송하기 위한 **토큰**.&#x20;

HTTP의 **stateless** 환경에서 **사용자 인증**과 **인가**를 처리할 때 주로 사용됩니다. 세션 방식과 달리 서버 측에서 세션을 저장하지 않고, 클라이언트가 JWT를 통해 인증 정보를 관리하므로 **서버의 부담을 줄일 수 있는 장점이 있습니다.**

1. **로그인 요청시 서버는 인증 정보를 확인 후 JWT를 생성하여 클라이언트에게 반환**
2. **클라이언트는 JWT를 로컬 스토리지나 세션 스토리지 등에 저장**
3. **이후 클라이언트는 요청을 보낼떄마다 HTTP헤더의 Authorization 필드에 포함시켜 서버로 전송**

* 장점: Stateless, 확장성
* 단점: 토큰 탈취 위험 -> Redis같은 인메모리 DB를 사용하여 블랙리스트 등을 관리



## OAuth2란 무엇이며, 어떤 인증 방식을 제공하나요?

* **제3자 서비스**가 사용자 정보를 접근할 수 있도록 허가하는 **인증 및 인가 프레임워크**
* 백엔드 중심 구조 기준으로



1. <mark style="color:purple;">**유저**</mark>가 애플리케이션에서 "Google로 로그인" 버튼을 클릭함
2. <mark style="color:purple;">**유저**</mark>의 브라우저는 <mark style="color:green;">**클라이언트**</mark>에 로그인 요청을 보냄
3. <mark style="color:green;">**클라이언트**</mark>는 <mark style="color:purple;">**유저**</mark>에게 `302 Redirect` 응답을 보내며 <mark style="color:red;">**인증서버**</mark>**로 리디렉션**
4. <mark style="color:purple;">**유저**</mark>는 <mark style="color:red;">**인증서버**</mark>에서 로그인 및 권한 동의를 진행함
5. <mark style="color:red;">**인증서버**</mark>는 **Authorization Code**를 <mark style="color:green;">**클라이언트**</mark>의 **redirect URI**로 리디렉션함
6. <mark style="color:green;">**클라이언트**</mark>는 Authorization Code를 수신함
7. <mark style="color:green;">**클라이언트**</mark>는 Authorization Code를 이용해 Google <mark style="color:red;">**인증서버**</mark>에 **Access Token, Refresh Token 요청**
8. <mark style="color:red;">**인증서버**</mark>는 **Access Token, Refresh Token**을 <mark style="color:green;">**클라이언트**</mark>에 응답
9. <mark style="color:green;">**클라이언트**</mark>는 Access Token을 사용해 Google API로부터 **사용자 정보를 조회**
10. 이후 <mark style="color:purple;">**유저**</mark>는 로그인된 상태로 <mark style="color:green;">**클라이언트**</mark>와 통신함. 필요한 경우 <mark style="color:green;">**클라이언트**</mark>가 토큰을 이용해 다시 사용자 정보에 접근

