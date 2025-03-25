---
description: Vault
---

# Vault (HashiCorp Vault)

## HashiCorp Vault 란?

Vault(HashiCorp Vault)는 비밀정보 관리와 데이터 암호화를 위한 중앙화된 플랫폼입니다. 애플리케이션의 민감한 데이터를 안전하게 저장하고 접근을 제어하는 오픈소스 보안도구입니다.

민감한 데이터:

* JWT secret
* API Key
* DB Password
* 인증서
* 암호화 키 등

Vault는 권한이 있는 사용자나 애플리케이션만 접근할 수 있도록 인증 및 권한부여를 수행하며, 감사 로그를 통해 누가 어떤 비밀에 접근했는지 추적할 수 있습니다.



## Vault를 사용하는 이유

일반적으로 애플리케이션 개발 시 민감한 값들을 application.yml 등의 설정 파일이나 환경변수에 직접 넣는 경우가 많습니다. 하지만 이러한 방식에는 문제점들이 있습니다.

* 소스 노출 위험: 설정 파일에 포함된 비밀값이 깃 저장소 등에 올라갈 수도 있습니다.
* 배포 및 관리 어려움: 코드나 설정에 하드코딩된 비밀값은 변경하려면 애플리케이션을 다시 빌드/배포해야 하므로 유연성이 떨어집니다.
* 접근 통제 부족: 개별 파일에 분산되어 관리되면 어떤 사용자가 어떤 비밀정보를 볼 수 있는지 중앙에서 통제하기 어렵습니다.
* 자동 ROTATION 부재: 고정된 비밀값은 장기간 변경되지 않을 수 있고, 유출 시 즉각 교체하기 어렵다는 문제가 있습니다.



위 문제들을 해결하기 위해 vault를 사용하며, 아래와 같은 이점들이 있습니다.

* 중앙집중 암호화 저장: 모든 비밀값을 내부적으로 암호화하여 저장하므로, 데이터가 저장소에 있거나 전송되는 중에도 안전합니다. (TLS 통신)
* 세분화된 접근제어: Vault는 정책에 기반한 권한 체계를 제공하여, 각 애플리케이션이나 사용자별로 필요한 비밀만 접근하도록 제어합니다. 이를 통해 최소 권한 원칙을 구현하고 비인가 접근을 차단합니다.
* 동적 생성 및 자동 만료: 비밀값을 필요할 때 동적으로 생성하고, 일정 시간 후 만료시키는 기능을 제공합니다. 예를 들어, 임시 DB 계정 비밀번호를 발급하고, 사용 후 자동 폐기할 수 있습니다.
* 감사 및 추적: Vault의 모든 접근은 감사 로그에 남기도록 설정되어 있어서, 누가 언제 비밀을 조회/갱신했는지 추적이 가능합니다.



## Vault 활용 주요 시나리오 (Spring Boot 관점)

### 데이터베이스 인증정보 관리

애플리케이션이 사용하는 DB의 사용자명과 비밀번호를 Vault의 KV(Key-Value) 저장소나 DB 시크릿 엔진에 보관하고, 애플리케이션은 이를 Vault로부터 받아 사용합니다. 예를 들어 Vault의 MySQL 시크릿 엔진을 활용하면 **애플리케이션 기동 시점마다 읽기전용 DB 계정 정보를 동적으로 발급**받아 사용할 수 있고, Vault가 이 계정의 **수명 및 회전을 관리**합니다​.  이를 통해 데이터베이스 자격증명의 노출을 막고 주기적인 비밀번호 변경도 자동화할 수 있습니다.

### API 키 및 서드파티 인증 토큰 관리

외부 API 연동에 필요한 **API 키, OAuth 토큰** 등을 Vault에 저장해 두고 Spring Boot에서 필요할 때 가져다 씁니다. 예를 들어 AWS 클라우드 서비스를 호출하기 위한 Access Key와 Secret Key를 Vault의 AWS 시크릿 엔진을 통해 관리하면, Vault가 **일회용 IAM 자격증명**을 발급해주어 애플리케이션이 이를 사용하는 패턴도 가능합니다. 이처럼 클라우드 자격증명, 서드파티 API 키 등을 Vault로 일원화하여 관리하면 키 유출 위험이 감소합니다.

### 애플리케이션 자체 비밀값 관리

JWT(JSON Web Token) 인증을 사용하는 Spring Boot 애플리케이션이라면, **토큰을 서명하고 검증하기 위한 Secret Key**(예: Access Token 및 Refresh Token 서명용 비밀키)를 Vault에 보관할 수 있습니다. 이를 통해 코드나 설정 파일에 토큰 서명키를 하드코딩하지 않아도 되며, 운영 환경에서 키를 교체해야 할 때 Vault의 값을 변경하는 것으로 애플리케이션에 주입되는 키를 갱신할 수 있습니다. Vault를 사용하면 이밖에도 애플리케이션에서 사용하는 **암호화 키, SSL/TLS 인증서의 개인키** 등을 안전하게 저장하고 필요 시 불러오는 형태로 활용할 수 있습니다​

### 환경별 설정 관리

Spring Boot 마이크로서비스 아키텍처에서 Vault는 Config Server처럼 사용되어 **환경별로 다른 비밀 설정값**을 제공하는 저장소로 쓰일 수 있습니다. 예를 들어 각 마이크로서비스의 개발(dev), 스테이징(staging), 운영(prod) 환경에 대한 별도의 Vault 경로를 두고, **환경에 맞는 DB 비밀번호나 API 키**를 저장해두면 애플리케이션은 현재 프로파일(profile)에 따라 Vault에서 올바른 값을 받아 사용할 수 있습니다. 이 방식은 Jenkins 등의 CI/CD 파이프라인에서 **배포 시점에 Vault로부터 비밀을 주입**받는 방식이나, 애플리케이션이 **구동 시 Vault에 접근**하여 설정을 로드하는 방식 모두 활용됩니다.



## 로컬 개발 환경에서 Vault 설치 및 설정(Docker-compose)

프로젝트 구조:

```yaml
project-root/
├── vault/
│   ├── config/
│   │   └── vault.hcl       # Vault 설정 파일
│   └── init/
│       └── init.sh         # 초기 비밀 데이터를 등록할 스크립트
└── docker-compose.yml
```



1. Vault 설정 파일 작성

```hcl
storage "file" { # Vault 데이터를 저장할 위치를 지정
  path = "/vault/data"   # Docker 내부 경로 (볼륨 마운트됨)
} # 여기서 /vault/data는 Docker 내부 경로고, docker-compose.yml에서 vault-data라는 볼륨으로 연결

listener "tcp" { #Vault의 리슨 포트을 지정하는 부분
  address = "0.0.0.0:8200"   # 모든 인터페이스에서 접근 허용
  tls_disable = 1            # 개발 환경에서는 TLS 생략
}

disable_mlock = true         # Linux 환경에서 메모리 잠금 기능 비활성화
ui = true                    # Vault UI 사용 가능 (http://localhost:8200/ui)
```



2. docker-compose.yml 작성

```yaml
version: '3.8'

services:
  vault:
    image: hashicorp/vault:latest
    container_name: vault
    ports:
      - "8200:8200"
    cap_add:
      - IPC_LOCK
    environment:
      VAULT_LOCAL_CONFIG: "" # 환경변수 JSON 설정. vault.hcl를 사용하기 떄문에 불필요.
    volumes:
      - ./vault/config:/vault/config #로컬 설정파일을 컨테이너에 전달 (Vault 설정 파일 vault.hcl)
      - ./vault/init:/vault/init #초기화 스크립트(init.sh)를 실행하기 위해 마운트
      - vault-data:/vault/data #Vault 비밀 데이터를 저장하는 디렉토리 (여기 저장되면 휘발성 아님)
    entrypoint: ["/bin/sh", "-c"] # Vault 서버를 실행할 때 사용할 명령어
    command: |
      vault server -config=/vault/config/vault.hcl 

  vault-init:
    image: hashicorp/vault:latest
    depends_on: #Vault 서버가 먼저 실행되고 나서 이 컨테이너가 실행되도록 하는 설정
      - vault
    volumes: #초기화 스크립트가 담긴 로컬 디렉토리를 컨테이너 안에 넣는 설정
      - ./vault/init:/vault/init
    entrypoint: ["/bin/sh", "/vault/init/init.sh"] #컨테이너가 실행될 때, init.sh 스크립트를 자동으로 실행하도록 하는 부분

volumes: #도커 볼륨 이름
  vault-data:
```



3. Vault 초기 설정 및 비밀 등록 스크립트

```sh
#!/bin/sh

# 1. Vault 서버가 뜰 때까지 기다림
until curl -s http://vault:8200/v1/sys/health | grep -q 'initialized'; do
  echo "[vault-init] waiting for vault to start..."
  sleep 1
done

# 2. Vault 초기화 (키 1개, 사용 시 1개만 필요)
vault operator init -key-shares=1 -key-threshold=1 > /vault/init/init.log

# 3. unseal 키 및 root token 추출
UNSEAL_KEY=$(grep 'Unseal Key 1:' /vault/init/init.log | awk '{print $NF}')
ROOT_TOKEN=$(grep 'Initial Root Token:' /vault/init/init.log | awk '{print $NF}')

# 4. Vault Unseal 잠금 해제
vault operator unseal $UNSEAL_KEY

# 5. Vault 로그인
export VAULT_TOKEN=$ROOT_TOKEN

# 6. KV 엔진 활성화 (secret/ 경로에 키-값 저장소 만들기)
vault secrets enable -path=secret kv

# 7. 비밀값 등록
vault kv put secret/myapp access-token-secret="ACCESS_SECRET" refresh-token-secret="REFRESH_SECRET"

echo "Vault Initialized and Secrets Loaded. Root Token: $ROOT_TOKEN"
```



4. 실행

```
Docker-compose up -d

vault 서버 컨테이너 실행됨
설정(vault.hcl)에 따라 포트 열리고 UI도 켜짐
vault-init 컨테이너가 Vault 상태 체크 후:
- 자동 초기화
- 잠금 해제(Unseal)
- Root 로그인
- secret/myapp 경로에 민감 정보 등록
Spring Boot는 그 경로를 통해 자동으로 비밀을 가져감
```





5. Spring Boot 연동

build.gradle 설정

```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.cloud:spring-cloud-starter-vault-config'
}

dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:2023.0.0"
    }
}
```

application.yml 설정

```yaml
spring:
  application:
    name: myapp  # Vault의 secret/<name> 경로와 연결됨
  config:
    import: vault://

  cloud:
    vault:
      uri: http://localhost:8200        # Vault 서버 주소
      authentication: TOKEN             # 토큰 인증 방식 사용
      token: root-token                 # 개발용 Root 토큰 (운영에선 AppRole을 사용해야 함)
      kv:
        backend: secret                 # 사용하는 KV 엔진 경로 (secret/)
        default-context: myapp          # secret/myapp 위치에서 값을 읽음

```



6. 비밀값 사용

```
@Value("${access-token-secret}")
private String accessTokenSecret;

@Value("${refresh-token-secret}")
private String refreshTokenSecret;

또는

@ConfigurationProperties(prefix = "jwt")
public class JwtProperties {
    private String accessSecret;
    private String refreshSecret;
    // getter/setter
}

# application.yml
jwt:
  accessSecret: ${access-token-secret}
  refreshSecret: ${refresh-token-secret}
```



## Spring Boot 프로젝트와 Vault 연동 방법

1. build.gradle 의존성 추가
2. application.yml 설정



## 값 사용하는 방법

Vault에 아래처럼 저장된 상태라고 가정:

vault kv put secret/myapp access-token-secret="ACCESS\_SECRET" refresh-token-secret="REFRESH\_SECRET"

### 방법 1

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TokenController {

    @Value("${access-token-secret}")
    private String accessTokenSecret;

    @Value("${refresh-token-secret}")
    private String refreshTokenSecret;

    // 이 값을 JWT 서명 등에 사용할 수 있음
}
```

### 방법 2

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "jwt")
public class JwtProperties {
    private String accessSecret;
    private String refreshSecret;

    // Getter/Setter 필수
}

```

```yaml
# application.yml
jwt:
  accessSecret: ${access-token-secret}
  refreshSecret: ${refresh-token-secret}
```



## 결과

Spring Boot를 실행하면 다음과 같은 일이 자동으로 발생:

1. Vault 서버에 `http://localhost:8200`로 연결
2. `secret/myapp` 경로에서 키-값을 읽음
3. `application.yml`이나 `@Value`, `@ConfigurationProperties`를 통해 프로퍼티 주입
