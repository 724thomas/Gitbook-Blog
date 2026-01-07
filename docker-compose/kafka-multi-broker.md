# Kafka, Multi Broker

Docker Desktop 버전 2이상 필요하다.



Docker Compose 설정파일 작성

* docker-compose.yml 파일을 작성하고, 해당 파일이 존재하는 디렉토리에서 docker-compose 실행.

```yaml
---
version: '2' # Docker compose 버전 지정
services:
  zookeeper-1:
    image: confluentinc/cp-zookeeper:latest #실전에서는 정확한 버전을 사용 권장
    environment:
      ZOOKEEPER_SERVER_ID: 1 #클러스터에서 주키퍼를 식별할 아이디. 
      ZOOKEEPER_CLIENT_PORT: 2181 #컨테이너 내부 포트
      ZOOKEEPER_TICK_TIME: 2000 #클러스터를 구성할때 동기화를 위한 틱 타임
      ZOOKEEPER_INIT_LIMIT: 5 #주키퍼 클러스터는 쿼럼이라는 과정을 통해 마스터를 선출. 리더에게 커넥션을 맺을때 지정할 초기 타임아웃이다.
      ZOOKEEPER_SYNC_LIMIT: 2 #주키퍼 리더와 나머지 서버들의 싱크 타임.
    ports:
      - "22181:2181"

  zookeeper-2:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_SERVER_ID: 2
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
    ports:
      - "32181:2181"

  zookeeper-3:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_SERVER_ID: 3
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
      ZOOKEEPER_INIT_LIMIT: 5
      ZOOKEEPER_SYNC_LIMIT: 2
    ports:
      - "42181:2181"

  kafka-1:
    image: confluentinc/cp-kafka:latest #실사용엔 지정된 버전 사용
    depends_on:
      - zookeeper-1 #서비스들의 우선순위를 지정해준다. zookeeper-1가 먼저 실행되어 있어야 컨테이너가 올라오게 된다.
      - zookeeper-2
      - zookeeper-3
    ports:
      - 29092:29092 #브로커의 내부 포트
    environment:
      KAFKA_BROKER_ID: 1 #브로커 아이디. 단일 브로커기 때문에 없어도 무방하다.
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181,zookeeper-2:2181,zookeeper-3:2181 #zookeeper에 커넥션하기 위한 대상을 지정한다.
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-1:9092,PLAINTEXT_HOST://localhost:29092 #외부에서 접속하기 위한 리스너이다.
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT #보안을 위한 프로토콜 매핑
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT #도커 내부에서 사용할 리스너 이름
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1 #트랜잭션 상태에서 복제 계수를 지정
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1 #트랜잭션 최소 ISR(InSyncReplicas)설정을 지정하는 것으로, 단순 작업하기 위해 복제 계수를 1로 설정.

  kafka-2:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
      - zookeeper-2
      - zookeeper-3
    ports:
      - "39092:39092"
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181,zookeeper-2:2181,zookeeper-3:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-2:9092,PLAINTEXT_HOST://localhost:39092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1

  kafka-3:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper-1
      - zookeeper-2
      - zookeeper-3
    ports:
      - "49092:49092"
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper-1:2181,zookeeper-2:2181,zookeeper-3:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-3:9092,PLAINTEXT_HOST://localhost:49092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
```



Docker-Compose 실행하기

```
docker-compose -f docker-compose.yml up -d
```

<figure><img src="/broken/files/A7k6qldwzFNVu4L24abn" alt=""><figcaption></figcaption></figure>

Docker 상태 로그 확인하기

```
docker ps
```

<figure><img src="/broken/files/V6SpMqEFlKi0xAQM0L2H" alt=""><figcaption></figcaption></figure>

컨테이너 ID로 로그를 확인한다

```
docker logs 41d38fe8b03c
```



Topic 생성하기

```
docker-compose exec kafka-1 kafka-topics --create --topic my-topic --bootstrap-server kafka-1:9092 --replication-factor 3 --partitions 2
```

<figure><img src="/broken/files/E9PsdoMDoO8O0MYW5pUD" alt=""><figcaption></figcaption></figure>

* docker-compose:
  * 명령어를 수행한다.
* exec:
  * 컨테이너 내에서 커맨드를 수행하도록 한다.
* kafka-1:
  * 우리가 설정으로 생성한 브로커(서비스) 이름이다.
* kafka-topics:
  * 카프카 토픽에 대한 명령을 실행한다.
* \--create:
  * 토픽을 생성하겠다는 의미이다.
* \--topic :
  * 생성할 토픽 이름을 지정한다.
* \--bootstrap-server service:port
  * bootstrap-server는 kafka-1 브로커 서비스를 나타낸다. 이때 서비스:포트 로 지정하여 접근할 수 있다.
* \--replication-factor 1:
  * 복제 계수를 지정한다.
  * 여기서는 1로 지정했다.
* \--partition:
  * 토픽내에 파티션 개수를 지정한다



생성된 토픽 확인하기

```
docker-compose exec kafka-1 kafka-topics --describe --topic my-topic --bootstrap-server kafka-1:9092 
```

<figure><img src="/broken/files/SGHpO2YBLBSqM2YJYiCk" alt=""><figcaption></figcaption></figure>

* docker-compose:
  * 명령어를 수행한다.
* exec:
  * 컨테이너 내에서 커맨드를 수행하도록 한다.
* kafka-1:
  * 우리가 설정으로 생성한 브로커(서비스) 이름이다.
* kafka-topics:
  * 카프카 토픽에 대한 명령을 실행한다.
* \--describe:
  * 생성된 토픽에 대한 상세 설명을 보여달라는 옵션이다.
* \--topic :
  * 생성한 토픽 이름을 지정한다.
* \--bootstrap-server service:port
  * bootstrap-server는 kafak-1 브로커 서비스를 나타낸다. 이때 서비스:포트 로 지정하여 접근할 수 있다.
* 결과로 kafka-1브로커의 토픽이름, 아이디, 복제계수, 파티션, 리더, 복제정보, isr 등을 확인할 수 있다.
* 복제 계수는 3으로 지정되었다.
* 그리고 파티션은 2개로 0, 1 이 있다.
* Leader의 결과를 통해서 해당 파티션의 리더가 어떤 브로커인지 알려준다.
* Replicas 는 데이터 복제에 대해서 알려준다.
* Isr: In sync replica 에 대해서 알려준다. (동기화된 복제본임을 알려준다.)



컨슈머 실행하기

```
docker-compose exec kafka-1 bash

[appuser@571d813de396 ~]$ kafka-console-consumer --topic my-topic --bootstrap-server kafka-1:9092
```

* 우선 docker-compose exec kafka-1 bash 를 통해서 컨테이너 내부의 쉘로 접속한다.
* 이후 kafka-console-consumer 를 이용하여 컨슘한다.
* 역시 컨슘할 토픽을 지정하고, 브로커를 지정하기 위해서 --bootstrap-server 를 이용했다.



프로듀서 실행하기

```
docker-compose exec kafka-1 bash 

[appuser@571d813de396 ~]$ kafka-console-producer --topic my-topic --broker-list kafka-1:9092

>hello world
>this is producer
```

컨슈머를 확인하면 hello world, this is producer 라는 메세지를 볼 수 있다.



참고: [https://devocean.sk.com/blog/techBoardDetail.do?ID=164016](https://devocean.sk.com/blog/techBoardDetail.do?ID=164016)
