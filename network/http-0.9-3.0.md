---
description: HTTP 0.9-3.0
---

# HTTP 0.9-3.0

## 1. HTTP 0.9 (1991)

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:red;">GET 요청만 존재</mark>
* <mark style="color:red;">Header X</mark>
* <mark style="color:red;">Status Code X</mark>
* <mark style="color:red;">HTML파일만 전송</mark>

## 2. HTTP 1.0 (1997)

<figure><img src="../.gitbook/assets/image (262).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:green;">Header</mark>
* <mark style="color:green;">Status Code</mark>
* <mark style="color:green;">new Method</mark>

## 3. HTTP 1.1

* <mark style="color:green;">Persistent connection</mark>
* <mark style="color:green;">Pipelining</mark>
* <mark style="color:green;">Chunk Transfer Encoding</mark>
* <mark style="color:green;">Caching && Conditional Request</mark>
* <mark style="color:red;">HTTP의 Head Of Line Blocking</mark>
* <mark style="color:red;">헤더 중복</mark>

### 3.1. Persistent Connection

<figure><img src="../.gitbook/assets/image (263).png" alt=""><figcaption></figcaption></figure>

* 매번 연결을 열고 닫지 않음.
* Close되기 전까지 Connection이 열려있음
* 여러 HTTP 요청과 응답에 TCP 연결을 한번만 수행

### 3.2. Pipelining

<figure><img src="../.gitbook/assets/image (264).png" alt=""><figcaption></figcaption></figure>

* 응답을 기다릴 필요가 없음
* 응답 속도가 빨라짐

### 3.3. Chunk Transfer Encoding

<figure><img src="../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

* 응답을 작은 Chunk로 나눠서 보냄
* 초기웹 페이지 로딩이  빨라짐

### 3.4. Caching && Conditional Request

<figure><img src="../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

* 필요없는 데이터 전송을 줄임

<details>

<summary>ETag</summary>

* ETag는 서버가 특정 리소스의 버전을 나타내기 위해 생성하는 해시 값 또는 고유 식별자

Server: 리소스 제공시 ETag 포함

```vbnet
HTTP/1.1 200 OK
ETag: "12345abcd"
```

**Client: 캐싱된 ETag로 조건부 요청**\
클라이언트는 서버로 리소스를 재요청할 때, 이전에 받은 ETag 값을 요청 헤더에 포함시켜 전송.

```mathematica
If-None-Match: "12345abcd"
```

**Server: 변경되지 않음**

```mathematica
HTTP/1.1 304 Not Modified
```

클라이언트는 기존에 캐싱된 리소스를 사용합니다.

**Server: 변경됨**

```vbnet
HTTP/1.1 200 OK
ETag: "67890efgh"
```

새로운 리소스 데이터를 제공하며 ETag를 업데이트합니다.



</details>

### 3.5. HTTP Head Of Line Blocking

<figure><img src="../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

* 요청에서 문제가 발생하면, 그 뒤에 요청들은 기다리는 문제

## 4. HTTP 2.0 (2015)

* <mark style="color:green;">Binary Framing Layer</mark>
* <mark style="color:green;">Multiplexing</mark>
* <mark style="color:green;">Stream Prioritization</mark>
* <mark style="color:green;">Server Push</mark>
* <mark style="color:green;">HPack Compression</mark>
* <mark style="color:red;">TCP의 Head Of Line Blocking</mark>

### 4.1. Binary Framing Layer

<figure><img src="../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

* HTTP/1.1에서 텍스트 기반의 요청 및 응답 방식을 이진(binary) 형식으로 변환하여 전송
* 프레임 독립성: 데이터를 프레임으로 쪼갬 (헤더 프레임, 데이터 프레임, ...)
* 빠른 전송, 병렬처리, 연결 오버헤드 감소, 헤더 크기 감소

### 4.2. Multiplexing

<figure><img src="../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

* 단일 TCP 연결에서 스트림 채널을 열어, 여러 HTTP 요청/응답을 병렬로 전송
* 하나의 요청이 지연되도 다른 요청에 영향을 주지 않음
* HTTP 1.1 TCP Head Of Line Blocking(FIFO 방식) 문제 해결
* TCP의 Head Of Line Blocking 문제 발생

<details>

<summary>Head Of Line Blocking</summary>

앞의 작업에서 문제가 발생하면 뒤에 있는 작업들이 Block 되는 상황



HTTP Head Of Line Blocking

요청1, 요청2, 요청3

**HTTP에서 만약 `요청1`에 문제가 발생(패킷 손실) 한다면?**

1. HTTP/1.1은 **단일 TCP 연결당 하나의 요청/응답만 처리**할 수 있음. **직렬화**된 요청/응답만 가능하기 때문에 **앞선 요청이 지연되면 이후 요청도 지연**
2. 요청1이 손실되었으므로, **요청2**와 **요청3**는 **같은 연결 위에 있기 때문에** 대기.
3. 이미 **요청2**가 도착했더라도, **앞선 요청1의 응답이 끝나지 않으면** 애플리케이션에 전달되지 않음.



TCP Head Of Line Blocking

요청1a, 요청2a, 요청3a, 요청1b, 요청2b, 요청3b, 요청1c, 요청2c, 요청3c

**TCP에서 만약 `요청3a`에 문제가 발생(패킷 손실)한다면?**

1. TCP는 **단일 바이트 스트림**으로 동작하므로, 모든 데이터는 순서대로 전달되어야 함.
2. **요청3a**가 손실되었으므로, **그 이후에 도착한 모든 패킷**(`요청1b`, `요청2b`, `요청3b`, ...)은 버퍼에 대기.
3. **요청1b**는 이미 도착했더라도 **앞선 순서(요청3a)**&#xAC00; 도착하지 않았기 때문에 애플리케이션에 전달되지 않음.

즉, **요청3a**의 손실로 인해 그 뒤에 도착한 **모든 데이터**(`요청1b`부터)가 처리되지 않는 것이 **TCP의 Head-of-Line 블로킹 문제**.

</details>



### 4.3. Stream Prioritization

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* 요청에 대한 우선순위 부여
* 서버측에서는 우선순위가 높은 요청 먼저 처리

### 4.4 Server Push

<figure><img src="../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

* 요청이 없거나 하나의 요청에 대한 여러 응답 가능

### 4.5 HPack Compression

<figure><img src="../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

* Hpack을 사용하여 과거 요청 헤더를 기억하고 압축



## 5. HTTP 3.0 (2022)

* QUIC

### 5.1. QUIC (by google)

<figure><img src="../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

* TCP 연결과정 생략
* UDP 기반으로 순차적 처리X
* TCP Head Of Line Blocking 해결. 손실된 패킷만 재전송



## 6. 정리

<figure><img src="../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>
