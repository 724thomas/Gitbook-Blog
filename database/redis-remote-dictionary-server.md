---
description: 레디스를 끝까지 파보자
---

# Redis: Remote Dictionary Server

## 1. Redis란 무엇인가?

메모리 기반 데이터 저장소로, 키값 구조를 기본으로 갖고 있는 NoSQL데이터베이스입니다.



### 1.1 특징

<details>

<summary>NoSql</summary>

* **스키마리스(Schema-less)**
  * 정해진 스키마 없이 데이터를 저장.
  * 데이터 구조가 자유로워, 데이터 모델 변경이 용이.
* **확장성(Scalability)**
  * **수평적 확장(Scale-out)**: 서버를 추가하여 성능을 확장 가능.
  * RDBMS는 주로 수직적 확장(Scale-up)만 가능.
* **고성능**
  * 특정 작업(읽기/쓰기)에 최적화된 구조로 높은 처리 속도를 제공.
  * 인덱스 구조 최적화 및 메모리 중심 설계.
* **유연한 데이터 모델**
  * JSON, Key-Value, Graph, Document 등 다양한 데이터 모델 지원.
* **CAP 이론 적용**
  * NoSQL은 **CAP 이론**에서 "일관성(Consistency)", "가용성(Availability)", "분할 허용성(Partition Tolerance)" 중 일부를 트레이드오프하며 설계됨.



참고 CAP 이론 : [https://wonjoon.gitbook.io/joons-til/books/learning-the-basics-of-large-scale-system-design-through-virtual-interview-cases/6.-key-value-system-design#id-3.1-cap-cap-theorem](https://wonjoon.gitbook.io/joons-til/books/learning-the-basics-of-large-scale-system-design-through-virtual-interview-cases/6.-key-value-system-design#id-3.1-cap-cap-theorem)

</details>

<details>

<summary> 메모리 기반</summary>

## 장점

* 초고속 데이터 접근
  * 메모리 접근 속도: 나노초(ns) 단위
  * 디스크 접근 속도: 밀리초(ms) 단위
* 휘발성
  * RAM은 휘발성이기 때문에 서버가 종료되면 데이터가 사라짐.
  * 데이터 영속성 옵션: RDB(Redis Database), AOF(Append-Only-File)
* 크기
  * RAM 크기에 따라 저장 가능한 데이터의 양이 제한
  * 메모리 관리 전략
    * INFO memory / MEMORY USAGE \<key> : 메모리 사용량 확인 명령어를 활용
    * 데이터 구조를 효율적으로 설계하여 메모리 사용량 최소화.
      * 데이터 구조를 효율적으로 저장
      * 중복 데이터 제거
      * 작은 데이터는 Hash에 저장
      * 데이터 TTL(Time-To-Live) 설정
    * 클러스터 구성으로 확장성 확보

## 단점

* 휘발성 데이터 (보완: RDB, AOF)
* 비용 (디스크 < RAM)

</details>

<details>

<summary>다양한 데이터 구조</summary>

**1. String**

* **특징**:
  * 가장 기본적인 데이터 타입.
  * 값은 문자열, 숫자, JSON 등 어떤 데이터라도 저장 가능.
  * 최대 크기는 512MB.
* **사용 예시**:
  * 캐싱 (예: 사용자 프로필, 설정값 저장).
  * 간단한 카운터 (INCR, DECR 명령어 사용).
  * JSON 데이터를 문자열로 저장.
* **주요 명령어**:
  * `SET key value`: 값을 설정.
  * `GET key`: 값을 조회.
  * `INCR key`: 값을 1 증가.
  * `APPEND key value`: 기존 값에 문자열을 추가.

***

**2. Hash**

* **특징**:
  * 필드-값(Field-Value) 쌍으로 이루어진 해시 테이블을 저장.
  * 하나의 키에 여러 필드-값 쌍을 저장하여 구조적 데이터를 관리.
* **사용 예시**:
  * 사용자 프로필 정보 저장 (예: `username`, `email`, `age` 등).
  * 간단한 객체 저장 (예: 상품의 이름, 가격, 수량 등).
* **주요 명령어**:
  * `HSET key field value`: 특정 필드의 값을 설정.
  * `HGET key field`: 특정 필드의 값을 조회.
  * `HGETALL key`: 모든 필드와 값을 조회.
  * `HDEL key field`: 특정 필드를 삭제.

***

**3. List**

* **특징**:
  * 값의 순서가 유지되는 데이터 구조 (Linked List).
  * FIFO(큐)와 LIFO(스택) 동작 모두 지원.
* **사용 예시**:
  * 작업 대기열 구현 (큐).
  * 최근 본 상품 목록 관리.
  * 채팅 기록 저장.
* **주요 명령어**:
  * `LPUSH key value`: 리스트의 앞에 값 삽입.
  * `RPUSH key value`: 리스트의 뒤에 값 삽입.
  * `LPOP key`: 리스트의 앞에서 값 삭제 및 반환.
  * `RPOP key`: 리스트의 뒤에서 값 삭제 및 반환.
  * `LRANGE key start stop`: 리스트의 특정 범위 값 조회.

***

**4. Set**

* **특징**:
  * 순서가 없는 고유한 값들의 집합.
  * 중복 값이 자동으로 제거됨.
* **사용 예시**:
  * 태그 관리 (중복 없는 값).
  * 소셜 네트워크의 친구 관계 관리.
  * 집합 연산 (교집합, 합집합, 차집합).
* **주요 명령어**:
  * `SADD key value`: 집합에 값 추가.
  * `SMEMBERS key`: 집합의 모든 값 조회.
  * `SREM key value`: 특정 값을 제거.
  * `SINTER key1 key2`: 두 집합의 교집합 반환.
  * `SUNION key1 key2`: 두 집합의 합집합 반환.

***

**5. Sorted Set (ZSet)**

* **특징**:
  * 값마다 점수(score)를 부여하여 정렬된 순서를 유지.
  * 점수에 따라 정렬되며 값은 중복되지 않음.
* **사용 예시**:
  * 순위표 구현 (예: 게임 점수 순위).
  * 추천 시스템에서 콘텐츠의 가중치 기반 정렬.
* **주요 명령어**:
  * `ZADD key score value`: 값을 특정 점수와 함께 추가.
  * `ZRANGE key start stop`: 정렬된 순서대로 범위 내 값 반환.
  * `ZRANGEBYSCORE key min max`: 점수 범위 내 값 반환.
  * `ZREM key value`: 특정 값을 삭제.
  * `ZREVRANGE key start stop`: 정렬된 역순으로 값 반환.

***

**6. Bitmaps**

* **특징**:
  * 비트 단위로 값을 저장하고 조작 가능.
  * 효율적인 메모리 사용.
* **사용 예시**:
  * 사용자 활동 기록 (예: 특정 날짜에 접속했는지 확인).
  * 플래그 값 저장.
* **주요 명령어**:
  * `SETBIT key offset value`: 특정 비트를 설정.
  * `GETBIT key offset`: 특정 비트를 조회.
  * `BITCOUNT key`: 설정된 비트의 수를 반환.

***

**7. HyperLogLog**

* **특징**:
  * 고유 값의 개수를 추정하는 데 사용되는 확률적 데이터 구조.
  * 메모리를 매우 적게 사용하며 정확도는 약간 떨어질 수 있음.
* **사용 예시**:
  * 대규모 데이터 세트에서 중복되지 않은 값의 개수 추정.
  * 예: 웹사이트의 유니크 방문자 수 계산.
* **주요 명령어**:
  * `PFADD key value`: 값을 추가.
  * `PFCOUNT key`: 고유 값의 개수 추정.
  * `PFMERGE key1 key2`: 두 HyperLogLog 병합.

***

**8. Geospatial**

* **특징**:
  * 위치 데이터를 저장하고 조회하는 기능 제공.
  * GPS 좌표 기반 데이터 처리.
* **사용 예시**:
  * 사용자 주변 상점 목록 제공.
  * 물류 시스템에서 가장 가까운 창고 찾기.
* **주요 명령어**:
  * `GEOADD key longitude latitude value`: 위치 데이터를 추가.
  * `GEODIST key value1 value2`: 두 지점 간 거리 계산.
  * `GEORADIUS key longitude latitude radius`: 특정 반경 내 값 조회.

</details>

<details>

<summary>멀티모델 데이터베이스</summary>

* **캐싱**: String, Hash
* **메시지 처리**: Pub/Sub, Streams
* **랭킹**: Sorted Set
* **분석**: HyperLogLog, Bitmaps
* **위치 기반 서비스**: Geospatial 데이터

사용:

1. **세션 관리**
   * Hash를 사용하여 사용자 세션 데이터를 효율적으로 저장.
   * 빠른 조회 및 업데이트 가능.
2. **게임 랭킹 시스템**
   * Sorted Set을 사용하여 실시간 랭킹 업데이트 및 검색.
3. **로그 및 이벤트 처리**
   * Streams를 통해 실시간 로그 데이터를 저장하고 처리.
4. **실시간 추천 시스템**
   * Set 또는 Sorted Set을 조합하여 사용자 기반 추천 알고리즘 구현.
5. **위치 기반 검색**
   * Geospatial 기능을 활용하여 근처의 장소 검색.
6. **데이터 분석 및 추적**
   * HyperLogLog와 Bitmaps를 사용하여 사용자 행동 추적 및 분석.

</details>



### 1.2 주요 적용 사례

<details>

<summary>캐싱 (Caching)</summary>

* **목적**: 데이터를 메모리에 저장하여 읽기/쓰기 성능을 크게 향상시키는 데 사용.
* **활용 예시**:
  * 웹 페이지 캐싱 (예: 사용자 프로필, 게시물)
  * API 응답 데이터 캐싱
  * 데이터베이스 쿼리 결과 캐싱
  * 계산 결과나 일시적으로 필요한 데이터 저장

</details>

<details>

<summary>세션 저장 (Session Store)</summary>

* **목적**: 사용자 세션 데이터를 관리하여 웹 애플리케이션의 상태를 유지.
* **활용 예시**:
  * 로그인 세션 관리 (JWT 토큰, 사용자 ID 등)
  * 쇼핑 카트 정보 유지
  * 사용자 맞춤형 설정 저장
* **장점**: Redis의 TTL(Time-To-Live)을 활용하여 세션 자동 만료 처리 가능.

</details>

<details>

<summary>Pub/Sub (Publish/Subscribe)</summary>

* **목적**: 실시간 메시징 시스템으로, 메시지를 발행(Publish)하고 구독(Subscribe)하는 방식으로 이벤트를 전달.
* **활용 예시**:
  * 채팅 애플리케이션
  * 실시간 알림 시스템 (예: 이메일 또는 SMS 알림)
  * 실시간 게임 상태 업데이트
  * IoT 장치 간 통신

</details>

<details>

<summary>분산 잠금 (Distributed Lock)</summary>

* **목적**: 분산 환경에서 리소스를 안전하게 제어하기 위해 사용.
* **활용 예시**:
  * 중복 작업 방지
  * 금융 거래와 같은 동시성 관리
  * 스케줄러 작업의 중복 방지
* **Redis 기능 활용**: `SETNX` 명령어와 키 만료(TTL) 설정.

</details>

<details>

<summary>순위표 관리 (Leaderboard)</summary>

* **목적**: 게임이나 애플리케이션에서 실시간 순위를 관리.
* **활용 예시**:
  * 게임 점수 순위 관리
  * 스포츠 경기 순위 관리
  * 유저 활동 점수(포인트) 순위 관리
* **Redis 기능 활용**: `Sorted Set` 데이터 타입을 사용해 점수를 기반으로 정렬.

</details>

<details>

<summary>실시간 데이터 처리</summary>

* **목적**: 실시간으로 업데이트가 필요한 데이터를 빠르게 처리.
* **활용 예시**:
  * 실시간 애널리틱스 (예: 사용자 클릭/뷰 카운트)
  * IoT 데이터 수집 및 처리
  * 실시간 주식 거래 데이터 관리

</details>

<details>

<summary>작업 큐 (Task Queue)</summary>

* **목적**: 작업을 대기열에 저장하고 순차적으로 처리.
* **활용 예시**:
  * 비동기 작업 처리 (예: 이메일 전송, 이미지 처리)
  * 대규모 배치 작업 관리
  * 비동기 API 호출 관리
* **Redis 기능 활용**: `List` 데이터 타입으로 큐를 구현 (`LPUSH`, `RPOP` 등).

</details>

<details>

<summary>지리 데이터 저장 및 처리 (Geospatial Data)</summary>

* **목적**: 위치 기반 서비스를 구현하기 위한 지리 데이터 관리.
* **활용 예시**:
  * 가까운 레스토랑, 가게, 서비스 찾기
  * 차량 공유 서비스의 위치 추적
  * 배달 경로 최적화
* **Redis 기능 활용**: `GEOADD`, `GEODIST`, `GEORADIUS` 명령어.

</details>

<details>

<summary>실시간 스트림 처리 (Redis Streams)</summary>

* **목적**: 실시간으로 데이터 스트림을 처리하고 소비자 그룹 간 데이터를 분배.
* **활용 예시**:
  * 로그 데이터 처리
  * 채팅 애플리케이션
  * 이벤트 중심 아키텍처
* **Redis 기능 활용**: Redis Streams API (`XADD`, `XREAD`).

</details>



### 1.3 데이터 구조

<details>

<summary>String</summary>

* **특징**
  * Redis의 기본 데이터 타입으로, 문자열, 숫자, JSON 등을 저장 가능.
  * 하나의 키에 최대 512MB까지 저장 가능.
  * 단순 값을 저장하거나 캐싱할 때 주로 사용.
* **명령어**
  * `SET key value`: 키에 값을 저장.
  * `GET key`: 키에 저장된 값을 반환.
  * `INCR key`: 값을 1 증가 (숫자형 데이터에 사용).
  * `DECR key`: 값을 1 감소.
  * `APPEND key value`: 기존 값에 문자열을 추가.
* **사용 사례**
  * 캐싱 (예: API 응답 결과).
  * 세션 데이터 저장.
  * 사용자 인증 토큰 저장.

</details>

<details>

<summary>Hash</summary>

* **특징**
  * 필드-값 쌍으로 구성된 데이터 구조.
  * 하나의 키로 여러 필드-값 쌍을 저장할 수 있음.
  * 필드별로 접근 및 수정이 가능.
* **명령어**
  * `HSET key field value`: 특정 필드에 값을 저장.
  * `HGET key field`: 특정 필드의 값을 가져옴.
  * `HGETALL key`: 모든 필드-값 쌍을 반환.
  * `HDEL key field`: 특정 필드를 삭제.
* **사용 사례**
  * 사용자 프로필 데이터 저장 (예: 이름, 나이, 이메일).
  * 구성 설정 저장 (예: 설정 이름과 값).

</details>

<details>

<summary>List</summary>

* **특징**
  * 순서가 있는 값들의 배열.
  * FIFO(큐) 또는 LIFO(스택) 구현 가능.
  * 중복 값을 허용함.
* **명령어**
  * `LPUSH key value`: 리스트의 왼쪽에 값을 추가.
  * `RPUSH key value`: 리스트의 오른쪽에 값을 추가.
  * `LPOP key`: 리스트의 왼쪽 값을 제거하고 반환.
  * `RPOP key`: 리스트의 오른쪽 값을 제거하고 반환.
  * `LRANGE key start stop`: 리스트의 특정 범위 값들을 반환.
* **사용 사례**
  * 작업 대기열 (Task Queue).
  * 최근 방문한 URL 목록.
  * 채팅 메시지 저장.

</details>

<details>

<summary>Set</summary>

* **특징**
  * 중복되지 않는 값들의 집합.
  * 값은 순서 없이 저장됨.
  * 빠른 삽입, 삭제, 검색 지원.
* **명령어**
  * `SADD key value`: 집합에 값을 추가.
  * `SMEMBERS key`: 집합의 모든 값을 반환.
  * `SREM key value`: 특정 값을 제거.
  * `SINTER key1 key2`: 두 집합의 교집합을 반환.
  * `SUNION key1 key2`: 두 집합의 합집합을 반환.
* **사용 사례**
  * 중복 없는 태그 저장.
  * 추천 시스템에서 유사 사용자 찾기.
  * 유니크한 방문자 IP 추적.

</details>

<details>

<summary>Sorted Set</summary>

* **특징**
  * 값과 함께 정렬 기준이 되는 점수(score)를 저장.
  * 점수를 기준으로 값들이 정렬됨.
  * 중복 값을 허용하지 않음.
* **명령어**
  * `ZADD key score value`: 값을 점수와 함께 추가.
  * `ZRANGE key start stop`: 점수 기준으로 정렬된 값을 반환.
  * `ZRANGEBYSCORE key min max`: 특정 점수 범위 내 값을 반환.
  * `ZREM key value`: 특정 값을 제거.
* **사용 사례**
  * 순위표 저장 (예: 게임 점수).
  * 타임스탬프 기반 정렬 데이터 저장 (예: 로그 데이터).
  * 추천 시스템에서 점수 기반 아이템 정렬.

</details>



### 1.4 메모리 기반 데이터 저장 원리

<details>

<summary>데이터의 압축 및 저장 방식</summary>

Redis는 `SDS(Simple Dynamic String)`이라는 동적 문자열을 사용해 String 데이터 타입을 저장

* **String**: SDS 구조로 저장.
* **Hash**: Small Hashes Optimization (entry가 작을 경우 배열로 저장).
* **List**: Quicklist (linked list + ziplist).
* **Set**: Compact Array 또는 HashTable.
* **Sorted Set**: Skiplist + HashTable.

</details>

<details>

<summary>데이터 만료 및 제거 정책</summary>

Redis는 데이터를 자동으로 삭제하여 메모리를 관리:

* **TTL(Time-to-Live)**: 키의 유효 기간을 설정하고 만료 시 삭제.
* **LRU(Least Recently Used) 정책**: 가장 오래된 데이터를 우선적으로 삭제.
* **LFU(Least Frequently Used) 정책**: 가장 적게 참조된 데이터를 삭제.

</details>

<details>

<summary>메모리 저장과 디스크 저장 간의 균형</summary>

Redis는 메모리 기반이지만, 데이터를 디스크로 동기화하여 영구적으로 저장 가능:

* **RDB (Redis Database)**: 주기적으로 데이터 스냅샷을 디스크에 저장.
* **AOF (Append-Only File)**: 모든 쓰기 연산을 로그 형태로 기록.

</details>



## 2. Redis 성능 최적화

### 2.1. 메모리 최적화

<details>

<summary>LRU 정책 (Least Recently Used)</summary>

더 이상 데이터를 저장할 공간이 없을 경우 메모리 내 데이터를 자동으로 삭제하는 **Eviction Policy**를 제공



**LRU 정책의 특징**

* **가장 오랫동안 사용되지 않은 키를 삭제**: 최근 사용되지 않은 키를 우선적으로 삭제해 메모리를 확보합니다.
* Redis는 사용된 키에 대한 **접근 시간 기록**을 효율적으로 관리해 LRU 계산에 활용합니다.

**Redis의 메모리 삭제 정책**

Redis는 다양한 메모리 삭제 정책을 지원하며, LRU는 그중 하나입니다. 설정 가능한 정책은 다음과 같습니다:

1. **noeviction**:
   * 메모리가 가득 찼을 때 새 데이터를 추가하지 않고 오류를 반환합니다.
   * 기본값입니다.
2. **allkeys-lru**:
   * 모든 키 중에서 LRU에 따라 가장 오래된 키를 삭제합니다.
3. **volatile-lru**:
   * 만료 시간이 설정된 키 중에서 LRU에 따라 가장 오래된 키를 삭제합니다.
4. **allkeys-random**:
   * 모든 키 중 무작위로 하나를 삭제합니다.
5. **volatile-random**:
   * 만료 시간이 설정된 키 중 무작위로 하나를 삭제합니다.
6. **volatile-ttl**:
   * 만료 시간이 설정된 키 중, 남은 TTL(Time To Live)이 가장 짧은 키를 삭제합니다.



**LRU 설정**

Redis의 LRU 정책은 `redis.conf` 파일 또는 명령어를 통해 설정할 수 있습니다:

*   설정 파일:

    ```bash
    maxmemory-policy allkeys-lru
    ```
*   명령어:

    ```bash
    CONFIG SET maxmemory-policy allkeys-lru
    ```

</details>

<details>

<summary>객체 압축</summary>

**데이터 구조의 효율적인 사용**

Redis의 데이터 구조는 사용 사례에 따라 메모리 효율성이 다릅니다.&#x20;

* **String 압축 (Bit-Packing)**:
  * 작은 정수값은 `int` 타입으로 저장됩니다.
  * 예: `"1234"`는 정수로 저장되어 8바이트보다 적은 메모리를 사용합니다.
* **Hash, Set, List, Sorted Set 압축**:
  * 작은 크기의 데이터는 \*\*Ziplist (압축 리스트)\*\*로 저장됩니다.
  * 조건: 요소의 수와 데이터 크기가 제한 내에 있어야 함.
  *   설정 예:

      ```bash
      hash-max-ziplist-entries 512   # Hash에서 압축 저장 가능한 최대 엔트리 수
      hash-max-ziplist-value 64      # Hash 값의 최대 크기
      list-max-ziplist-size -2       # List의 압축 저장 크기
      zset-max-ziplist-entries 128   # Sorted Set의 최대 엔트리 수
      zset-max-ziplist-value 64      # Sorted Set의 최대 값 크기
      ```

**2.2 Redis Object Encoding**

Redis는 데이터 구조에 대해 적절한 인코딩을 사용하여 메모리를 줄입니다:

* **int**: 작은 정수값에 사용.
* **embstr**: 작은 문자열에 사용. 더 작은 메모리를 소비하며, 속도도 빠름.
* **raw**: 큰 문자열에 사용.
* **ziplist**: 작은 Hash, List, Sorted Set에 사용.
* **linkedlist**: 큰 List에 사용.
* **hashtable**: 큰 Hash, Set에 사용.

**2.3 Key 및 Value 크기 최적화**

* **짧은 키와 값 사용**:
  * Redis는 키와 값 모두 메모리에 저장되므로, 짧은 키 이름을 사용하면 메모리를 절약할 수 있습니다.
  * 예: `user:1234:info` → `u:1234:i`
* **불필요한 데이터 제거**:
  * 사용하지 않는 데이터를 명시적으로 삭제 (`DEL` 명령어 사용).

**2.4 Redis RDB 및 AOF 압축**

* **RDB 압축**:
  * Redis는 스냅샷을 저장할 때 데이터 파일을 자동으로 압축합니다.
* **AOF 압축**:
  * AOF 파일을 리라이트(rewrite)하여 파일 크기를 줄입니다.
  *   명령어:

      ```bash
      bash코드 복사BGREWRITEAOF
      ```

</details>

<details>

<summary>TTL</summary>

* 모든 키에 만료 시간을 설정

```bash
EXPIRE key 3600
```

</details>



### 2.2. Redis 데이터 파티셔닝 (Sharding)

<details>

<summary>Key-Based Partitioning (키 기반 파티셔닝)</summary>

* **방식**: 각 키를 특정 규칙(hash function)을 통해 나눈 뒤 특정 Redis 노드에 할당.
* **예시**:
  * `hash(key) % n` (n = Redis 노드 개수)
  * `key.hashCode()`를 기반으로 파티션을 결정.
* **장점**:
  * 단순하고 빠르며 구현이 쉬움.
* **단점**:
  * 노드 수 변경 시 데이터 재배치 필요.

</details>

<details>

<summary>Range-Based Partitioning (범위 기반 파티셔닝)</summary>

* **방식**: 키 값이나 데이터 값이 특정 범위에 속하면 해당 범위를 담당하는 Redis 노드에 저장.
* **예시**:
  * `키가 "A~M"`은 Node 1에 저장.
  * `키가 "N~Z"`는 Node 2에 저장.
* **장점**:
  * 특정 데이터 그룹을 다루기 쉬움.
* **단점**:
  * 데이터가 불균등하게 분포될 가능성.
  * 노드 간 부하가 불균형해질 수 있음.

</details>

<details>

<summary>Consistent Hashing (일관성 해싱)</summary>

* **방식**: 해싱된 키 값을 원형 해시 공간에 배치하여 가장 가까운 노드에 데이터를 저장.
* **장점**:
  * 노드 추가/삭제 시 데이터 이동이 최소화.
* **단점**:
  * 구현이 상대적으로 복잡.

</details>



### 2.3. RDB와 AOF

<details>

<summary><strong>RDB (Redis Database Snapshot)</strong></summary>

* **특징**:
  * 메모리 상태를 특정 시점(Snapshot)으로 저장.
  * 데이터베이스를 덤프 파일(`dump.rdb`)로 생성.
  * 디스크에 데이터를 저장하기 때문에 I/O가 많이 발생.
* **장점**:
  1. **빠른 복구**:
     * RDB 파일은 데이터를 한 번에 읽어오므로 복구 속도가 빠릅니다.
  2. **적은 성능 영향**:
     * 데이터 스냅샷은 주기적으로 백그라운드에서 실행되어 Redis의 성능에 상대적으로 영향을 덜 미칩니다.
  3. **단순성**:
     * 설정 및 관리가 간단하며, 주로 정기적인 백업에 사용.
* **단점**:
  1. **데이터 손실 가능성**:
     * 마지막으로 생성된 스냅샷 이후에 Redis가 종료되면 해당 구간의 데이터는 손실됩니다.
  2. **고용량 환경에서 I/O 부하**:
     * 큰 메모리 덤프는 디스크와 CPU 자원을 많이 소모할 수 있습니다.
* **사용 시기**:
  * 데이터 손실 허용 범위가 몇 분 이상 가능한 애플리케이션.
  * 주기적인 백업용으로 적합.

</details>

<details>

<summary><strong>AOF (Append-Only File)</strong></summary>

* **특징**:
  * 모든 쓰기 작업(Set, Hash, List 등)을 로그 형식으로 파일에 기록.
  * 디스크에 기록된 명령어를 재실행하여 데이터 복구.
  * 파일 이름은 `appendonly.aof`.
* **장점**:
  1. **데이터 무결성**:
     * 실시간 쓰기 작업을 로그로 기록하여 데이터 손실을 최소화.
  2. **세밀한 복구 가능**:
     * 마지막 명령어까지 복구할 수 있어 데이터 유실 가능성이 적음.
* **단점**:
  1. **파일 크기 증가**:
     * 모든 명령어를 기록하기 때문에 RDB에 비해 파일 크기가 커질 수 있음.
  2. **성능 저하 가능성**:
     * 빈번한 디스크 쓰기 작업으로 인해 CPU 및 I/O 부하 증가.
* **사용 시기**:
  * 데이터 유실이 거의 허용되지 않는 서비스(예: 금융 데이터, 실시간 분석).
  * 쓰기 빈도가 적고 안정적인 복구가 중요한 경우.

</details>

<details>

<summary>RDB &#x26;&#x26; AOF 조합</summary>

**조합 전략**:

1. RDB는 **정기적인 백업**을 위한 스냅샷 생성.
2. AOF는 **실시간 데이터 무결성**을 보장.
3. Redis 복구 시 AOF가 있으면 이를 사용해 복구, RDB는 보조 수단으로 활용.

</details>



### 2.4. Redis 클러스터를 통한 확장성 확보

<details>

<summary>Redis 클러스터의 기본 구조</summary>

* **샤딩(Sharding)**:
  * 데이터를 키의 해시 값을 기준으로 분할하고 노드에 분배.
  * CRC16 해싱 알고리즘과 16384개의 슬롯 사용.
* **마스터-슬레이브 구조**:
  * 각 노드는 마스터 또는 슬레이브로 구성.
  * 마스터는 쓰기/읽기를 처리, 슬레이브는 읽기 전용으로 동작.
* **Replica Failover**:
  * 마스터 노드 장애 시 슬레이브 노드가 자동으로 마스터로 승격.

</details>

<details>

<summary>Redis 클러스터 성능 최적화</summary>

* **데이터 균등 분배**:
  * 키의 해시 태그를 사용해 데이터 분포를 균등화.
* **클러스터 상태 모니터링**:
  * `CLUSTER INFO` 명령어로 클러스터 상태 확인.
* **슬롯 리밸런싱**:
  * `redis-cli --cluster rebalance`로 데이터 재분배.
* **Redis Cluster Proxy 사용**:
  * 클라이언트와 클러스터 간의 통신을 최적화.

</details>

<details>

<summary>클러스터 장애 처리</summary>

* **Redis Sentinel과 통합**:
  * Sentinel을 활용해 장애 감지 및 자동 복구 지원.
* **Manual Failover**:
  * `CLUSTER FAILOVER` 명령어로 수동으로 마스터 변경.

</details>



### 2.5. Redis Sentinel을 이용한 고가용성 구성

<details>

<summary>Redis Sentinel이란?</summary>

Redis Sentinel은 Redis의 고가용성을 지원하는 도구로, 다음과 같은 기능을 제공

1. **모니터링**: Redis 마스터와 슬레이브의 상태를 지속적으로 확인합니다.
2. **자동 장애 조치(Failover)**: 마스터가 다운되면 슬레이브 중 하나를 새로운 마스터로 승격합니다.
3. **알림**: 관리 도구나 사용자에게 Redis 상태 변화를 알립니다.
4. **구성 관리**: Redis 클라이언트에게 새로운 마스터 정보를 제공합니다.

</details>

<details>

<summary>Redis Sentinel의 주요 구성 요소</summary>

* **Sentinel 인스턴스**: Redis Sentinel 프로세스가 실행되며, 모든 Redis 인스턴스를 모니터링합니다.
* **Redis Master**: 쓰기 작업을 처리하는 주요 Redis 인스턴스입니다.
* **Redis Slave**: 읽기 전용 작업을 처리하며, 마스터의 데이터를 복제합니다

</details>

<details>

<summary>Redis Sentinel의 동작 원리 &#x26; 예시</summary>

* **마스터 모니터링**: Sentinel은 주기적으로 마스터의 상태를 PING 명령으로 확인합니다.
* **장애 감지**: 특정 시간 동안 마스터로부터 응답이 없으면 장애로 간주합니다.
* **합의(Failover 결정)**: 다수의 Sentinel이 장애를 감지하면, 슬레이브 중 하나를 새로운 마스터로 승격합니다.
* **클라이언트 알림**: 새로운 마스터 정보를 클라이언트에 전달합니다.



* **마스터 장애 감지**: Sentinel은 `PING` 명령을 사용해 마스터의 상태를 확인합니다. 일정 시간 동안 응답이 없으면 마스터 장애로 간주합니다.
* **슬레이브 승격**: 장애가 감지되면, Sentinel은 슬레이브 중 하나를 새로운 마스터로 승격합니다.
* **구성 업데이트**: 클라이언트는 Sentinel을 통해 새로운 마스터 정보를 수신하여 연결을 유지합니다.

</details>

<details>

<summary>Redis Sentinel 장단점</summary>

**장점:**

1. 자동 장애 복구
2. 다중 슬레이브를 지원하여 읽기 성능 향상
3. 설정 및 운영의 단순성

**단점:**

1. 완전한 분산 시스템은 아니며, 클러스터링에 비해 확장성이 낮음.
2. 장애 복구 중 데이터 손실 가능성 (비동기 복제의 한계).

</details>



### 2.6. I/O 멀티플렉싱과 성능

<details>

<summary>I/O 멀티플렉싱이란?</summary>

I/O 멀티플렉싱은 단일 스레드가 동시에 여러 네트워크 소켓을 관리할 수 있는 기법입니다. Redis는 이 기술을 활용하여 다수의 클라이언트 요청을 효율적으로 처리합니다.

* **작동 방식**:
  * 단일 스레드가 `epoll`(Linux) 또는 `select`(기타 OS)와 같은 시스템 호출을 사용해 여러 소켓의 읽기/쓰기 이벤트를 감지.
  * 이벤트가 발생한 소켓만 처리하여 불필요한 대기 시간을 제거.
* **장점**:
  * 스레드 간 컨텍스트 스위칭 비용 없음.
  * 낮은 CPU 사용량으로 높은 처리량 가능.

</details>

<details>

<summary>Redis의 싱글 스레드 모델</summary>

Redis는 단일 스레드 기반으로 동작하며, 명령어 처리, 네트워크 통신, 데이터베이스 작업 등을 하나의 스레드에서 수행합니다.

* **이유**:
  * 멀티스레딩 환경에서의 데이터 락과 동기화 문제 회피.
  * 간단하고 예측 가능한 성능 제공.
* **제약**:
  * CPU 집약적인 작업(예: 복잡한 Lua 스크립트 실행)으로 인해 성능 병목이 발생할 수 있음.

</details>

<details>

<summary>I/O 멀티플렉싱 최적화 요소</summary>

**a. 네트워크 이벤트 감지 최적화**

* Redis는 기본적으로 Linux에서 `epoll`을 사용하며, 다른 플랫폼에서는 `select`나 `kqueue`를 활용.
* Linux 환경에서 Redis를 실행하면 `epoll`의 효율적인 비동기 이벤트 처리를 기본적으로 이용.

**b. `tcp-backlog` 설정**

* Redis는 클라이언트의 연결 대기열 크기를 설정하는 `tcp-backlog` 값을 조정할 수 있습니다.
* 적절한 크기로 설정하면 네트워크 트래픽이 많을 때도 대기열 초과를 방지.

**c. 클라이언트 동시성 관리**

* **`maxclients`**: Redis가 동시에 처리할 수 있는 최대 클라이언트 수를 설정.
  * 기본값은 10,000개이며, 시스템 파일 디스크립터 제한(`ulimit -n`)에 의존.
  * 서버의 메모리 및 네트워크 리소스를 고려해 설정.

</details>

<details>

<summary>Redis I/O 멀티플렉싱의 성능 장점</summary>

* **낮은 대기 시간**: 요청당 처리 시간이 짧아 클라이언트가 빠르게 응답을 받을 수 있음.
* **높은 처리량**: 수십만 TPS(Transactions Per Second) 처리 가능.
* **적은 리소스 사용**: 싱글 스레드 기반이므로 멀티스레드 모델에 비해 CPU 사용량이 적음.

</details>

<details>

<summary>성능 병목 문제와 해결책</summary>

**a. CPU 병목**

* Redis는 단일 스레드로 동작하므로 CPU 사용량이 높은 작업(Lua 스크립트, 복잡한 데이터 처리 등)은 병목을 초래할 수 있음.
* **해결책**:
  * 복잡한 작업은 애플리케이션 레벨에서 처리.
  * Redis 클러스터를 구성하여 CPU 부하를 분산.

**b. 네트워크 병목**

* 클라이언트 요청 수가 급증하거나 대역폭이 부족할 경우 성능이 저하될 수 있음.
* **해결책**:
  * 네트워크 대역폭 모니터링.
  * 데이터 압축을 통한 트래픽 최적화.

**c. 디스크 I/O 병목**

* AOF(Append-Only File) 및 RDB(Snapshot) 쓰기 작업 중 디스크 I/O 병목 발생 가능.
* **해결책**:
  * SSD 사용으로 디스크 I/O 성능 향상.
  * RDB와 AOF 설정을 적절히 조정하여 병목 완화.

</details>



### 2.7. Redis의 메모리 사용량 모니터링 및 분석

<details>

<summary>Redis 메모리 모니터링 주요 명령어</summary>

**1.1 `INFO memory`**

* Redis 메모리 상태를 확인하는 가장 기본적인 명령어입니다.
* 주요 출력 정보:
  * `used_memory`: Redis가 사용하는 총 메모리 (바이트 단위).
  * `used_memory_rss`: 운영 체제에서 보고한 실제 메모리 사용량 (Resident Set Size).
  * `used_memory_peak`: Redis가 실행된 이후 사용한 최대 메모리.
  * `used_memory_lua`: Lua 스크립트 실행에 사용된 메모리.
  * `maxmemory`: 설정된 최대 메모리 제한 (없다면 무제한).
  * `mem_fragmentation_ratio`: 메모리 조각화 비율 (`used_memory_rss / used_memory`).

**1.2 `MEMORY STATS`**

* Redis 메모리 사용에 대한 세부 통계를 제공합니다.
* 주요 항목:
  * `peak.allocated`: 할당된 최대 메모리.
  * `dataset.bytes`: 실제 데이터가 차지하는 메모리.
  * `overhead.bytes`: 메타데이터와 기타 Redis 내부 구조가 차지하는 메모리.

**1.3 `MEMORY USAGE <key>`**

* 특정 키가 차지하는 메모리를 확인합니다.
* 출력: 바이트 단위 메모리 사용량.

**1.4 `MEMORY DOCTOR`**

* Redis 메모리 문제를 진단하는 도구로, 설정과 메모리 사용에 대한 조언을 제공합니다.

</details>

<details>

<summary>Redis 메모리 사용량 분석</summary>

**2.1 메모리 사용량의 주요 구성 요소**

* **데이터 저장 공간**: 실제 키-값 데이터가 차지하는 메모리.
* **오버헤드**:
  * 데이터 구조에 따른 메모리 사용량 (예: Hash의 엔트리, Tree 구조 등).
  * 데이터베이스 메타데이터 (RDB 스냅샷, AOF 버퍼 등).
* **캐싱 및 버퍼**:
  * 네트워크 버퍼, 클라이언트 큐, 레플리케이션 큐.

**2.2 메모리 조각화**

* `mem_fragmentation_ratio`로 파악하며, 1.0에 가까울수록 메모리 조각화가 적음.
* 조각화 비율이 높으면 Redis 프로세스의 메모리 요청이 운영 체제의 메모리 관리 방식과 잘 맞지 않을 가능성이 있음.

**2.3 데이터 구조 최적화**

* 각 데이터 구조(String, Hash, List 등)의 메모리 효율성을 이해하고 사용 목적에 맞게 선택.
* 예: 많은 필드를 가진 데이터를 저장할 때 String보다 Hash가 메모리 효율적임.

</details>

<details>

<summary>Redis 메모리 관련 도구</summary>

* **Prometheus와 Grafana**
  * Redis Exporter를 사용해 Redis의 메모리 관련 메트릭을 시각화.
* **ElastiCache Metrics**
  * AWS ElastiCache 사용 시, CloudWatch를 통해 메모리 사용량 모니터링.

</details>



## 3. Redis 백업 및 복구

### 3.1. RDB (Snapshot) 방식

RDB (Redis Database Snapshot)는 Redis의 백업 방식 중 하나. 현재 메모리에 저장된 데이터를 특정 시점의 상태로 디스크에 저장하는 방식. 스냅샷 파일은 `.rdb` 확장자를 가지며, 복구 시 해당 파일을 로드하여 Redis를 특정 시점의 상태로 복원할 수 있음.

<details>

<summary>RDB의 주요 특징</summary>

* **스냅샷 방식**: 메모리 상태를 특정 시점에 캡처하여 저장.
* **고성능**: 백업 작업이 Redis의 메인 스레드와 분리되어 수행되므로, 운영 중인 Redis의 성능에 큰 영향을 주지 않음.
* **복구 속도**: 스냅샷 파일을 메모리에 로드하는 방식으로 빠른 복구 가능.
* **지속성**: 주기적으로 저장된 데이터를 보존하여 시스템 장애나 재시작 시 복구에 활용.

</details>

<details>

<summary>RDB 백업 방법</summary>

**(1) 자동 백업**

Redis는 `save` 설정에 따라 특정 조건에서 자동으로 RDB 파일을 생성합니다.

*   `redis.conf` 파일에서 `save` 옵션을 설정:

    ```plaintext
    save 900 1   # 900초(15분) 동안 1번 이상의 키 변경이 발생하면 백업
    save 300 10  # 300초(5분) 동안 10번 이상의 키 변경이 발생하면 백업
    save 60 10000 # 60초(1분) 동안 10,000번 이상의 키 변경이 발생하면 백업
    ```
* Redis는 기본적으로 `dump.rdb`라는 파일명으로 RDB 파일을 저장.

**(2) 수동 백업**

수동으로 스냅샷을 생성하는 명령어:

*   `BGSAVE`: 백그라운드에서 RDB 스냅샷 생성.

    ```plaintext
    BGSAVE
    ```

    * Redis 서버는 새로운 프로세스를 생성하여 RDB 파일을 생성.
    * 비동기적으로 동작하여 클라이언트 요청에 영향을 주지 않음.
*   `SAVE`: 즉시 RDB 스냅샷 생성.

    ```plaintext
    SAVE
    ```

    * 현재 프로세스에서 동작하며, 모든 클라이언트 요청이 일시 중단될 수 있음.
    * 운영 환경에서는 권장되지 않음.

</details>

<details>

<summary>RDB 복구 방법</summary>

**(1) Redis 재시작**

* Redis는 시작 시 `dump.rdb` 파일을 자동으로 로드하여 데이터를 복구합니다.
*   RDB 파일이 저장된 디렉토리를 `redis.conf` 파일에서 설정:

    ```plaintext
    dir /path/to/dump/
    ```

    * `dir` 디렉토리 안에 `dump.rdb` 파일이 존재하면 Redis가 자동 복구.

**(2) 수동 복구**

*   `dump.rdb` 파일을 Redis 데이터 디렉토리에 복사한 후 Redis를 재시작:

    ```bash
    cp /backup/dump.rdb /var/lib/redis/dump.rdb
    systemctl restart redis
    ```
*   복구 후 데이터 확인:

    ```plaintext
    redis-cli
    KEYS *
    ```

</details>

<details>

<summary>RDB의 장점, 단점</summary>

* **성능 영향 최소화**: 백그라운드에서 수행되므로 애플리케이션 성능 저하를 방지.
* **복구 속도**: RDB 파일을 로드하여 신속하게 복구 가능.
* **간결한 파일 구조**: 파일 크기가 작고, 전송 및 저장이 용이.



* **데이터 손실 가능성**: 백업 시점 이후부터 장애 발생 시점까지의 데이터는 손실 가능.
* **고정된 백업 주기**: 실시간 데이터 변경에 대한 완전한 대응이 어려움.

</details>



### 3.2. Redis AOF (Append-Only File) 방식

AOF(Append-Only File)는 Redis에서 데이터를 백업하고 복구하는 주요 메커니즘 중 하나로, 모든 쓰기 명령을 순차적으로 파일에 기록하는 방식&#x20;

<details>

<summary>AOF의 주요 특징</summary>

* **내구성 보장**: AOF는 Redis가 재시작되거나 비정상 종료되더라도 데이터를 복구할 수 있도록 보장합니다.
* **명령어 기반 기록**: 모든 쓰기 명령어(예: `SET`, `LPUSH`)를 순차적으로 기록합니다.
* **명령 재실행**: AOF 파일에 기록된 명령을 재실행하여 Redis를 복구합니다.
* **가독성**: AOF 파일은 사람이 읽을 수 있는 명령어 형태로 기록됩니다.

</details>

<details>

<summary>AOF의 동작 원리</summary>

* **명령어 기록**:
  * Redis는 클라이언트의 쓰기 요청(예: `SET`, `INCR`)을 AOF 파일에 추가합니다.
* **동기화(flush)**:
  * AOF 파일은 특정 동기화 정책에 따라 디스크에 기록됩니다.
* **복구**:
  * Redis가 재시작되면 AOF 파일을 읽고 저장된 명령어를 순서대로 실행하여 데이터 상태를 복구합니다.

</details>

<details>

<summary>AOF 동기화 정책</summary>

AOF는 디스크 쓰기 성능과 데이터 안정성 사이의 균형을 맞추기 위해 세 가지 동기화 정책을 제공합니다:

1. **`always`**:
   * 각 명령이 실행될 때마다 AOF 파일을 동기화합니다.
   * **가장 안전**하지만 성능 저하가 발생할 수 있습니다.
2. **`everysec`** _(기본값)_:
   * 1초에 한 번 AOF 파일을 동기화합니다.
   * **안정성과 성능**의 균형을 제공합니다.
3. **`no`**:
   * Redis가 직접 동기화를 하지 않고 운영 체제(OS)에 맡깁니다.
   * 성능은 뛰어나지만 데이터 손실 위험이 있습니다.

</details>

<details>

<summary>Redis 클러스터와 AOF</summary>

* Redis 클러스터 환경에서도 AOF를 사용할 수 있습니다.
* 각 노드가 독립적으로 AOF 파일을 유지합니다.
* 복제된 노드의 AOF를 통해 데이터 일관성을 확보할 수 있습니다.

</details>



### 3.3. Redis 데이터 복구 전략

Redis 데이터 복구 전략은 Redis의 백업 방식(RDB, AOF)과 고가용성 기능(Sentinel, 클러스터링)을 기반으로 데이터를 신속하고 안정적으로 복구하는 방법을 포함

<details>

<summary>RDB 파일을 이용한 복구</summary>

* **개념**: RDB(Snapshot)는 Redis 메모리 상태를 특정 시점에 스냅샷으로 저장한 파일입니다.
* **복구 절차**:
  1. Redis 서버를 중지합니다.
  2. 백업해 둔 RDB 파일(`dump.rdb`)을 Redis 데이터 디렉토리에 복사합니다.
     * 일반적으로 디렉토리는 `/var/lib/redis/`입니다.
  3. Redis 서버를 다시 시작하면 RDB 파일이 자동으로 로드됩니다.
* **장점**:
  * 빠른 복구 가능.
  * 데이터 크기가 작아 백업 및 복구 속도가 빠름.
* **단점**:
  * 마지막 스냅샷 이후의 데이터 손실 가능

</details>

<details>

<summary>AOF 파일을 이용한 복구</summary>

* **개념**: AOF(Append-Only File)는 Redis 명령어를 순차적으로 기록한 파일입니다.
* **복구 절차**:
  1. Redis 서버를 중지합니다.
  2. 백업해 둔 AOF 파일(`appendonly.aof`)을 Redis 데이터 디렉토리에 복사합니다.
  3. Redis 설정 파일에서 `appendonly yes`가 설정되어 있는지 확인합니다.
  4. Redis 서버를 다시 시작합니다. Redis는 AOF 파일을 재생하여 데이터를 복구합니다.
* **장점**:
  * 데이터 손실이 거의 없음(매번 명령어를 기록).
* **단점**:
  * 복구 속도가 RDB보다 느림.
  * 파일 크기가 커질 수 있음.

</details>

<details>

<summary>RDB와 AOF 병합을 통한 복구</summary>

* **개념**: Redis는 RDB와 AOF를 함께 사용하여 백업을 최적화할 수 있습니다.
  * 주기적으로 RDB 스냅샷을 저장하고, AOF를 사용해 RDB 이후의 변경 사항을 기록합니다.
* **복구 절차**:
  1. RDB 파일로 초기 상태 복구.
  2. AOF 파일로 최신 상태로 동기화.
* **장점**:
  * 데이터 손실을 최소화하면서 복구 속도도 일정 수준 유지.

</details>

<details>

<summary>Redis Sentinel을 이용한 자동 복구</summary>

* **개념**: Sentinel은 Redis의 고가용성 관리 도구로, 마스터 장애 시 자동으로 복구를 수행합니다.
* **복구 절차**:
  * Sentinel이 장애를 감지하면, 다른 슬레이브 노드를 마스터로 승격합니다.
  * 클라이언트는 새로운 마스터 노드에 연결하여 작업을 지속할 수 있습니다.
* **장점**:
  * 다운타임 최소화.
  * 수동 개입 없이 자동 복구.
* **단점**:
  * 설정 및 관리 복잡성.

</details>

<details>

<summary>Redis 클러스터를 이용한 복구</summary>

* **개념**: 클러스터 모드는 데이터가 여러 노드에 분산 저장되므로, 일부 노드 장애 시에도 데이터를 복구할 수 있습니다.
* **복구 절차**:
  * 장애가 발생한 노드의 데이터를 복제본(slave 노드)에서 복구.
  * 클러스터 내부에서 자동으로 데이터 리밸런싱 수행.
* **장점**:
  * 데이터 복원력 높음.
  * 대규모 애플리케이션에 적합.
* **단점**:
  * 클러스터 설정 및 운영이 복잡.

</details>



### 3.4. Redis의 백업 자동화

<details>

<summary>Redis 백업 방법 이해</summary>

Redis는 다음 두 가지 주요 백업 방식을 제공합니다:

* **RDB (Redis Database File):**
  * 특정 시점의 데이터를 스냅샷으로 저장합니다.
  * 성능에 영향을 적게 미치지만, 데이터 손실 가능성이 있습니다 (스냅샷 간 데이터 변화).
* **AOF (Append-Only File):**
  * 모든 쓰기 작업을 로그 파일로 저장합니다.
  * 데이터 손실 가능성이 적지만, 파일 크기가 커지고 I/O 부하가 증가할 수 있습니다.

자동화는 주로 RDB 스냅샷 방식을 활용하며, 필요에 따라 AOF와 조합할 수 있습니다.

</details>

<details>

<summary>RDB 백업 자동화</summary>

* **백업 주기 최적화**: 데이터 중요도에 따라 RDB와 AOF 주기를 조정.
* **다중 위치 저장**: 온프레미스와 클라우드에 동시에 저장.
* **주기적인 복구 테스트**: 백업 파일로 복구 테스트를 수행해 신뢰성 검증.
* **보안 강화**: 백업 파일을 암호화하여 저장.
* **성능 고려**: 백업 중 Redis 성능 저하를 최소화하도록 설정.

</details>



## 4. Redis 고급 기능

### 4.1. Redis Streams (데이터 스트림 처리)

<details>

<summary>Redis Streams의 주요 개념</summary>

* **Stream (스트림)**
  * 시간 순서대로 추가되는 데이터의 시퀀스.
  * 데이터는 고유한 ID로 식별되며, 이는 타임스탬프와 시퀀스 번호로 구성됨 (예: `1670958850000-0`).
* **Entry (항목)**
  * Stream에 저장된 단일 데이터 항목.
  * 각 항목은 키-값 쌍으로 구성됨.
  * 예: `XADD mystream * temperature 23 humidity 80`
* **Consumer (소비자)**
  * Stream 데이터를 읽고 처리하는 클라이언트.
* **Consumer Group (소비자 그룹)**
  * Stream 데이터를 병렬로 처리하기 위한 소비자 그룹.
  * 동일한 그룹 내의 소비자들이 데이터를 나눠서 처리.
* **Producer (생산자)**
  * Stream 데이터에 항목을 추가하는 클라이언트.

</details>

<details>

<summary>Redis Streams의 특징</summary>

1. **실시간 데이터 처리**
   * 실시간으로 데이터가 들어오고 소비자들이 이를 처리할 수 있음.
2. **내장 메시징 큐**
   * Kafka 같은 메시징 큐 역할을 수행 가능.
   * Pub/Sub보다 확장성이 뛰어남.
3. **내구성 보장**
   * 데이터는 디스크에 저장되어 재시작 시에도 손실되지 않음.
4. **유연한 데이터 처리**
   * 데이터를 여러 소비자에게 분산 처리 가능.
   * 소비자 그룹을 통해 데이터 처리가 병렬로 이루어짐.

</details>

<details>

<summary>Redis Streams의 사용 사례</summary>

* **실시간 로그 처리**
  * 애플리케이션 로그를 실시간으로 수집하고 분석.
* **이벤트 기반 시스템**
  * 사용자 활동 추적, 알림 시스템 등.
* **IoT 데이터 수집**
  * 센서에서 발생하는 데이터를 실시간으로 수집.
* **메시지 큐**
  * Kafka나 RabbitMQ와 비슷한 용도로 사용.

</details>

<details>

<summary>Redis Streams의 장점과 한계</summary>

**장점**

* 간단한 설정과 사용법.
* 고속 데이터 쓰기/읽기 성능.
* Redis의 다른 기능들과 자연스럽게 통합.

**한계**

* Redis는 메모리 기반이므로 대용량 스트림 데이터는 비용이 증가.
* Kafka 같은 전용 메시징 시스템만큼의 고급 기능은 부족

</details>



### 4.1. Redis Modules (RedisSearch, RedisGraph 등)

<details>

<summary>RedisSearch</summary>

RedisSearch는 Redis를 기반으로 하는 고급 검색 엔진입니다. 다음과 같은 기능을 제공합니다:

* **풀텍스트 검색**: 자연어 기반의 텍스트 검색 지원.
* **필터링과 정렬**: 검색 결과를 특정 필드나 조건에 따라 필터링하고 정렬.
* **인덱싱**: Redis 데이터의 효율적인 검색을 위해 인덱스를 생성.
* **JSON 문서 검색**: RedisJSON과 결합하여 JSON 데이터를 검색.
* **범위 쿼리**: 숫자, 날짜, 텍스트 등의 필드에 대해 범위 쿼리 지원.

**주요 사용 사례:**

* 검색 엔진 개발.
* 데이터 기반의 빠른 분석 및 결과 제공.

</details>

<details>

<summary>RedisGraph</summary>

RedisGraph는 그래프 데이터를 저장하고 쿼리할 수 있는 그래프 데이터베이스 모듈입니다.

* **Cypher 쿼리 언어 지원**: Neo4j와 호환되는 Cypher 언어를 사용.
* **노드와 엣지**: 그래프의 노드와 엣지를 저장하고 관계를 쿼리.
* **고성능**: 메모리 기반의 그래프 저장 및 처리로 빠른 성능.
* **동적 그래프 구조**: 실시간으로 변경되는 그래프를 다룰 수 있음.

**주요 사용 사례:**

* 소셜 네트워크 관계 분석.
* 추천 시스템.
* 경로 탐색 및 그래프 데이터 분석.

</details>

<details>

<summary>RedisJSON</summary>

RedisJSON은 JSON 데이터를 Redis에 저장하고 조작할 수 있는 모듈입니다.

* **JSON 포맷 저장**: Redis에 구조화된 JSON 데이터를 저장.
* **부분 업데이트**: JSON 문서의 특정 필드만 수정 가능.
* **RedisSearch와 통합**: JSON 데이터를 RedisSearch와 함께 사용해 검색 성능 향상.
* **쿼리 지원**: JSONPath를 사용한 복잡한 쿼리 가능.

**주요 사용 사례:**

* JSON API 데이터 저장.
* 복잡한 데이터 모델링 및 검색.

</details>

<details>

<summary>RedisTimeSeries</summary>

RedisTimeSeries는 시계열 데이터를 효율적으로 처리하기 위한 모듈입니다.

* **시계열 데이터 저장**: 시간과 관련된 데이터를 효율적으로 관리.
* **다운샘플링**: 데이터의 크기를 줄이면서 중요한 정보만 유지.
* **자동 집계**: 데이터를 그룹화하고 집계 처리.
* **알람 설정**: 특정 조건에 대한 경고 기능 제공.

**주요 사용 사례:**

* IoT 데이터 관리.
* 실시간 로그 분석 및 모니터링.

</details>

<details>

<summary>RedisBloom</summary>

RedisBloom은 확률 데이터 구조를 구현한 모듈로, 메모리 효율적이고 고속의 데이터 구조를 제공합니다.

* **Bloom Filter**: 특정 데이터가 존재하는지 여부를 빠르게 확인.
* **Cuckoo Filter**: 동적 크기 지원과 삭제 가능한 데이터 구조.
* **Count-Min Sketch**: 데이터 빈도 추적.
* **Top-K 알고리즘**: 가장 빈번하게 등장하는 데이터 추적.

**주요 사용 사례:**

* 데이터 중복 제거.
* 실시간 추천 시스템.
* 빈도 기반 데이터 분석.

</details>



## 5. Redis와 애플리케이션

### 5.1. Redis를 사용한 캐싱 전략

<details>

<summary>Redis를 캐싱으로 사용하는 이유</summary>

* **고성능:** Redis는 메모리 기반 데이터 저장소로 매우 빠른 읽기/쓰기 속도를 제공합니다.
* **유연한 데이터 구조:** 다양한 데이터 구조(String, Hash, List, Set, Sorted Set 등)를 제공하여 다양한 캐싱 시나리오에 적합합니다.
* **TTL(Time to Live):** 데이터 만료를 설정하여 자동으로 데이터를 제거할 수 있습니다.
* **확장성:** Redis 클러스터를 통해 수평 확장이 가능합니다.
* **다양한 언어 지원:** Redis는 여러 언어와의 손쉬운 통합을 지원합니다.

</details>

<details>

<summary>데이터 읽기 캐싱 (Read Cache)</summary>

* **설명:** 데이터베이스에서 자주 조회되는 데이터를 Redis에 저장하고, 요청이 들어오면 Redis에서 먼저 데이터를 확인합니다.
* **예시:**
  1. 클라이언트 요청 → Redis 조회
  2. Redis에 데이터가 없으면 → 데이터베이스에서 조회 후 Redis에 저장 (Cache Miss)
  3. Redis에 데이터가 있으면 → 바로 반환 (Cache Hit)
* **코드 예제 (Java + Spring):**

```java
// Redis에서 캐싱된 데이터 조회
String data = redisTemplate.opsForValue().get(key);
if (data == null) {
    // 데이터베이스 조회
    data = databaseService.getData(key);
    // Redis에 데이터 저장
    redisTemplate.opsForValue().set(key, data, Duration.ofMinutes(10)); // TTL 설정
}
return data;
```

</details>

<details>

<summary>데이터 쓰기 캐싱 (Write Cache)</summary>

* **설명:** 데이터베이스에 데이터를 저장하거나 업데이트할 때, Redis에도 데이터를 동시에 업데이트합니다.
* **예시:**
  1. 클라이언트가 데이터를 업데이트하면
  2. 데이터베이스와 Redis 모두 데이터를 갱신합니다.
* **장점:** 데이터의 최신 상태를 캐시에 유지할 수 있습니다.
* **단점:** 데이터베이스와 Redis 간의 일관성 문제가 발생할 수 있습니다.

</details>

<details>

<summary>Lazy Loading (지연 로딩)</summary>

* **설명:** 캐시에 데이터가 없을 때만 데이터베이스에서 데이터를 가져와 캐시에 저장합니다.
* **장점:** 필요할 때만 캐시를 채우므로 초기 캐시 오버헤드가 적습니다.
* **단점:** Cache Miss가 발생하면 초기 응답 속도가 느릴 수 있습니다

</details>

<details>

<summary>Write-Through Cache</summary>

* **설명:** 데이터를 데이터베이스에 저장할 때 Redis에도 동시에 저장합니다.
* **장점:** 캐시와 데이터베이스 간의 일관성이 높습니다.
* **단점:** 쓰기 작업에서 약간의 오버헤드가 발생합니다.

</details>

<details>

<summary>Read-Through Cache</summary>

* **설명:** 데이터베이스를 직접 조회하지 않고 Redis를 통해 데이터를 읽어옵니다.
* **장점:** 데이터 접근이 일관되게 Redis를 통해 이루어집니다.
* **단점:** Cache Miss가 많으면 Redis에서 데이터를 지속적으로 로드해야 합니다.

</details>

<details>

<summary>Cache Aside (주변 캐시 패턴)</summary>

* **설명:** 애플리케이션이 Redis와 데이터베이스를 직접 제어하며, 데이터가 필요한 경우 캐시를 먼저 확인하고 없으면 데이터베이스를 조회합니다.
* **장점:** 애플리케이션의 요구에 따라 유연하게 동작합니다.
* **단점:** 캐시와 데이터베이스 간의 일관성 문제를 개발자가 직접 처리해야 합니다.

</details>

<details>

<summary>TTL(Time to Live) 설정</summary>

* Redis는 키에 만료 시간을 설정할 수 있습니다. 예를 들어, 캐시 데이터가 10분 동안만 유효하도록 설정할 수 있습니다.
*   **명령어 예시:**

    ```bash
    SET key value EX 600 # 600초(10분) TTL
    ```
* 만료 시간을 활용하여 오래된 데이터를 자동으로 삭제해 메모리 사용량을 최적화할 수 있습니다.

</details>

<details>

<summary>캐싱 전략 선택 기준</summary>

* **데이터 일관성:** 데이터베이스와 Redis 간의 동기화가 필요한지 여부.
* **캐시 데이터 크기:** 저장하려는 데이터의 크기와 Redis 메모리 용량.
* **TTL 요구사항:** 데이터의 수명이 얼마나 중요한지.
* **읽기/쓰기 비율:** 읽기와 쓰기의 비율에 따라 전략을 선택.
* **요청 패턴:** 데이터 요청이 특정 패턴을 따르는지 분석.

</details>







