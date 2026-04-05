---
description: 'Sigma Rule 마스터하기: 기초부터 실무 운영까지'
---

# SIGMA Rule

## Part 1. 가장 먼저 잡아야 할 큰 그림: 왜 Sigma인가?

### 1.1 Sigma Rule의 정의와 탄생 배경

#### SIEM 파편화 문제: 왜 플랫폼마다 쿼리가 다른가?

{% hint style="info" %}
SIEM(Security Information and Event Management)

기업 내의 방화벽, 서버, 네트워크 장비 등 다양한 시스템에서 발생하는 방대한 로그를 한곳으로 수집하고 분석하여 사이버 위협을 탐지하는 '보안 관제 중앙 통제실' 역할.
{% endhint %}

표준화된 오픈소스 빅데이터 처리 기술이 마땅히 없었기떄문에, \
SIEM 벤더사들이 데이터를 저장하고 검색하는 아키텍처를 서로 다르게 설계.\
그로 인해, SIEM 파편화 문제가 발생했습니다.

예를 들어:

* Splunk: 대용량 텍스트에서 데이터를 읽는 시점(Schema-on-read)에 구조를 부여
* Elastic: 검색 엔진 기반(Lucene)의 강력한 사전 인덱싱 방식 채택

{% hint style="info" %}
Schema-on-read(Splunk)

데이터를 수집할 때는 형태를 따지지 않고 일단 원본 텍스트 그대로 밀어 넣은 뒤, 나중에 분석가가 검색하는 시점에 비로소 구조를 부여하는 방식.
{% endhint %}

{% hint style="info" %}
Inverted Index 및 Schema-on-write(Elastic)

특정 키워드를 빠른 속도로 찾아주는 검색 엔진을 바탕으로,\
데이터를 수집하는 시점에 체계적으로 데이터를 저장해주는 방식.
{% endhint %}



#### "한 번 작성하여 어디든 적용한다(Write Once, Use Anywhere)"의 철학

파편화의 문제를 해결하고자, 특정 벤터에 종속되지 않는 범용 로그 탐지 포맷인 Sigma Rule이 등장.

{% hint style="info" %}
Sigma

소프트웨어 개발에서 하나의 소스코드를 작성한 뒤 다양한 운영체제에 맞게 컴파일 하는 것처럼, 하나의 YAML 파일로 작성된 탐지 논리를 Splunk, Elastic, Sentinel 등 다양한 환경의 쿼리로 변환 가능.
{% endhint %}



#### Sigma Rule의 역할: YARA, Snort와의 결정적 차이점 이해

기업의 IT 인프라를 안전하게 지키기 위해 감시해야하는 구역은 크게 3가지:

* 파일 - YARA
* 네트워크 - Snort
* 시스템 - Sigma

파일과 네트워크에서는:

* YARA(Yet Another Ridiculous Acronym ???) - 파일 기반 탐지 표준\
  서버 또는 시스템에 저장된 파일 내부를 들여다보며, 악성코드 파일이 가진 특정한 텍스트 문자열이나 파이트(Byte) 패턴을 찾아내는 데 특화.
* Snort - 네트워크 기반 탐지 표준\
  네트워크 길목에 서서 네트워크 회선을 타고 들어오고 나가는 패킷의 헤더와 페이로드를 실시간으로 검사하여 침입 시도를 차단.

YARA와 Snort가 각각 파일과 네트워크를 방어하지만, 현대의 해커들은 이를 우회하는 방법을 찾아냅니다.

예를 들어, 하드디스크에 악성파일을 다운로드 하지 않고, 윈도우에 기본 탑재된 관리자 도구(PowerShell 등)를 메모리상에서만 악용하여 시스템을 파괴하는 경우, YARA와 Snort로는 이를 알아채기 어렵습니다.

그래서 이벤트 로그를 분석하여 위협을 탐지하는 포맷인 Sigma 등장.

* Sigma - 시스템 로그 기반 탐지 표준\
  시스템이나 사용자가 이미 어떤 행동을 마치고 남긴 '발자국', 즉 엔드포인트 장비나 애플리케이션에서 생성된 이벤트 로그의 패턴을 분석하는 데 사용.

> \[SigmaHQ Official Documentation - What is Sigma: [https://sigmahq.io/docs/basics/what-is-sigma.html](https://www.google.com/search?q=https://sigmahq.io/docs/basics/what-is-sigma.html)]
>
> Nextron Systems - Sigma: Generic Signatures for SIEM Systems: [https://www.nextron-systems.com/2018/02/10/sigma-generic-signatures-for-siem-systems/](https://www.google.com/search?q=https://www.nextron-systems.com/2018/02/10/sigma-generic-signatures-for-siem-systems/)]





### 1.2 탐지 엔지니어링을 위한 선행 지식

보안 관제에서 탐지 룰을 설계하기 위해서 분석 대상이 되는 원천 데이터의 성격을 파악하는게 가장 선행되어야합니다.

{% hint style="info" %}
탐지 룰

로그와 텔레메트리 데이터 속에서 해커의 공격이나 비정상적인 행위를 자동으로 식별해 내기 위해 미리 정의해 둔 '논리적인 조건식'.

비유하자면 경찰이 범죄자를 잡기 위해 구체적인 인상착의를 적어둔 '지명수배 전단지'와 같은 역할을 수행.

예:

IF: 문서 편집 프로그램(winword.exe)이 자식 프로세스 파워셀(powershell.exe)을 은닉 모드로 실행한다면,\
THEN: 보안관제팀에 '즉각 경보를 울려라'와 같이 특정 행위 패턴을 조건으로 명시하는 것.

이 룰을 시스템에 적용하여, 보안 분석가는 대량의 활동 데이터들 사이에서 시스템을 위협하는 악성 활동을 찾아냄.
{% endhint %}

```yaml
Sigma YAML (특정한 위협을 잡기 위해 작성된 탐지 룰. ie.지명수배 전단지)
title: Suspicious PowerShell Encoded Command
id: 12345678-1234-1234-1234-123456789012 # 전역적으로 고유한 식별자(UUID)입니다.
status: experimental # 룰의 현재 상태(test, experimental, stable 등)를 나타냅니다.
description: Detects the execution of PowerShell with Base64 encoded arguments. # 이 룰이 무엇을 찾는지 설명합니다.
author: My Name # 룰 작성자의 이름이나 소속을 적습니다.
date: 2026/04/06 # 작성 또는 마지막 수정 날짜입니다.
tags:
    - attack.execution # MITRE ATT&CK 전술(Execution)과 맵핑합니다.
    - attack.t1059.001 # MITRE ATT&CK 세부 기법(PowerShell)과 맵핑합니다.
logsource:
    category: process_creation # 대상 로그의 종류를 프로세스 생성 로그로 한정합니다.
    product: windows # 대상 운영체제를 윈도우로 지정합니다.
detection:
    selection: # 탐지할 핵심 조건을 정의하는 부분입니다.
        Image|endswith: '\powershell.exe' # 실행 파일 경로가 powershell.exe로 끝나는지 확인합니다.
        CommandLine|contains: # 커맨드라인 인자에 아래 문자열 중 하나라도 포함되어 있는지 확인합니다.
            - ' -enc '
            - ' -EncodedCommand '
    condition: selection # 위에서 정의한 selection 조건에 모두 일치하면 알람을 발생시킵니다.
falsepositives:
    - Legitimate administrative scripts # 정상적인 시스템 관리 스크립트가 오탐을 낼 수 있음을 명시합니다.
level: high # 탐지되었을 때 경보의 심각도(low, medium, high, critical)를 설정합니다.
```

```json
Sigma YAML로 잡아낸 원본 이벤트 로그 예시
{
  "EventID": 4688,
  "LogName": "Security",
  "Computer": "Server-01.company.local",
  "SubjectUserName": "Admin",
  "NewProcessName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "CommandLine": "powershell.exe -enc JABzAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAE0AZQBtAG8AcgB5AFMAdAByAGUAYQBtACgAWwBDAG8AbgB2AGUAcgB0AF0AOgA6AEYAcgBvAG0AQgBhAHMAZQA2ADQAUwB0AHIAaQBuAGcAKAAiAEgA..."
}
```

#### 로그(Log) vs 텔레메트리(Telemetry)의 차이

데이터는 크게 2가지로 분류됩니다.

* 로그(Log):\
  시스템에서 접속, 오류, 파일 생성 등 특정 이벤트가 발생했을 때마다 단편적으로 남기는 정적인 기록.
* 텔레메트리(Telemetry):\
  원격 측정이라는 의미로, 엔드포인트 장비가 프로세스의 흐름이나 메모리 상태 등을 동영상처럼 연속해서 중앙으로 스트리밍하는 동적인 데이터의 흐름을 뜻



#### 위협의 흔적: IOC, IOA 그리고 TTP

IOC, IOA, TTP는 로그와 텔레메트리(증거)에서 찾아내고자하는 위협의 실체.

{% hint style="info" %}
Log & Telemeter와 IOC, IOA, TTP의 관계

로그와 텔레메트리는 사진(Log) EHsms CCTV 비디오(Telemetry)와 같은 기록 매체.\
IOC, IOA, TTP는 사진과 비디오를 분석하여 식별해야내는 범인의 지문(IOC), 범행을 시도하려는 수상한 동작(IOA), 그리고 범인 고유의 치밀한 침투 수법(TTP)에 해당

예:

해커의 악성 C\&C 서버 IP 주소나 다운로드된 파일의 해시값 같은 단순한 IOC는 웹 서버나 방화벽이 단편적으로 남긴 Log로 식별 가능.

반면, 해커가 정상적인 윈도우 프로세스에 몰래 악성 코드를 주입하고 내부망의 다른 서버로 조용히 이동을 시도하는 복잡한 IOA, TTP를 파악하기 위해서는 로그로는 부족.

고차원적인 전술과 행위의 맥락(Context)를 끊긱지 않고 추적하기 위해 EDR과 같은 장비가 시스템 내부의 메모리 변화와 프로세스 실행 트리를 동영상 처럼 연속해서 스트리밍해 주는 풍부한 텔레메트리(Telemetry) 데이터가 뒷받침돼야함.

비유:

로그와 텔레메트리라는 '생선'를 도마 위에 올린 뒤, Sigma Rule '레시피'를 적용하여, 최종적으로 IOA, TTP '가시'를 발라내는 과정.
{% endhint %}



방대한 데이터 속에서 찾아내야하는 위협의 흔적들은 크게 3가지로 분류.

* IOC(Indicator of Compromise, 침해 지표):\
  해커가 시스템에 이미 침입하고 난 뒤 남기고간 과거의 흔적.\
  악성코드 해시값, 해커가 사용한 C\&C 서버의 IP 주소나 악성 도메인 등이 대표적.\
  방어벽에 IOC 값들을 등록하여 차단할 수 있지만, 공격자가 IP주소나 파일의 해시값을 바꾸면 무용지물이기에 수명이 짧다는 단점.
* IOA(Indicator of Attack, 공격 지표):\
  현재 진행 중인 비정상적인 행위의 악의적 의도와 맥락에 초점을 둔 능동적인 지표.\
  방어자는 특정 악성코드의 해시값을 쫓는 대신 비정상적인 행위 패턴 자체를 추적.\
  예를 들어, "파워셀이 숨김 모드로 실행되어 암호화 명령어를 처리한다" 등
* TTP(Tactics, Techniques, and Procedures, 전술, 기법, 절차)\
  해커가 공격 목표를 달성하기 위해 사용하는 근본적인 '전략과 행동 메뉴얼'을 의미.\
  해커가 초기 침투(Tactic)을 하기 위해 이메일 스피어 피싱 기법(Technique)을 사용하고, 침투 후 백도어를 설치하기 위해 윈도우 레지스트리를 조작하는 일련의 과정(Procedure) 전체가 TTP에 속함.

> - CrowdStrike - IOC vs IOA: What is the Difference?: [https://www.crowdstrike.com/cybersecurity-101/indicator-of-compromise-ioc-vs-ioa/](https://www.google.com/search?q=https://www.crowdstrike.com/cybersecurity-101/indicator-of-compromise-ioc-vs-ioa/)
> - SANS Institute - The Pyramid of Pain: [https://www.sans.org/tools/the-pyramid-of-pain/](https://www.sans.org/tools/the-pyramid-of-pain/)
> - MITRE ATT\&CK - Enterprise Tactics (TTP 개념 이해): [https://attack.mitre.org/tactics/enterprise/](https://attack.mitre.org/tactics/enterprise/)



#### 현대 보안 관제의 핵심: SIEM, EDR, XDR의 역할 분담

단일 경계망만 방어해서는 막을 수 없음\
데이터를 수집하고 분석하는 영역을 기능과 범위에 따라 세 가지 솔루션으로 나누어 책임을 분담.

* SIEM(Security Information and Event Management)\
  인프라 전체를 조망하는 '중앙 관제탑' 역할을 수행.\
  방화벽, 라우터, 웹 서버, 사내 어플리케이션 등, 장비에서 발생하는 단편적인 정적 로그(Log)들을 중앙의 단일 데이터베이스로 끌어옵니다.\
  수집된 데이터를 바탕으로 상관 분석(Correlation Analysis)를 수행하여, 개별 로그로는 알아채기 힘든 조직 전체 규모의 거시적인 해킹 시나리오를 식별.\
  깊고 세밀한 동작까지는 들여다보기 어렵지만, 전체 보안 현황을 파악하고 장기적인 침해 지표를 검색하는 두뇌 역할.
* EDR (Endpoint Detection and Response)\
  개별 임직원의 PC를 깊게 밀착 감시하는 '현장 특수 요원'.\
  SIEM이 넓은 범위의 정적 로그를 모은다면, EDR은 운영체제 커널 수준에서 프로세스의 실행 트리, 메모리 인젝션, 레지스트리 조장 등 미세하고 동적인 텔레메트리(Telemetry) 데이터를 실시간으로 스트리밍받아 분석.\
  해커가 백신을 우회하는 신종 악성코드를 심거나, 파일 없이 메모리 상에서만 동작하는 공격(Fileless Attack)을 시도할 때 이를 즉각적으로 탐지하고 해당 장비를 네트워크에서 격리하는 역할.
* XDR (Extended Detection and Response)\
  XDR은 EDR이 가진 가시성에, SIEM의 넓은 통합 시야를 결합하여 진화시킨 '확장된 차세대 탐지 및 대응 플랫폼'.\
  공격자들은 엔드포인트 장비 하나만 노리지 않고, 이메일, 네트워크, 클라우드 인프라, 사용자 계정 등 다양한 경로를 넘나들며 복합적인 공격 전개.\
  XDR은 각 보안 도구들 사이에 있던 데이터 사일로(Silo) 장벽을 허물고, 모든 영역의 텔레메트리를 하나의 통합된 플랫폼으로 연결.\
  머신러닝과 자동화 기술을 통해 파편화된 경보들을 하나의 공격 스토리로 연결하여, 방어팀이 여러 화면을 오가며 분석해야 했던 시간을 줄이고, 즉각 대응 가능.

{% hint style="info" %}
XDR = SIEM + EDR?

EDR은 PC나 서버(엔드포인트) 내부만 깊게 감시하다 보니, 해커가 장비를 빠져나가 네트워크나 클라우드로 숨어버리면 전체 공격 동선을 놓치는 한계가 있습니다.\
그래서, EDR이 가진 심증 분석 및 즉각적인 차단 기능을 네트워크, 이메일, 클라우드, 사용자 계정 영역까지 확장하여, 해커의 경로를 하나의 스토리로 묶어낸 진화형 모델이 XDR.

SIEM - 장기적인 거시적 데이터 분석을 수행하는 형태\
XDR - 신속한 실시간 위협 대응
{% endhint %}



#### MITRE ATT\&CK 프레임워크와 탐지 설계의 연결 고리

MITRE ATT\&CK 프레임워크는 전 세계에서 발생한 실제 해킹 침해 사고들을 분석하여, 공격자들의 Tactics와 Techniques를 체계적으로 분류해 놓은 지식 사전.\
방어자들은 이 프레임워크를 바탕으로 해커가 왜, 어떻게 움직이는지 공통된 언어로 소통할 수 있게됐습니다.

이를 바탕으로, 로그 전체를 조회하지 않고, 실제 해커의 행동 메뉴얼 및 기록을 바탕으로, 어떤 로그를 봐야하고, 어떤 Sigma Rule을 작성해야할지 우선순위를 정할 수 있게됩니다.

예를 들어, 도둑이 환풍구를 통해 들어왔던 메뉴얼이 존재한다면, 환풍구를 집중적으로 감시하는 룰을 만들 수 있습니다.



### Part 2. Sigma Rule 구조 해부: YAML 속에 담긴 탐지 논리

#### 2.1 Sigma Rule 파일의 전체 구조와 Metadata

* YAML 기반 포맷 이해와 규칙 작성의 기본 골격
* Metadata의 중요성: Title, Status, Tags가 운영에 미치는 영향
* Rule Lifecycle: 테스트에서 배포까지의 상태 관리

#### 2.2 Logsource: 탐지의 출발점 설정하기

* Category, Product, Service 필드 완벽 이해
* "어떤 로그를 볼 것인가?"에 따른 룰의 유효성 판단
* Windows Sysmon vs Native Event Log의 로그 소스 차이점

#### 2.3 Detection & Condition: 탐지 엔진의 심장

* Selection과 Filter의 조합을 통한 정교한 타격
* 매칭 연산자 딥다이브: Contains, StartsWith, Regex
* \[핵심] Value Modifiers 활용법 (Base64Offset, Windash 등)
* Condition 문법: 1 of them, selection and not filter 등 논리 구조 짜기

### Part 3. 실전! 공격 행위를 규칙으로 번역하기

#### 3.1 공격자 사고방식으로 탐지 룰 설계하기

* CTI(위협 인텔리전스) 보고서에서 탐지 포인트 추출하기
* "정상 행위와 무엇이 다른가?" - 제외 조건(Filtering) 설정의 기술
* 추상적인 공격 기법을 구체적인 로그 패턴으로 치환하는 법

#### 3.2 주요 Use Case별 룰 작성 실습

* LOLBins(rundll32, mshta) 등 기본 도구 오남용 탐지
* PowerShell 및 스크립트 기반 공격 패턴 식별
* 권한 상승 및 방어 회피(Defense Evasion) 흔적 찾기

### Part 4. 백엔드 변환과 환경 최적화: 이론을 실천으로

#### 4.1 Sigma CLI와 변환 엔진(Backend) 이해

* Sigma 자체는 실행 엔진이 아니다: 변환기의 역할
* 신규 표준 도구 `sigma-cli` 설치 및 기본 사용법
* 특정 SIEM(Splunk, ELK, Sentinel)으로의 쿼리 변환 실습

#### 4.2 데이터 정규화와 필드 맵핑의 한계

* Sigma 필드명과 실제 SIEM 필드명이 다를 때의 대응 (Pipeline)
* 변환 과정에서 발생하는 논리적 손실과 주의사항
* 환경별 최적화(Tuning)가 필요한 이유

### Part 5. 퀄리티 컨트롤과 운영: 지속 가능한 보안 관제

#### 5.1 오탐(FP)과 미탐(FN)의 구조적 관리

* Precision(정밀도) vs Recall(재현율)의 트레이드 오프
* 경보 피로도(Alert Fatigue)를 줄이기 위한 튜닝 전략
* "탐지되는 룰"을 넘어 "운영 가능한 룰"로 다듬기

#### 5.2 Rule 검증 및 품질 관리 프로세스

* 문법 검증(Validation)과 유효성 체크리스트
* CI/CD를 활용한 룰 배포 자동화의 기초
* 실무에서 자주 겪는 함정: Regex 남용과 와일드카드 매칭의 위험성
