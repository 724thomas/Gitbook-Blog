---
description: 비밀번호 해싱 및 Salt
---

# pw hashing && Salt

## 1. 개요

해커가 데이터베이스를 유출했을 때, 비밀번호가 평문(plain text)으로 저장돼 있으면 큰 문제가 됩니다. 또한, 간단히 **MD5나 SHA1**만 적용한 해시도 이미 취약하다는 것이 알려져 있습니다. 따라서, 더 안전한 방식인 **솔트(Salt)** 와 **강력한 해싱 알고리즘**(예: BCrypt, Argon2, PBKDF2 등)을 적용해야 합니다.



## 2. 왜 Salt가 필요한가?

### 2.1. 해싱만으로는 부족

단순히 **SHA256(password)** 같은 방식으로 비밀번호를 해싱한다면, 공격자는 **레인보우 테이블(Rainbow Table)** 이나 **사전(Dictionary) 공격**을 통해 빠르게 역추적할 수 있습니다. 특히, **사용자들이 자주 사용하는 비밀번호**(`123456`, `qwerty`, `password` 등)를 미리 해시해둔 목록이 공격자에게 있기 때문입니다.

예:

```rust
password -> SHA256 -> 5e884898da28047151d0e56f8dc6292773603d0d…
```

만약 DB에서 이 해시를 발견하면, 해당 값이 “password”라는 것을 쉽게 알 수 있습니다.

### 2.2. Salt로 레인보우 테이블 무력화

**Salt**는 **무작위로 생성된 문자열**을 의미합니다. 비밀번호 해싱 시 \*\*(비밀번호 + Salt)\*\*를 결합해 해싱하면, 레인보우 테이블 공격을 어렵게 만듭니다. 예를 들어:

```ini
salt = "OJrT9vLm"  // 무작위
password = "password"
hashed = SHA256("password"+"OJrT9vLm") = ...
```

이렇게 하면, 기존에 만들어진 레인보우 테이블엔 “password”에 대한 해시만 있을 뿐, “passwordOJrT9vLm”에 대한 해시는 없으므로 공격이 훨씬 어려워집니다.



## 3. 해싱 알고리즘 선택

### 3.1. 단순 해시(SHA256, SHA512) 문제점

SHA256, SHA512 같은 단방향 해시는 빠른 계산을 목표로 합니다. 이는 곧 **대규모 병렬 공격**에 노출되기 쉬움을 의미합니다. GPU 클러스터가 수십억 번 해시 연산을 시도하여, 해시를 깰 수도 있기 때문입니다.

### 3.2. BCrypt, Argon2, PBKDF2

#### BCrypt

* **Password-Based Key Derivation Function**으로, 내부에 Salt가 포함됨
* 연산량을 조절할 수 있어, 하드웨어가 좋아져도 해시 계산이 빨라지지 않도록 “비용(cost) 파라미터”를 높일 수 있음
* **스프링 시큐리티** 등 대부분 Java 프레임워크에서 지원

#### Argon2

* **2015년 비밀번호 해싱 콘테스트 우승**
* GPU/ASIC 공격에 더 강인하도록 설계됨
* 다양한 파라미터(메모리 사용량, 병렬 처리, iteration 횟수)로 공격 난이도 조정 가능

#### PBKDF2

* Key derivation 함수. SHA2나 SHA3 계열 해시를 반복 적용
* NIST 표준이며, 많은 라이브러리에서 지원
* 기존 시스템 호환성 좋음

실무에서는 **BCrypt**나 **Argon2**를 가장 권장합니다. PBKDF2도 여전히 안전하지만, Argon2가 좀 더 현대적인 공격에 대비가 잘 되어있다는 평이 많습니다.



## 4. Salt를 어떻게 DB에 저장하나?

### 4.1. 해싱 함수가 자동으로 관리

BCrypt나 Argon2 구현 라이브러리는 내부적으로 **Salt를 자동 생성**하고, 결과 해시에 Salt를 같이 인코딩해 둡니다. 예컨대, BCrypt 해시 문자열은 `$2a$10$yMolcIs0l637n10OV0pHIe` 식으로 시작해, 그 안에 cost와 Salt가 함께 들어 있습니다.

이 경우 DB에 “(해시된 비밀번호) = $2a$10$…” 형태를 하나만 저장해두면 됩니다. Salt를 별도 컬럼에 저장할 필요가 없죠.

### 4.2. 수동 관리

PBKDF2 같이 Salt를 수동으로 생성해, DB의 별도 컬럼(`salt`)에 저장하는 경우도 있습니다. 이때 해시값(`hashed_password`)와 Salt를 각각 컬럼으로 저장해둔 뒤, **검증 시** user가 입력한 password + salt로 해시를 다시 계산해 비교합니다. 예:

```java
String hashed = PBKDF2(password + userSalt);
if (hashed.equals(user.getHashedPassword())) {
    // 비밀번호 일치
}
```

단, 라이브러리에 따라 편의성이 조금씩 다릅니다. 어떤 방식을 택하든, 핵심은 **Salt를 반드시 저장**하고, **각 사용자마다 유니크**해야 한다는 점입니다.



## 5. Java 예시: BCrypt 사용

스프링 부트를 예로 들면, **BCryptPasswordEncoder** 클래스를 흔히 사용합니다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 5.1. 비밀번호 해싱

```java
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
String rawPassword = "password123";
String encoded = encoder.encode(rawPassword);
System.out.println(encoded);

// 결과 예시: $2a$10$kG0NZlWxIedCLq3DMSmRZ.HjOTm8xqWGXmmr9b.4Z3/cvTzGrKobS
```

문자열 앞부분 `$2a$10$…`에서 10은 cost factor(연산 강도)입니다. 값이 높을수록 해싱 연산이 느려지고, brute-force 공격도 어려워집니다.

### 5.2. 비밀번호 검증

```java
public boolean checkPassword(String rawPassword, String encodedPassword) {
    return encoder.matches(rawPassword, encodedPassword);
}
```

BCrypt가 internally Salt를 저장하고 있으므로, `matches()` 메서드 호출 시 알아서 Salt를 추출해 다시 해싱해 비교합니다.

**(BCrypt와 Salt를 가팅 쓸 수도 있습니다.)**



## 6. 테이블 스키마 예시

MVP 수준에서는 **User** 테이블에 `password` 컬럼 하나만 있으면 됩니다. 이 필드에 해시된 결과(BCrypt 문자열 등)를 저장합니다.

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL, --또는 password_hash. "해시된 패스워드"
    name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    -- ...
);
```

* `password`에는 절대 **평문**이나 **단순 해시**를 저장하면 안 됩니다.
* `VARCHAR(255)`는 충분히 긴 해시 문자열을 담을 수 있도록 잡았습니다. BCrypt 해시 길이는 약 60자지만, Argon2나 PBKDF2 문자열은 더 길어질 수 있습니다.

실무에선 이 컬럼명을 `password_hash`라고 두기도 합니다. 헷갈리지 않도록 “해시된 패스워드”임을 명시하는 편이 낫습니다.



## 7. 회원가입/로그인 로직

### 7.1. 회원가입 (Register)

1. 사용자가 회원가입 폼에서 `email`, `rawPassword`, `name` 등을 입력
2. 서버는 `BCryptPasswordEncoder`로 `rawPassword`를 해싱
3. DB에 **해시 결과**를 저장: `user.setPassword(encodedPassword)`

```java
@PostMapping("/register")
public ResponseEntity<?> register(@RequestBody RegisterRequest req) {
    if (userRepository.existsByEmail(req.getEmail())) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
                .body("이미 존재하는 이메일입니다.");
    }

    String hashed = encoder.encode(req.getPassword());
    UserEntity user = new UserEntity();
    user.setEmail(req.getEmail());
    user.setPassword(hashed);
    user.setName(req.getName());
    // ...

    userRepository.save(user);
    return ResponseEntity.ok("등록 성공");
}
```

### 7.2. 로그인 (Login)

1. 사용자가 로그인 폼에서 `email`, `rawPassword`를 입력
2. DB에서 해당 email의 `UserEntity`를 찾고, `encodedPassword = user.getPassword()`를 가져옴
3. `encoder.matches(rawPassword, encodedPassword)`로 검증
4. 일치하면 로그인 성공, JWT 발급 또는 세션 생성

```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest req) {
    UserEntity user = userRepository.findByEmail(req.getEmail())
            .orElseThrow(() -> new RuntimeException("존재하지 않는 사용자"));

    if (!encoder.matches(req.getPassword(), user.getPassword())) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body("비밀번호가 일치하지 않습니다.");
    }
    // 세션 생성 혹은 JWT 발행
    // ...
    return ResponseEntity.ok("로그인 성공");
}
```



## 8. 비밀번호 변경 & 리셋(Reset) 로직

### 8.1. 비밀번호 변경(Authenticated)

로그인한 사용자가 “비밀번호 변경”을 시도할 때,

1. 현재 비밀번호 확인 → **encoder.matches()**
2. 새 비밀번호 규칙 검사
3. 새 비밀번호를 다시 **encode** 후 DB 업데이트

### 8.2. 비밀번호 재설정(비로그인/분실)

* 사용자가 “비밀번호 찾기”를 하면, 서버는 **비밀번호 재설정 토큰**(JWT or 랜덤 문자열)을 생성해 이메일로 전송
* 사용자가 이메일에 담긴 링크를 클릭하면, 새 비밀번호를 입력할 수 있는 페이지로 연결
* 서버는 **토큰**을 검증해 유효하면, 새 비밀번호를 encode해 저장

이 토큰 또한 **DB나 캐시**에 저장해 1회성으로 사용 가능하도록 관리해야 하며, 만료시간도 설정해두어야 합니다.



