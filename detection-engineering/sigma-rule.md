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



#### Sigma 자체는 실행 엔진이 아니다: 변환기의 역할

Sigma Rule 자체는 시스템을 실시간으로 검사하는 백신 프로그램이나 탐지 엔진이 아닙니다.\
그저 해커의 행위를 정의해놓은 범용적인 Sigma YAML 문서이며, 스스로 로그를 조회하거나 알람을 울릴 수 있는 기능은 없습니다.

따라서, Sigma Rule이 실제로 탐지 시스템에서 동작하려면, 이 논리를 Splunk, Elastic 등 각 조직이 실제로 사용중인 SIEM(또는 EDR, XDR)의 고유한 쿼리 언어로 번역을 해줘야합니다.



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
Sigma Rule의 탐지 대상이 되는 과거 가장 기본의 원본 이벤트 로그(Log) 예시 
{
  "EventID": 4688,
  "LogName": "Security",
  "Computer": "Server-01.company.local",
  "SubjectUserName": "Admin",
  "NewProcessName": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "CommandLine": "powershell.exe -enc JABzAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAE0AZQBtAG8AcgB5AFMAdAByAGUAYQBtACgAWwBDAG8AbgB2AGUAcgB0AF0AOgA6AEYAcgBvAG0AQgBhAHMAZQA2ADQAUwB0AHIAaQBuAGcAKAAiAEgA..."
}
```

```json
Sigma Rule 탐지 대상이 되는 최신 버전의 원본 이벤트 로그(Log) 예시
{
  "EventID": 1,
  "RuleName": "technique_id=T1059.001,process_creation",
  "UtcTime": "2026-04-06 07:12:24.123",
  "ProcessGuid": "{A1B2C3D4-E5F6-7890-ABCD-1234567890EF}", // 핵심: 고유 식별자
  "ProcessId": 4512,
  "Image": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
  "FileVersion": "10.0.19041.1",
  "Description": "Windows PowerShell",
  "Product": "Microsoft® Windows® Operating System",
  "Company": "Microsoft Corporation",
  "CommandLine": "powershell.exe -enc JABzAD...",
  "CurrentDirectory": "C:\\Windows\\system32\\",
  "User": "COMPANY\\Admin",
  "LogonGuid": "{L1M2N3O4-P5Q6-R7S8-T9U0-V1W2X3Y4Z5A6}", // 로그인 세션 추적
  "LogonId": "0x3E7",
  "TerminalSessionId": 1,
  "IntegrityLevel": "High", // 권한 수준
  "Hashes": "SHA256=85E9A7E562D760A880D1A721...", // 파일 무결성 확인용
  "ParentProcessGuid": "{X9Y8Z7W6-V5U4-T3S2-R1Q0-P9O8N7M6L5K4}", // 부모 추적용
  "ParentProcessId": 1024,
  "ParentImage": "C:\\Windows\\explorer.exe",
  "ParentCommandLine": "C:\\Windows\\Explorer.EXE"
}
```

#### 로그(Log) vs 텔레메트리(Telemetry)의 차이

데이터는 크게 2가지로 분류됩니다.

* 로그(Log):\
  시스템에서 접속, 오류, 파일 생성 등 특정 이벤트가 발생했을 때마다 단편적으로 남기는 정적인 기록.
* 텔레메트리(Telemetry):\
  원격 측정이라는 의미로, 엔드포인트 장비가 프로세스의 흐름이나 메모리 상태 등을 동영상처럼 연속해서 중앙으로 스트리밍하는 동적인 데이터의 흐름을 뜻.\
  부모-자식 관계를 프로세스 트리(Tree) 구조로 즉시 파악 가능

{% hint style="info" %}
텔레메트리(Telemetry)는 하나의 로그가 아니라 일련의 로그들을 하나의 그룹으로 묶은 것인가?

물리적으로 하나의 파일 덩어리로 묶여있는 로그는 아니고, 언제든 다시 조립될 수 있도록 연결고리를 가지고 있는 데이터입니다.

A, B, C프로세스가 모두 끝날때까지 기다렸다가 한번에 보내는 방식이 아니라, 각각의 이벤트가 발생하는 즉시 로그들을 연속적으로 스트리밍하듯이 보냅니다.

일반적인 정적 로그와 달리, 텔레메트리 스트림의 각 개별 로그 내부에는 고유 식별표가 있습니다.\
A 프로세스가 실행되는 순간 Process GUID라는 UUID가 부여되고, 이후 자식인 B 프로세스가 실행되면, B의 생성 로그에 A 프로세스의 Process GUID가 같이 저장됩니다.

SIEM이나 XDR 플랫폼은 파편화된 로그들의 GUID들을 바탕으로 다시 조립합니다. 프로세스 트리 그룹으로 나타나게 됩니다.
{% endhint %}

```json
Sigma Rule의 탐지 대상이 되는 원본 이벤트 텔레메트리(Telemetry) 예시
{
  "AgentMetadata": {
    "AgentId": "EDR-AGENT-9912",
    "OS": "Windows 10",
    "HostName": "Server-01"
  },
  "EventDetails": {
    "EventType": "ProcessActivity",
    "Action": "ProcessRollup",
    "Timestamp": "2026-04-06T16:04:24Z",
    "ProcessGuid": "{a1b2c3d4-e5f6-g7h8-i9j0}", // 이 프로세스의 고유 ID
    "ParentProcessGuid": "{x9y8z7w6-v5u4-t3s2-r1q0}", // 부모 프로세스를 즉시 추적 가능
    "Image": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
    "CommandLine": "powershell.exe -enc JABzAD0ATgBlAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAE0AZQBt...",
    "Hashes": {
      "MD5": "5EB63BB735E6903A84022650849D6A71",
      "SHA256": "85E9A7E562D760A880D1A721C07C1C6C52098D3B8960E185077F5B6F55D766C2"
    },
    "User": "COMPANY\\Admin",
    "Signer": "Microsoft Windows", // 디지털 서명 여부 (정상 파일 확인)
    "BehavioralChains": [ // 텔레메트리의 핵심: 연관된 행위들을 묶음
      {
        "Type": "NetworkConnect",
        "DestIP": "104.26.10.19",
        "DestPort": 443
      },
      {
        "Type": "FileCreate",
        "Path": "C:\\Users\\Public\\malware.exe"
      },
      {
        "Type": "RegistryModify",
        "Target": "HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Run\\Infect"
      }
    ]
  }
}
```

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

### SIEM, EDR, Sigma Rule, XDR의 역사적인 등장 배경과 순서

1\. SIEM (1990년대 후반\~2000년대)

* 배경: 방화벽, IDS, 서버 등 장비가 늘어나며 로그가 파편화됨. "로그를 한곳에 모아 전체 흐름을 보자"
* 논리: Log(정적 기록) 중심. 누가 언제 어디에 접속했는가를 상관 분석
* 한계: PC 내부에서 벌어지는 행위는 로그에 남지 않아 놓치게됨

2\. EDR (2013년\~)

* 배경: SIEM이 보지 못하는 PC 내부의 사각지대를 노린 공격이 급증하여, 단말기 내부의 모든 로그를 찍자
* 논리: Telemetry(동적 행위) 중심. 어떤 프로세스가 메모리에서 무엇을 했는가를 실시간 스트리밍
* 한계: 보안 관제사가 봐야할 툴이 너무 많아짐(SIEM, EDR,..). 벤더마다 룰 형식이 달라 관리가 어려움

3\. Sigma Rule (2017년\~)

* 배경: A사 SIEM에서 쓴 룰을 B사 EDR에서도 쓰고 싶다는 요구
* 논리: 언어통일을 하여 어떤 솔루션이든 번역해서 쓸 수 있는 공통 문법(YAML)을 만듦.
* 결과: 전문가들이 만든 탐지 로직을 전 세계 보안 커뮤니티가 공유. (RCS에서 룰을 검증)

4\. XDR (2018년\~)&#x20;

* 배경: SIEM은 무겁고 분석이 느리고, EDR로는 네트워크 침투를 알 수 없음. 같이 써야함
* 논리: 계층 간 통합하여 EDR(엔드포인트) + NDR(네트워크) + Cloud(클라우드)를 하나의 플랫폼으로 묶음
* 특징: SIEM처럼 모든 로그를 긁어오는게 아니라 "보안에 핵심적인 데이터(EDR+네트워크 등)"만 연결해 공격 시나리오를 자동으로 완성. SIEM보다 빠르고 EDR보다 넓음.

#### MITRE ATT\&CK 프레임워크와 탐지 설계의 연결 고리

MITRE ATT\&CK 프레임워크는 전 세계에서 발생한 실제 해킹 침해 사고들을 분석하여, 공격자들의 Tactics와 Techniques를 체계적으로 분류해 놓은 지식 사전.\
방어자들은 이 프레임워크를 바탕으로 해커가 왜, 어떻게 움직이는지 공통된 언어로 소통할 수 있게됐습니다.

이를 바탕으로, 로그 전체를 조회하지 않고, 실제 해커의 행동 메뉴얼 및 기록을 바탕으로, 어떤 로그를 봐야하고, 어떤 Sigma Rule을 작성해야할지 우선순위를 정할 수 있게됩니다.

예를 들어, 도둑이 환풍구를 통해 들어왔던 메뉴얼이 존재한다면, 환풍구를 집중적으로 감시하는 룰을 만들 수 있습니다.



#### 정리

* 서버, 시스템의 모든 로그 데이터는 중앙 데이터베이스 역할을 하는 SIEM에 모입니다. 침해지표(IOC), 공격지표(IOA), 전술기법절차(TTP)를 추출하기 위한 원본 데이터들입니다.
* 로그 데이터에서 악의적인 기록, 행동, 패턴을 찾아내기 위해 일종의 조건식인 Sigma YAML을 작성하게 됩니다.
* Sigma Rule은 특정 조건 검색 쿼리가 벤더사에 종속되지 않는 범용 로그 탐지 포맷이고, SIEM 벤더사에 맞게 변환기가 있어야하지만, 결국 Sigma Rule이 있다면 벤더사를 변경해도 변환기로 새롭게 적용 가능합니다.
* Sigma Rule로 탐지된 결과값은 JSON으로 변환이 됩니다.



## Part 2. Sigma Rule 구조 해부: YAML 속에 담긴 탐지 논리

### 2.1 Sigma Rule 파일의 전체 구조와 Metadata

#### YAML 기반 포맷 이해와 규칙 작성의 기본 골격

Sigma Rule은 YAML 포맷 기반으로 작성이됩니다. 이 파일의 뼈대는 크게 3핵심 블록으로 분류됩니다.

* 메타데이터 블록: 규칙의 이름과 작성자 등 기본 정보표를 달아두는 영역
* 로그소스 블록: 이룰이 어떤 시스템의 로그를 분석할지 대상을 좁혀주는 영역
* 디텍션 블록: 해커의 악의적인 행위 패턴이나 침해 지표를 논리적인 조건식으로 묶어낸 영역

```yaml
title: Suspicious PowerShell Encoded Command Execution
id: 5a4a5b4c-1234-abcd-9876-000000000000
related:
    - id: 11112222-3333-4444-5555-666677778888
      type: derived
status: experimental
description: 악성 해커가 명령어를 숨기기 위해 Base64로 인코딩된 인자와 함께 파워셸을 실행하는 행위를 탐지합니다.
license: Sigma Public License v1.0
author: My Name
date: 2024/01/01
modified: 2026/04/06
references:
    - https://attack.mitre.org/techniques/T1059/001/
tags:
    - attack.execution
    - attack.t1059.001
level: high
logsource:
    category: process_creation
    product: windows
    service: security
detection:
    selection1:
        Image|endswith:
            - '\powershell.exe'
            - '\pwsh.exe'
    selection2:
        CommandLine|contains:
            - ' -enc '
            - ' -EncodedCommand '
    filter_admin:
        ParentImage|endswith: '\IT_admin_tool.exe'
    condition: (selection1 and selection2) and not filter_admin
falsepositives:
    - 정상적인 사내 IT 관리자가 서버 점검을 위해 자동화 스크립트를 돌릴 때 발생하는 로그일 확률이 존재함.
```

{% hint style="info" %}
해석

title: 의심스러운 파워셸 인코딩 명령어 감지

id: 고유 식별 번호

related: derived. 구버전 룰에서 파생된 업그레이드된 규칙

status: 충분히 검증되지 않은 룰

description: Base64로 인코딩하여 파워셸을 실행하는 패턴을 탐지하는 룰

license, author, date, modified: 메타정보.

references: MITRE ATT\&CK 공식 문서 링크 제공

tags: MITRE ATT\&CK의 실행 전술 및 T1059.001 기법을 방어하는 룰.

level: 심각 레벨의 탐지 알림.'

logsource: 윈도우OS에서 security 로그 채널에서 새롭게 생성된 로그들이 대상.

detection: 조건, 생성된 프로세스가 \powershell.exe 또는 pwsh.exe 이고, 커멘드라인에 -enc 또는 -EncodedCommand가 포함되어있고, 부모 프로세스가 \IT\_admin\_tool.exe가 아닌 것

falsepositives: 오탐 확률이 존재.
{% endhint %}



#### Metadata의 중요성: Title, Related, Status, date, modified, references, tags, Tags가 운영에 미치는 영향

* title & description:\
  시그마 룰의 제목과 상세 설명
* id: \
  전세계적 고유한 UUID
* related: \
  과거룰에서 파생됬거나, 여러규칙이 병합되었을때, 또는 동일한 룰을 중복 개발 방지
* status: \
  experimental, test, stable, deprecated로, 오탐 여부/실전 투입/폐기 를 표시
* tags:\
  MITRE ATT\&CK 프레임워크의 특정 전술 및 기법과 1:1로 연결. 태그 데이터를 집계하여 전체 방어망의 시각적인 커버리지 지도를 그림으로, 방어 사각지대(Blind Spot)을 식별
* level:\
  심각도의 우선 순위. low, medium, high, critical.
* references:\
  룰의 뼈대를 설계할때 참고했던 위협 인텔리전스(CTI) 보고서나 공격 원리 파악시 관련 기술 문서 URL 작성.



#### Rule Lifecycle: 테스트에서 배포까지의 상태 관리

* experimental, test:\
  경보는 울리지 않고, 백그라운드에서 조용히 로그 매칭 결과만 수집하겠다.
* stable:\
  충분한 테스트 후, 오탐이 없는 것이 검증되면, stable 상태로 승격시켜 24시간 실시간 모니터링 알람에 투입.
* deprecated:\
  애플리케이션의 버전업 등 IT환경의 변화로 더 이상 유효하지 않게 된 규칙. 운영에서 배제.



### 2.2 Logsource: 탐지의 출발점 설정하기

메타데이터가 룰의 신분증이면, Logsource 블록은 탐지 룰이 IT 인프라 시스템 중에서 정확히 "어느 위치에 설치된 어떤 종류의 CCTV(Log)를 볼 것인가"를 지정합니다.

#### Category, Product, Service, Definition 필드 완벽 이해

* Category:\
  수집하려는 로그의 본질적인 행위 종류. eg. process\_creation, network\_connection, dns\_query, file\_event, ... 등
* Product: \
  운영체제나 제품군. eg windows, linux, macos, aws, m365, zeek, apache,... 등
* Service:\
  Product안에서 구체적으로 어떤 특정 채널이나 서브 시스템에서 발생하는 로그를 볼 것인가.\
  Product안에서 security, system, sysman,... 등
* Definition(선택사항):\
  추가적인 정보를 입력해야할때, 사람이 읽을 수 있는 텍스트로 적어두는 필드.



#### "어떤 로그를 볼 것인가?"에 따른 룰의 유효성 판단

완벽하게 작성된 Detection 조건식도, 적절한 로그 소스를 잘못 선택하면 의미가 없어집니다.\
잘못된 logsource를 지정하게 되면 결국 전혀 쓸모 없는 Dead Rule이 되어버립니다.

따라서 자신이 잡고자 하는 TTP가 발생할 때, IT 시스템의 어느 장비가 어떤 형태의 로그를 남기는지 가시성(Visibility)의 본질을 꿰뚫고 있어야 정확한 탐지룰을 설계할 수 있습니다.



#### Windows Sysmon vs Native Event Log의 로그 소스 차이점

* Native Event Log (service: security):\
  윈도우 운영체제에 내장되어있는 순정 보안 로그.\
  Event ID가 남지만, 깊이 있는 정보는 제공해주지 않습니다.
* Sysmon (service: sysmon):\
  마이크로소프트에서 무료 제공하지만 관리자가 별도로 설치해야하는 강력한 시스템 모니터링 도구.\
  EDR 솔루션에 비빌만큼 깊고 세밀한 커널 수준의 텔레메트리 제공.\
  프로세스가 생성될때(Event) SHA256 해시값을 기본적으로 기록해주고, 해커가 악성코드를 주입하는 고도화된 행위나 프로세스별 네트워크 연결 기록까지 모조리 기록합니다.

### 2.3 Detection & Condition: 탐지 엔진의 심장

#### Selection과 Filter의 조합을 통한 정교한 타격

* Selection(선택):\
  찾고자 하는 악성 의심 로그의 특징을 정의하는 그물망 역할
* Filter:\
  Selection만으로는 오탐 알림이 폭주하게 됨으로, 완벽하게 보완하기 위해 필터 변수를 추가로 정의.

#### 매칭 연산자 딥다이브: Contains, StartsWith, Regex

기본적으로는 대소문자를 구분하지 않는 정확한 일치 방식을 사용하지만, 해커의 기법들을 유연하게 잡아내기 위해 파이프( | ) 기호를 활용한 확장 연산자들을 제공합니다.

* |contains:\
  핵심 키워드가 포함되어 있는가
* |startswith / |endswith:\
  파일의 실행 경로를 검사할때 쓰이고, 경로의 시작점 또는 파일 확장자가 특정 문자로 끝나는지 검사합니다.
* |re:\
  정규표현식을 포함하여 더욱 고도화된 형태의 패턴 매칭을 수행할 수 있습니다.



#### \[핵심] Value Modifiers 활용법 (Base64Offset, Windash 등)

해커들은 백신을 속이기 위해 윈도우 환경에서 명령어를 Base64로 인코딩하여 숨기곤 합니다. Base64의 수학적 특성상 앞에 오는 문자열 길이에 따라 동일한 원본 명령어라도 인코딩 결과값이 3가지 전혀 다른 문자열 패턴으로 변형되는 문제가 발생합니다.

탐지룰에 |base64offset 변형자를 달아주면, Sigma Compiler 엔진이 알아서 3가지의 변형된 Base64결과값을 자동으로 게산하여 검색 쿼리에 덧붙여줍니다.

해커들은 윈도우 커멘드라인은 옵션을 줄 때 슬래시( / )와 하이픈( - )을 섞어쓰기도 하는데 |windash 변형자를 적용하면 /enc와 -enc를 따로 적어주지 않아도 시스템이 두 가지 기호를 동일한 것으로 위급하여 우회 시도를 차단합니다.

{% hint style="info" %}
Value Modifier를 기본값으로 두면 모든 우회 시도를 잡을 수 있지 않나?

기본값으로 하지 않는 이유:

1. False Positive
2. SIEM 검색 엔진의 성능 과부하
3. 맥락과 문법적 타당성 훼손:\
   Image: 'C:\Windows\Temp\malware.exe'를 찾고 싶은데 windash로 인해 ( \\, /, -)이 있는 버전들로 만들어져 버려서, 애초에 존재하지 않는 조건이 됩니다.
{% endhint %}

#### Condition 문법: 1 of them, selection and not filter 등 논리 구조 짜기

Selection과 Filter 변수들은 최종적으로 condition의 필드를 통해 완벽한 논리 구조로 결합됩니다.\
기본적으로 and, or, not 같은 논리 연산자와 묶음 괄호() 를 사용하여 수학 공식처럼 탐지 조건을 한줄로 만듭니다.

예:

* (selection1 or selection2) and not filter\_admin
* 1 of selectrion\_\*: selection으로 시작하는 변수들 중 1개
* all of them



## Part 3. 실전! 공격 행위를 규칙으로 번역하기

### 3.1 공격자 사고방식으로 탐지 룰 설계하기

#### CTI(위협 인텔리전스) 보고서에서 탐지 포인트 추출하기

Cyber Threat Intelligence는 글로벌 보안 업체들이 최신 해킹 그룹의 침해 사고를 분석하여, 그들의 범행 수법을 기록해 둔 '사건 기록부'입니다.

단편적인 IOC를 복사하여 방화벽에 적용하는 수준이 아닌, TTP를 바탕으로 "어떤걸 어떤식으로 조합하여 악용했는가?"를 바탕으로 탐지 포인트를 추출합니다.

예: "해커가 PowerShell을 이용해 Pastebin에서 악성 페이로드를 다운로드 했다"

* 파워셸 프로세스 실행(image)
* 외부 네트워크 연결(Network)
* 특정 url 패턴(CommandLine)

세 가지 교집합을 통해 Sigma의 Selection 조건으로 설계할 수 있습니다.

#### "정상 행위와 무엇이 다른가?" - 제외 조건(Filtering) 설정의 기술

최근 해커들은 탐지 솔루션을 회피하기 위해 외부에서 악성코드를 가져오지 않고, 시스템에 깔려 있는 정상 관리자 도구(cmd powershell, psexec 등)를 무기로 돌변시켜 '기생형 공격(Living off the Land, LotL)' 기법을 사용합니다.

무작정 룰을 지정하게 됐을때는 오탐이 발생할 수 있고, 구분이 어려울 수 있기 때문에, 해당 회사의 정상적인 업무 패턴 등 과거 로그 데이터 분석을 통해 확실히 알고 있어야합니다.

#### 추상적인 공격 기법을 구체적인 로그 패턴으로 치환하는 법

보고서에서 읽은 해킹 개념을 SIEM/EDR 엔진이 읽을 수 있는 로그의 언어로 번역을 해야합니다.

예를 들어, MITRE ATT\&CK 프레임워크에 "해커가 관리자 비밀번호를 훔치기 위해 OS 크리덴셜 덤프(Credential Dumping) 기법을 사용했다" 라고 했을때,\
검색엔진은 크리덴셜 덤프를 이해하지 못하기에, 이게 어떤 이벤트로 발현되는지 알고 있어야 합니다.

방어자는 이 기법이 실제로 "어떤 미상의 프로세스가 윈도우의 핵심 인증 프로세스인 lsass.exe의 메모리 영역에 강제로 접근하여 읽기 권한을 요청하는 행위" 라는 것을 알고 있어야합니다.

### 3.2 주요 Use Case별 룰 작성 실습

#### Case1: LOLBins(rundll32, mshta) 등 기본 도구 오남용 탐지

해커들이 기생형 공격(Living off the Land) 방식을 사용할때 악용되는 윈도우 내장 도구들을 LOLBins(Living off the Land Binaries)라고 하며, 대표적으로 자바스크립트나 VBScript를 실행할 수 있는 mshta.exe와 DLL을 강제로 실행시켜주는 rundl132.exe가 있습니다.

정상적인 디지털 서명이 있는 신뢰할 수 있는 시스템 파일이므로, 이 파일 자체를 차단(YARA 룰 등)하는 것은 시스템을 마비시킵니다.

그래서, 이 파일들이 실행되고 뒤따라오는 행동들을 파악하여 '비정상적인 커멘드라인 인자를 추적하는 방식으로 룰을 설계해야합니다.

#### Case2: 권한 상승 및 방어 회피(Defense Evasion) 흔적 찾기

해커가 시스템 침투후 가장 먼저 일반 사용자 계정의 권한을 최고 관리자로 끌어올리는 '권한 상승'과 자신을 추적하는 보안 솔루션의 눈과 귀를 멀게하는 '방어 회피'입니다.

이 단계에서는 해커가 윈도우의 기본 보안 시스템이나 로그 기록 장치를 강제로 끄거나 지우려고하고, 대표적인 방어 회피 시그널은 webtutil.exe를 사용하여 윈도우 이벤트 로그 삭제를 시행하는 것입니다. \
또한, 윈도우 디펜터의 실시간 감시 기능을 완전히 무력화하기 위해 레지스트리 값을 강제로 변조하거나, Set-MpPreference -DisableRealtimeMonitoring $true 명령어를 내리는 것도 좋은 탐지 포인트입니다.
