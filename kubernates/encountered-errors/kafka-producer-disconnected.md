# kafka producer: disconnected

설명:

1. bitnami의 kafka 프로젝트 설치 후, producer의 동작을 확인하는 과정에서 에러 발생.
2.

    ```
    WARN [Producer clientId=console-producer] Bootstrap broker kafka-controller-2.kafka-controller-headless.kafka.svc.cluster.local:9092 (id: -3 rack: null) disconnected (org.apache.kafka.clients.NetworkClient)
    ```





원인:

* bitnami/kafka 프로젝트 내에서 발생하는 이슈로 파악된다.
* ```
  https://github.com/bitnami/charts/issues/18659
  ```
* <pre><code><strong>https://github.com/bitnami/charts/issues/18793
  </strong></code></pre>
* SASL 과 SCRAM-SHA256이 정상적으로 이용되지 않아서 발생한 에러.



해결:

1. 기본 SASL 프로토콜을 SASL\_PLAINTEXT에서 PLAINTEXT로 바꿔서 적용한 후 설치.
2.  ```
    # 헬름 차트로 카프카 설치
    ```

    ```
    helm install kafka --set volumePermissions.enabled=true,replicaCount=3,listeners.client.protocol=PLAINTEXT oci://registry-1.docker.io/bitnamicharts/kafka
    ```

    (client.properties를 이용하지 않는다.)
