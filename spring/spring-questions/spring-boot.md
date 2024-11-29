---
description: 스프링 Data JPA
---

# Spring Boot

**Spring과 Spring Boot의 차이점**

## Spring Framework와 Spring Boot의 주요 차이점을 설명해주세요.

* 설정 방식과 초기화 과정의 단순화

Spring Framework

* 개발자가 애플리케이션 설정을 직접 처리해야합니다.

Spring Boot

* 자동 설정(Auto-Configuration): 개발자는 최소한의 설정만으로 애플리케이션을 빠르게 실행
* 임베디드 서버: Tomcat 같은 임데디드 웹 서버를 제공하여 별도의 WAS 설정없이 독립 실행
* 스타터 종속성: 프로젝트에 필요한 공통 기능을 포함하는 starter dependencies 제공. 간단한 종족성 추가만으로 여러 기능을 쉽게 설정
* Production-Ready-Features: Spring Boot Actuator 등 운영 환영에서 유용한 기능들을 내장



**Spring Boot의 특징과 장점**

## Spring Boot의 특징과 주요 기능을 설명해주세요.

* 자동 설정: spring-boot-starter-data-jpa 를 추가하면 HIbernate와 JPA 설정 적용.
* 임베디드 서버: Tomcat, Jetty와 같은 내장 서버를 제공하여 별도의 WAS 없이 독립적으로 실행
* 스타터 종속성: 프로젝트 공통으로 필요한 기능을 포함하는 starter dependencies 제공
* 운영 환경 지원: 모니터링을 위한 actuator 등 운영환경에서 필요한 여러가지 기능 내장





**Spring Boot 설정과 구성**

## Spring Boot에서 사용되는 기본 구성 파일의 이름과 역할은 무엇인가요?

* 애플리케이션 속성 관리: application.properties / application.yml을 통해 파일에서 설정한 속성들이 자동으로 로드.
* 환경별 설정: application-{profile}-properties / .yml 파일로 개발 환경/운영 환경 설정분리



## Spring Boot 애플리케이션을 사용자 정의 포트로 실행하는 방법은 무엇인가요?

application.properties / .yml 파일에서 server.port=8081



## Spring Boot에서 프로파일(Profile)을 사용하는 방법은 무엇인가요?

서로 다른 환경(예: 개발, 테스트, 운영)에 맞는 설정을 분리하여 관리할 수 있는 기능을 제공



## Spring Boot에서 속성을 정의하고 접근하는 방법을 설명해주세요.

* application.properties, application.yml 파일에 속성을 정의합니다
* @Value 를 통해 속성 값을 개별적으로 주입받습니다.



## spring-boot-starter-parent의 역할은 무엇인가요?

Spring Boot 프로젝트의 **부모 POM(Project Object Model) 파일**로서, Maven을 사용하는 프로젝트에서 **기본 설정**을 제공하는 역할



## Spring Boot에서 HTTP/2 지원을 활성화하는 방법은 무엇인가요?

* HTTP/2는 Spring Boot에서 HTTPS 환경에서만 활성화.
* `application.properties` 또는 `application.yml` 파일에 `server.http2.enabled=true`를 추가.
* SSL 인증서를 설정 필요.



## Spring Boot에서 로그 레벨을 조정하는 방법은 무엇인가요?

* SLF4J와 Logback을 사용하여 로그를 관리합니다.
* application.properties/.yml에서 설정하거나, Logger 객체를 사용합니다.



**임베디드 서버와 배포**

## 임베디드 컨테이너와 WAR 배포의 차이점을 설명해주세요.

임베디드 컨테이너

* 애플리케이션 자체가 웹 서버(서블릿 컨테이너)를 포함하고 있어, **독립 실행형 JAR 파일**로 실행할 수 있는 방식
* Spring Boot에서는 Tomcat, Jetty 등의 서블릿 컨테이너를 애플리케이션에 내장.

WAR 배포

* 애플리케이션이 **독립 실행형**이 아닌, 외부에 설치된 **서블릿 컨테이너** 또는 **WAS(Web Application Server)**&#xC5D0; 배포되는 방식



**Spring Boot의 기타 기능**

## Spring Initializr를 사용하여 Spring Boot 애플리케이션을 생성하는 방법은 무엇인가요?

Spring Boot 애플리케이션을 빠르게 설정하고 생성할 수 있는 웹 도구

[https://start.spring.io](https://start.spring.io)에서 사용



## Spring Boot Actuator의 역할과 기능을 설명해주세요.

* **Spring Boot 애플리케이션의 상태 모니터링**, **메트릭 수집**, **애플리케이션 관리**를 위한 다양한 기능을 제공하는 **production-ready** 도구
* 운영 환경에서 애플리케이션의 **헬스 체크**, **로그 모니터링**, **애플리케이션 설정 확인** 등을 손쉽게 할 수 있도록 도와줍니다



## Spring Boot에서 Docker를 활용하는 방법을 설명해주세요.

* **Docker**를 사용하면 Spring Boot 애플리케이션을 **컨테이너화**하여 어디서나 일관된 환경에서 실행할 수 있습니다.
* `Dockerfile`을 작성하여 애플리케이션을 패키징한 후, **Docker 이미지**를 빌드하고 실행합니다.
* **Docker Compose**를 사용하면 Spring Boot와 다른 서비스(예: 데이터베이스)를 함께 실행할 수 있습니다.
