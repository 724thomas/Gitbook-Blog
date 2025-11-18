# Page 1

## 사전 예약 기능 문서화

### 목적

사전 예약 기능은 서비스 런칭 전 사용자들의 관심도를 파악하고, 초기 사용자 풀을 확보하기 위한 목적으로 구현되었습니다. 주요 기능은 다음과 같습니다:

1. **사전 예약 정보 수집**: 전화번호와 SNS URL을 통한 사용자 정보 수집
2. **실시간 알림**: 사전 예약 등록 시 슬랙을 통한 즉시 알림 전송
3. **콘솔 조회 지원**: 관리자가 콘솔에서 사전 예약 정보를 조회할 수 있도록 약관 타입 추가



### 먼저 반성해야할 점:

당연한 내용들인데, 급하게 개발을 하느라 고려하지 못한 내용은 다음과 같습니다.

1. **데이터 검증**: 서버에서 입력 데이터에 대한 철저한 검증 로직 (개발은 해놨는데 어노테이션을 뺴먹음)
2. **전화번호 암호화**: 개인정보인 전화번호를 암호화하여 저장하여 개인정보보호법 준수 및 데이터 유출시 피해 최소화



### 작업 내용

#### 1. 데이터 무결성 보장

```kotlin
    indexes = [
        Index(
            name = "ux_pre_registration_phone",
            columnList = "phone_number",
            unique = true,
        ),
    ],
```

* **전화번호 유니크 인덱스**: 동일한 전화번호로 중복 등록을 방지하기 위해 데이터베이스 레벨에서 유니크 제약 조건을 설정했습니다. 이는 애플리케이션 레벨의 중복 체크와 함께 이중 방어 메커니즘을 제공합니다.

#### 2. 입력 데이터 검증

```kotlin
    @field:NotBlank(message = "전화번호는 필수 입니다")
    @field:Pattern(
        regexp = "^01[0-9]{8,9}$",
        message = "유효하지 않은 휴대폰 번호 형식입니다.",
    )
    val phoneNumber: String,
```

* **전화번호 형식 검증**: 한국 휴대폰 번호 형식(01X-XXXX-XXXX)에 맞는 정규식을 사용하여 유효한 전화번호만 수집하도록 했습니다.
* **SNS URL 길이 제한**: 최대 200자로 제한하여 데이터베이스 스키마와 일치시켰습니다.

```kotlin
    @field:Size(max = 200, message = "SNS URL은 최대 200자 까지 입력 가능합니다.")
    val snsUrl: String?,
```

#### 3. 개인정보 처리 동의

```kotlin
    @field:AssertTrue(message = "개인정보 수집 및 이용에 동의해야 합니다.")
    val consentAgreed: Boolean? = null,
```

* **동의 필수 체크**: `@AssertTrue` 어노테이션을 사용하여 개인정보 수집 및 이용에 대한 동의를 필수로 받도록 구현했습니다.

#### 4. 중복 등록 방지 로직

```kotlin
        if (preRegistrationRepository.existsByPhoneNumber(entity.phoneNumber)) {
            throw BadRequestException(ExceptionCode.DUPLICATED_PRE_REGISTRATION, "이미 사전등록된 번호입니다.")
        }
```

* **사전 중복 체크**: 저장 전에 `existsByPhoneNumber` 메서드를 통해 중복 여부를 확인하여 사용자에게 명확한 에러 메시지를 제공하고, 불필요한 데이터베이스 작업을 방지합니다.

#### 5. 슬랙 알림 기능

```kotlin
        aroundServiceNotiSlackSender.sendSlackMessage(
            SendSlackMessageRequest(
                title = "사전 등록 접수",
                messageType = MessageType.INFO,
                message = """
                    *전화번호* : ${saved.phoneNumber}
                    *SNS URL* : ${saved.snsUrl ?: "-"}
                    *누적* : ${count} 건
                """.trimIndent(),
            ),
        )
```

* **누적 건수 포함**: 슬랙 메시지에 누적 등록 건수를 포함하여 실시간으로 사전 예약 현황을 파악할 수 있도록 했습니다.
* **조건부 빈 생성**: `@ConditionalOnProperty`를 사용하여 슬랙 웹훅 설정이 없을 경우 빈이 생성되지 않도록 하여 로컬 환경에서의 오류를 방지.

```kotlin
@ConditionalOnProperty(
    value = ["app.slack.webhook.path.nockplace-service-****"],
    matchIfMissing = false,
)
```

**슬랙 웹훅 설정**

슬랙 알림 기능을 위해 다음과 같은 설정이 필요합니다:

1.  **환경 변수 설정** (`api.properties`):

    ```
    SLACK-WEBHOOK-PATH-NOCKPLACE-SERVICE-NOTI= ******
    ```
2.  **헬름차트 환경 변수 설정**: 각 서비스의 `values.yaml` 파일 (dev/prod 환경 모두)에 다음 환경 변수를 추가했습니다:

    ```yaml
    - name: SLACK-WEBHOOK-PATH-NOCKPLACE-SERVICE-****
      valueFrom:
        configMapKeyRef:
          name: global-confi****
          key: app.slack.webhook.path.nockplace-service-****
    ```

    이 설정은 다음 서비스들의 dev/prod 환경에 적용되었습니다:

    * `homeknockzone-console-api`
    * `homeknockzone-around-api`
    * `homeknockzone-around-biz-api`
    * `homeknockzone-batch`
    * `homeknockzone-event`

이를 통해 각 환경에서 슬랙 알림이 정상적으로 동작하도록 구성했습니다.

#### 6. 콘솔 조회를 위한 약관 타입 추가

```kotlin
    PRE_REGISTRATION("사전등록 시, 필요한 약관", TermsServiceType.NOCKPLACE),
```

* **약관 카테고리 추가**: 콘솔에서 사전 등록 시 사용되는 약관을 조회할 수 있도록 `PRE_REGISTRATION` 카테고리를 추가했습니다.

#### 7. 엔티티 생성 패턴

```kotlin
    companion object {
        fun create(phoneNumber: String, snsUrl: String?, consentAgreed: Boolean) =
            PreRegistration(
                phoneNumber = phoneNumber,
                snsUrl = snsUrl,
                consentAgreed = consentAgreed,
            )
    }
```

* **팩토리 메서드 패턴**: `companion object`의 `create` 메서드를 통해 엔티티 생성을 캡슐화로 향후 엔티티 생성 로직 변경 시 유지보수성을 높입니다.

#### 8. 성능 이슈 가능성

```kotlin
        val count = preRegistrationRepository.findAll().size
```

* **전체 조회 후 카운트**: 현재는 기존 메서드 `findAll().size`를 재사용하여 전체 건수를 조회하고 있습니다. 이는 데이터가 많아질 경우 성능 문제를 야기할 수 있습니다만 행복한 고민이겠죠!!

### 시간이 있으면 고도화할 수 있는 방안들과 각각의 장단점

#### 1. 성능 최적화: count() 메서드 사용

**방안**: `findAll().size` 대신 `count()` 메서드를 사용

**장점**:

* 데이터베이스에서 직접 COUNT 쿼리를 실행하여 메모리 사용량 감소
* 대용량 데이터에서도 빠른 응답 시간 보장
* 네트워크 트래픽 감소

**단점**:

* 구현 변경이 필요 (현재는 단순하지만)

#### 2. 슬랙 알림 실패 처리

**방안**: 슬랙 알림 전송 실패 시 재시도 로직 또는 큐를 통한 비동기 처리

**장점**:

* 일시적인 네트워크 오류에 대한 복원력 향상
* 알림 실패로 인한 데이터 손실 방지
* 사용자 경험 개선 (알림 실패가 등록 실패로 이어지지 않음)

**단점**:

* 구현 복잡도 증가
* 추가 인프라 필요 (메시지 큐 등)
* 재시도 로직 관리 필요

**구현 방안**:

* Spring의 `@Async`를 활용한 비동기 처리
* RabbitMQ, Kafka 등의 메시지 큐 활용
* 실패 시 로깅 및 모니터링 강화

#### 3. 전화번호 암호화

**방안**: 개인정보인 전화번호를 암호화하여 저장

**장점**:

* 개인정보보호법 준수 강화
* 데이터 유출 시 피해 최소화
* 보안 수준 향상

**단점**:

* 암호화/복호화 오버헤드(Minor)
* 키 관리 복잡도 증가
* 검색 및 조회 시 성능 저하 가능성
* 기존 데이터 마이그레이션 필요

**구현 방안**:

* JPA의 `@Convert` 어노테이션을 활용한 자동 암호화/복호화
* AES-256 등 강력한 암호화 알고리즘 사용

#### 4. 등록 제한 기능

**방안**: IP 주소 또는 디바이스 기반 등록 제한 (예: 1일 1회 제한)

**장점**:

* 악의적인 대량 등록 방지 (다른 사람의 번호로 신청할 수 있음)
* 데이터 품질 향상
* 서버 부하 감소

**단점**:

* 구현 복잡도 증가
* 공유 IP 환경에서의 정당한 사용자 제한 가능성
* 추가 저장소 필요 (Redis 등)

**구현 방안**:

* Redis를 활용한 IP/디바이스 기반 제한
* Rate Limiting 라이브러리 활용

#### 5. 데이터 만료 및 정리

**방안**: 일정 기간(예: 1년) 경과 후 자동으로 데이터 삭제 또는 보관

**장점**:

* 개인정보 보관 기간 준수
* 데이터베이스 용량 관리
* 법적 요구사항 준수

**단점**:

* 배치 작업 구현 필요
* 데이터 복구 불가능
* 비즈니스 요구사항과의 충돌 가능성

**구현 방안**:

* Spring Batch를 활용한 주기적 정리 작업
* `createdAt` 기준으로 만료된 데이터 삭제

#### 6. 통계 및 대시보드

**방안**: 사전 예약 통계를 제공하는 API 및 대시보드 구현

**장점**:

* 실시간 현황 파악
* 데이터 기반 의사결정 지원
* 비즈니스 인사이트 도출

**단점**:

* 추가 개발 리소스 필요
* 성능 최적화 필요 (집계 쿼리 등)

**구현 방안**:

* 일별/주별/월별 통계 API 제공

### 결론

현재 구현된 사전 예약 기능은 기본적인 요구사항을 충족하며, 데이터 무결성과 개인정보 보호를 고려한 설계가 되어 있습니다. 특히 다음과 같은 점이 잘 구현되었습니다:

1. **중복 방지**: 데이터베이스 레벨과 애플리케이션 레벨의 이중 방어
2. **개인정보 보호**: 동의 필수 체크 및 적절한 데이터 구조
3. **실시간 알림**: 슬랙을 통한 즉시 알림 기능

만약, 고도화를 하게된다면 다음과 같은 개선이 필요합니다:

1. **즉시 개선 필요**: `findAll().size`를 `count()`로 변경하여 성능 최적화
2. **단기 개선**: 전화번호 정규화 및 슬랙 알림 실패 처리
3. **중장기 개선**: 전화번호 암호화, 등록 제한, 데이터 만료 정리, 통계 기능
