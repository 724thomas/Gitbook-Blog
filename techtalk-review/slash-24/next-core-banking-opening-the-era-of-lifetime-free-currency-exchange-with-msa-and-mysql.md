---
description: Next 코어뱅킹, MSA와 MySQL로 여는 평생 무료 환전 시대
---

# Next Core Banking: Opening the Era of Lifetime Free Currency Exchange with MSA and MySQL

## 💱 토스뱅크가 만든 "24시간 환전"의 비밀: MSA와 MySQL 기반 코어뱅킹 아키텍처 혁신기

***

### 📌 개요

토스뱅크는 "24시간 365일 100% 우대 환율 환전"이라는 혁신적인 서비스를 선보이며, 기존 은행의 복잡하고 폐쇄적인 환전 시스템에 도전장을 내밀었습니다. 이 글에서는 해당 서비스의 아키텍처 설계 배경부터, 기술적 도전, 장애 대응 전략, 테스트 자동화, 그리고 운영 성과까지 토스뱅크 실무 개발팀의 관점에서 심층 분석합니다.

***

### 📦 기존 은행 시스템 아키텍처: 왜 문제였나?

#### 🧱 모놀리식 구조의 한계

[![Image](https://private-user-images.githubusercontent.com/55742497/489562256-df8dc4ce-7dfb-41ac-9b3c-efc6b668a782.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjIyNTYtZGY4ZGM0Y2UtN2RmYi00MWFjLTliM2MtZWZjNmI2NjhhNzgyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTFjYjc0YWFhZDM0MDhjYTU5NTJiYzA3NGJiNDM0Y2IxYTk1NjI1YWZkNDgyMmZiNDc5Njg0NDU5MmIyYjA3OWEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.ly6qoPW6wx5dEOpxkU-YsNH9yVncgnu7jjydd9qSGoE)](https://private-user-images.githubusercontent.com/55742497/489562256-df8dc4ce-7dfb-41ac-9b3c-efc6b668a782.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjIyNTYtZGY4ZGM0Y2UtN2RmYi00MWFjLTliM2MtZWZjNmI2NjhhNzgyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTFjYjc0YWFhZDM0MDhjYTU5NTJiYzA3NGJiNDM0Y2IxYTk1NjI1YWZkNDgyMmZiNDc5Njg0NDU5MmIyYjA3OWEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.ly6qoPW6wx5dEOpxkU-YsNH9yVncgnu7jjydd9qSGoE)

기존 은행 시스템은 채널계(고객 트래픽 수신)와 계정계(비즈니스 로직 처리)로 구분되며, 대부분 거대한 단일 서버(Monolithic) 아키텍처로 구성되어 있었습니다.

* **장점**: 트랜잭션 처리 일관성 확보
* **단점**: 단일 장애 지점(SPOF), 서비스 확장성 부족

#### ☁️ 토스 초기 코어뱅킹 구조

* Kafka, Redis 등의 모던 스택을 도입했으나, 코어뱅킹 자체는 여전히 모놀리식
* 하나의 서비스 장애가 전체 서비스에 영향을 주는 구조

***

### 🔄 기술 전환의 필요성: 왜 MSA + MySQL인가?

#### 💥 기존 시스템의 병목 사례: 지금이자받기 서비스

[![Image](https://private-user-images.githubusercontent.com/55742497/489562601-cc021852-f206-4cde-99e9-d8a858622bf9.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjI2MDEtY2MwMjE4NTItZjIwNi00Y2RlLTk5ZTktZDhhODU4NjIyYmY5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWNiMDE4NmEyMjdjMDE4M2UzNzhmMTBlNzJmN2Q4MzJkYjc4YTA4ZDY2ZjVmZGI4ZTY2Y2NkYjQ2ZTg1Zjk3YTYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.9NEp1YfeTzIyIQDJz5VNOKdtZMFJ7_NNB0twNpbBpcg)](https://private-user-images.githubusercontent.com/55742497/489562601-cc021852-f206-4cde-99e9-d8a858622bf9.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjI2MDEtY2MwMjE4NTItZjIwNi00Y2RlLTk5ZTktZDhhODU4NjIyYmY5LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWNiMDE4NmEyMjdjMDE4M2UzNzhmMTBlNzJmN2Q4MzJkYjc4YTA4ZDY2ZjVmZGI4ZTY2Y2NkYjQ2ZTg1Zjk3YTYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.9NEp1YfeTzIyIQDJz5VNOKdtZMFJ7_NNB0twNpbBpcg)

* MSA 구조에서 각각의 마이크로서비스가 Oracle DB를 공유
* 특정 서비스의 폭주로 인해 관련 없는 서버들까지 영향을 받음
* Oracle의 CPU 코어 단위 과금 구조로 인해 독립 DB 구성이 어려움

#### 🔍 오라클의 장단점 분석

* **장점**: 신뢰성, 복구성, 트랜잭션 처리, 대규모 데이터 처리 성능
* **단점**: 단일 장애 지점, 고비용, 스케일아웃 제약

#### 🧪 대안 탐색

* **트랜잭션 관리**: Redis 분산락, Kafka Saga Pattern 등 Application Layer 중심
* **확장성**: MySQL Replication 기반 Scale-out
* **안정성**: MVCC 기반 동시성 제어 가능
* **커뮤니티와 운영 노하우**: MySQL도 성숙 단계에 도달

***

### 🧱 새로운 외화예금 아키텍처: 완전한 MSA 기반

#### 🔁 기존 원화예금 아키텍처

* 개설, 입금, 출금 등 모든 업무가 하나의 코어뱅킹 서버에 집중

#### 🔧 새롭게 설계된 외화예금 아키텍처

* 업무 단위로 쪼개진 MSA 구조
* **기술 스택**: Kotlin, Spring, MySQL, Redis, Kafka
* **전면 재설계**: 데이터 모델링, API, 인프라 전부 재구축

[![Image](https://private-user-images.githubusercontent.com/55742497/489562902-20bc46b1-b24c-48f2-8a69-eca7aa15bb00.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjI5MDItMjBiYzQ2YjEtYjI0Yy00OGYyLThhNjktZWNhN2FhMTViYjAwLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTJhYTgzN2Y1MWY2NDgxZWFmYzk0MWQzNDdlOTRlNDk1NmEwNGZhZTlhYmEwMTRhZTBiZWVkMzRiOWU3ZDBmMzEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.YPTDJqUPjp1G4jzcGhmTH3EH2iXcBj4raWQvi_s9z64)](https://private-user-images.githubusercontent.com/55742497/489562902-20bc46b1-b24c-48f2-8a69-eca7aa15bb00.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjI5MDItMjBiYzQ2YjEtYjI0Yy00OGYyLThhNjktZWNhN2FhMTViYjAwLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTJhYTgzN2Y1MWY2NDgxZWFmYzk0MWQzNDdlOTRlNDk1NmEwNGZhZTlhYmEwMTRhZTBiZWVkMzRiOWU3ZDBmMzEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.YPTDJqUPjp1G4jzcGhmTH3EH2iXcBj4raWQvi_s9z64)

***

### ⚙️ 트랜잭션 아키텍처 설계 전략

#### 💡 기본 전략: 느슨한 연결 + 핵심은 엄격

* **고객 잔액**: 동기 트랜잭션, 분산락 사용
* **내부 회계 처리**: Kafka 기반 비동기 트랜잭션

#### 🔁 환전 프로세스 트랜잭션 흐름

1. 고객이 환전 요청
2. 외환 서버: 환율 확정 → 원화 출금 요청
3. 원화 서버: 출금 수행
4. 외화 서버: 입금 수행 → 고객 잔액 반영 완료

[![Image](https://private-user-images.githubusercontent.com/55742497/489563144-a37ac96a-eec9-4432-9736-21a8ba2c9532.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjMxNDQtYTM3YWM5NmEtZWVjOS00NDMyLTk3MzYtMjFhOGJhMmM5NTMyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTcyYTJkMmU0MTA4MTUzMWYxZDVlMDhmMDM5MjlhNTgzMjY2ZTk2ZDcwNWRmMjk3YWE4OTk3N2RlZjk2NjQwOGUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.VxBM1ARNvib07EeTQfGdDfBZGB_A6hl0rJADp7Va-KI)](https://private-user-images.githubusercontent.com/55742497/489563144-a37ac96a-eec9-4432-9736-21a8ba2c9532.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjMxNDQtYTM3YWM5NmEtZWVjOS00NDMyLTk3MzYtMjFhOGJhMmM5NTMyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTcyYTJkMmU0MTA4MTUzMWYxZDVlMDhmMDM5MjlhNTgzMjY2ZTk2ZDcwNWRmMjk3YWE4OTk3N2RlZjk2NjQwOGUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.VxBM1ARNvib07EeTQfGdDfBZGB_A6hl0rJADp7Va-KI)

1. 이후 회계 서버에서 Kafka 통해 은행 계좌 업데이트

[![Image](https://private-user-images.githubusercontent.com/55742497/489563367-766907dc-4553-48ac-909e-c67fc0b77e4a.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjMzNjctNzY2OTA3ZGMtNDU1My00OGFjLTkwOWUtYzY3ZmMwYjc3ZTRhLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWZlYmRlMDJiYmRkNWQ5NDY4NWM5MWM5MDYxOGM0MGQwMjYzMzQzYzRiNWVhZWRlNTIwMmMxMjUwM2I0YmFhNzMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.vUXEPL0IhwIOyLuyKAAjOpxvP7ofiBLApvg_WLqnVgE)](https://private-user-images.githubusercontent.com/55742497/489563367-766907dc-4553-48ac-909e-c67fc0b77e4a.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjMzNjctNzY2OTA3ZGMtNDU1My00OGFjLTkwOWUtYzY3ZmMwYjc3ZTRhLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWZlYmRlMDJiYmRkNWQ5NDY4NWM5MWM5MDYxOGM0MGQwMjYzMzQzYzRiNWVhZWRlNTIwMmMxMjUwM2I0YmFhNzMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.vUXEPL0IhwIOyLuyKAAjOpxvP7ofiBLApvg_WLqnVgE)

***

### 🔐 동시성 제어 및 계좌 검증

#### 🔒 동시성 제어

* Redis 분산락 → 빠른 Lock 획득으로 Fail-fast 유도
* DB 레벨에서 비관적 Lock 병행 적용

#### ✅ 거래 가능 계좌 검증

* 해지 여부, 사고 계좌, 사망자 계좌 등 10여 가지 항목 동시 검증
* **기술 포인트**: 코루틴 활용하여 레이턴시 최소화

[![Image](https://private-user-images.githubusercontent.com/55742497/489563572-c2369c81-cf10-4470-b089-e63df0a3d033.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjM1NzItYzIzNjljODEtY2YxMC00NDcwLWIwODktZTYzZGYwYTNkMDMzLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTdiNDViYjVkODY1N2UwNGE1MmQxYzFlODMyN2QyMjcxY2RjMjFhODk3OWVhOTBjY2FhNjRlNGQ5OTdiMjAzMDcmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.TDrOE8_LKwVpRIK_wO7uCDjiwwLUSE_UkXQDMakBaOQ)](https://private-user-images.githubusercontent.com/55742497/489563572-c2369c81-cf10-4470-b089-e63df0a3d033.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5NDU2NTMsIm5iZiI6MTc1Nzk0NTM1MywicGF0aCI6Ii81NTc0MjQ5Ny80ODk1NjM1NzItYzIzNjljODEtY2YxMC00NDcwLWIwODktZTYzZGYwYTNkMDMzLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA5MTUlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwOTE1VDE0MDkxM1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTdiNDViYjVkODY1N2UwNGE1MmQxYzFlODMyN2QyMjcxY2RjMjFhODk3OWVhOTBjY2FhNjRlNGQ5OTdiMjAzMDcmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.TDrOE8_LKwVpRIK_wO7uCDjiwwLUSE_UkXQDMakBaOQ)

***

### 🌙 무중단 서비스: 잔액 대사 전략

#### ⏰ 일반 은행 방식

* 자정\~새벽 거래 차단 → 정확한 잔액 산출
* 단점: 고객 경험 저하

#### 🧠 토스의 2가지 전략

#### 📅 방식 1: 보정 기반

* 전일자 스냅샷 테이블 + 이후 거래 내역 보정

#### 🧾 방식 2: 구조 개선 기반

* 일자별/계좌별 잔액 테이블 운영
* 스냅샷 시점의 잔액을 하둡 이관된 데이터로 계산

→ 고객은 자정에도 결제/환전 가능

***

### 🧪 테스트 자동화 전략

#### 🤖 테스트 3단계

1. **단위 테스트**: 단 하나의 케이스 실패 시 라이브 반영 불가
2. **E2E 시뮬레이션 서버**: 실제 API 시나리오 자동 호출
3. **운영 이상 탐지 서비스**: 해지 계좌 입금, 중복 이자 등 이상 트랜잭션 탐지

→ 5명 개발자가 6개월 만에 안정적인 외화예금 시스템 구축 가능했던 비결

***

### 🚀 성과 요약

#### 🥇 성과 1: 금융권 최초 MSA + MySQL 기반 코어뱅킹

* 3,000 TPS 이상을 실시간 처리 중
* 안정적인 운영 입증 (Grafana 메트릭 기반)

#### ⚡ 성과 2: 세계에서 가장 빠른 환전

* 평균 응답 시간: 30ms (원화 송금 280ms보다 9배 이상 빠름)

#### 🌙 성과 3: 무중단, 24시간 365일 환전 가능

* 데이터 아키텍처 구조적 혁신 기반

***

### 💡 교훈 및 인사이트

* **기술적 결단**: Oracle이라는 업계 표준을 넘어서기 위한 과감한 기술 전환
* **설계 원칙**: 트랜잭션은 반드시 느슨하게, 핵심은 엄격하게
* **검증 철학**: 무조건 자동화 → 품질과 속도 모두 확보
* **서비스 철학**: 고객 경험을 중심으로 아키텍처를 디자인할 수 있다

***

### 📚 주요 기술 키워드 정리

| 키워드          | 설명                                                         |
| ------------ | ---------------------------------------------------------- |
| MSA          | Microservice Architecture. 모놀리식 구조 대비 유연한 배포와 독립적인 스케일링 가능 |
| Saga Pattern | 비동기 분산 트랜잭션 처리 방식. Kafka 기반으로 구현                           |
| Redis 분산락    | 여러 서버 간 동시성 제어를 위한 빠른 락 처리 방식                              |
| MVCC         | 다중 버전 동시성 제어. MySQL/InnoDB의 기본 동작 방식                       |
| 잔액 대사        | 고객 계좌와 은행 계좌의 금액이 일치하는지 확인하는 프로세스                          |
| 코루틴          | Kotlin에서 제공하는 경량 스레드 처리 방식. 레이턴시 최소화에 효과적                  |
| E2E 테스트 서버   | 고객 시나리오를 코드로 정의하고 실제 서버에 자동 테스트 요청                         |

***

### 🧭 마무리

이번 토스뱅크 외화예금 프로젝트는 단순한 기능 개발을 넘어, **금융 시스템 아키텍처의 패러다임 전환**을 실현한 사례였습니다. 이를 통해 실무적으로 얻은 인사이트는 아래와 같습니다:

* 더 이상 오라클은 정답이 아니다
* 트랜잭션은 DB가 아닌 어플리케이션 레이어에서도 충분히 관리할 수 있다
* 테스트는 개발보다 더 중요한 단계일 수 있다
* 고객 경험이 아키텍처의 출발점이 될 수 있다



## 학습한 것

* **하둡(Hadoop)**
  * 대규모 데이터를 **분산 저장 + 병렬 처리**할 수 있는 시스템.
  * 금융 시스템에서는 운영 DB(MySQL) 부하를 줄이기 위해 **잔액 대사, 이상거래 탐지, 리포트** 같은 집계/검증 작업에 활용.
  * 즉, **MySQL = 실시간 트랜잭션**, **Hadoop = 무거운 집계/검증**으로 역할 분리.
  * 자정(24시)에도 거래를 멈추지 않고 **스냅샷 기반 검증**을 가능하게 함.
* **락(Concurrency Control)**
  * 토스뱅크는 **Redis 분산락 + DB 비관적 락**을 조합.
  * 비관적 락을 쓴 이유: 금융거래는 **충돌 발생 시 롤백(낙관적 락 방식)** 자체가 리스크 → 안전성 최우선.
  * Redis를 활용해 빠른 Fail-fast 처리 + DB에서 최종 보호.
* **테스트 자동화 전략**
  * 과거: Excel 기반 수기 검증 + 케이스별 테스트 코드 반복 작성 → 비효율.
  * 현재:
    * (1) 단위 테스트 (Unit test)
    * (2) **E2E 시뮬레이션 서버** – 실제 API 시나리오 자동 호출
    * (3) 운영 환경 **이상거래 탐지 서비스**
  * 이 3단계 자동화로, 소규모 팀(5명)도 6개월 만에 안정적 시스템 오픈 가능.
* **E2E 서버**
  * 단위 테스트와 달리, **현실 세계 고객 요청을 그대로 재현**하는 테스트 서버.
  * 실제 API를 호출해 전체 시나리오(환전, 송금 등)를 자동으로 검증.
  * 코드 수정 시 자동 실행 → 반복 테스트 코드 작성 불필요.
  * 과거 수기 테스트/반복 작성의 한계를 극복한 핵심 인프라.
* **24시간 무중단 거래**
  * 전통 은행: 자정에 거래를 멈추고 원장 대사.
  * 토스: **스냅샷/보정/일자별 잔액 테이블** 전략으로 거래 중단 없이 잔액 검증 → 24/365 서비스 가능.



## 후기

먼저, 하둡(Hadoop)은 단순히 빅데이터 저장소가 아니라 **운영 DB의 부하를 분리하고, 잔액 대사·이상거래 탐지 같은 무거운 집계 작업을 처리하는 핵심 인프라**라는 걸 깨달았다. 그래서 MySQL은 실시간 트랜잭션에 집중하고, 하둡은 분석과 검증을 담당하는 구조로 역할이 명확히 나뉜다는 걸 배웠다. 덕분에 은행 시스템에서도 자정에 거래를 멈추지 않고 스냅샷 기반 검증으로 24시간 무중단 서비스를 구현할 수 있다는 점이 인상 깊었다.

또한 동시성 제어에서는 왜 낙관적 락이 아니라 비관적 락을 선택했는지 이해하게 되었다. 금융 거래는 충돌이 발생했을 때 단순히 롤백하는 것만으로는 위험이 크기 때문에, **안전성을 최우선으로 하는 비관적 락**이 적합했다. 여기에 Redis 분산락을 함께 사용해 빠르게 실패를 감지(Fail-fast)하고, DB 비관적 락을 최종 방어선으로 두는 전략이 매우 현실적이라는 점을 배웠다.

테스트 자동화 부분도 큰 깨달음이었다. 과거에는 수많은 케이스를 Excel과 테스트 코드로 반복 검증했지만, 이제는 **단위 테스트 → E2E 시뮬레이션 서버 → 운영 환경 이상거래 탐지**라는 3단계 자동화 체계를 갖췄다는 점이다. 특히 E2E 서버를 통해 실제 고객이 요청할 시나리오를 자동으로 실행하고 검증하는 방식은, 반복적인 테스트 코드 작성의 한계를 극복한 좋은 사례라는 걸 알게 되었다.

마지막으로, 전통적인 은행이 자정에 시스템을 멈추는 이유와 토스가 이를 어떻게 극복했는지를 보며, **데이터 관리 전략 하나가 고객 경험을 180도 바꿀 수 있다**는 걸 다시금 깨달았다. 스냅샷, 보정, 일자별 잔액 테이블 같은 접근이 단순 기술적 선택이 아니라, 24/365 무중단 금융 서비스라는 결과로 이어졌다는 점이 큰 인사이트였다.
