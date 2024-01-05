# service-name-headless?

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

1. service-name-headless

* 이 서비스는 `ClusterIP`가 `None`으로 설정되어 있습니다. 이는 서비스가 로드 밸런싱을 제공하지 않으며, 클라이언트가 서비스 이름을 DNS 조회할 때 각 포드의 실제 IP 주소 목록을 반환받는다는 것을 의미합니다.
* 클라이언트는 이 목록을 사용하여 특정 포드에 직접 연결할 수 있으며, 이 방식은 스테이트풀 서비스(예: 데이터베이스)에 적합합니다.
* `service-name-headless`는 주로 포드 간의 직접적이고 특정한 통신을 위해 사용됩니다.



2. service-name

* 이 서비스에는 유효한 `ClusterIP` (예: `10.16.11.125`)가 할당되어 있습니다. 이 IP는 서비스에 접근하는 통합된 진입점으로 작동하며, 서비스로 들어오는 요청을 서비스에 연결된 포드 간에 로드 밸런싱합니다.
* 일반 서비스는 클라이언트 요청을 여러 포드 중 하나로 자동으로 분배하기 때문에, 상태가 없는(stateless) 애플리케이션에 적합합니다.
* `app-db-mysql`은 일반적으로 클라이언트가 특정 포드에 직접 연결할 필요가 없는 경우 사용됩니다.

