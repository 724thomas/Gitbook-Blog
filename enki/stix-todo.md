---
description: 'STIX 2.1 지식 사전: 위협 인텔리전스 완벽 가이드'
---

# STIX 2.1 (Structured Threat Information Expression)

## Part 1: STIX 2.1 기초 철학과 아키텍처

{% hint style="info" %}
데이터의 진화 흐름: 원시 데이터 -> 정보 -> 지능 -> 공유

1. 텔레메트리 수집 (PC -> EDR) -> SCO와 Observed Data\
   PC 엔드포인트에서 발생하는 프로세스 생성(SCO) 등의 raw 텔레메트리가 EDR에 수집(Observed Data).\
   STIX는 이걸 SCO(STIX Cyber-observable Object)로 표현. 이를 언제 관찰했는지 묶어주는 것이 Observed Data SDO.
2. 의미있는 데이터 추출 (Sigma Rule 탐지) -> Indicator\
   텔레메트리 속에서 악의적인 행위를 찾아내기 위해 Sigma Rule을 작성하여 EDR에 적용.\
   STIX는 탐지 룰 자체를 Indicator SDO로 표현. STIX 패터닝외에 stix, pcre, snort, yara, sigma suricata도 지원.
3. 행동/패턴 식별 (IOC, IOA, TTP 도출) -> Attack Pattern, Malware, Threat Actor\
   Sigma Rule을 바탕으로 알람들을 분석하고, 해커들의 공격임이 밝혀짐.\
   STIX는 Indicator를 바탕으로 큰 그림을 그리게 됩니다. Relationship SRO를 통해 타겟을 연결.
4. CTI 공용어 (STIX 포맷팅) -> Bundle 포장 및 공유\
   도출된 위협 인텔리전스를 공유하기 위해 STIX 포맷으로 변환.\
   STIX는 생성된 모든 객체(Indicator, Attack Pattern, Malware, Obeserved Data, Relationship 등)를 하나의 컨테이너로 묶기 위해 Bundle 객체 사용.
{% endhint %}



### 1.1 STIX와 TAXII

#### 사이버 위협 인텔리전스(CTI) 표준화의 필요성

* 사이버 위협 인텔리전스: \
  해커들의 공격 동기, 사용 도구, 침해 지표(IOC) 등 방어를 위해 수집 및 분석된 모든 정보를 의미.
* 표준화의 이유: \
  보안 조직들이 사이버 위협 인텔리전스(CTI)를 서로 일관되고 기계가 읽을 수 있는 방식으로 공유하기 위해서는 공통된 언어 규격이 필요.
* STIX의 역할:\
  공통된 언어 규격을 위해 만들어진 언어이자 데이터를 구조화하는 직렬화 포맷.
* 도입 효과:\
  보안 커뮤니티는 미래에 발생할 컴퓨터 기반 공격을 더 잘 이해하고, 빠르고 효과적으로 예측 및 대응할 수 있습니다.\
  협업 기반의 위협 분석, 자동화된 위협 정보 교환, 자동화된 탐지 및 차단 능력 향상.

> Page 12

#### STIX(표현 언어)와 TAXII(전송 프로토콜)의 역할 분담

* 전송 불가지론(Transport-Agnostic):\
  STIX 2.1은 본질적으로 전송 방식에 구애받지 않도록 설계. 데이터의 구조와 직렬화 방식만을 정의.
* TAXII의 역할: \
  STIX객체를 네트워크를 통해 서로 안전하게 전송하기 위해 특별히 설계된 애플리케이션 계층 통신 프로토콜

{% hint style="info" %}
TAXII vs HTTPS

TAXII는 기존의 안전한 웹 표준인 HTTPS 위에서 동작하는 RESTful API 규격.

데이터를 직렬화: JSON, 암호화: TLS, 전송: HTTP.

TAXII: "사이버 위협 정보를 주고받기 위해 어떤 URL 엔드포인트를 써야 하고, 어떤 메시지 패턴을 사용해야 하는지"를 정의한 API 아키텍처. (예. GET: 해당 URL로 STIX 데이터를 받음, POST: STIX 데이터를 서버에 저장)
{% endhint %}

#### STIX 1.x, 2.0에서 2.1로의 진화와 주요 변경 사항 (SCO의 독립, Cyber Observable Container의 폐기 등)

* STIX 1.x:\
  XML 스키마가 사용되어서 파싱이 어려웠고, 객체 안에 다른 객체를 깊게 중첩할 수 있어 구조가 까다로움.\
  별개의cybOX(Cyber Observable eXpression) 규격을 사용하여 러닝 커브가 높음
* STIX 2.0:\
  XML에서 벗어나 JSON으로 포맷을 의무화.\
  데이터를 SDO와 SCO(노드), SRO(간선)로 나누어 연결하는 그래프 기반 데이터 모델 도입\
  Cyber Observable 데이터가 Observed Data의 자식으로 있어야만 하는 한계. (graph within a graph).
* STIX 2.1:\
  구형 Cyber Observable Container 방식을 폐기하고, SCO(STIX Cyber-observable Object)를 일반 SDO처럼 그래프의 최상위 독립 객체로 승격.\
  SCO가 독립하면서 SRO를 사용해 SCO끼리도 직접적인 관계 연결이 가능해졌습니다. \
  또한 7개의 새로운 객체들이 도입되었으며, 데이터 작성자의 확신을 나타내는 신뢰도 속성이 추가됐습니다.

```json
// Graph within a graph (STIX 1.X): 데이터 중복 발생
{
  "type": "threat-actor",
  "name": "해커 A",
  "used_ip": {
    "type": "ipv4-addr",
    "value": "198.51.100.3"  // <-- 첫 번째 등장
  }
},
{
  "type": "threat-actor",
  "name": "해커 B",
  "used_ip": {
    "type": "ipv4-addr",
    "value": "198.51.100.3"  // <-- 두 번째 등장 (복사 붙여넣기)
  }
},
{
  "type": "malware",
  "name": "악성코드 C",
  "communicates_with_ip": {
    "type": "ipv4-addr",
    "value": "198.51.100.3"  // <-- 세 번째 등장 (복사 붙여넣기)
  }
}

// Graph-based Model (STIX 2.1): 재사용 및 정규화

// 1. 공통으로 쓸 증거물(SCO)을 딱 "하나만" 만듭니다.
{
  "type": "ipv4-addr",
  "id": "ipv4-addr--1234",
  "value": "198.51.100.3"
},

// 2. 고차원 개념(SDO)들을 만듭니다.
{ "type": "threat-actor", "id": "actor--A", "name": "해커 A" },
{ "type": "threat-actor", "id": "actor--B", "name": "해커 B" },
{ "type": "malware",      "id": "malware--C", "name": "악성코드 C" },

// 3. 관계(SRO)를 통해 화살표(간선)만 그어줍니다.
{ "type": "relationship", "source_ref": "actor--A", "target_ref": "ipv4-addr--1234" },
{ "type": "relationship", "source_ref": "actor--B", "target_ref": "ipv4-addr--1234" },
{ "type": "relationship", "source_ref": "malware--C", "target_ref": "ipv4-addr--1234" }
```



## 1.2 그래프 기반 데이터 모델 (Graph-Based Model)

#### 인텔리전스를 연결하는 노드(Node)와 간선(Edge)의 이해

STIX는 노드와 간선으로 연결된 그래프입니다.

* 노드:\
  STIX Domain Objects(SDO)와 STIX Cyber-observable Objects(SCO)가 그래프의 노드를 정의
* 간선:\
  외부 STIX Relationship Objects(SRO)와 내부 속성에 포함된 내장된 관계(embedded relationships)가 간선을 정의.

이 그래프 모델로 인해 분석가들은 테이블 형태로는 불가능 했던 복잡한 상관관계를 일관되고 모듈화된 구조로 그려낼 수 있게 됐습니다.

#### STIX Domain Objects (SDO): 위협의 '맥락'과 '개념'

위협 분석가들이 위협 환경을 이해할 때 생성하거나 다루는 행동 및 구성 개념을 나타내는 고차원적인 인텔리전스 객체. Attack Pattern, Campaign, Malware, Threat Actor 등 18개의 객체가 정의.

"이 사건의 배후는 누구인가?", "어떤 전술을 사용했는가?"와 같은 해석과 컨텍스트를 담는 객체.

#### STIX Cyber-observable Objects (SCO): 관찰된 '사실'과 '증거'

네트워크나 호스트에서 발생한 관찰된 사실(observed facts)을 나타내는 객체. 이 객체들은 네트워크나 호스트에서 '무엇이 일어났는지'에 대한 사실만을 기록하며, '누가', '언제', '왜' 했는지는 캡처하지 않습니다.\
IPv4, File, Process, Windows Registry Key 등이 포함됩니다.

순수한 현장의 증거물로, SDO(맥락)가 SCO(사실)를 뒷받침하는 컨텍스트로 사용합니다.\
"우리가 수집한 이 증거(SCO)가 결국 해커 조직(SDO)의 소행이다" 라는 고차원적인 이해를 전달합니다.

#### STIX Relationship Objects (SRO): 객체 간의 '관계' 연결

SRO는 SDO끼리, SCO끼리, 또는 SDO와 SCO를 서로 연결하여 위협 환경에 대한 보다 완전한 이해를 형성하는 객체. 대표적인 SRO로는 두 객체간 관계를 서술하는 Relationship 객체와, 특정 지표를 목격했음을 알리는 Sighting 객체가 있습니다.

Indicator(SDO)가 Malware(SD)를 indicates라는 SRO를 연결함으로써, "이 탐지 룰로 저 악성코드를 잡을 수 있다"는 구체적인 활용 방안이 완성.

#### SDO, SCO, SRO 이해하기

* SDO:\
  사건의 배후, 동기, 범행 수법 등 분석가의 고차원적인 분석과 해석이 들어간 지능 정보
* SCO:\
  해킹이 발생한 네트워크나 PC에서 발견된 감정이나 해석이 1%도 섞이지 않은 100% 순수한 증거물
* SRO:\
  용의자와 증거물을 서로 연결해주는 빨간색 실(관계).

{% hint style="info" %}
예시1:\
랜섬웨어(Ransomware) 악성코드가 명령제어(C2) 서버인 '198.51.100.3' IP 주소와 통신한다.\
SDO: Malware - 분석가가 악성코드라고 부여한 '맥락'\
SCO: IPv4-Addr - 현장에서 발견된 팩트이자 증거물인 IP주소\
SRO: communicates-with - 악성코드가 IP주소와 통신한다는 것을 나타내는 관계

예시2:\
"해킹 그룹 'Kimsuky'가 특정 대상을 속이기 위해 '스피어 피싱(Spear Phishing)' 기법을 사용했다."\
SDO1: Kimsuky - 배후로 지목된 해킹 그룹(용의자)\
SDO2: Spear Phishing - 구체적인 범행 수법(TTP)\
SRO: uses - 위협 행위자가 해당 공격 패턴을 사용하여 공격을 수행함을 나타냄

예시3:\
"악성 도메인 'evil.com'(SCO)이 IP 주소 '10.0.0.1'(SCO)로 리졸브(해석)(SRO)된다."

예시4:\
"최근 벌어지는 'Operation Ghost' 캠페인(SDO)은 유명 블로그 플랫폼의 취약점인 'CVE-2014-0160'(SDO)을 표적으로 삼고 있다(SRO)."

예시5:\
"해커가 구축한 '봇넷(Botnet)(SDO)' 인프라가 피해자들에게 'Zeus' 트로이목마 악성코드(SDO)를 유포(전달)(SRO)하고 있다."



SDO vs SCO: "이 객체가 감정이나 목적이 없는 단순한 IT 자산인가?"
{% endhint %}

> Page 13, 14

### 1.3 데이터 직렬화(Serialization)와 전송

#### JSON 기반의 필수 구현(MTI) 포맷 이해하기

&#x20;STIX는 특정 저장소나 직렬화 방식에 독립적으로 정의되어 있습니다. 하지만 시스템 간의 상호운용성을 위해 STIX 2.1의 필수 구현 직렬화 포맷은 \[RFC7493], \[RFC8259]에 정의된 UTF-8 인코딩 JSON으로 지정되어 있습니다.

"STIX를 지원하는 제품을 만들때 다른 포맷을 지원해도 되지만, JSON은 무조건 파싱하고 생성할 수 있어야 한다"

#### 속성 명명 규칙 (소문자, 언더스코어, `_ref`, `_bin` 등의 접미사 규칙)

STIX를 코드로 작성하거나 파싱할 때 반드시 지켜야 하는 엄격한 명명 규칙 존재. 변수명만 보고 데이터의 성격을 유추하기 위한 목적입니다. 또한 파서의 성능과 직결되기 때문에 시스템의 처리 속도를 높이고 오류를 줄여줍니다.

* 기본 규칙:\
  모든 타입 이름, 속성 이름, 리터럴(값)은 소문자. (IANA 레지스트리의 정규 이름 등 외부 표준을 참조할 떄는 예외)
* 단어 구분:\
  속성 이름의 단어는 ( \_ ) 구분. (eg. created\_by\_ref)\
  타입 이름과 문자열 열거형은 ( - ) 구분. (eg. attack-pattern)
* 길이 제한: 모든 타입, 속성 객체, 어휘 용어의 이름은 3\~250자이고 알파벳으로 시작

{% hint style="info" %}
핵심 접미사(suffix) 규칙

* \_ref(단일 참조):\
  속성 값이 다른 STIX 객체의 ID 참조를 하나만 포함할때 사용 (created\_by\_ref = id)
* \_refs(다중 참조):\
  속성 값이 여러 STIX 객체의 ID 참조 리스트를 포함할때 사용 (object\_marking\_refs = list(id))
* \_bin(바이너리):\
  속성 값이 바이너리 값을 포함할때 사용
* \_hex(16진수):\
  속성 값이 16진수 값을 포함할때 사용
* _enc(인코딩):_\
  _원본 속성의 값이 기본(UTF-8)과 다른 대체 인코딩을 사용할 경우, 어떤 인코딩을 썼는지 명시하기 위해 사용._\
  _원본 속성이 없을 때는_ enc 속성도 존재해서는 안됩니다.
{% endhint %}

> Page 32

***

## Part 2: 데이터 타입과 공통 규칙

### 2.1 16가지 공통 데이터 타입 (Common Data Types)

STIX 2.1은 사이버 위협 데이터를 표현하기 위해 총 16가지의 공통 데이터 타입을 정의합니다. (이 규격을 어기면 파싱 에러가 발생할 수 있습니다.)

#### 기본 타입: boolean, integer, float, string

* boolean: true/false
* integer: -2\*\*53+1, 2\*\*53-1
* float: 실수형
* string: UTF-8 문자의 연속

#### 식별 및 참조: identifier, external-reference

* identifier (\<object-type>--\<UUID>): \
  STIX 객체를 고유하게 식별하는 ID. (eg. indicator--e2e1a340-4415...)
* external-reference (외부 참조):\
  STIX 외부에 있는 정보(취약점 데이터베이스, 보고서 PDF 링크 등)를 가리킬 때 사용.\
  source\_name은 필수이고, url, external\_id, hash, description 중 하나는 필수.

```json
// external-reference to a VERIS Community Database
{
    ...
    "external_references": [
        {
            "source_name": "veris",
            "external_id": "0001AA7F-C601-424A-B2B8-BE6C9F5164E7",
            "url": "https://github.com/vz-risk/VCDB/blob/125307638178efddd3ecfe2c267ea434667a4eea/data/json/validated/0001AA7F-C601-424A-B2B8-BE6C9F5164E7.json",
            "hashes": {
                "SHA-256": "6db12788c37247f2316052e142f42f4b259d6561751e5f401a1ae2a6df9c674b"
            }
        }
    ],
...
}
```

#### 암호화 및 데이터: hashes, hex, binary

* hashes:\
  하나 이상의 암호화 해시값을 담는 딕셔너리(Key: Val). Key는 STIX에서 지정한 어휘를 사용해야합니다. (eg. MD5, SHA-256. 가능하면 무조건 SHA-256 해시를 포함할 것을 권장)
* hex:\
  16 진수 문자로 인코딩된 바이트 배열. 문자열 길이는 짝수여야하며, a\~f, 0\~9만 허용. 커스텀 속성에 쓸 때는 이름이 \_hex로 끝나야합니다.
* binary: 바이트 시퀀스를 나타내며, JSON에서는 Base64로 인코딩된 문자열로 저장됩니다. 커스텀 속성에 쓸 때는 \_bin로 끝나야합니다.

#### 구조화 타입: dictionary, list, kill-chain-phase, timestamp

* dictionary:\
  Key는 ASCII 문자(a-z, 0-9, 하이픈, 언더스코어) 로만 구성되어야하며, 길이는 250 이하. 비어있는 딕셔너리는 금지. 값이 없다면 속성 자체를 생략.
* list:\
  리스트 안의 값들은 모두 동일한 타입이어야하고, 비어있는 리스트는 사용 금지.
* kill-chain-phase:\
  공격의 특정 단계를 나타냅니다. kill\_chain\_name과 phase\_name 두 가지 필수 속성으로 구성됩니다.
* timestamp:\
  RFC 3339 규격을 따라야하며, UTC 시간임을 나타내는 Z로 끝나야합니다. (eg. 2016-01-20T12:31:12.123Z)

#### 어휘 타입: enum, open-vocab

* enum:\
  STIX 스펙에서 하드코딩해둔 단어 목록.
* open-vocab (개방형 어휘):\
  STIX가 추천하는 단어 목록. 가능하면 추천 목록에서 고르는 것이 좋지만, 상황에 맞게 새로운 단어를 써도 에러가 발생하지 않습니다.

#### 2.2 모든 객체의 공통 속성 (Common Properties)

* `type`, `spec_version`, `id`, `created`, `modified` 등 필수/선택 속성
* 객체 생성자(`created_by_ref`)와 신뢰도(`confidence`) 지정 방법

> Page 28\~35

***

### Part 3: 식별(ID)과 버전 관리(Versioning)

#### 3.1 객체 식별자(ID) 체계의 비밀

* SDO, SRO, Meta 객체를 위한 UUIDv4 생성 규칙
* SCO를 위한 결정론적(Deterministic) UUIDv5 생성 규칙과 중복 제거

#### 3.2 버전 관리와 변경 이력 추적

* `created`와 `modified` 타임스탬프의 상관관계
* 폐기(`revoked`) 속성의 활용과 영구적 의미
* \[심화] 새로운 버전으로 업데이트할 것인가, 완전히 새로운 객체를 만들 것인가? (Material change 기준)

***

### Part 4: SDO (Domain Objects) 상세 사전

#### 4.1 공격의 주체와 동기 파악

* Threat Actor: 위협 행위자의 유형, 역할, 동기, 기술 수준
* Intrusion Set: 침입 세트의 특징과 캠페인과의 차이점
* Campaign: 특정 목표를 향한 공격 웨이브와 기간

#### 4.2 공격 기법과 도구

* Attack Pattern: CAPEC과 연계된 공격 패턴 설명
* Malware: 멀웨어 제품군(Family)과 인스턴스(Instance)의 차이
* Tool: 정상 소프트웨어를 악용하는 해킹 도구 명시

#### 4.3 탐지, 사고, 그리고 대응

* Indicator: STIX 패터닝 언어를 품은 탐지 지표
* Observed Data: 시스템에서 관찰된 순수한 원시 데이터의 기록
* Course of Action: 위협 완화 및 대응 방안
* Incident (Stub): 향후 확장을 위한 사고 객체

#### 4.4 피해자 및 인프라

* Identity: 개인, 조직, 그룹 또는 클래스의 신원 정보
* Location: 공격자나 피해자의 지리적 영역 표현
* Infrastructure: 공격에 사용되거나 방어에 사용되는 시스템/서비스
* Vulnerability: CVE 등 소프트웨어 결함 정보

#### 4.5 분석 및 보고

* Report: 여러 STIX 객체를 엮어 만든 종합 위협 보고서
* Grouping: 분석 과정을 묶어주는 논리적 컨텍스트
* Note: 분석가의 추가적인 컨텍스트나 텍스트 메모
* Opinion: 타 조직의 데이터에 대한 동의/비동의 평가
* Malware Analysis: 정적/동적 분석 결과와 메타데이터

***

### Part 5: SCO (Cyber-observable Objects) 상세 사전

#### 5.1 네트워크 자산 관찰

* IPv4/IPv6 Address, MAC Address, Domain Name, URL
* Network Traffic: 흐름, 바이트/패킷 카운트, 프로토콜 계층

#### 5.2 호스트 자산 관찰

* File: 해시, 크기, 이름 및 디렉토리 정보
* Directory, Process, Windows Registry Key, Mutex
* Software, User Account

#### 5.3 기타 디지털 자산

* Artifact: Base64로 인코딩된 페이로드 및 파일 내용
* Email Message & Email Address
* X.509 Certificate

#### 5.4 사전 정의된 객체 확장 (Predefined Extensions)

* NTFS, PDF, Raster Image, Windows PE Binary 파일 확장
* HTTP Request, ICMP, TCP, Network Socket 트래픽 확장
* Windows Process, Windows Service, UNIX Account 계정 확장

***

### Part 6: SRO (Relationship Objects) 매듭짓기

#### 6.1 Generic Relationship

* `source_ref`와 `target_ref`를 통한 그래프 엣지 생성
* 공식 스펙이 정의한 SDO/SCO 간의 관계 요약

#### 6.2 Sighting (목격)

* 지표나 멀웨어를 '언제, 어디서, 얼마나' 보았는가?
* `sighting_of_ref`, `where_sighted_refs`, `observed_data_refs`의 활용

***

### Part 7: Meta Objects와 Bundle

#### 7.1 Data Markings (데이터 보호와 권한)

* Marking Definition: TLP(Traffic Light Protocol) 및 Statement 지정
* Object-Level 마킹과 Granular(세밀한) 마킹의 차이점 및 적용법

#### 7.2 다국어 번역과 포장

* Language Content: 기존 객체를 수정하지 않고 다국어(번역) 텍스트 덧붙이기
* Bundle: 의미적 연결 없이 객체들을 한 번에 묶어 전송하는 임시 컨테이너

***

### Part 8: 확장(Extensions)과 커스터마이징

#### 8.1 STIX 2.1 Extension Definition

* 새로운 SDO, SCO, SRO 창조하기 (`new-sdo`, `new-sco`)
* 기존 객체에 속성 추가하기 (`property-extension`)
* 확장 스키마(Schema) 작성 및 UUID 참조 방식

#### 8.2 과거의 유산 (Deprecated)

* STIX 2.0 방식의 Custom Properties, Custom Objects가 폐기된 이유

***

### Part 9: STIX Patterning (탐지 규칙 언어)

#### 9.1 STIX 패턴의 문법 구조

* 패턴 표현식(Pattern Expressions)과 대괄호 `[ ]`의 의미
* 객체 경로 구문(Object Path Syntax) 작성법 (예: `file:hashes.'SHA-256'`)

#### 9.2 연산자와 한정자

* 비교 연산자(Comparison Operators): `=`, `!=`, `IN`, `MATCHES`(정규식), `ISSUBSET` 등
* 논리 연산자(Boolean Operators)와 관찰 연산자(Observation Operators: `FOLLOWEDBY`)
* 시간/반복 한정자(Qualifiers): `WITHIN`, `REPEATS`, `START/STOP`

***

### Part 10: 어휘와 열거형 (Vocabularies & Enumerations)

#### 10.1 Open Vocabularies (-ov)

* 산업군(industry-sector), 위협 행위자 역할(threat-actor-role), 공격 동기(attack-motivation) 등 확장 가능한 어휘 목록 활용법

#### 10.2 Enumerations (-enum)

* 해시 알고리즘, 암호화 알고리즘, 윈도우 레지스트리 데이터 타입 등 엄격하게 지켜야 하는 고정된 어휘 목록

***

### Part 11: 적합성(Conformance)과 실전 적용

#### 11.1 소프트웨어의 STIX 준수 요건

* Producer(생성자)와 Consumer(소비자)가 지켜야 할 필수 기능

#### 11.2 STIX Patterning 적합성 레벨

* Level 1 (기본), Level 2 (다중 관찰), Level 3 (전체 호환)의 차이 및 시스템(NIDS, SIEM 등)별 적용 사례
