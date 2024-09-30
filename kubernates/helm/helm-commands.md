---
description: Helm 명령어
---

# Helm commands

* helm --help : helm과 관련된 명령어
  * 보통 사용하는 건 install, uninstall, upgrade, search, repo 정도이다.
* helm repo --help : helm repo와 관련된 명령어



helm만 설치되어있을때, 레포지토리가 아무것도 없는 상태이기 때문에, 레포지토리를 추가한다.

* helm repo add bitnami https://charts.bitnami.com/bitnami
  * binami에는 helm 패키지들이 모여있는 레포지토리이다.
  * 이 레포지토리에는 각종 chart가 들어있다.
* helm repo list
  * helm에 저장되어있는 레포지토리를 볼 수 있다. (위 커맨드 후엔 bitnami만 있을거다)
* helm search repo
  * 레포지토리에 들어있는 모든 차트들을 검색
* helm search repo mariadb
  * mariadb가 레포지토리에 있는지 확인한다
*   helm show chart bitnami/mariadb

    * mariadb의 차트가 어떤 내용을 담고 있는지 확인(간단한 내용)

    <figure><img src="../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>
* helm inspect values bitnami/mariadb
  * mariadb의 차트가 어떤 내용을 담고 있는지 확인(상세한 내용)
  * 각종 yaml파일들이 reference하고 있는 values 파일 내용을 볼 수 있다
* helm install app-db bitnami/mariadb
  * bitnami/mariadb를 app-db 이름으로 설치한다.
  * 주의해야할점은 install이라고 설치되는게 아니라, application으로 동작가능한 상태를 만드는 것
  * app-db : 배포되는 이름 커스터마이징된 이름
*   helm install app-db --set auth.rootPassword=secretpassword bitnami/mariadb

    * inspect value안에 있는 값들을 바꿔서 배포도 가능하다.

    <figure><img src="../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>
* helm list
  * 운영중인 chart 패키지들을 확인할 수 있다
* helm uninstall app-db
  * app-db 제거
