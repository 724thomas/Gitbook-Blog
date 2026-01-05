# Eliminating Unnecessary Polling with SSE (Server-Sent Events) Push

[https://youtu.be/ovGgdPPUZ2I?si=4VC6c6ncNmcn5gbh](https://youtu.be/ovGgdPPUZ2I?si=4VC6c6ncNmcn5gbh)

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 요약

* Polling의 대체재 - HTTP SSE
* message broker 전략을 어떻게 가져갈지 고민
* SSE connection이 정상 close가 됬는지 모니터링 도구 필요

### 1. 배경

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 준 실시간성으로 계좌 정보를 가져오기 위해 Polling 방식으로 구성
* 사용자 증가 → Polling 방식이 부담 (CPU 50% 이상)

### 2. 해결

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

계좌 API를 요청할때 재조회 이벤트가 요청이 있을때만 호출.

#### Push Server 구현

* 현재 토스 증권에서는 WebSocket과 SSE를 케이스를 나눠서 구현하고 있음
* WebSocket
  * 양방향 통신
  * 송 / 수신량 데이터량 많음
  * 시세 위주 사용
* ServerSentEvents
  * 불필요한 polling 제거 시
  * `개인화된` 데이터를 이용한 이벤트 푸쉬

**현재 구성된 polling을 제거하는 방향이기에 SSE 채택**

#### SSE 구현

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Client side message 전략

* 브로드캐스팅(BroadCasting) - Fan out 방식
* 유니캐스팅(Unicasting)

#### 브로드캐스팅 실제 사용 사례

*   코드

    ```java
    private val channel: Sinks.Many<ServerSentEvent<T>> = Sinks.many().multicast().directAllOrNothing()

    override fun doProcess(consumer: Consumer<ReactiveSubscription.Message<String, String>>)
        get() = Consumer { msg ->
            val sseMsg = JsonUtil.fromJson(msg.message, object: TypeReference<SseMessage<T>>() {})
            channelMeterRegistry.recordMessageSub(sseMsg.header.sentTime)

            // send to client
            super.send(
                channel,
                ServerSentEvent.builder<T>()
                    .data(sseMsg.msg)
                    .id(msg.channel)
                    .build()
            )
        }

    override fun afterPropertiesSet() {
        // subscribe inbound message
        super.onMessage(CHANNEL_NAME)
            .subscribe()
    }

    fun connect(): Flux<ServerSentEvent<T>> {
        return Flux.merge(
            super.connect(channel) {},
        )
    }
    ```

    ```java
    @RestController
    class BroadcastEventController(
        @Qualifier(ChannelConfiguration.BROADCAST_CHANNEL)
        private val broadcastChannelHandler: BroadcastChannelHandler<String>
    ) {
        @GetMapping(value = ["/api/v1/live-chat"], produces = [MediaType.TEXT_EVENT_STREAM_VALUE])
        fun eventConnect(): Flux<ServerSentEvent<String>> {
            return broadcastChannelHandler.connect()
        }
    }
    ```
* 이벤트성 단일 채팅방 (실시간 의견 기능)



<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 클라이언트3는 메시지를 API Server로 전송
* API Server는 메시지를 메시지 브로커로 전송
* 메시지브로커로부터 SSE 서버가 메시지를 수신
* 수신한 메시지는 모두 클라이언트에게 전송

하지만 유저별 어떻게..? 개인화된 데이터를 이용한 이벤트 푸쉬는??

#### 유니캐스트 사용 사례

*   코드

    ```java
    class UnicastChannelHandler<T> : Any(
        private val channelMeterRegistry: ChannelMeterRegistry,
        private val inboundStream: Flux<String>,
    ): ChannelOutBoundHandler<ServerSentEvent<T>> {

        private val CHANNEL = Sinks.many().unicast().onBackpressureBuffer<ServerSentEvent<T>>()
        private val TYPE_REFERENCE = object : TypeReference<SseMessage<T>>() {}

        fun connect(): Flux<ServerSentEvent<T>> {
            return Flux.merge(
                // channel connect
                super.connect(CHANNEL) {
                },

                inboundStream
                    .map { JsonUtil.fromJson(it, TYPE_REFERENCE) }
                    .map { sseMsg ->
                        channelMeterRegistry.recordMessageSub(sseMsg.header.sentTime)
                        ServerSentEvent.builder<T>()
                            .data(sseMsg.msg)
                            .id(sseMsg.header.key)
                            .build()
                    }
            )
        }
    }
    ```

    ```java
    fun createInboundStreamFromRedisPubSub(
        connection: StatefulRedisClusterPubSubConnection<String, String>,
        channel: String
    ): Flux<String> {
        return Flux.create { sink: FluxSink<String> ->
            val pubSubListener = object : RedisPubSubAdapter<String, String>() {
                override fun message(messageChannel: String, message: String?) {
                    if (channel == messageChannel && message != null) {
                        sink.next(message)
                    }
                }
            }

            val (_, subscriber) =
                shardConnection(connection.async().upstream(), channel)
                    ?: return@create sink.error(UnreachableCodeException("redis connection error"))

            subscriber.statefulConnection.addListener(pubSubListener)
            subscriber.subscribe(channel)

            sink.onDispose {
                subscriber.unsubscribe(channel)
                subscriber.statefulConnection.removeListener(pubSubListener)
            }
        }
    }
    ```

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 유저별 단독 채널 생성
* 클라이언트는 본인 채널만 구독하면 됨

유니캐스트 실제 사용 사례

* 보유자산 polling 제거

<figure><img src="../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

3초 주기의 폴링을 없앨 수 없다. 그래서 중간에 SSE Server를 두고 재조회 이벤트가 발행되었을 때만 보유 종목 API를 호출하여 30%정도의 성능 개선

### 브로드캐스팅 문제 발생(feat. 안드로이드 푸쉬알람 대체)

* 배경

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

* FCM Push가 Capacity가 넘어가는 상황
* 모든 푸쉬 이벤트를 전부 FCM을 발행하도록 되어있음

### 해결

* 접속중: SSE
* 미접속중: FCM Push
* Redis Pub/Sub을 사용하여 접속/미접속 사용자 구별

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## Server Side Message

브로커 선택 전략:

* Kafka:
  * 접속/비접속 사용자 모두 메시지가 발행됨 → 불필요한 메시지 발생을 위한 필터링이 추가적으로 필요(At most once)
* Redis pub/sub:
  * SSE 서버는 Redis Pub/Sub에 연결된 사용자만 Subscribe 요청을 하게됨.
  * Publisher는 Redis Pub/Sub에만 요청을 하게 됨.
  * redis pub/sub Brokered Throughput 성능 : `60K`
* NATS:
  * Brokered Throughput을 더 높이기 위한 옵션
  * 단일, 클러스터, 슈퍼 클러스터드 기능 제공

### NATS

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

* 동작 방식
  1. NATS 2에게 PUB msg를 보냄
  2. NATS 2 → NATS 1로 msg 보냄
  3. NATS 1 → 메시지 전송

**NATS 2는 NATS 1에게 Subscriber가 있는지 어떻게 알았을까?**

* NATS Server Node를 사용: 본인의 라우팅 정보, 타 서버로 라우팅할 정보가 적힌 장부

**장부에 기록하는 타이밍?**

* 구독 or 해지 시점

**Unsubscribe는 어떻게 동작하나?**

* Fan-out 방식으로 NATS 2에서 시작하여 모든 노드(NATS 1, NATS 3)에 장부를 업데이트

## 결과

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
