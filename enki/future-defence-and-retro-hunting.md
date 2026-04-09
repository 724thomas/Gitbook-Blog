# Future Defence and Retro Hunting

## 실시간 탐지의 과정

### 1. \[생산자의 배포]

KISA 등의 기관이 위협 인텔리전스를 담은 Report(SDO) 객체를 만들고, 그 안에 Malware(SDO), Infrastructure(SDO), 그리고 이를 탐지할 Indicator(SDO), 데이터 유출을 막기 위한 Marking Definition 꼬리표를 달고 Bundle에 포장하여 TAXII 서버에 올립니다.

{% content-ref url="until-stix-report-is-built.md" %}
[until-stix-report-is-built.md](until-stix-report-is-built.md)
{% endcontent-ref %}

{% hint style="info" %}
\[현재 STIX 그래프 상태]&#x20;

`[Node 1: SDO]` Threat Actor - name: "알 수 없는 해커 조직" \
`[Node 2: SDO]` Location - region: "eastern-europe" (동유럽) \
`[Node 3: SDO]` Infrastructure - name: "메인 C2 서버" \
`[Node 4: SDO]` Identity - name: "금융권 (타겟)" \
`[Node 5: SDO]` Vulnerability - name: "방화벽 장비 결함", external\_references: \[{"CVE-2024-0001"}] \
`[Node 6: SDO]` Malware - name: "초기 침투용 악성코드" \
`[Node 7: SCO]` IPv4-Addr - value: "198.51.100.1" (메인 C2 IP) \
`[Node 8: SDO]` Indicator - name: "메인 C2 탐지 룰", pattern: "\[ipv4-addr:value = '198.51.100.1']" \
`[Node 9: SDO]` Report - name: "KISA 위협 동향 보고서" (Node 1\~8 포함) \
`[Node 10: Meta]` Marking-Definition - name: "TLP:AMBER"

\[SRO (Edges)] \
`[Edge A]` Node 1 (Threat Actor) --(owns)--> Node 3 (Infrastructure) \
`[Edge B]` Node 3 (Infrastructure) --(located-at)--> Node 2 (Location) \
`[Edge C]` Node 1 (Threat Actor) --(targets)--> Node 4 (Identity: 금융권) \
`[Edge D]` Node 1 (Threat Actor) --(targets)--> Node 5 (Vulnerability) \
`[Edge E]` Node 1 (Threat Actor) --(uses)--> Node 6 (Malware) \
`[Edge F]` Node 6 (Malware) --(uses)--> Node 3 (Infrastructure) \
`[Edge G]` Node 8 (Indicator) --(indicates)--> Node 6 (Malware) \
`[Edge H]` Node 3 (Infrastructure) --(consists-of)--> Node 7 (IPv4-Addr: 메인 C2 IP)
{% endhint %}

### 2. \[방어자의 수신]

우리 회사의 CTI플랫폼(Threadt Intelligence Platform. TIP)은 해당 TAXII를 구독하며 주기적으로 폴링하다가, 업데이트된 Bundle을 다운로드합니다.

{% hint style="info" %}
신규 SDO 생성

`[Node 11: SDO]` Identity - name: "우리 회사 (방어자/수신자)"
{% endhint %}



### 3. \[무기화 및 배포]

CTI 플랫폼은 Bundle 안의 Indicator(SDO) 객체를 열고, 그 안의 pattern 속성을 바탕으로 EDR용 Sigma Rule을 자동 생성하여 사내 장비에 배포합니다.



### 4. \[탐지 및 트리아지]

EDR이 텔레메트리(로그)를 수집하여 알람을 띄웁니다. SOC 분석가가 Tirage(오탐/정탐 검증)을 수행합니다.



### 5. \[정탐 확정 및 대응]

해커의 악정 행위 기록은 Observed Data(SDO)로 생성되며, 내부에는 악성 IPv4-Addr(SCO), 통신 기록인 Network-Traffic(SCO)가 포함됩니다.\
EDR을 통해 네트워크를 격리하고, 이 조치 내역을 Course of Action(SDO)으로 만든 뒤, mitigate(SRO) 화살표를 긋습니다. \[Course of Action] --(mitigates)--> \[Indicator, Malware]

{% hint style="info" %}
\[새로운 STIX 객체]\
`[Node 12: SCO]` Network-Traffic - dst\_port: 443 _(신규)_ \
`[Node 13: SDO]` Observed-Data - object\_refs: \[Node 7(C2 IP), Node 12(Traffic)] _(신규: 탐지된 로그)_ \
`[Node 14: SDO]` Course-of-Action - name: "감염 PC 네트워크 격리 조치" _(신규)_

\[새로운 SRO 객체 (Edges)] \
`[Edge I]` Node 14 (Course-of-Action) --(mitigates)--> Node 6 (Malware) _(신규)_
{% endhint %}



### 6. \[목격 보고 - Feedback]

탐지에 성공했으며, KISA가 준 원본 Indicator를 참조하는 Sighting(SRO) 객체를 생성합니다.\
Sighting안에는 EDR에서 만들어진 Observed Data(SDO)와 회사의 Identity(SDO)를 연결합니다.\
해당 피드백 세트를 다시 Bundle로 묶어 TAXII 서버로 올립니다.

{% hint style="info" %}
\[새로운 STIX 객체]\
`[Node 15: SRO]` Sighting - count: 1 _(신규: 목격 보고서)_

\[새로운 SRO 객체 (Edges)] \
`[Edge J]` Node 15 (Sighting) --(sighting\_of)--> Node 8 (원본 Indicator) _(신규)_ \
`[Edge K]` Node 15 (Sighting) --(where\_sighted)--> Node 11 (우리 회사 Identity) _(신규)_ \
`[Edge L]` Node 15 (Sighting) --(observed\_data)--> Node 13 (탐지 로그 Observed-Data) _(신규)_
{% endhint %}



### 7. \[새로운 단서 발견]

격리된 PC의 로그를 정밀 분석하던 중, 해커가 예비용으로 숨겨둔 새로운 C2 IP와 연결하려는 정황을 포착합니다.

{% hint style="info" %}
\[새로운 STIX 객체]\
`[Node 16: SCO]` IPv4-Addr - value: "203.0.113.5" (예비용 C2 IP) _(신규)_ \
`[Node 17: SDO]` Observed-Data - object\_refs: \[Node 16] _(신규: 예비용 C2 연결 시도 로그)_
{% endhint %}



### 8. \[새로운 지표 생성 및 공유]

우리 CTI 플랫폼은 새 단서를 바탕으로 새로운 STIX 객체들을 만듭니다.

IPv4-Addr(SCO):  새로운 IP 객체\
Indicator(SDO) 생성: 이 IP를 탐지하기 위한 새로운 룰\
indicates(SRO) 생성: 이 룰이 그 악성코드를 가리킨다. \[New Indicator] --(indicates)--> \[원본 Malware]

이 새로운 객체들을 Bundle로 포장하여 TAXII 업로드 합니다.

{% hint style="info" %}
\[새로운 STIX 객체]\
`[Node 18: SDO]` Indicator - name: "예비용 C2 탐지 룰", created\_by\_ref: "Node 11 (우리 회사)" _(신규)_

\[새로운 SRO 객체 (Edges)] \
`[Edge M]` Node 18 (새로운 Indicator) --(indicates)--> Node 6 (원본 Malware) _(신규: 새 룰이 기존 악성코드를 잡음)_ \
`[Edge N]` Node 3 (원본 Infrastructure) --(consists-of)--> Node 16 (예비용 C2 IP) _(신규: KISA가 알려준 인프라가 사실 이 IP도 포함하고 있었음을 우리가 밝혀냄!)_
{% endhint %}



## 레트로 헌팅(소급 탐지)의 과정

### 1. \[생산자의 배포]&#x20;

KISA 등의 기관이 위협 인텔리전스를 담은 `Report`(SDO) 객체를 만들고, 그 안에 `Malware`(SDO), `Infrastructure`(SDO), 그리고 이를 탐지할 `Indicator`(SDO), 데이터 유출을 막기 위한 `Marking Definition` 꼬리표를 달고 `Bundle`에 포장하여 TAXII 서버에 올립니다.



### 2. \[방어자의 수신]&#x20;

우리 회사의 CTI 플랫폼(Threat Intelligence Platform, TIP)은 해당 TAXII를 구독하며 주기적으로 폴링하다가, 업데이트된 `Bundle`을 다운로드합니다.



### 3. \[무기화 및 과거 로그 스캔 (Retro-Scan)]&#x20;

CTI 플랫폼은 `Bundle` 안의 `Indicator`(SDO) 객체를 열고, 그 안의 `pattern` 속성을 바탕으로 EDR의 '과거 로그 검색용' 쿼리 시그마 룰을 자동 생성하여 사내 로그 장부에 스캔 명령을 내립니다.



### 4. \[과거 위협 탐지 및 트리아지]&#x20;

EDR 장비가 "3주 전(20일 전) 인사팀 PC에서 이 지표와 일치하는 통신이 있었습니다!" 라는것을 찾아냅니다.\
SOC 분석가가 이 오래된 로그를 보며 Triage(오탐/정탐 검증)를 수행합니다.



### 5. \[침해 사실 확정 및 사고 대응 (IR)]&#x20;

해커가 이미 과거에 침투했음(정탐)을 확정합니다. \
당시의 악성 행위 기록은 `Observed Data`(SDO)로 생성되며, 내부에는 악성 `IPv4-Addr`(SCO), 통신 기록인 `Network-Traffic`(SCO)가 포함됩니다.&#x20;

이미 뚫린 상태이므로 단순 차단을 넘어 즉각적인 침해 사고 대응(Incident Response) 모드로 돌입하여 피해 범위를 조사하고 시스템을 격리/복구합니다.&#x20;

이 조치 내역을 `Course of Action`(SDO)으로 만든 뒤, `mitigates`(SRO) 화살표를 긋습니다. `[Course of Action] --(mitigates)--> [Indicator, Malware]`



### 6. \[과거 목격 보고 - Feedback]&#x20;

탐지에 성공했으며, KISA가 준 원본 `Indicator`를 참조하는 `Sighting`(SRO) 객체를 생성합니다. \
`Sighting` 안에는 EDR에서 찾아낸 `Observed Data`(SDO)와 회사의 `Identity`(SDO)를 연결합니다. \
해당 피드백 세트를 다시 `Bundle`로 묶어 TAXII 서버로 올립니다.



### 7. \[수평적 이동(Lateral Movement) 등 새로운 단서 발견]&#x20;

3주 전 감염되었던 인사팀 PC를 정밀 포렌식(분석) 하던 중, 해커가 그 PC를 징검다리 삼아 사내 '사내 데이터베이스 서버'로 은밀하게 이동(수평적 이동)할 때 사용한 새로운 내부 탈취 도구(스크립트)나 추가 C2 도메인을 포착합니다.



### 8. \[새로운 지표 생성 및 공유]&#x20;

우리 CTI 플랫폼은 이 과거의 흔적에서 찾은 새 단서를 바탕으로 새로운 STIX 객체들을 만듭니다.

* `Domain-Name`(SCO) 또는 `File`(SCO): 새롭게 발견된 악성 도메인이나 해커의 도구 객체
* `Indicator`(SDO) 생성: 이 새로운 단서를 탐지하기 위한 새로운 룰
* `indicates`(SRO) 생성: 이 룰이 그 악성코드를 가리킨다. `[New Indicator] --(indicates)--> [원본 Malware]`
