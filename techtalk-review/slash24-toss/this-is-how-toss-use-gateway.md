---
description: 토스는 Gateway 이렇게 씁니다.
---

# This is how toss use Gateway

## 🎯 서문 — “게이트웨이를 제품처럼 운영한다”는 말의 진짜 의미

본 글은 제공해주신 발표 스크립트를 바탕으로, **장애 상황 → 기술적 원인 분석 → 아키텍처 설계 결정 배경 → 해결책 → 교훈/인사이트**의 흐름으로 재구성한 **실무 지향 기술 블로그**입니다. 문서 전반에 걸쳐 **스프링 클라우드 게이트웨이(Spring Cloud Gateway, 이하 SCG)**, **WebFlux/Reactor Netty**, **BFF(Backend For Frontend)**, **Ingress/Egress 분리**, **JWT/mTLS/OIDC**, **서킷브레이킹**, **Elasticsearch/Prometheus/Grafana/Pinpoint**, **Z-score 이상 감지**, **Gateway Bot** 등 스크립트의 모든 핵심 요소를 포함했습니다. 신입 개발자도 이해할 수 있도록 **난해 키워드에는 주석**을 덧붙였습니다.

> \[주석: WebFlux] Spring의 비동기/논블로킹 웹 스택. Netty 위에서 Reactor(리액티브 스트림)을 사용. 동시성에 강함.
>
> \[주석: Reactor Netty] Netty 기반 고성능 네트워킹과 Reactor를 결합한 런타임. 이벤트 루프 모델.
>
> \[주석: BFF] 프론트엔드 종류(앱/웹/SSR)별로 전용 백엔드(또는 게이트웨이)를 둬서 관심사를 분리하는 패턴.
>
> \[주석: Ingress/Egress] Ingress는 **들어오는** 트래픽(사용자→클러스터), Egress는 **나가는** 트래픽(클러스터→외부).
>
> \[주석: OIDC] OpenID Connect. OAuth2 위에 사용자 인증 계층을 더한 표준. 사내/사설 웹 SSO에 자주 사용.

***

## 🧭 왜 게이트웨이인가 — 공통 로직의 집중과 단일 제어지점

초기에는 **클라이언트 → 서비스 직접 호출** 구조도 문제없습니다. 그러나 서비스가 늘고 트래픽이 커지면 **인증/유저 정보/보안 정책**과 같은 **공통 로직 변경**을 모든 서비스에 배포해야 하고, 한 서비스라도 누락되면 **장애**로 이어집니다.

**게이트웨이**는 이 공통 로직을 **전·후처리 파이프라인**으로 끌어올려 **단일 제어지점**을 형성합니다.

* **정의(스크립트 인용 요지)**: “API 게이트웨이는 라우팅과 프로토콜 변환을 담당하는 마이크로서비스의 중개자. 서비스는 클라이언트와 독립적으로 확장 가능하며, 보안/모니터링의 단일 제어지점을 제공.”
* **핵심 가치**
  * 보안/인증/인가 **정책 일원화**
  * 로깅/모니터링/트레이싱 **일관 수집**
  * 공통 로직의 **중복 제거** 및 **배포 단순화**

> \[주석: 단일 제어지점] 정책/가시화/트래픽 제어를 한 곳에서 강제·관찰할 수 있는 구조.

***

## 🧱 Route = Predicate + Filter — 게이트웨이의 동작 단위

SCG에서 **Route**는 **Predicate(매칭)** + \*\*Filter(전/후처리)\*\*로 구성됩니다.

* **Predicate**: `Path`, `Method`, `Host`, `Header`, `Query` 등으로 요청을 구분.
* **Filter(Pre/Post)**:
  * Pre: 입력 위생(Sanitization), 인증/인가(JWT, mTLS, 서명), 내부 헤더 주입(공통 정보), 트레이스 ID 생성.
  * Post: 응답 헤더 정책(CORS/CSP), 개인정보 마스킹, 로깅 샘플링, 캐시 힌트.

```
# 예시: SCG 라우트 (요지)
- id: user-profile
  predicates:
    - Path=/api/v1/users/**
    - Method=GET
  filters:
    - name: SanitizeParams
    - name: VerifyJwt
    - name: InjectPassport
    - name: CircuitGuard
  uri: http://user-service
```

> \[주석: Predicate] 요청을 어떤 라우트가 처리할지 구분하는 조건. 다중 조건 가능, 모두 만족해야 매칭.
>
> \[주석: Filter 순서] 전처리 안전 → 인증/인가 → 컨텍스트 주입 → 라우팅 → 후처리 순으로 배치하는 것이 일반적.

***

## 🧩 모놀리식 게이트웨이의 문제와 BFF로의 분해

게이트웨이에 모든 로직이 쌓이면 **웹/앱/SSR 요구사항이 충돌**하며 빅볼 오브 머드화합니다. 스크립트가 제시한 해결은 **BFF**입니다.

[![Image](https://private-user-images.githubusercontent.com/55742497/503246923-5563801c-3424-49e3-b2f0-d07147b3b994.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjA5NzEyNzAsIm5iZiI6MTc2MDk3MDk3MCwicGF0aCI6Ii81NTc0MjQ5Ny81MDMyNDY5MjMtNTU2MzgwMWMtMzQyNC00OWUzLWIyZjAtZDA3MTQ3YjNiOTk0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEwMjAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMDIwVDE0MzYxMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ3YTU5N2I1NmYwYTBhMzBlNDdiNjA2MTc1ZGI3ODhhOTMwYmJlOThhZjY1MGM3NTQyZGIyYjQ3M2YxODc0YzgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.dsY2q_qhe3K5xY4tXoz5fd6KgzDF5ZstcLAHNB0dNtA)](https://private-user-images.githubusercontent.com/55742497/503246923-5563801c-3424-49e3-b2f0-d07147b3b994.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjA5NzEyNzAsIm5iZiI6MTc2MDk3MDk3MCwicGF0aCI6Ii81NTc0MjQ5Ny81MDMyNDY5MjMtNTU2MzgwMWMtMzQyNC00OWUzLWIyZjAtZDA3MTQ3YjNiOTk0LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEwMjAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMDIwVDE0MzYxMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWQ3YTU5N2I1NmYwYTBhMzBlNDdiNjA2MTc1ZGI3ODhhOTMwYmJlOThhZjY1MGM3NTQyZGIyYjQ3M2YxODc0YzgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.dsY2q_qhe3K5xY4tXoz5fd6KgzDF5ZstcLAHNB0dNtA)

* **BFF(Backend For Frontend)(웹/앱/SSR 전용 게이트웨이)**
  * 웹 전용: 브라우저 특성(JWT, CORS, 캐시)을 반영.
  * 앱 전용: 앱 채널 보안(본문 암호화, 단명 서명) 반영.
  * SSR 전용: 서버사이드 렌더링의 토큰 스코핑/호출 패턴 반영.
* **효과**: 관심사 분리, 배포 리스크 축소, 정책의 단순화.

> \[주석: 빅볼 오브 머드] 분해와 경계 없이 로직이 복잡하게 얽힌 대규모 코드/시스템.

***

## ↔️ Ingress/Egress 분리 — 방향성에 따른 정책 분화

**Ingress**(사용자/외부 → 클러스터)와 **Egress**(클러스터 → 외부/계열사)는 **보안·신뢰·실패 모델**이 다릅니다. 스크립트는 **각 계열사별 Egress 게이트웨이**를 별도로 둔다고 명시합니다.

* **Ingress 게이트웨이**: 사용자 인증·입력 정규화·레이트리미팅·서킷브레이킹(엣지 백프레셔).
* **Egress 게이트웨이**: 파트너별 mTLS/CA 핀닝/요청 변환/감사 로그와 재시도/타임아웃 전략 분리.

[![Image](https://private-user-images.githubusercontent.com/55742497/503247083-c0063bfb-14bb-4be4-a605-dc9bb15ad5dd.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjA5NzEyNzAsIm5iZiI6MTc2MDk3MDk3MCwicGF0aCI6Ii81NTc0MjQ5Ny81MDMyNDcwODMtYzAwNjNiZmItMTRiYi00YmU0LWE2MDUtZGM5YmIxNWFkNWRkLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEwMjAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMDIwVDE0MzYxMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWM4NWQ1ZTM4NzQ5MmFjOTkyNGU1ZDU2MGU3MTc5MjM2YTRlYzZkNDU0MWQyOGIyNjI1N2Y5YTMzOTg3MzNhNWYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.1ZwMWudF5hUDy3kJWBxrqs58q1evWkUIV8CFlFM_HBA)](https://private-user-images.githubusercontent.com/55742497/503247083-c0063bfb-14bb-4be4-a605-dc9bb15ad5dd.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjA5NzEyNzAsIm5iZiI6MTc2MDk3MDk3MCwicGF0aCI6Ii81NTc0MjQ5Ny81MDMyNDcwODMtYzAwNjNiZmItMTRiYi00YmU0LWE2MDUtZGM5YmIxNWFkNWRkLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEwMjAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMDIwVDE0MzYxMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWM4NWQ1ZTM4NzQ5MmFjOTkyNGU1ZDU2MGU3MTc5MjM2YTRlYzZkNDU0MWQyOGIyNjI1N2Y5YTMzOTg3MzNhNWYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.1ZwMWudF5hUDy3kJWBxrqs58q1evWkUIV8CFlFM_HBA)

***

## 🧰 토스의 게이트웨이 포트폴리오 — 목적에 맞는 역할군

스크립트에 등장하는 게이트웨이:

* **App Gateway**: 앱 채널, **본문 암호화+서명 검증**.
* **Public(Web) Gateway**: 웹 채널, **JWT 검증**.
* **Secure Gateway**: **mTLS 인증서 검증**.
* **SSR Gateway**: SSR 노드 서버가 접근하는 API 중계, **토큰 스코프 확인**.
* **Internal Service Gateway** / **Internal Secure Gateway**: 사내 API/툴, OIDC SSO 및 mTLS.
* **Egress Gateways**: 계열사/외부사별 아웃바운드.

개발자는 **목적에 맞는 게이트웨이**를 선택하여 라우트를 추가, 필요한 필터를 조합해 요청 플로우를 완성합니다.

***

## ⚙️ 런타임 스택 — SCG + WebFlux + Reactor Netty + Kotlin Coroutines

* \*\*SCG(Spring Cloud Gateway)\*\*는 **WebFlux** 위에서 돌아가며 **Reactor Netty**로 논블로킹 I/O 처리.
* **Kotlin Coroutines**를 필터 개발에 **보조적으로** 활용해 복잡한 비동기 흐름 가독성을 개선.
* **Istio/Envoy**와 유기적으로 연동(인그레스/이그레스, 사이드카 필터, B3 트레이스).

> \[주석: 논블로킹] 스레드가 I/O 대기에서 멈추지 않음. 적은 스레드로 많은 동시 요청 처리.

***

## 🧼 전처리 1 — Request 처리 데이터 Sanitize

**Sanitize** : 클라이언트로부터 올바르지 않은 요청이 올 경우, 이를 지워주거나 올바른 값으로 바꿔주는 것을 의미

게이트웨이는 **첫 신뢰 경계**입니다. 악의적/오류 입력을 **정규화 또는 차단**합니다.

* 파라미터 범위 클램프(예: `size ∈ [1, 100]`), 인코딩/경로 정규화, 금지 헤더 제거.
* JSON 스키마 검증 및 잘못된 타입/필드 제거.
* 허용 헤더 화이트리스트 운영.

> \[주석: 클램프] 값이 최소/최대 범위를 벗어나지 못하도록 강제.

***

## 🧵 전처리 2 — Trace-ID/MDC/B3로 트랜잭션 상관관계 확보

게이트웨이가 **Trace-ID**를 생성/전파하면, 마이크로서비스 전반에서 **동일한 트랜잭션 단위**로 로그를 묶을 수 있습니다.

* **MDC**에 Trace-ID 저장 → 서비스 로그 상관관계 유지.
  * MDC: Mapped Diagnostic Context: Log에 Context를 추가해주는 기능(SLF4J)
* **B3 스팬**으로 마이크로서비스 간 호출 연결.

> \[주석: MDC] Logback/SLF4J 등에서 제공하는 스레드 지역 컨텍스트. 분산 환경에서는 헤더로 전파.

***

## 📨 공통 정보의 중복 호출 제거 — Internal Header 주입

한 트랜잭션에서 여러 서비스가 동일한 \*\*공통 정보(예: 사용자 동의 여부)\*\*를 반복 조회하면 **지연/비용**이 증가합니다. 게이트웨이가 **1회 조회** 후 `X-User-Consent: true` 같은 **내부 헤더**로 전파합니다.

* **효과**: 중복 요청 제거, tail latency 감소, 공통 API 부하 완화.

***

## 🪪 Toss Passport — Netflix Passport에서 영감 받은 단명 컨텍스트 토큰

**Toss Passport**는 **사용자/디바이스 컨텍스트**를 담은 **ID 토큰**입니다.

* 앱이 **유저 식별 키**와 함께 API 요청 → 게이트웨이가 인증 서버와 통신해 **Passport 발급**.
* Passport에는 **디바이스 정보/유저 정보/동의/위험도**가 포함. 게이트웨이가 **직렬화**하여 하위 서비스에 전파.
* 서비스는 유저 API를 재호출하지 않고 **Passport**만으로 필요한 컨텍스트 접근.
  * 통합 DTO 생성 + 헤더 전파

> \[주석: 단명 토큰] 유효기간이 매우 짧아 탈취/재사용 위험을 줄임.

***

## 🔐 앱 채널 보안 — 본문 암호화 + 초단기 서명 + Anti-Replay

토스 앱은 요청 본문을 **앱에서 암호화**해 전송하고, **App Gateway**에서만 **복호화**합니다. 또한 매 요청에 **초단기 유효 서명**을 포함합니다.

* **복호화 이후** 게이트웨이에서 인증/인가 로직 수행.
* **서명 검증**: 발급 시각/만료/nonce/재사용 여부 검사로 **지연·리플레이·위변조** 차단.
* **의심 시 대응**: FTS(행위 분석)와 연동하여 **계정 비활성화/정밀 차단**.

> \[주석: 리플레이 공격] 이전에 유효했던 요청을 재전송해 권한을 남용하는 공격.

***

## 🧾 웹/SSR 보안 — JWT + SSR 게이트웨이 스코프 검증

SSR/웹은 본문 종단암호화가 어렵기에 **JWT**를 사용합니다.

* **Public Gateway**가 JWT **서명·만료·중복 사용** 검증 → 안전한 요청만 SSR로 전달.
* **SSR Gateway**는 토큰으로 유저 정보를 가져와 **스코프/리소스 접근 허용 범위**를 확인 후 업스트림 호출.
* 외부 사에서 토스의 서비스에 접근하는 경우를 위해 OAuth2를 활용한 인증/인가 처리도 지원함
* 내부 **토큰 검증 서비스**와 질의해 **유효성·권한 범위** 재확인.

> \[주석: 스코프] 토큰이 허용받은 작업/리소스 집합.

***

## 🔏 내부/파트너 — mTLS + X.509 SAN 기반 세분 인가

내부 및 파트너 호출은 **mTLS**로 보호합니다.

* **엣지(Envoy)**: 클라이언트 인증서 체인/CA 유효성 검증 → 인증서 정보를 **신뢰 헤더**로 전달.
* **게이트웨이**: X.509 인증서를 파싱하여 \*\*SAN(Subject Alternative Name)\*\*에서 호출자 식별자를 추출하고, **Host/Path**와 결합해 인가 판단.

> \[주석: SAN] 인증서 주체 외에 추가 이름(도메인/URI/이메일 등)을 담는 확장 필드.

***

## 🪪 내부 웹 SSO — OIDC FLOW + Redis 세션 스토어

사내 웹은 **게이트웨이가 OIDC 플로우를 통해 Provider에서 Access Token을 가져오고 Redis Session Store을 통해 관리.**

로그인 UI를 각 팀이 만들 필요 없이 **페이지별 인증/인가**를 손쉽게 붙일 수 있고, OIDC 미지원 OSS도 게이트웨이 뒤에서 보호 가능합니다(예: Kibana, Pinpoint, 대시보드 등).

***

## 🔭 행위 기반 방어 — FDS와의 연동, 정밀 차단

요청 단위 검증을 넘어 **유저 행위 로그**를 바탕으로 **브루트포스/스크래핑/앱 분석 시도** 등 이상 패턴을 탐지합니다.

탐지 시 게이트웨이에 **유저/IP/디바이스/특정 API 단위**로 정밀 차단 정책을 내려 **서비스 전체 차단 없이** 위험을 억제합니다.

> \[주석: IPS] Intrusion Prevention System. 침입 시도를 실시간 차단하는 보안 시스템.

***

***

## 🧯 장애 사례 2 — 캐스케이딩 지연 → 스레드/소켓 고갈

**현상**: 특정 업스트림의 지연/오류가 재시도·팬아웃으로 증폭되어 **전역 자원 고갈**로 확산.

**기술적 원인 분석**

* **서킷브레이킹**이 **호스트 단위**에만 있거나, **엣지 백프레셔**가 부족.
* 라우트/기능 단위의 **정밀 타임아웃·오류율 기반 차단**이 늦음.

**해결책**

* **다층 브레이킹**:
  * **Envoy/Istio(인프라)**: 호스트 단위 응급 차단(거친 방패)
  * **SCG/서비스(앱)**: 라우트·기능 단위 정밀 차단(정밀 방패), **half-open** 프로빙
* **엣지에서 빨리 실패**를 돌려 클라이언트 재시도 폭주 억제

```
# 예시: Resilience4j 스타일(개념)
circuitBreaker:
  failureRateThreshold: 50
  slowCallRateThreshold: 50
  waitDurationInOpenState: 30s
  permittedNumberOfCallsInHalfOpenState: 10
  slidingWindowSize: 100
  minimumNumberOfCalls: 50
timeout:
  duration: 800ms   # 게이트웨이→업스트림
```

**교훈/인사이트**

* 인프라(빠른 호스트 차단) + 앱(정밀 라우트 차단)을 **동시에** 운영.
* 타임아웃은 **계층별로 차등**(클라이언트↔GW, GW↔Upstream, 내부 hop) 설정.

> \[주석: Half-open] 열림(Open) 상태에서 제한된 수의 트래픽으로 회복 여부를 시험하는 상태.

***

## 🧪 운영 구조의 진화 — 모노레포에서 분리, 그리고 PR-테스트-카나리

**과거**: 모노레포에 모든 게이트웨이 코드+자바 라우트가 섞여 증적/검증/배포 간섭이 빈번.

**현재**:

* 게이트웨이별 **저장소 분리**, 공통 로직은 **라이브러리화**.
* 라우트 설정 JAVA에서 **YAML로 전환 + 설정 저장소 분리**.
* **사내 서비스 라우팅 설정 → PR(YAML로 변환되어 PR생성) → CI(스키마/행동 테스트)** → Dev 자동 갱신(1분) → Live 카나리.
* Live환경은 개발 환경과 다르게 Route 설정을 자동으로 갱신하는 것은 위험(Master 브랜치 기반)
  * 발견되지 않은 설정 오류나 환경 차이 때문에 에러가 발생할 수 있기 때문
  * 따라서 카나리 배포방식으로 점진적 배포
* **Gateway Bot**으로 라이브==마스터 **검증 증적** 확보.

***

## 🚨 장애 사례 1 — “머지는 했는데, Live에 라우트가 없다(404 폭증)”

**현상**: 라우트 설정을 `master`에 머지했으나, **실제 게이트웨이 배포 누락**으로 라이브에 반영되지 않아 **404**가 쏟아짐.

**기술적 원인 분석**

* 라우트가 **자바 코드**에 섞여 있던 시절엔 **가독성·검증·증적 추적**이 어려웠음.
* “머지 = 라이브 반영”으로 착각하는 **휴먼 프로세스 갭** 존재.

**해결책**

1. **YAML 마이그레이션 + 설정 전용 저장소 분리**
2. **SC Config Server**로 게이트웨이가 설정을 임포트(코드/설정 완전 분리)
3. **사내 라우트 UI**에서 템플릿 기반으로 수정 → **PR 자동 생성**
4. **CI**에서 라우트 목록/테스트 케이스로 **Predicate/Filter 유효성** 검증
5. 개발환경은 **1분 주기 자동 갱신**으로 바로 검증
6. 라이브는 **자동 갱신 금지 + 카나리 배포**로 점진 반영
7. **Gateway Bot**이 **`master` 최신 커밋 vs 라이브 적용 커밋**을 주기 비교 → **드리프트 알림**

**교훈/인사이트**

* **설정=코드** 원칙과 **드리프트 탐지 자동화**가 중요.
* “머지”와 “적용”은 별개 단계. **카나리**로 위험을 낮춰야 한다.

***

## 📜 로깅 — Elasticsearch + Envoy 로그 + 트랜잭션 상관관계

* **게이트웨이 로그**: `routeId`, `method`, `URI`, `status` 등을 ElasticSearch에 남김
  * 요청이 어떤 Route로 들어왔는 지, 업스트림으로 어떤 URI를 호출했는지 확인 가능
* **Envoy 로그**: `source/destination`, 실패 사유, L7 코드, 각 Hop마다의 네트워크 레이턴시 등 **풍부한 맥락** 제공.
* **Envoy에서 지원하는 B3 Span**로 **분산 경로 상관관계** 확보 → 문제 지점 특정 용이.

***

## 📈 메트릭 — 시스템/애플리케이션/라우트/브레이커, 그리고 Z-score

#### System Metric, JVM Metric 모두 Prometheus가 수집

* System Metric : Node Exporter로 수집
* Application Metric : Spring Actuator로 수집
* **시스템**: CPU/메모리/네트워크(RX/TX)로 인프라 압력 감지.
* **애플리케이션**: JVM 스레드 블록, 세대별 메모리 할당, 이벤트 루프 상태.
* **라우트별**: 상태코드 분포, 평균/최대 응답 시간, 호출 수, **서킷 오픈/타임아웃**.
* **Z-score 이상 감지**: 4xx/5xx 오류 급증 스파이크를 통계적으로 포착해 **Slack 경보**.

> \[주석: Z-score] (현재값 − 과거평균) / 표준편차. 값이 클수록 이례적.

***

## 🛰️ 트레이싱 — Pinpoint로 “그림처럼” 흐름과 콜스택을 본다

**Pinpoint**로 트랜잭션 경로를 시각화합니다. 실패 지점이 드러나면 **콜스택 시작 위치/실행 시간/스레드 이름**으로 **원인 서비스/라인**을 좁혀갑니다. 게이트웨이-서비스-데이터스토어까지 **종단 간** 분석이 가능합니다.

* Pinpoint: 네이버 OS. Application Performance Mangement 도구.
* 트랜잭션에서 트래픽이 어떻게 흐르는지 한번에 볼 수 있어서 문제 지점 쉽게 파악 가능.

[![Image](https://private-user-images.githubusercontent.com/55742497/503247251-21a1e4e4-d5a5-4edd-b1d1-6bfe142a554a.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjA5NzEyNzAsIm5iZiI6MTc2MDk3MDk3MCwicGF0aCI6Ii81NTc0MjQ5Ny81MDMyNDcyNTEtMjFhMWU0ZTQtZDVhNS00ZWRkLWIxZDEtNmJmZTE0MmE1NTRhLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEwMjAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMDIwVDE0MzYxMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWE2ZDUwYTZlZTYxODQzYWJhYTcxMmFlM2JhMzEyYzRhNDhlMDJjNTJiM2ZiZjVlMmNkY2I5YmZjOTAwMTQ3ZGYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.V1MBLyYg9GJNMLA_CLKTJwFFVSWCb7Cwc06c3EjGmAM)](https://private-user-images.githubusercontent.com/55742497/503247251-21a1e4e4-d5a5-4edd-b1d1-6bfe142a554a.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjA5NzEyNzAsIm5iZiI6MTc2MDk3MDk3MCwicGF0aCI6Ii81NTc0MjQ5Ny81MDMyNDcyNTEtMjFhMWU0ZTQtZDVhNS00ZWRkLWIxZDEtNmJmZTE0MmE1NTRhLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEwMjAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMDIwVDE0MzYxMFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWE2ZDUwYTZlZTYxODQzYWJhYTcxMmFlM2JhMzEyYzRhNDhlMDJjNTJiM2ZiZjVlMmNkY2I5YmZjOTAwMTQ3ZGYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.V1MBLyYg9GJNMLA_CLKTJwFFVSWCb7Cwc06c3EjGmAM)
