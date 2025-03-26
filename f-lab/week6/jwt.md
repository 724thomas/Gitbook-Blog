---
description: JSON WEB TOKEN
---

# JWT

## 1. 개요

**JWT**는 세션 정보를 서버 메모리에 저장할 필요 없이, **토큰** 자체에 인증 정보를 담아 **무상태(stateless)** 로 인증을 구현할 수 있다는 장점이 있습니다.



## 2. 기본 개념

### 2.1. JSON Web Token 구조

**JWT**는 “Header, Payload, Signature” 세 부분이 `.`으로 구분된 문자열 형태를 취합니다:

```bash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9  <-- Header (base64)
. eyJzdWIiOiIxMjMiLCJyb2xlIjoiVVNFUiIsImlhdCI6MTYy...  <-- Payload (base64)
. SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  <-- Signature (base64)
```

1. **Header**: 알고리즘(alg=HS256), 토큰 타입(typ=JWT) 등 메타데이터
2. **Payload**: 실제 인증 정보(사용자 ID, 만료시간, role 등)
3. **Signature**: 위 Header와 Payload를 비밀키(Secret)로 서명

클라이언트는 이 문자열(Access Token)을 `Authorization: Bearer <token>` 헤더에 실어 서버에 요청합니다.

### 2.2. 왜 토큰 기반?

* **Stateless**: 서버가 세션을 관리하지 않고, 모든 인증 정보가 토큰 안에 있음
* **확장성**: 여러 서버 간 세션 공유 없이 토큰만 검증하면 됨
* **JWT**는 표준화: 여러 언어/플랫폼에서 쉽게 지원



## 3. JWT Payload

### 3.1. 필수 등록된 클레임 vs. 커스텀 클레임

**JWT** 사양에는 “등록된(Registered) 클레임” 이라는 권장 필드가 있습니다:

* **iss**(토큰 발급자), **sub**(주체), **aud**(토큰 대상), **exp**(만료시간), **iat**(발행시간), 등.

이와 별도로, 애플리케이션이 필요한 **커스텀 클레임**(예: userId, role)을 넣을 수 있습니다. 예:

```json
{
  "sub": "userId_123",
  "role": "USER",
  "iat": 1676790800,
  "exp": 1676794400,
  "customField": "anyvalue"
}
```

### 3.2. 과도한 정보 넣지 말 것

JWT Payload는 **서명 전**이긴 해도, **클라이언트에게 노출**됩니다(브라우저가 base64 디코딩 가능). 그러므로 \*\*민감 정보(비밀번호, 신용카드, 개인정보)\*\*를 절대 넣으면 안 됩니다. Payload는 **ID, 권한**, 만료 관련 정보 등 최소한의 식별 데이터만 담아야 합니다.

### 3.3. 만료 시간(exp)

JWT는 보통 **exp**(만료시간)를 반드시 넣어 유효기간을 제한합니다. 예: 30분(Access Token) 정도. 만료 후엔 재발급(Refresh Token) 과정을 거쳐야 합니다.



## 4. JWT LifeCycle (발급, 만료, 재발급)

### 4.1. Access Token과 Refresh Token

이커머스나 모바일 앱에서 흔히:

1. **Access Token**: 짧은 유효기간(예: 30분) → API 호출 시 사용
2. **Refresh Token**: 길게 유효(예: 2주 \~ 4주) → Access Token 만료 시, Refresh Token으로 재발급 요청

이 방식을 통해 보안과 편의를 균형 있게 취할 수 있습니다.

* **Access Token**이 짧게 만료되면, 탈취돼도 피해가 제한됨.
* 유저는 매 30분마다 로그인하긴 번거로우니, **Refresh Token**으로 자동 재발급 가능.

### 4.2. 발급 시점

1. 사용자 로그인(이메일+비밀번호 검증) 성공
2. 서버가 **Access Token**(JWT) + **Refresh Token**(JWT or DB 저장)을 발급
3. 클라이언트(브라우저/앱)에 저장(쿠키/스토리지). 이후 API 호출할 때 Access Token 전송.

### 4.3. 만료 및 재발급

* 클라이언트가 API 호출 시 “Access Token 만료” 응답(401 or 403) → 클라이언트는 **Refresh Token** 전송해 `/auth/refresh` API로 새 Access Token 획득
* 만약 Refresh Token도 만료되었거나 무효화되었다면, 다시 로그인해야 함.

### 4.4. Refresh Token 저장 방법

* **DB**나 **Redis**에 Refresh Token(또는 토큰 ID) 저장 → 만료 전 로그아웃/무효화 가능
* **JWT** 자체로 Refresh Token을 발급하기도 하지만, 무효화가 까다로움(“Blacklisting” 필요)



## 5. JWT 발급/검증 알고리즘

### 5.1. HMAC (HS256 등)

* 대칭키(Secret) 기반.
* Header: `{"alg":"HS256","typ":"JWT"}`
* Signature = HMAC-SHA256(Header, Payload, Secret)

### 5.2. RSA or ECDSA

* 비대칭키(Private/Public) 방식.
* 서버가 Private Key로 서명, 클라이언트나 게이트웨이가 Public Key로 검증.
* 대규모/마이크로서비스에서 Public Key 배포해 서명 검증할 때 편리.

MVP 단계에서 HS256(대칭키)라도 충분하지만, 배포환경/보안 요구에 따라 RSA나 ECDSA가 더 안전/유연합니다.



## 6. 서버에서 JWT 검증 흐름

1. **클라이언트**: `Authorization: Bearer <JWT>` 헤더로 API 호출
2. **서버**: 토큰 parse → Header, Payload, Signature 분리
3. **Signature 검증**: HS256이면 Secret으로 HMAC, RSA이면 Public Key로 verify
4. **exp**(만료시간), **nbf**(시작시간) 등 클레임 체크
5. 유효하면 Payload의 userId, role 등 꺼내서 컨트롤러에 넘김.

만약 서명 실패 or exp 지남 → 401 Unauthorized 응답.



## 7. 세션less(Stateless) 장점 vs. 주의점

#### 7.1. 장점

* 서버 확장에 유리 (N대 서버 로드밸런싱 시 세션 스티키 필요 없음)
* 서버 재시작해도 로그인 세션 유지 (DB 세션 저장 불필요)

#### 7.2. 주의점

* **토큰 탈취** 시 만료 전까지 API 남용 가능 → 짧은 만료시간과 HTTPS 보안 필요
* **토큰 강제 만료**가 쉽지 않다(서버에선 토큰 자체를 신뢰, “블랙리스트” 등 부가 로직 필요)
* **Refresh Token 저장** 방법이 중요(쿠키, HttpOnly, DB?).



## 8. JWT in E-Commerce Project

### 8.1. User Login Flow

1. **POST /auth/login**: email, password 전송
2. 서버가 **BCrypt** 등으로 PW 검증 → OK시 `AccessToken + RefreshToken` 발급
3. 클라이언트는 AccessToken(30분 만료) 메모리/스토리지, RefreshToken(2주 만료)을 쿠키(httponly)나 DB에 저장

### 8.2. Subsequent API Calls

* **GET /user/profile** or **POST /cart/add** → HTTP Header `Authorization: Bearer <AccessToken>`
* 서버 middleware(Interceptor)에서 **JWT 검증**.
* 통과 → controller에서 userId, role 등 활용.

### 8.3. Token Expiry

* 30분이 지나면 AccessToken 만료. API 호출 시 401 → 클라이언트가 RefreshToken 사용 `/auth/refresh` → 새 AccessToken 받음.



## 9. 만료 전 토큰 재발급(토큰 로테이션)

**Refresh Token 로테이션** 기법:

1. 클라이언트가 한 번 Refresh Token 사용 시, 서버에서 새로운 Refresh Token 발급 & DB 업데이트
2. 이전 Refresh Token 즉시 무효화
3. 만약 이전 토큰이 재사용되면, 보안 위험이 감지됨 → 강제 로그아웃

이 기법은 Refresh Token 도난을 신속히 감지·차단하기 위함. 구현 난이도 있지만 보안 향상.



## 10. 로그아웃 처리

JWT 자체는 “서버상 세션 상태”가 없으므로, 클라이언트 측에서 토큰 삭제하면 끝… 이지만 **도난** 상황에서 강제 만료가 필요할 수 있습니다.

* **Token blacklist**: 만료 시간 전까지 “이 토큰은 무효” 목록에 등록해 매 요청 시 체크.
* **Refresh Token** DB 관리: Access Token 만료돼도 Refresh Token이 무효면 재발급 불가 → 사실상 로그아웃.

이커머스는 보안이 중요하므로, Refresh Token을 DB에 저장·관리하는 방식이 안전합니다.



## 11. Implementation Example (Java Spring Boot)

```java
@PostMapping("/auth/login")
public LoginResponse login(@RequestBody LoginRequest req) {
    // 1. UserRepository.findByEmail -> password match
    // 2. Generate AccessToken (15m), RefreshToken (7d)
    String accessToken = jwtService.generateAccessToken(user.getId());
    String refreshToken = jwtService.generateRefreshToken(user.getId());
    // 3. Save refreshToken in DB or return in Cookie
    return new LoginResponse(accessToken, refreshToken);
}
```

```java
@Aspect
@Component
public class JwtAuthAspect {
    @Around("@annotation(RequireLogin)")
    public Object checkToken(ProceedingJoinPoint pjp) {...}
}
```

* Aspect or Filter checks “Authorization: Bearer ...” → decode JWT → validate.
* If valid, set user context. If invalid, throw 401.



## 12. Example Payload

```json
{
  "sub": "user:42",         // user id
  "role": "USER",           
  "iat": 1676790800,        
  "exp": 1676794400,        
  "iss": "my-ecom-service"  
}
```

* **sub**: subject (“user:42”)
* **role**: “USER” or “ADMIN”
* **iat**: issued at timestamp
* **exp**: expiration timestamp
* **iss**: issuer name



## 13. 두 가지 큰 접근법

### 13.1. 100% Stateless (No Blacklist)

**개념**:

* 서버는 **Access Token**(단기 만료) + **Refresh Token**(장기 만료)를 발급.
* 로그인 후, 서버는 **어떤 토큰도 저장**하지 않음(= 완전 무상태).
* 각 요청 시 Access Token만 서명 검증→Payload→유효? OK, 만료? → Refresh Token으로 재발급(서명만 확인).

**장점**:

* 서버 부담 최소: 토큰 상태를 전혀 저장·추적하지 않으므로 확장성 높음.
* 단순 구현: Redis나 DB가 필요 없음.

**단점**:

* **토큰 강제 만료**(로그아웃·권한 회수)가 어렵다. Access Token은 만료 전까지 유효.
* **Refresh Token**도 빼앗기면 장기 사용 가능(만료 전까지).

### 13.2. Blacklisting (Redis / DB 저장)

**개념**:

* 여전히 **Access+Refresh** 발급하되, **서버(또는 Redis)** 에 토큰 정보(또는 토큰 ID)를 저장.
* 만료 전 “강제 무효화”가 필요할 때, **BlackList** (또는 **AllowList**)를 활용.
  * BlackList: 무효 처리할 토큰을 등록. API 호출 시 토큰이 블랙리스트에 있으면 거부.
  * AllowList: 발급된 토큰만 허용. 만료 전이라도 DB/Redis에서 삭제→무효화.

**장점**:

* **실시간 토큰 무효화** 가능(로그아웃, 관리자 강제 만료).
* 분실/도난, 비밀번호 변경, 권한 변경 시 즉시 차단 가능.

**단점**:

* 서버가 토큰 목록을 저장/조회 → 부하 증가.
* DB/Redis 운영 필요 → 무상태(stateless) 이점 일부 상실.



## 14. 시나리오 비교

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

### 시나리오 1. **User가 정상적으로 로그아웃**

**A. 100% Stateless**

1. 클라이언트 측에서 Access Token/Refresh Token 삭제 (ex. 쿠키·스토리지 clear).
2. 서버는 따로 할 일이 없음(Stateless).
3. BUT, 이미 발급된 토큰이 어디선가 남아있으면(도난), 만료 전까지 여전히 유효. → **강제 만료 안됨**.

**B. BlackList**

1. 클라이언트 측에서 토큰 삭제.
2. 서버에 “이 Refresh Token(또는 Access Token)을 무효화” 요청 → DB/Redis에 “Blacklisted token” 등록 or “Refresh Token row” 삭제.
3. 향후 해당 토큰으로 요청 오면 거부.
4. **실시간**으로 토큰 무효화됨.

**차이**: Stateless에선 로그아웃 시 토큰을 버리는 것 외에 서버단 처리 불가. BlackList 방식은 **즉시 무효화** 가능.

***

### 시나리오 2. **Access Token 도난, 사용자 위험**

**A. 100% Stateless**

1. 공격자는 Access Token 만료 전까지 자유롭게 API 호출 가능.
2. 사용자가 “비밀번호 변경” 혹은 “다시 로그인” 해도, **도난된 Access Token**이 만료될 때까지는 유효.
3. 만약 Refresh Token도 도난되면, 매우 위험(장기 사용).

**B. BlackList**

1. 사용자는 “모든 기기에서 로그아웃” 같은 기능 → 서버에서 **해당 토큰(혹은 Refresh Token)** 등록 → **API 거부**.
2. 도난된 토큰 재사용 시, BlackList 체크로 즉시 401 차단.
3. 보안성 향상.

**차이**: 완전 Stateless에선 **도난 후** 대책은 “기다려서 토큰 만료” 뿐. BlackList는 “즉시 차단” 가능.

***

### 시나리오 3. **비밀번호 변경 / 회원 권한(ROLE) 변경**

**A. 100% Stateless**

* 기존 Access Token에 적힌 `role` 등은 만료 전까지 그대로 유효.
* 비밀번호 바꿔도 Token Payload는 바뀌지 않으므로, 만료 시점까지 API 가능.
* Refresh Token 재발급 시점 이후부터 새 토큰에만 새로운 `role` 적용.

**B. BlackList**

* “비밀번호 변경 이벤트” → 서버가 **기존 토큰 모두 무효화**(BlackList)
* 새로 로그인/Refresh 해야만 새 토큰(`role` 갱신)
* 즉시 권한 변경 반영.

**차이**: Stateless 방식에서 즉시 반영 불가, Access Token 만료 기다려야. BlackList는 “지금부터 옛 토큰 안됨”이 가능.

***

### 시나리오 4. **한 기기(토큰)만 강제 만료**

**A. 100% Stateless**

* 불가능. 특정 토큰만 골라 만료시킬 방법 없음.
* 모두가 새 토큰 발급받을 때까지 기다려야 하거나, 짧은 Access Token으로 리스크 줄이는 수밖에.

**B. BlackList**

* 서버가 특정 토큰 ID / jti(고유 식별자) / Refresh Token row를 DB/Redis에서 제거 or “blacklist”
* 나머지 기기는 문제없이 사용, 특정 기기는 즉시 만료

**차이**: Stateless에선 토큰별 조절 불가. BlackList는 토큰 단위로 세밀히 제어 가능.

***

### 시나리오 5. **모든 기기(모든 토큰) 만료(“로그아웃 다른 기기”)**

**A. 100% Stateless**

1. 불가능에 가깝다.
2. 방법: “비밀번호 변경” → 이후 새 로그인. 기존 토큰들은 만료 전까지 유효.

**B. BlackList**

1. “전체 토큰” = Refresh Token 테이블(또는 user별 token list)을 전부 삭제/BlackList 등록
2. 즉시 전 디바이스가 401 Unauthorized → 재로그인 필수.

***

### 시나리오 6. **Refresh Token마저 도난**

**A. 100% Stateless**

* 도난된 Refresh Token이 만료 전이면 공격자 무제한 Access Token 재발급 가능.
* 유저가 비밀번호 바꾸거나 재로그인해도, 이미 발급된 Refresh Token은 유효(만료 전).
* 치명적.

**B. BlackList**

* Refresh Token DB나 Redis에 저장. 도난 사실 알면 즉시 해당 Refresh Token row 삭제 → 더 이상 재발급 불가.
* 안전.

***

### 시나리오 7. **Token Expired 자동 만료**

**A. 100% Stateless**

* 서버는 Payload의 `exp` 검사 → 만료면 401.
* 클라이언트가 Refresh Token으로 새 Access Token 받음.
* 별도 DB 검사 없이 편리.

**B. BlackList**

* 마찬가지로 `exp` 검사.
* 또한, DB의 “issued\_at”이나 “valid\_until” 필드와 비교할 수도 있음.
* 만료되면 401, refresh needed.

(이 부분은 두 방식 크게 다르지 않음. BlackList도 `exp`로 토큰 만료 자동 반영.)

***

### 시나리오 8. **토큰 만료 직전 재발급 (Rotation)**

**A. 100% Stateless**

* 서버는 Access Token 재발급 시, 기존 Access Token 무효화 불가능.
* Token Rotation(Refresh 시 옛 Token 파기) → 크게 의미 없음, 여전히 옛 Access Token 유효.

**B. BlackList**

* Rotation 시 **이전 Access Token** / Refresh Token **BlackList**에 추가 → 즉시 무효.
* 새 Token만 유효.

***

### 시나리오 9. **오래된 Refresh Token, 중간에 무효화?**

**A. 100% Stateless**

* 원칙적으로 무효화 불가, 만료 기다리거나, user가 전면 재로그인.
* 장기 Refresh Token은 유출 시 대형 사고.

**B. BlackList**

* 어떤 사유(예: 보안사고)로 Refresh Token을 모두 폐기. DB/Redis row 삭제.
* 즉시 효력 상실.

***

### 시나리오 10. **서버 재부팅 / Autoscaling**

**A. 100% Stateless**

* 매우 편함. 서버 10대를 늘리거나 재부팅해도, Token 검증은 동일 Secret 공유하면 OK.
* 세션 동기화 불필요.

**B. BlackList**

* BlackList(토큰 상태) 저장을 **Redis/DB** 등 **공유 저장소**에 둬야 함.
* 서버 재부팅 / Autoscaling 시, 그 저장소만 제대로 연결되면 문제 없음.
* 다만, 부하·관리비 조금 증가.
