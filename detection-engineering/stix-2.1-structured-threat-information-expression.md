---
description: >-
  사이버 공격에 대해 서로 대화하기 위해 사용하는 공용어 STIX는 파편화되고 수동적이었던 보안 정보 공유 체계를 '표준화, 자동화,
  맥락화'하여 전 세계적인 집단 사이버 방어를 가능하게 만든 혁신적인 뼈대
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

STIX가 등장하기 전, 보안 담당자들은 비정형 데이터의 늪, 맥락의 부재, 솔루션 간의 호환성 부족으로 어려움을 겪고 있었습니다.

* 비정형 데이터의 늪(수작업의 한계):\
  위협 정보가 PDF 보고서, 워드 문서, 뉴스 기사, 이메일 등 텍스트 형태로 배포되었습니다.\
  보안 담당자가 이를 일일이 눈으로 읽고 악성 IP나 해시값을 복사해서 보안 장비(방화벽, 백신 등)에 수동으로 입력해야 했습니다.
* 맥락(Context)의 부재:\
  침해치표(IOC)만 공유되는 경우가 많았고, "이 IP가 차단 목록에 있기는 한데, 이게 랜섬웨어 관련인지, 북한 해커 조직 관련인지, 어떤 취약점을 노리는건지" 알 수 가 없었습니다.
* 솔루션 간의 호환성 부족:\
  A 보안업체와 B보안업체가 위협 정보를 표현하는 방식이 제각각이었고, 서로 데이터가 연동되지 않아 정보의 파편화가 생겼습니다.

STIX는 보안 데이터를 "기계가 읽을 수 있는 표준 규격"으로 만들고, 관계성을 부여하여 위 문제점들을 해결합니다.

* 기계화 및 자동화 실현:\
  STIX는 규격화된 JSON형태로, PDF를 읽을 필요 없이 보안 장비(TIP, SIEM, EDR 등)가 즉각적으로 데이터를 파싱하고 이해할 수 있습니다.
* 풍부한 맥락(Context) 제공:\
  단순히 악성 IP(SCO)만 전달하지 않고, 이 IP가 어떤 악성코드(Malware)에 사용되었고, 배후의 그룹(Threat Actor)은 누구이며, 이를 막기 위한 대응 방안(Course of Action)은 무엇인지 관계 객체를 통해 하나의 스토리로 엮어서 전달합니다.
* 벤더 종속성 탈피 (호환성 확보):\
  전 세계 주요 보안 벤더들이 STIX 표준을 지원하기 시작하여, 서로 다른 회사의 장비를 사용하더라도 STIX 규격만 맞추면 언어 장벽 없이 위협 인텔리전스를 편하게 교환할 수 있게 됐습니다.



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



### 1.2 그래프 기반 데이터 모델 (Graph-Based Model)

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

### 2.2 모든 객체의 공통 속성 (Common Properties)

#### `type`, `spec_version`, `id`, `created`, `modified` 등 필수/선택 속성

* type(필수):\
  STIX 객체의 유형을 결정하는 문자열(eg. malware, ipv4-addr)
* spec\_version(SDO/SRO 필수, SCO 선택):\
  객체를 표현하는데 사용된 STIX 스펙의 버전(eg. "2.1")
* id: 객체 식별자
* created(SDO/SRO 필수, SCO 제외):\
  객체의 첫 번째 버전이 생성된 시간. 밀리초 정밀도를 가져야하며, 새로운 버전이 생성되더라도 수정되면 안됨.
* modified(SDO/SRO 필수, SCO 제외):\
  현재 특정 버전이 마지막으로 수정된 시간. created와 같거나 그 이후.

#### 객체 생성자(`created_by_ref`)와 신뢰도(`confidence`) 지정 방법

* created\_by\_ref(선택):\
  객체를 처음 생성한 엔티티의 ID를 가리키는 Embbedded Relationship. Identity객체의 ID가 들어가야합니다.\
  속성을 생략할 수 있으며, 익명으로 정보를 공유할 수 있습니다.
* confidence (선택):\
  작성자가 자신이 만든 데이터의 정확성에 대해 가지는 확신(신뢰도) 수준. 0\~100의 정수.\
  공식 신뢰도 척조(Admiralty Credibility, WEP, DNO Scale 등)와 1:1로 매핑해둔 규범적 표(Normative mapping)이 존재.

> Page 28\~35

***

## Part 3: 식별(ID)과 버전 관리(Versioning)

### 3.1 객체 식별자(ID) 체계의 비밀

STIX 식별자의 기본 형태는 "object-type--UUID"입니다.\
object-type은 식별하려는 객체의 type 속성과 정확히 일치하는 소문자 문자열이고,\
UUID는 RFC 4122를 준수하는 유효한 UUID여야합니다.

{% hint style="info" %}
파서나 분석가는 식별자 문자열만 보고도 DB를 보지 않고 이건 XX객체를 가리키는 ID라는걸 알 수 있습니다.
{% endhint %}

#### SDO, SRO, Meta 객체를 위한 UUIDv4 생성 규칙

STIX Domain Objects, STIX Relationship Objects, STIX Meta Objects, STIX Bundle Object는 식별자의 UUID 부분에 무작위 생성 방식인 UUIDv4를 사용해야합니다.

{% hint style="info" %}
SDO와 SRO는 분석가의 주관적인 해석과 맥락이 포함된 인텔리전스입니다. 동일한 내용 보고서를 A, B기관에서 작성했더라도, 서로 다른 독립적인 분석 결과로 간주해야합니다.
{% endhint %}

#### SCO를 위한 결정론적(Deterministic) UUIDv5 생성 규칙과 중복 제거

순수한 디지털 증거물을 나타내는 STIX Cyber-observable Objects(SCO)는 식별자의 UUID 부분에 결정론적 방식인 UUIDv5를 사용해야합니다.

UUIDv5 생성시 3가지 규칙이 있습니다.

1. 네임스페이스 고정:\
   SCO의 UUIDv5 생성을 위한 네임스페이스는 "00abedb4-aa42-466c-9c01-fed23315a9b7"을 사용합니다.
2. 기여 속성 (ID Contributing Properties):\
   SCO 스펙마다 ID 생성에 기여하는 속성 목록이 정해져 있습니다. (File 객체는 hashes/name 등)
3. 정규화 (Canonicalization):\
   띄어쓰기나 속성 순서 떄문에 해시값이 달라지는 것을 방지하기 위해, 기여 속성들은 \[JCS] (JSON Canonicalization Scheme)에 따라 엄격하게 문자열로 변환(Stringify)된 후 UUIDv5 해시에 입력되어야 합니다.

만약 모든 속성이 Optional이라 비어있으면 UUIDv4를 돌립니다.

{% hint style="info" %}
A, B기관이 각각 수집한 198.51.100.3 IP는 동일한 객체여야합니다. 이 객체들의 중복을 제거하고 의미적 동등성을 보장하기 위해 입력값이 같으면 항상 동일한 출력값을 반환하는 UUIDv5를 사용합니다.
{% endhint %}

{% hint style="info" %}
UUIDv4 & UUIDv5

* UUIDv4:\
  완전한 난수와 의사 난수를 사용하여 생성. 매번 완전히 새로운 고유 ID가 생성됩니다. (2\*\*122 조합수)\
  DB 기본키, 세션ID, 트랜잭션ID 등에서 사용
* UUIDv5:\
  특정 네임스페이스(Namespace)와 이름(Name)을 SHA-1 알고리즘으로 해싱하여 생성됩니다.\
  입력값이 같다면 ID는 항상 같기에 예측 가능합니다.
{% endhint %}

{% hint style="info" %}
\[JCS] JSON Canonicalization Scheme [https://datatracker.ietf.org/doc/rfc8785/](https://datatracker.ietf.org/doc/rfc8785/)\
모양은 다르지만 의미가 같은 JSON 데이터를 입력 받아서 완벽하게 동일한 단 하나의 표준 JSON 문자열을 출력합니다.
{% endhint %}

> Page 25

### 3.2 버전 관리와 변경 이력 추적

STIX 객체(SDO, SRO)는 revoked, created, modified를 통해 버전 관리를 합니다.

#### `created`와 `modified` 타임스탬프의 상관관계

* 초기 생성:\
  객체가 처음 생성되었을 때(첫 번째 버전), created와 modified 속성은 동일한 타임스템프를 가짐.
* 버전 업데이트:\
  객체 작성자가 오타 수정/새로운 정보를 추가하여 버전을 업데이트하면 id와 created 시간은 그대로 유지한 채 modified 시간만 현재 수정 시간으로 갱신.
* 무결성 규칙:\
  created <= modified 조건이 성립해야하고, 동일한 id의 객체들을 여러개 받았을 때는 modified 값이 가장 최신인 것을 사용해야합니다.

{% hint style="info" %}
동일한 ID를 가진 객체들이 많다는 것은 버전 관리를 실패한 것이 아닌가?

1. 도메인의 특수성:\
   모든 것들이 이력으로 남아있어야합니다. 변경 이력 그 자체로도 중요한 단서가 되기 때문에 매번 새로운 객체(동일한 ID)를 만들게 됩니다.
2. 중앙 서버가 없는 글로벌 분산 환경:\
   CTI(사이버 위협 인텔리전스) 생태계에서는 데이터를 연관 기관들로 "뿌리기"때문에, 객체가 수정되었다고 적용하기가 어렵습니다.
{% endhint %}

#### 폐기(`revoked`) 속성의 활용과 영구적 의미

boolean 값을 가진 속성이고, 이 값이 true가 되면, 해당 객체(및 모든 과거 버전 포함)가 "더 이상 유효하지 않음"으로 판정됩니다.

한번 revoked가 된 객체는 영구적으로 폐기가 되어서, 같은 id의 새로운 버전을 절대 생성할 수 없습니다.

{% hint style="info" %}
SCO같은 경우 UUIDv5를 쓰고 있는데, 한번 폐기한 객체(198.51.100.3, IP)는 같은 ID를 사용하지 못하니, 해당 UUID는 평생 사용 불가한가?

SCO같은 경우는 버전 관리를 하지 않습니다. 그 이유는 '순수한 증거물(팩트)' 이기에 새롭게 변화되는 경우가 없습니다.

만약 악성 IP였다가, 정상적으로 돌아오는 상황이 된다면, SCO가 아닌 SDO를 폐기하는 방법을 채택해야합니다.

예를 들어,

1. 과거 상황: Indicator SDO -> indicates SRO -> IPv4-Addr
2. IP 정상화: "저 IP는 악성이야"라는 SDO를 폐기.
3. 다시 악성화: 새로운 Indicator SDO와 indicates SRO를 다시 IPv4-Addr에 연결합니다.
{% endhint %}

> Page 38, 39, 40

***

## Part 4: SDO (Domain Objects) 상세 사전

### 4.1 공격의 주체와 동기 파악

#### Threat Actor (위협 행위자): 해킹의 진짜 배후

악의적인 의도를 가지고 활동하는 것으로 추정되는 실제 개인, 그룹 또는 조직을 의미합니다.\
Threat Actor SDO에는 공격자의 동기(primary\_motivation, secondary\_motivation), 능력치(sophistication), 자원 수준(resource\_level), 조직 내 역할(roles) 등 프로파일링 할 수 있는 속성들을 제공합니다.

분석가는 해커의 개인적인 동기(personal\_motivation)까지도 기록할 수 있고, 통일된 용어를 Open Vocabulary에서 연동하여 사용합니다.

> Page 109. Properties\
> Page 261. Threat Actor Role Vocabulary

#### Intrusion Set (침입 세트): 범행 조직의 시그니처(고유 수법)

단일 조직이 조율하는 것으로 여겨지며, 공통적인 특성(인프라, 도구, TTP 등)을 공유하는 적대적 행동과 리소스의 그룹화된 세트입니다.

> Page 74. Intrusion Set\
> Page 75. Properties

Threat Actor를 특정할 수는 없지만, 공격들이 항상 A라는 악성코드와 B라는 IP대역을 쓰고, C라는 취약점만 노리는 걸 보니 "동일한 놈들의 소행이다"라고 묶어주는 캔버스입니다.

Threat Actor와의 차이점:\
Intrusion Set은 Threat Actor 그 자체가 아니라, 해커가 누구인지 밝혀내지 못해도, 새로운 해킹 활동을 특정 intrusion set의 활동으로 귀속시킬 수 있습니다.

Campaign과의 차이점:\
캠페인은 특정 목표를 달성하기 위해 '일정 기간' 동안만 진행되는 공격 웨이브입니다. 반면, Intrusion Set는 전체 공격 패키지를 의미하며, 목적을 달성하기 위해 여러 Campaign에 걸쳐 반복적으로 사용될 수 있습니다.

#### Campaign: 특정 목표를 향한 공격 웨이브와 기간

특정 타겟(피해자) 집단을 상대로, 특정 기간 동안 악의적인 활동이나 공격('웨이브')의 묶음을 지칭합니다.

캠페인은 잘 정의된 목표를 가지고 있고, 시작 시점(first\_seen)과 종료 시점(last\_seen)을 속성으로 명확히 기록하여 시간의 흐름에 따른 공격 변화를 추적할 수 있습니다.

> Page 51. Campaign
>
> Page 52. Properties



#### Possible Cases

* Threat Actor들은 기존에 쓰던 Intrusion Set을 재사용하여 Campaign을 벌일 수 이는 경우
* Campaign을 성공시키기 위해 새로운 인프라를 구축하는경우
* Threat Actor가 봄에는 Intrusion Set A를 사용하여 Campaign을 벌이고, 여름에는 Intrusion Set B를 사용하여 Campaign을 벌이는 경우
* Threat Actor가 Intrusion Set을 개발하여 다른 목적의 Campaign A, Campaign B를 진행한 경우

#### Practices

<details>

<summary>[Node 1] Campaign (이름: "Operation Dark Night")<br>[Node 2] Vulnerability (이름: "CVE-2021-12345")<br>[Node 3] Identity (이름: "대한민국 금융권", identity_class: "class")<br>[Edge A] Node 1 --(targets)--> Node 2<br>[Edge B] Node 1 --(targets)--> Node 3</summary>

"Operation Dark Night(SDO)"라는 캠페인은 "대한민국 금융권(SDO)"을 타겟(SRO)으로 삼으며 "CVE-2021-12345" 취약점(SDO)을 타겟팅하고 있습니다.

</details>

<details>

<summary>[Node 1] Threat Actor (이름: "OceanLotus")<br>[Node 2] Intrusion Set (이름: "Silent Chollima")<br>[Node 3] Campaign (이름: "Crypto Heist 2024")<br>[Node 4] Malware (이름: "AppleJeus")<br>[Edge A] Node 3 --(uses)--> Node 4<br>[Edge B] Node 3 --(attributed-to)--> Node 2<br>[Edge C] Node 2 --(attributed-to)--> Node 1</summary>

"Crypto Heist 2024"(SDO) 캠페인은 "AppleJeus"(SDO)라는 악성코드/툴을 사용(SRO)합니다.\
"Crypto Heist 2024"(SDO)는 "Silent Chollima"(SDO) 침입 세트에 귀속(SRO)되며, "Silent Chollima"(SDO) 침입 세트의 배후는 "OceanLotus"(SDO)입니다.

</details>

<details>

<summary>[Node 1] Infrastructure (이름: "해킹된 워드프레스 서버", type: "hosting-malware")<br>[Node 2] Malware (이름: "Emotet")<br>[Node 3] IPv4-Addr (값: "203.0.113.10")<br>[Edge A] Node 1 --(delivers)--> Node 2<br>[Edge B] Node 1 --(consists-of)--> Node 3</summary>

"해킹된 워드프레스 서버"(SDO)는 악성코드 호스팅 서버이고, "Emotet"(SDO) 악성코드를 유포(SRO)합니다.\
"해킹된 워드프레스 서버"(SDO)는 "203.0.113.10"(SCO) IP로 구성(SRO)됩니다.

</details>



### 4.2 공격 기법과 도구

해커들은 특정한 패턴(전술)을 가지고 악의적인 소프트웨어(무기)를 개발하거나, 때론 정상적인 '도구'를 악용하여 목표를 달성합니다. 이는 STIX 2.1에서 세 가지 객체로 나누어 프로파일링합니다.

#### Attack Pattern: CAPEC과 연계된 공격 패턴 설명

공격자가 목표를 손상시키기 위해 시도하는 '방법(전술 및 기법')을 설명하는 TTP의 한 유형입니다. \
스피어 피싱, SQL 인젝션, 버퍼 오버플로우 등 해커가 뚫고 들어가거나 구체적인 행위 방식을 나타냅니다.

동일한 용어를 위해 표준화된 공격 사전(Taxonomy)와 연계됩니다.\
external\_references 속성을 사용하여 MITRE의 CAPEC (Common Attack Pattern Enumeration and Classification) ID나 ATT\&CK 메트릭스 ID를 참조합니다.\
(CAPEC ID를 지정할 떄는 source\_name 속성을 capec으로, external\_id를 CAPEC-\[id]로 지정합니다.)

{% hint style="info" %}
* 스피어 피싱: 특정 개인이나 조직을 정교하게 노리고 악성 이메일을 보내는 기법.
* SQL Injection: 웹 애플리케이션의 입력창에 SQL 쿼리문을 삽입하여 데이터를 유출하는 기법
* Buffer Overflow: 메모리 버퍼의 처리 용량을 초과하는 데이터를 의도적으로 입력하여 프로그램 흐름을 조작하거나 악성 코드를 실행하게 만드는 기법
{% endhint %}

> Page 47

#### Malware: 멀웨어 제품군(Family)과 인스턴스(Instance)의 차이

Malware 객체는 기밀성, 무결성 또는 가용성을 손상시키기 위해 시스템에 은밀하게 삽입되는 악성 코드.

Family vs Instance:  Malware 객체는 is\_family(bool) 속성을 가지고,

* true: WannaCry 라는 랜섬웨어 제품군 전체를 아우르는 개념.
* false: 특정 WannaCry.exe 파일 한개를 선언. (무조건 하나의 sample\_refs와 매핑)

> Page 84

#### Tool: 정상 소프트웨어를 악용하는 해킹 도구 명시

공격을 수행하기 위해 해커가 사용할 수 있는 "합법적인 소프트웨어".

이미 설계되어 있거나 설치되어 있는 합법적인 툴을 악용하는 기법입니다.\
(Tool은 악성 코드의 특성을 묘사하는데 절대 사용되서는 안되며, 방어자가 공격에 대응하기 위한 조치(Course of Action) 도구로 사용되어서도 안됩니다)

> Page 114

<details>

<summary>"해커가 스피어 피싱 메일을 보냈는데, 그 안에 버퍼 오버플로우 취약점을 터뜨리는 PDF 악성코드가 들어있었다."</summary>

\[Node 1] Attack pattern ("Spear Phishing")\
\[Node 2] Malware ("Overflow PDF")\
\[Node 3] Vulnerability ("Buffer Overflow")\
\[Edge A] Node 1 --(delivers) --> Node2\
\[Edge B] Node 2 --(exploits) --> Node3

</details>



### 4.3 탐지, 사고, 그리고 대응

#### Indicator: STIX 패터닝 언어를 품은 탐지 지표

지표 객체는 특정 활동을 식별하기 위해 사용되는 탐지 룰. 마치 검색 쿼리이고, 쿼리를 통해 나온 검색 결과가 SCO입니다.\
탐지 룰은 시그마 룰(sigma)을 포함한 yara, snort, suricata, pcre, stix가 있습니다.

{% hint style="info" %}
EDR에서 파생된 여러개의 연관된 로그(부모-자식 프로세스)들은 각각 개별적인 Process SCO로 등록됩니다. 이 SCO들은 서로 parent\_ref 속성으로 꼬리를 물고 하나의 '프로세스 트리'를 형성하게 됩니다.

Indicator SDO는 탐지 룰이 아닌 탐지 룰을 담기 위한 객체이고, 탐지 결과는 항상 SCO(팩트)입니다.\
다만, Observed Data 객체는 SDO이고 object\_refs에 SCO(팩트)들이 참조되어등록이 됩니다.\
참조되어 등록된 SCO들이 트리 형태를 만들 수도 있습니다.
{% endhint %}

```
Sigma(Indicator, SDO) 탐지룰로 탐지한(Sighting, SDO) 기록(Observed Data, SDO)에는 
관련된 로그1(Log, SCO), 로그2(Log, SCO), 로그3(Log, SCO)이 있고,
각각의 로그의 메타데이터를 봤을때(예. GUID) 부모가 존재하고, 하나의 프로세스 트리를 이루고 있다.
 
Indicator (SDO) 
   └── Sighting (SRO) 
        └── Observed Data (SDO) 
             │
             ├── (object_refs 참조) ──> [Log 1: SCO (winword.exe)]
             │                                   ▲
             │                                   │ (parent_ref로 부모를 가리킴)
             │                                   │
             ├── (object_refs 참조) ──> [Log 2: SCO (cmd.exe)]
             │                                   ▲
             │                                   │ (parent_ref로 부모를 가리킴)
             │                                   │
             └── (object_refs 참조) ──> [Log 3: SCO (powershell.exe)]
```

{% hint style="info" %}
sigma 예시:

* `pattern_type`: `"sigma"`
* `pattern`: `"title: Suspicious PowerShell Execution\nlogsource:\n category: process_creation... (실제 시그마 룰 코드)"`

yara 예시:

* `pattern_type`: `"yara"`
* `pattern`: `"rule malicious_pdf { strings: $magic = \"%PDF-\" condition: $magic }"`
{% endhint %}

* STIX Patterning:\
  STIX 자체 패터닝 언어를 사용하여 "파일 해시가 A이고 통신 목적지가 B인 경우를 찾아라"와 같은 조건을 기계가 읽을 수 있게 정의.
* 유효기간 (Valid From/Until):\
  탐지 룰이 언제부터 언제까지 유효한지 명시합니다.

> Page 64, 66

#### Observed Data: 시스템에서 관찰된 순수한 원시 데이터의 기록

관찰된 데이터 객체는 특정 시점에 네트워크나 호스트에서 실제로 관찰된 사실들의 기록입니다.

SCO를 objects 속성안에 품고, 언제 처음 봤는지 (first\_observed), 언제 마지막으로 봤는지 (last\_observed) 타임스탬프를 찍습니다. \
number\_observed를 통해 "해당 시간동안 몇번 탐지됐는지"를 통해 공격 강도를 기록할 수도 있습니다.

> Page 100

#### Course of Action: 위협 완화 및 대응 방안

대응 방안 객체는 위협을 예방(Prevent), 탐지(Detect), 대응(Respond) 또는 완화(Mitigate)하기 위해 취해야 할 조치를 설명합니다.\
description속성에 human-readable 텍스트 기반 권고 사항을 담고 있고("방화벽에서 해당 IP를 차단하세요"),\
주로 Attack Pattern, Malware, Vulnerability와 SRO(mitigate)로 연결됩니다.

{% hint style="info" %}
CoA가 있으면 자동으로 방화벽을 차단하는 기능을 만들면 되지 않나?

이미 존재합니다. CACAO(Collaborative Automated Course of Action Operations)같은 별도의 조치 자동화 표준이 존재합니다. STIX는 "위협 정보를 표현"하는데에 주 목적이 있어서 "정보 공유"에 집중합니다.
{% endhint %}

> Page 56, 57

#### Incident (Stub): 향후 확장을 위한 사고 객체

사고 객체는 보안상 위협이 실제로 발생하여 피해가 확인된 사건을 나타냅니다.\
어떤 사고가 발생했는지 이름을 붙이고, Campaign, Malware, Observed Data 등을 하나로 묶어주는 종합 보고서의 제목 역할을 주로 수행합니다.

> Page 62

### 4.4 피해자 및 인프라

#### Identity: 개인, 조직, 그룹 또는 클래스의 신원 정보

Identity 객체는 실세계에서 '주체'를 나타내는 객체입니다.

피해자를 묘사하거나 작성자의 신원을 나타낼때(created\_by\_ref) 참조되기도 합니다.

> Page 60

#### Location: 공격자나 피해자의 지리적 영역 표현

Location 객체는 위협 이벤트나 관련된 주체의 '물리적인 지리적 위치'를 설명합니다.

위경도를 찍을 수도 있고, Region-Country-City 같은 행정 구역 단위로 추상화를 할 수 도 있습니다.

> page 82

#### Infrastructure: 공격에 사용되거나 방어에 사용되는 시스템/서비스

Infrastructure 객체는 사이버 작전을 수행하기 위해 물리적 또는 노리적으로 구축된 시스템, 하드웨어 장치, 소프트웨어 서비스, 네트워크 등을 나타냅니다.

인프라는 해커의 공격 인프라 뿐만 아니라, 방어자의 방어 인프라(사내 방화벽, 메일 게이트웨이,...)에서도 사용됩니다.

인프라는 SDO의 객체로 만들어지지만 consists-of 필드에 SCO 객체들(IP주소, 도메인 등)이 연결되어야 합니다.

> Page 78

#### Vulnerability: CVE 등 소프트웨어 결함 정보

Vulnerabliity 객체는 공격자가 시스템이나 네트워크에 대한 무단 액세스 권한을 얻기 위해 악용(Exploit)할 수 있는 소프트웨어, 하드웨어 또는 아키텍처 상의 실수나 논리적 오류를 의미.

새로운 취약점 점호를 부여하지 않고, CVE(Common Vulnerabilities and Exposures)와 연동하도록 권장합니다.\
(취약점이 CVE ID를 가지고 있다면 external\_reference 속성에 source\_name: cve로 하고, external\_id를 CVE-2021-12345형식으로 기입합니다.)

Malware/Attack Pattern(SDO) --> Expliots(SRO) --> Vulnerability(SDO) --> Mitigates(SRO) --> Course of Action(SDO)

```
"해커 조직이 **[Location: 동유럽]**에 위치한 봇넷 **[Infrastructure: C2 서버]**를 구축했습니다. 
이들은 **[Identity: 금융권 클래스]**를 타겟으로 삼았으며, 
이들의 사내망을 뚫기 위해 방화벽 장비의 알려진 결함인 **[Vulnerability: CVE-2024-0001]**을 집중적으로 
공격(Exploits)했습니다."

[Node 1: SDO] Threat Actor 
  - name: "알 수 없는 해커 조직"

[Node 2: SDO] Location 
  - region: "eastern-europe" (동유럽)

[Node 3: SDO] Infrastructure 
  - name: "C2 서버"

[Node 4: SDO] Identity 
  - name: "금융권"

[Node 5: SDO] Vulnerability 
  - name: "방화벽 장비 결함"
  - external_references: [{"source_name": "cve", "external_id": "CVE-2024-0001"}]
  
[Edge A] Node 1 --(owns)--> Node 3
[Edge B] Node 3 --(located-at)--> Node 2
[Edge C] Node 1 --(targets)--> Node 4
[Edge D] Node 1 --(targets)--> Node 5
```

> Page 120

### 4.5 분석 및 보고

#### Report: 여러 STIX 객체를 엮어 만든 종합 위협 보고서

Report 객체는 '월간 위협 동향 보고서', '침해사고 분석 리포트'를 JSON 포맷으로 만든 것.\
object\_refs에 SDO, SCO, SRO들을 가르킵니다.

#### Grouping: 분석 과정을 묶어주는 논리적 컨텍스트

Grouping 객체는 전문가가 조사를 진행하는 동안 관련 자료들을 임시로 모아두는 '폴더' 역할.\
"이번주 발생한 의심스러운 악성 코드 샘플들 묶음" 같이 공식적이지 않습니다.

#### Note: 분석가의 추가적인 컨텍스트나 텍스트 메모

Note 객체는 다른 STIX 객체 위에 붙는 '포스트 잇'입니다.

STIX 객체는 작성자 제외 수정이 어렵기 때문에, 활용되는 객체입니다.

#### Opinion: 타 조직의 데이터에 대한 동의/비동의 평가

Opinion 객체는 STIX 객체에 붙일 수 있는 별점(리뷰) 시스템입니다.

#### Malware Analysis: 정적/동적 분석 결과와 메타데이터

Malware Analysis 객체는 Malware를 샌드박스 시스템이나 리버스 엔지니어가 뜯어본 '부검 결과서'입니다.

{% hint style="info" %}
Malware vs Malware Analysis

Malware: "이것은 Emotet이라는 악성코드다"\
Malware Analysis: "그 Emotet 파일을 Cuckoo 샌드박스에서 윈도우 10 환경으로 돌려봤더니 이런 행동을 하더라"
{% endhint %}

> Page 101, 58, 97, 105, 90

***

## Part 5: SCO (Cyber-observable Objects) 상세 사전

### 5.1 네트워크 자산 관찰

SCO 객체들은 철저히 팩트만을 기록합니다. 네트워크 자산 관찰 파트라고 할 수 있으며, 보안 장비(방화벽, IPS)가 뱉어내는 로그들입니다.

판단/의미부여를 하기전에 물리적 현상 그 자체를 묘사합니다. \
"A컴퓨터에서 B컴퓨터로 몇 바이트의 데이터가 어떤 프로토콜을 타고 흘러갔다."

#### 기본 네트워크 식별자 객체

* IPv4-Addr / IPv6-Addr: 단일 IP 주소나 서브넷 (CIDR)을 나타냅니다.
* MAC-Addr: 하드웨어 고유한 물리적 주소를 나타냅니다.
* Domain-Name: www.example.com 같은 도메인 이름을 나타냅니다. 내장된 resolves\_to\_refs 속성으로 IP객체(SCO)들을 가리킵니다.
* URL: 도메인을 넘어선 전체 웹 경로(https://example.com/login.php)를 나타냅니다. 피싱 메일 분석이나 악성코드 다운로드 경로를 기록할 때 필수적입니다.

#### Network Traffic: 통신의 모든 것을 담는 블랙박스

Network Traffic 객체는 IP/MAC 주소는 단순한 주소에서 발생한 기록들을 담아내는 거대한 SCO입니다.\
TCP/IP 계층 구조의 거의 모든 정보를 표현할 수 있습니다.

* Source & Destination: src\_ref, dst\_ref
* Port: src\_port, dst\_port
* Byte/Packet Count: src\_byte\_count, dst\_byte\_count / src\_packets, dst\_packets
* Protocols: ipv4 / tcp / http

{% hint style="info" %}
"우리 내부 PC(`192.168.1.50`)가 외부 악성 도메인(`evil.com`)으로 HTTP 통신을 500바이트 보냈다"



\[Node 1: SCO] IPv4-Addr (value: "192.168.1.50") -- 내부 PC\
\[Node 2: SCO] Domain-Name (value: "evil.com") -- 외부 도메인

\[Node 3: SCO] Network-Traffic

* src\_ref: Node 1의 ID (출발지)
* dst\_ref: Node 2의 ID (목적지)
* dst\_port: 80
* protocols: \["ipv4", "tcp", "http"]
* src\_byte\_count: 500
{% endhint %}

### 5.2 호스트 자산 관찰

#### File & Directory

* File 객체: 파일의 정적 속성을 정의합니다. extensions 속성을 통해 OS 전용 정보 등 메타데이터를 붙일 수 있습니다.
* Directory 객체: 파일이 위치하는 주소. 주요 속성으로 path, contains\_refs 있습니다.

> Page 143

#### Process

Process 객체는 다른 SCO들을 하나로 묶는 허브 역할을 수행합니다.

* 프로세스의 원본 파일 -> image\_ref 속성
* 누가 실행했는가 -> creator\_user\_ref 속성
* 부모 프로세스 -> parent\_ref 속성

> Page 174

#### Windows Registry Key, Mutex

* Windows Registry 객체:\
  해커가 PC가 재부팅되어도 악성코드가 다시 실행되도록 설정할때 건드리는 윈도우 설정 DB.
* Mutex 객체:\
  운영체제에서 프로세스 동시 접근을 제어하는 자물쇠. \
  유저가 악성코드를 여러번 실행하게 되면 발각될 확률이 높기때문에 싱글톤으로 관리되게끔 뮤텍스를 생성.

#### Software, User Account

* Software 객체:\
  시스템에 설치된 프로그램. CPE(Common Platform Enumeration)을 cpe 속성에 기록하여 취약점 연동.
* User Account 객체: \
  해커가 탈취한 계정 정보

### 5.3 기타 디지털 자산

#### Artifact (아티팩트): Base64로 인코딩된 페이로드 및 파일 내용

File의 실제 내용물을 담는 객체.\
JSON형식인 STIX에 Binary 데이터를 직접 넣을 수 없기에, base64 인코딩을 거친 텍스트를 payload\_bin 속성에 넣습니다. 크기가 커지면 대신, url 속성에 virustotal 다운로드 링크 주소를 적습니다.

#### Email Message & Email Address

* Email-Addr: 단일 이메일 주소
* Email-Message: 이메일 메시지. subject, body, from\_ref, to\_refs 속성

> Page 135

#### X.509 Certificate

백신을 우회하기 위해 프로그램이 신뢰할 수 있는 개발자에 의해 만들어졌다는 디지털 인증서.

> Page 188

### 5.4 사전 정의된 객체 확장 (SCO 내부: Predefined Extensions 속성)

#### 파일 객체를 위한 확장: NTFS, PDF, Raster Image, Windows PE Binary&#x20;

* NTFS 확장: 파일이 윈도우 NTFS 파일 시스템에 저장될 떄의 특성을 캡처
* PDF 확장: PDF 파일의 버전, 최적화 여부, 메타데이터를 기록.
* Raster Image 확장: 이미지 파일의 메터데이터를 기록
* Windows PE Binary 확장: 윈도우 실행파일 전용 확장.

#### 네트워크 통신 객체를 위한 확장 HTTP Request, ICMP, TCP, Network Socket&#x20;

* HTTP Request 확장: L7의 HTTP 요청을 분석합니다.
* ICMP 확장: 핑 통신 등에 쓰이는 ICMP 프로토콜의 icmp\_type\_hex, code\_hex바이트를 16진수로 기록
* TCP 확장: TCP 통신 세션 전체에서 관찰된 src, dst\_flags\_hex를 캡처하여 비정상적인 스캔 공격을 증명
* Network Socket 확장: 통신이 맺어진 운영체제의 '소켓'상태와 주소 패밀리 정보 묘사.

#### 프로세스와 계정 객체를 위한 확장 Windows Process, Windows Service, UNIX Account

* Windows Process 확장: 윈도우 프로세스만의 특성인 메모리 보호 기법적용 여부 등을 기록
* Windows Service 확장: 프로세스가 윈도우 '서비스'로 동작할때의 정보
* Unix Account 확장: 유닉스/리눅스 계정 전용 확장.

{% hint style="info" %}
왜 객체가 아닌 Extended 속성일까?

파일로그 하나를 표현하다고 하였을때, NTFS(파일시스템정보), File, PE Binary를 모두 객체로 만들고 SRO로 모두 연결을 해야하는데, 이 작업은 불필요하게 그래프를 더 크게 만들게 됩니다.
{% endhint %}

> Page 147, 149, 150, 151, 169, 171, 173, 172, 177, 178, 185

***

## Part 6: SRO (Relationship Objects) 매듭짓기

### 6.1 Generic Relationship

STIX 생태계에서 SDO(도메인 객체)와 SCO(관찰 대상 객체)를 연결하기 위해 SRO(관계 객체)를 활용합니다.\
SRO는 SDO 또는 SCO만 가리킬 수 있습니다.\
관계 객체는 기본적으로 "A가 B를 어떻게 한다"라는 명제를 만들게 됩니다.

#### `source_ref`와 `target_ref`를 통한 그래프 엣지 생성

* relationship\_type(필수): 관계의 종류입니다. (eg. uses, indicates, targets,...)
* source\_ref(필수): 화살표의 출발점
* target\_ref(필수): 화살표 도착점

> Page 122

#### 공식 스펙이 정의한 SDO/SCO 간의 관계 요약

STIX 2.1은 객체들 사이에서 쓸 수 있는 Vocabulary를 정의하였고, 시스템간 데이터 교환을 위해 권장합니다.

대표적인 관계:

* uses: 해커(Threat Actor)나 캠페인(Campaign)이 악성코드(Malware)나 인프라(Infrastructure)를 사용할 때.
* targets: 악성코드나 해커가 특정 취약점(Vulnerability)이나 기업/국가(Identity, Location)을 노릴 때.
* indicates: 탐지 룰(Indicator)이 악성코드나 해커의 활동을 잡아낼 때.
* communicates-with: 악성코드가 C2 서버(IP, 도메인 등 SCO)와 네트워크 통신을 할 때
* mitigates: 방어 조치(Course of Action)가 악성코드 취약점을 막아낼 때
* related-to: '연관은 있는데 뭐라고 정의해야할지/뭔지 모르겠다' 라는 만능 통일 동사

{% hint style="info" %}
Indicator가 Campaign을 직접 indicate한다 라고 허용합니다.

위 얘기는 탐지룰이 Campaign을 탐지하는게 아니라, TTP를 탐지하게 되는 것이어서 냉정하게 틀린 얘기입니다.\
하지만, STIX는 복잡한 중단 단계를 생략하고 직관적으로 "이 룰이 뜨면 그 해킹 캠페인이야" 라고 핵심만 전달할 수 있도록, STIX는 논리적 건너뛰기(Shortcuts)를 공식적으로 허용합니다.
{% endhint %}

> Relation Volcabulary: Page 281
>
> Shortcut: Page 121



### 6.2 Sighting (목격)

#### 지표나 멀웨어를 '언제, 어디서, 얼마나' 보았는가?

Sighting 객체는 '특정 위협이 실제 목격되었다' 라는 동적인 사전(Event)를 기록하는 특수한 형태의 SRO입니다.

#### `sighting_of_ref`, `where_sighted_refs`, `observed_data_refs`의 활용

목격 사건의 '육하원칙'을 정의하기 위해 세 가지 강력한 참조(\_ref) 속성을 제공합니다.

* sighting\_of\_ref(필수): 우리가 목격한 대상(SDO)이 무엇인지. Indicator, Malware, Tool, Threat Actor,...
* where\_sighted\_refs: 목격한 피해자, 기관(Identity) 또는 물리적 지역(Location)을 가리키는 리스트 속성.
* observed\_data\_refs: 목격 주장을 뒷받침하는 원시 데이터(Raw Data) 리스트 속성. Obeserved Data(SDO)만 가리킬 수 있습니다.

```json
{
  "type": "sighting",
  "spec_version": "2.1",
  "id": "sighting--ee20065d-2555-424f-ad9e-0f8428623c75",
  "created": "2016-04-06T20:08:31.000Z",
  "modified": "2016-04-06T20:08:31.000Z",
  "first_seen": "2015-12-21T19:00:00Z",
  "last_seen": "2015-12-21T19:00:00Z",
  
  "count": 50, //(논리적인 목격 횟수. 반면, Observed Data:number_observed는 물리적인 로그의 갯수)
  
  "sighting_of_ref": "indicator--8e2e2d2b-17d4-4cbf-938f-98ee46b3cd3f",
  "where_sighted_refs": ["identity--b67d30ff-02ac-498a-92f9-32f845f448ff"],
  "observed_data_refs": ["observed-data--b67d30ff-02ac-498a-92f9-32f845f448cf"]
}
```

> Page 126

***

## 중간 점검: STIX 2.1 분석 연습

### \[Case1] 은밀한 인프라와 악성코드 관계 (Page 284)

```json
[
  {
    "type": "infrastructure",
    "spec_version": "2.1",
    "id": "infrastructure--d09c50cf-5bab-465e-9e2d-543912148b73",
    "name": "Example Target List Host",
    "infrastructure_types": ["hosting-target-lists"]
  },
  {
    "type": "malware",
    "spec_version": "2.1",
    "id": "malware--3a41e552-999b-4ad3-bedc-332b6d9ff80c",
    "is_family": true,
    "malware_types": ["bot"],
    "name": "IMDDOS"
  },
  {
    "type": "domain-name",
    "spec_version": "2.1",
    "id": "domain-name--3c10e93f-798e-5a26-a0c1-08156efab7f5",
    "value": "evil-example.com"
  },
  {
    "type": "relationship",
    "spec_version": "2.1",
    "id": "relationship--37ac0c8d-f86d-4e56-aee9-914343959a4c",
    "relationship_type": "uses",
    "source_ref": "malware--3a41e552-999b-4ad3-bedc-332b6d9ff80c",
    "target_ref": "infrastructure--d09c50cf-5bab-465e-9e2d-543912148b73"
  },
  {
    "type": "relationship",
    "spec_version": "2.1",
    "id": "relationship--81f12913-1372-4c96-85ec-E9034ac98aba",
    "relationship_type": "consists-of",
    "source_ref": "infrastructure--d09c50cf-5bab-465e-9e2d-543912148b73",
    "target_ref": "domain-name--3c10e93f-798e-5a26-a0c1-08156efab7f5"
  }
]
```

IMDDOS 이름을 가진 악성 코드 패밀리 봇입니다. \
공격 타겟 리스트를 받아 오기 위해 hosting-target-lists 인프라를 사용(uses)하고 있습니다. \
인프라는 evil-example이라는 도메인 이름으로 구성되어 있습니다.



### \[Case 2] SOC(보안 관제 센터)의 다급한 알림 Page(127)

```json
[
  {
    "type": "sighting",
    "spec_version": "2.1",
    "id": "sighting--ee20065d-2555-424f-ad9e-0f8428623c75",
    "first_seen": "2023-10-21T19:00:00Z",
    "last_seen": "2023-10-21T19:00:00Z",
    "count": 50,
    "sighting_of_ref": "indicator--8e2e2d2b-17d4-4cbf-938f-98ee46b3cd3f",
    "where_sighted_refs": ["identity--b67d30ff-02ac-498a-92f9-32f845f448ff"],
    "observed_data_refs": ["observed-data--b67d30ff-02ac-498a-92f9-32f845f448cf"]
  },
  {
    "type": "observed-data",
    "spec_version": "2.1",
    "id": "observed-data--b67d30ff-02ac-498a-92f9-32f845f448cf",
    "first_observed": "2023-10-21T19:00:00Z",
    "last_observed": "2023-10-25T19:58:16Z",
    "number_observed": 50,
    "object_refs": [
      "file--30038539-3eb6-44bc-a59e-d0d3fe84695a"
    ]
  },
  {
    "type": "file",
    "spec_version": "2.1",
    "id": "file--30038539-3eb6-44bc-a59e-d0d3fe84695a",
    "name": "suspicious_payload.exe",
    "hashes": {
      "SHA-256": "4bac27393bdd9777ce02453256c5577cd02275510b2227f473d03f533924f877"
    }
  }
]
```

식별 기관( b67d30ff)에서 suspicious\_payload.exe라는 파일이 50번 실행된 로그(21일 19:00  \~ 25일 19:58)가 발견됐습니다.\
탐지 룰(8e2e2d2b)과 동일한 패턴이라는 것을 확인했습니다 (21일 19:00).\
(Observed-data는 25일까지 실행됐는데, sighting은 업데이트가 안돼있습니다).

***

## Part 7: Meta Objects와 Bundle

데이터가 어떻게 사용되고 공유될 수 있는지에 대한 제한, 권한 및 기타 지침을 나타냅니다.

### 7.1 Data Markings (데이터 보호와 권한)

#### Marking Definition: TLP(Traffic Light Protocol) 및 Statement 지정 (객체)

데이터 마킹 규칙 그 자체를 정의하는 독립된 메타 객체입니다. 절대로 Versioning을 할 수 없습니다.

* Statement (서술형 마킹): \
  저작권, 이용약관 같은 텍스트 기반 마킹 문구를 정의합니다. 사람이 읽고 참고하기 위한 목적을 가졌습니다.
* TLP(Traffic Light Protocol):\
  전 세계 보안 커뮤니티의 표준 정보 공유 등급인 TLP(White, Green, Amber, Red)를 지정합니다.

#### Object-Level 마킹과 Granular(세밀한) 마킹의 차이점 및 적용법 (Embedded Property)

* Object-Level Marking (객체 전체 마킹):\
  STIX 객체 전체와 그 안 모든 속성을 한번에 마킹을 적용. object\_marking\_refs 속성에 마킹 객체 ID를 포함.
* Granular Marking (세밀한 부분 마킹):\
  객체 전체가 아니라 객체 내부의 각각 특정 속성에 마킹을 적용. granular\_markings 속성에 selectors 리스트에 어떤 속성에 마킹을 적용할지 경로 지정.

```json
{
  "type": "campaign",
  "id": "campaign--83422c77-904c-4dc1-aff5-5c38f3a2c55c",
  "name": "Operation Ghost",
  "description": "이 캠페인은 북한의 특정 부대에서 주도한 것으로...",
  "labels": ["top-secret-op", "finance-target"],
  
  // 1. Object-Level 마킹: 객체 전체는 TLP:GREEN 이다.
  "object_marking_refs": [
    "marking-definition--34098fce-860f-48ae-8e50-ebd3cc5e41da" 
  ],
  
  // 2. Granular 마킹: 특정 필드만 콕 집어서 다른 마킹을 적용한다!
  "granular_markings": [
    {
      "marking_ref": "marking-definition--089a6ecb-cc15-43cc-9494-767639779123",
      "selectors": ["description", "labels.[0]"] 
    }
  ]
}
```



### 7.2 다국어 번역과 포장

#### Language Content: 기존 객체를 수정하지 않고 다국어(번역) 텍스트 덧붙이기

Language Content 객체는 원본 STIX 객체를 수정하지 않도고 다국어 번역 텍스트를 덧붙일 수 있는 메타 객체.\
Versioning이 들어가서, 해당 버전의 테스트만 유효합니다.

작성자가 원본 STIX 객체에 번역본도 넣고 싶을때 속성이 아닌 Language Content 객체를 추가 생성해야 합니다.

#### Bundle: 객체들을 한 번에 묶어 전송하는 임시 컨테이너

SDO, SCO, SRO 및 여러 객체들을 TAXII 서버나 이메일로 전송하기 위해서는 JSON 파일로 묶기 위해 Bundle을 사용합니다.

* 의미적 연결 없음: 하나의 Bundle에 포함된 객체들은 항상 연관된 정보는 아닙니다. 연관성을 부여하려면 Grouping/Report SDO를 사용합니다.
* 일회성: 수신자가 Bundle을 받으면, 내용물만 저장하고 Bundle은 폐기합니다.
* 객체가 아니다: Bundle은 객체가 아닙니다. 공통 속성이나 마킹을 가질 수 없고, SRO로 연결될 수도 없습니다.

```json
{
  "type": "bundle",
  "id": "bundle--5d0092c5-5f74-4287-9642-33f4c354e56d",
  "objects": [
    { "type": "indicator", "id": "indicator--8e...", "name": "악성 IP 지표" },
    { "type": "malware", "id": "malware--31...", "name": "Poison Ivy" },
    { "type": "relationship", "id": "relationship--44...", "relationship_type": "indicates" }
  ]
}
```

> Page 194, 213

***

## Part 8: 확장(Extensions)과 커스터마이징

### 8.1 STIX 2.1 Extension Definition (확장 정의 메타 객체)

새로운 객체를 만들거나 기존 객체에 속성을 추가하려면, 가장 먼저 "확장을 만들 것이다" 라는 설계도 객체(Extension Definition)를 만들어야합니다.

핵심 속성:

* schema (필수): 이 확장이 어떤 데이터 구조를 가지는지 정의한 문서의 URL을 작성합니다.
* version (필수): 확장의 버전입니다.
* extension\_types (필수): 이 확장이 어떤 종류인지(새로운 객체/속성 추가)를 배열로 선언합니다.

{% hint style="info" %}
확장이란?

기존 스마트폰에서 새로운 앱을 다운받아 기능을 추가하고 싶은 상황에서,

Extention Definition은 앱을 스토어에 등록하기 위한 "이 앱은 무슨 기능을 하고, 어떤 데이터를 다룬다"같은 정보를 생성해야합니다.

확장은 새로운 앱을 설치하는 것으로 기존 '스마트 폰'에 기능을 확장하는 것과 같은 맥락입니다.
{% endhint %}

#### 새로운 SDO, SCO, SRO 창조하기 (`new-sdo`, `new-sco, new-sro`)

STIX에 존재하지 않는 새로운 객체를 만들 때 사용합니다. \
extension\_types:

* new-sdo: 새로운 도메인 객체 생성
* new-sco: 새로운 관찰 대상 객체 생성
* new-sro: 새로운 관계 객체 생성

#### &#x20;기존 객체에 속성 추가하기 (`property-extension`)

* Nested Property Extension (property-extension) - 권장\
  객체 내부의 extensions 딕셔너리 안에 확장 속성들을 안전하게 '격리'시켜서 추가하는 방식. 충돌 위험이 없습니다.

{% hint style="info" %}
왜 객체로 만든 후에 SRO로 연결하지 않고 nested property를 사용하는가?

1. 내재적 속성의 원칙:\
   내장된 관계는 특정 속성이 객체의 내재적인 부분이며, 제3자가 별도로 주장하거나 점수를 매길 필요가 없는 경우에 사용.\
   사람이라는 객체는 직업과 취미를 연결할 수는 있지만, 그 사람의 혈액형이나 DNA 정보는 고유한 특성입니다.
2. 그래프 최적화 (Cluttering 방지):\
   STIX는 기술적인 세부 데이터는 객체 내부에 숨겨두고, 거시적인 흐름만 그래프 화살표로 볼수 있게 최적화되어있습니다.
3. 강력한 스키마 기반 검증(효율성):\
   SRO를 바탕으로 DB를 탐색하지 않고, 해당 객체 하나에 붙어있는 속성을 통해 확장 정보를 한번에 볼 수 있기에 더 효율적입니다.

Embedded를 판단하기 위해서 고려할 것:

1. Embedded: 이 정보는 이 객체에 종속적이다.
2. SRO: 이 정보는 나중에 다른 객체와도 연결될 수 있다.
{% endhint %}

* Top-Level Property Extension (toplevel-property-extension) - 주의\
  객체의 최상위 레벨에 속성을 직접 꽂아 넣는 방식. Extension Definition 객체 안에 extension\_properties라는 리스트를 만들어 "루트 레벨에 어떤 이름의 속성들을 추가할지" 명시합니다. 충돌 위험 존재.

{% hint style="info" %}
A회사가 최상위에 rank를 추가하고, B회사도 최상위에 rank를 추가하면 충돌이 발생하여 가급적이면 property-extension방식 사용을 권장.
{% endhint %}

#### 확장 스키마(Schema) 작성 및 UUID 참조 방식

설계도(Extension Definition) 객체를 만들었다면, 설계도에 맞게 데이터를 넣기 위해\
객체의 extensions 딕셔너리 안에서 설계도 객체의 id를 key값으로 사용하여 연결합니다.

```json
기존 Indicator 객체에 우리 회사만의 독성 지수(toxicity) 추가하기
// 1. 설계도 창조 (Extension Definition)
{
  "type": "extension-definition",
  "spec_version": "2.1",
  "id": "extension-definition--d83fce45-ef58-4c6c-a3f4-1fbc32e98c6e",
  "name": "ACME Corp Toxicity Score",
  "schema": "https://acme.com/schema/toxicity/v1/",
  "version": "1.0.0",
  "extension_types": ["property-extension"]
}

// 2. 실제 객체에 설계도 적용 (UUID 참조)
{
  "type": "indicator",
  "id": "indicator--e97bfccf-8970-4a3c-9cd1-5b5b97ed5d0c",
  "name": "Poison Ivy 탐지 룰",
  
  "extensions": {
    // 확장의 ID(UUID)를 Key로 사용하여 맵핑!
    "extension-definition--d83fce45-ef58-4c6c-a3f4-1fbc32e98c6e": {
      "extension_type": "property-extension",
      "toxicity": 8,
      "rank": 5
    }
  }
}
```

> Page 205\~210, 239,&#x20;

***

## Part 9: STIX Patterning (탐지 규칙 언어)

### 9.1 STIX 패턴의 문법 구조

STIX 패턴 언어는 Indicator 객체의 pattern 속성에 들어가는 특수한 문자열 문법.\
STIX 패턴으로 기존에 알 수 없던 복합적인 부가 이벤트까지 묘사할 수 있습니다. \
(기존 YARA, SNORT는 파일/네트워크 패킷 하나만 검사 vs STIX 패턴은 파일과 네트워크 통신이 동시에 일어나는 복합적인 이벤트 묘사.)

{% hint style="info" %}
Sigma Rule은 특정 벤더사에 종속되지 않도록 만들어진 언어인데, 왜 STIX Patterning이라는게 또 만들어진걸까?

Sigmam, YARA, Snort는 각자의 영역에서 완벽한 벤더 중립 표준이지만, 서로의 영역을 넘나들지는 못합니다. (Sigma 룰안에서 YARA 문법을 쓸 수 없는 것처럼.)\
STIX Patterning으로 모든 영역의 데이터를 하나의 문법안에서 동시에 엮을 수 있게 됩니다.

크로스 도메인 탐지 예시:\
"악성 해시 파일을 발견한 뒤 -> 120초 이내에 특정 레지스트리 키를 변경하는 행위를 잡아라."\
이런 복합적인 시나리오를 Sigma나 YARA 단독으로 표현하기 어렵습니다.

\[file: ...] FOLLOWEDBY \[registry: ...]
{% endhint %}

{% hint style="info" %}
그럼 Sigma를 버리고 STIX Patterning만 쓰면 되는거 아닌가?

현장(SOC)에서는 여전히 Sigma, YARA, Snort를 쓰게 됩니다. 그 이유는 STIX Patterning은 구조가 방대하고 복잡하여 실제 보안 장비 엔진(엔드포인트/방화벽)에서 실시간으로 직접 돌리기엔 무거울 수 있습니다.

"거시적인 공격 시나리오(TTPs)를 교환할 때는 STIX Patterning을 쓰고, 우리 회사 SIEM장비에 당장 룰을 적용할 때는 Sigma 포맷을 활용"
{% endhint %}

#### 패턴 표현식(Pattern Expressions)과 대괄호 `[ ]`의 의미

STIX 패턴의 기본 단위는 관찰 표현식(Observation Expression)이고, 대괄호 \[ ]로 표현식을 감싸게 됩니다.

* 대괄호 \[ ] 의 의미:\
  단일 관찰 데이터(Single Observation, 하나의 Observed Data SDO 인스턴스)를 의미.
* 구조의 확장:\
  대괄호 안에는 <객체 경로> = <값> 형태의 비교 표현식들이 AND / OR로 묶여 들어갑니다.

{% hint style="info" %}
해커가 악성 도메인에 접속 한 뒤 (관찰 이벤트 로그)\
특정 레지스트리를 변조하는 연속 공격 (관찰 이벤트 로그)

-> \[ 악성 도메인 접속 ] FOLLOWEDBY \[ 레지스트리 변조 ]
{% endhint %}

#### 객체 경로 구문(Object Path Syntax) 작성법 (예: `file:hashes.'SHA-256'`)

대괄호 안에 "어떤 객체의 어떤 속성을 검사할 것인가?"를 지시하는 문법

* 기본 문법 ( : 와 . ):\
  <객체 타입>:<속성 이름> 형태로 작성합니다.\
  하위 속성으로 파고들 때는 점(.)을 사용. (network-traffic:src\_port)\
  속성 이름 안에 ( - 또는 . )이 포함되어 있다면 (' ')로 감싸야합니다. (file:hashes.'SHA-256')
* 리스트(배열) 속성 검사 ( \[\*] ):\
  속성이 리스트일 경우, 인덱스 번호( \[0], \[1] )를 쓰거나 별표 \[\*]를 써서 리스트 안의 모든 항목 중 하나라도 일치하면 True라고 지시할 수 있습니다.\
  (directory:contains\_refs\[\*].name = 'malware.exe'

{% hint style="info" %}
"HA-256 해시가 `aec0...`인 파일이거나, 악성 도메인 `evil.com`으로 통신하는 네트워크 로그를 찾아라"

\[ file:hashes.'SHA-256' = 'aec070645fe53ee3b3763059376134f058cc337247c978add178b6ccdfb0019f' OR network-traffic:dst\_ref.value = 'evil.com' ]

\*패턴은 Indicator 객체의 pattern 속성에 순수한 유니코드 문자열(Unicode string)으로 들어갑니다.

\*관찰 표현식에서 AND 로 묶인 비교 표현식은 동일한 SCO에서 출발해야합니다.\
\[ network-traffic:src\_port = 80 AND file:size = 100 ] -> \[ network-traffic:... ] AND \[ file:... ]
{% endhint %}

> Page 223, 229, 230, 220, 225, 227

### 9.2 연산자와 한정자

#### 비교 연산자(Comparison Operators): `=`, `!=`, `IN`, `MATCHES`(정규식), `ISSUBSET` 등

* 기본 연산자: =, !=, >, <, >=, <=를 지원합니다.
* IN: set중 하나와 일치하는지 확인합니다.
* LIKE: 와일드카드. SQL의 LIKE 문법과 동일합니다.
* MATCHES: 정규식 표현. PCRE 표준을 따르는 정규표현식 검색을 지원합니다.
* ISSUBSET / ISSUPERSET: IP 대역 전용으로 IPv4나 IPv6 주소가 특정 서브넷(CIDR)에 포함되는지/포함하는지 계산합니다.
* EXISTS: 속성 자체 존재 여부 판별

#### 논리 연산자(Boolean Operators)와 관찰 연산자(Observation Operators: `FOLLOWEDBY`)

* 논리 연산자 (AND, OR): \
  대괄호 내부에서 비교 연산식들을 묶습니다.\
  AND로 묶인 조건들은 동일한 단일 관찰 데이터(SCO)안에서 모두 참이어야 합니다.
* 관찰 연산자 (AND, OR, FOLLOWEDBY):\
  대괄호와 대괄호 사이 서로 다른 관찰 이벤트 들을 묶어줍니다.

#### 시간/반복 한정자(Qualifiers): `WITHIN`, `REPEATS`, `START/STOP`

관찰 표현식의 맨 뒤에 붙어서 "이 사건이 언제, 얼마나 자주 일어났는가?"라는 제약 조건을 걸어줍니다.

* REPEATS x TIMES: 이벤트가 정확히 x번 반복 관찰되어야 합니다.
* WITHIN x SECONDS: 여러 이벤트들이 x초내에 모두 발생해야 합니다.
* START x STOP y: 절대적인 타임스탬프 구간입니다.

{% hint style="info" %}
상황: "동일한 IP에서 의심스러운 엑셀 파일을 다운로드한 뒤(`FOLLOWEDBY`), 300초 이내에(`WITHIN`) 5번 연속으로(`REPEATS`) 레지스트리 키를 조작하는 행위를 탐지하라"

( \[file:name LIKE '%.xls' AND file:size > 1024] \
FOLLOWEDBY \
\[windows-registry-key:key = 'HKEY\_CURRENT\_USER\Software\Run'] REPEATS 5 TIMES ) \
WITHIN 300 SECONDS
{% endhint %}

> Page 222

***

## Part 10: 어휘와 열거형 (Vocabularies & Enumerations)

STIX 2.1의 어휘 사전은 이름 끝에 -enum인 닫힌 사전과 -ov가 붙는 열린 사전이 있습니다.

* Open Vocabulary (-ov):\
  업계에서 널리 쓰이는 공통 용어드를 가이드로 제공하지만, 스펙 문서에 없는 신조어나 회사 내부 용어가 필요하다면 확장해서 쓸 수 있습니다.\
  (eg. attack-motivation-ov / industry-sector-ov / threat-actor-role-ov)
* Enumerations (-enum):\
  어떤 시스템이든 동일한 방식으로 동작해야 하는 기계적인 설정이나 암호화 방식들을 정의할 때 사용.\
  스펙 문서의 목록에 있는 값만 사용해야하며, 임의로 값을 추가하거나 변경하면 STIX 유효성 검사에 실패하게 됩니다.\
  (eg. encryption-algorithm-enum / windows-registry-datatype-enum / entension-type-enum)

{% hint style="info" %}
악성코드 샘플을 `AES-128`로 암호화해서 공유하고 싶습니다. 그런데 `encryption-algorithm-enum` 목록을 보니 `AES-256-GCM`밖에 없네요. 그냥 `"encryption_algorithm": "AES-128"`이라고 적어서 보내면 안 되나요?

안 됩니다.\
`-enum`은 닫힌 사전이므로, 목록에 없는 `AES-128`을 입력하는 순간 그 STIX 문서는 규격 위반(Invalid)이 되어 다른 보안 장비에서 파싱 에러를 뿜어내게 됩니다 (Page 234, Line 3266). \
만약 꼭 `AES-128`을 써야 한다면, 기존 `Artifact` 객체의 표준 암호화 필드를 쓰는 대신, 새로운 `Extension Definition`을 만들어 독자적인 암호화 속성을 추가(`property-extension`)하는 방식으로 우회해야 합니다.
{% endhint %}

> Page 234

***

## Part 11: 적합성(Conformance)과 실전 적용

### 11.1 소프트웨어의 STIX 준수 요건

STIX 생태계에서 소프트웨어는 데이터를 만들어내는 생성자와 데이터를 읽어들이는 소비자로 나뉩니다.

### Producer(생성자)와 Consumer(소비자)가 지켜야 할 필수 기능

생성자의 조건:

* JSON 필수: 직렬화 포맷인 JSON으로 데이터를 인코딩해야 합니다.
* 필수 속성 포함: 각 객체 스팩에 "Required"라고 적힌 속성은 단 하나도 빠짐없이 채워 넣어야 합니다.
* 객체 지원: 최소한 1개 이상의 STIX Object를 생성할 수 있어야 합니다.

소비자의 조건:

* 파싱 필수: 수신한 STIX JSON 데이터의 모든 '필수(Required)' 속성을 튕겨내지 않고 파싱(읽기)할 수 있어야 합니다.
* JSON 필수: JSON 포맷을 기본적으로 읽어 들일 수 있어야 합니다.

### 11.2 STIX Patterning 적합성 레벨

전 세계 수많은 보안 장비들은 각자의 역할에 따라 CPU와 메모리 성능이 천차만별입니다. STIX는 장비의 체급에 맞춰 패턴을 3단계로 나누어 소화할 수 있도록 가이드라인을 제공합니다.

STIX 탐지 패턴 언어를 구현하는 소프트웨어는 그들이 처리할 수 있는 문법의 깊이에 따라 Level 1부터 Level 3까지 나뉩니다. ("우리 장비는 어디까지 이해할 수 있습니다")

1. Level 1: Basic Conformance (기본 / 무상태 탐지)\
   가장 빠르고 가볍습니다. 패터닝 사양의 최소 필수 측면을 준수하는 소프트웨어
2. Level 2: Basic Conformance plus Observation Operators (다중관찰)\
   최소 필수 측면을 지원하면서 다중 관찰에 대해 작동할 수 있는 소프트웨어.
3. Level 3: Full Conformance (전체 호환 / 시계열 탐지)\
   패터닝 사양의 모든 기능을 준수합니다.
