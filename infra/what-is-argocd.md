# What is ArgoCD

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## 요약

ArgoCD는 Kubernetes 환경에서 GitOps를 실현하는 핵심 도구로, Git을 단일 진실 소스로 삼아 클러스터의 실제 상태를 자동 또는 수동으로 동기화한다. CI가 Docker 이미지 빌드와 tag 생성까지 담당한다면, ArgoCD는 Git의 변경 사항을 기준으로 배포를 수행하는 CD 역할에 집중한다. 이를 위해 argocd-server, repo-server, application-controller로 구성된 아키텍처가 Git 변경 감지, 템플릿 렌더링, Diff 계산, Sync 수행 등 전체 배포 과정을 자동화한다. 반드시 이해해야 하는 핵심은 “Git 변경이 곧 배포 변경”이며, kubectl 기반 수동 조작은 ArgoCD Self Heal에 의해 되돌려진다는 점이다.

실무에서는 서비스 단위 및 환경별(Application 단위)로 관리 구조를 세분화하여 Dev/Stage/Prod 간 Sync 정책을 다르게 구성한다. image tag 변경은 실제 배포를 트리거하는 핵심 요소이며, Helm chart나 values.yaml 변경 또한 Git 변경만으로 재배포가 이루어진다. Secret은 Git에 평문으로 저장할 수 없기 때문에 Sealed Secrets, External Secrets Operator, SOPS 등 외부 보안 시스템을 활용해 GitOps 구조에서도 안전하게 관리한다.&#x20;

고급 기능으로는 Argo Rollouts를 통한 카나리·블루그린 배포, Multi-Cluster 전략, 환경별 Sync 정책 설계, Helm 기반 설정 분리 등 조직의 배포 안정성을 강화하는 다양한 요소들이 있다. ArgoCD는 단순 배포 도구가 아니라 운영 방식 자체를 선언형·자동화 기반으로 재정의하는 GitOps 엔진이며, 이를 정확히 이해하면 실무 배포 파이프라인에 적응하는데 도움이 된다.



## Table of Contents

1. \#ArgoCD란 무엇인가
   1. \###GitOps 개념과의 관계
      * _**GitOps는 무엇이며 왜 등장했는가**_
      * _**DevOps와 GitOps의 차이**_
      * _**ArgoCD가 GitOps를 실현하는 방식**_
   2. \###ArgoCD가 해결하는 문제들
      * _**기존 CI/CD 구조의 한계**_
      * _**배포의 일관성과 자동화 부족 문제**_
      * _**쿠버네티스 환경에서의 복잡성 해결**_
2. \#ArgoCD의 구성 요소
   1. \###ArgoCD 아키텍처
      * _**argocd-server 역할**_
      * _**repo-server 역할**_
      * _**application-controller 역할**_
   2. \###ArgoCD Application 개념
      * _**왜 서비스별로 Application을 나누는가**_
      * _**환경별(dev/stage/prod)로 Application을 분리하는 이유**_
      * _**Monorepo 구조에서의 Application 묶음 관리**_
3. \#GitOps 환경에서의 배포 흐름 이해
   1. \###master 변경 → 이미지 빌드 → tag 업데이트 → ArgoCD Sync
      * _**image tag란 무엇인가**_
      * _**tag는 언제 어떻게 변경되는가**_
      * _**CI와 CD의 역할 분리**_
   2. \###Helm Chart & values.yaml 변경에 따른 동작
      * _**Helm chart 구조 변경 시 ArgoCD가 감지하는 방식**_
      * _**values.yaml 변경이 자동으로 재배포되는 이유**_
      * _**개발자가 Jenkins/ArgoCD를 건드리지 않아도 배포가 되는 원리**_
4. \#ArgoCD Sync 메커니즘의 깊은 이해
   1. \###자동 Sync vs 수동 Sync
      * _**두 방식의 실무적 차이점**_
      * _**환경별 추천 Sync 정책**_
   2. \###Prune, Self Heal, Diff 계산
      * _**Prune이 필요한 이유**_
      * _**Self Heal의 강력함과 주의점**_
      * _**ArgoCD가 Diff를 계산하는 방식**_
   3. \###OutOfSync, 롤백 동작 원리
      * _**OutOfSync는 무엇을 의미하는가**_
      * _**Sync를 통해 어떤 일이 일어나는가**_
      * _**Git 롤백과 ArgoCD 롤백의 차이**_
5. \#실무 적용 사례와 고급 기능
   1. \###여러 클러스터를 관리하는 구조
      * _**관리 클러스터 vs 타깃 클러스터 구조**_
      * _**클러스터 추가 방법과 운영 전략**_
   2. \###Secret을 안전하게 관리하는 방법
      * _**Sealed Secrets**_
      * _**External Secrets Operator**_
      * _**SOPS**_
   3. \###ApplicationSet과 Argo Rollouts
      * _**ApplicationSet의 필요성과 구조**_
      * _**Rollouts로 카나리/블루그린 구현하기**_
   4. \###Monorepo에서 ArgoCD 운영하기
      * _**서비스 디렉토리 구조 설계**_
      * _**ApplicationSet 자동 생성 전략**_
      * _**서비스 변경만 배포하도록 구성하는 방법**_



## ArgoCD란 무엇인가

ArgoCD는 쿠버네티스 환경에서 GitOps를 구현하기 위한 대표적인 오픈소스 CD(Continuous Delivery/Deployment) 도구이다.\
CI/CD라는 용어는 개발자가 자주 접하지만, 두 개념은 역할이 다르다.\
CI는 코드를 빌드하고 테스트하며 Docker 이미지를 생성하는 과정이고, CD는 이렇게 생성된 결과물을 실제 배포 환경에 반영하는 과정이다.\
ArgoCD는 CD만 담당하는 도구이며, Git에 있는 선언형 정의를 기준으로 쿠버네티스 클러스터의 상태를 지속적으로 유지한다.

***

### GitOps 개념과의 관계

#### _**GitOps는 무엇이며 왜 등장했는가**_

GitOps는 인프라 및 애플리케이션 상태를 Git에 선언해두고, 이를 자동으로 클러스터에 반영하는 운영 방식이다.\
전통적인 CI/CD 시스템은 Jenkins가 직접 클러스터에 접근해 배포를 “밀어넣는(push)” 방식이었으며, 이는 보안 및 관리 측면에서 여러 문제를 발생시켰다.\
반면 GitOps는 Git을 단일 진실 소스(single source of truth)로 삼고, ArgoCD 같은 도구가 클러스터 내부에서 Git 상태를 “끌어오는(pull)” 방식으로 배포를 수행한다.

#### _**DevOps와 GitOps의 차이**_

DevOps는 개발과 운영의 협업을 위한 넓은 문화적 개념이다.\
GitOps는 이 철학을 **Git 기반 자동화**로 구체화한 기술적 실천 방법이다.\
즉, DevOps가 개념이라면 GitOps는 그중 배포/운영을 실천하는 방식이며, ArgoCD는 이를 구현하는 핵심 도구이다.

#### _**ArgoCD가 GitOps를 실현하는 방식**_

ArgoCD는 Git에 선언된 상태(desired state)를 쿠버네티스 클러스터의 실제 상태(actual state)와 지속적으로 비교한다.\
변경이 감지되면 OutOfSync 상태로 표시하고, 사용자가 수동 혹은 자동 Sync 정책에 따라 클러스터를 Git과 동일한 상태로 맞춘다.\
결국 “Git에서 정의된 everything-as-code”가 실제 운영 환경을 결정하게 된다.

***

### ArgoCD가 해결하는 문제들

#### _**기존 CI/CD 구조의 한계**_

기존 Jenkins 기반 CD는 다음과 같은 단점을 가진다:

* Jenkins가 Kubernetes 클러스터에 직접 접근해야 함
* 배포 히스토리가 Git이 아니라 Jenkins에만 남음
* Service별/환경별 배포 관리가 어려움
* 여러 클러스터를 관리하기 복잡함

#### _**배포의 일관성과 자동화 부족 문제**_

쿠버네티스는 환경 설정, Helm chart, values 등 서로 연결된 요소가 많다.\
사람이 수동으로 배포하면 실수가 쉽게 일어난다.\
ArgoCD는 Git을 기준으로 자동으로 상태를 유지하기 때문에 일관성이 확보된다.

#### _**쿠버네티스 환경에서의 복잡성 해결**_

쿠버네티스는 Deployment, ReplicaSet, Pod, ConfigMap, Secret 등 수십 개의 리소스를 다뤄야 한다.\
ArgoCD는 이를 UI에서 시각화하고 diff를 제공하며, 에러가 나면 즉시 확인할 수 있도록 돕는다.



## ArgoCD의 구성 요소

ArgoCD는 쿠버네티스 내부에서 동작하는 애플리케이션이며, 여러 내부 구성 요소가 협업하여 GitOps CD 기능을 제공한다.\
이 구성 요소를 이해하면 ArgoCD가 어떻게 Git과 클러스터 상태를 지속적으로 동기화하는지 자연스럽게 이해할 수 있다.

***

### ArgoCD 아키텍처

#### _**argocd-server 역할**_

ArgoCD의 진입점이다.

* UI 대시보드를 제공해 애플리케이션 상태, Sync 여부, 리소스 구조 등을 시각적으로 확인할 수 있다.
* CLI, REST API 요청을 받아 인증 및 인가 처리를 수행한다.
* 사용자의 Sync 요청, 롤백 요청 등을 받아 application-controller에게 전달한다.

즉, 사용자가 ArgoCD를 바라보는 모든 인터페이스(웹/UI/API)는 argocd-server를 통해 이루어진다.

#### _**repo-server 역할**_

repo-server는 GitOps의 핵심 엔진이다.

* Git 리포지토리를 clone 혹은 pull 하여 최신 상태를 가져온다.
* Helm chart, Kustomize, Jsonnet 등을 렌더링하여 실제 쿠버네티스 manifest를 생성한다.
* 이 manifest는 application-controller가 클러스터와 비교하는 기준이 된다.

repo-server를 별도 Pod로 운영하는 이유는 Git 렌더링 과정이 무겁기 때문이다.\
분리함으로써 병렬 처리 및 성능을 확보할 수 있다.

#### _**application-controller 역할**_

ArgoCD의 두뇌이자 GitOps 동기화의 핵심이다.

* repo-server가 렌더링한 manifest를 읽는다.
* 실제 클러스터 상태를 Kubernetes API를 통해 조회한다.
* 두 상태를 비교(Diff)하여 OutOfSync 여부를 판단한다.
* Sync 정책에 따라 자동/수동으로 클러스터에 변경을 적용한다.
* Health Check를 통해 Pod readiness, ReplicaSet 안정성 등을 모니터링한다.

application-controller가 있기 때문에 ArgoCD는 언제나 클러스터 상태를 Git과 동일하게 유지하려고 한다.

***

### ArgoCD Application 개념

ArgoCD에서 가장 중요한 리소스가 바로 _Application_ 이다.\
Application은 하나의 배포 단위를 의미하며, 어떤 Git 경로를 사용할지, 어떤 클러스터에 배포할지, 어떤 Helm values를 사용할지를 명확하게 정의한다.

***

#### _**왜 서비스별로 Application을 나누는가**_

서비스별 Application 분리는 GitOps의 핵심 원칙과 운영 효율성 모두에서 중요한 요소이다.

* 서비스별로 독립적으로 배포되어야 한다
* 각 서비스는 서로 다른 리소스(deployment/service/configmap 등)를 가진다
* 롤백, Sync, diff도 서비스 단위로 수행하는 것이 안전하다
* 오류가 발생했을 때 어느 서비스인지 구분 가능해야 한다

예를 들어 around-api, privacy-api, batch-service는 서로 완전히 별도의 실행 환경과 설정을 가지므로 각각을 Application으로 정의해야 한다.

***

#### _**환경별(dev/stage/prod)로 Application을 분리하는 이유**_

환경은 배포 전략이 다르고 위험도도 다르다.\
따라서 동일 서비스라도 환경별로 Application을 따로 운영하는 것이 정석이다.

실무 기준으로 보면:

* dev: 자동 sync, prune on
* stage: 자동 sync 또는 제한적 자동 sync
* prod: manual sync, prune off

또한 환경별 values.yaml을 적용해야 하므로 Application을 구분해주어야 한다.

***

#### _**Monorepo 구조에서의 Application 묶음 관리**_

Monorepo를 사용할 경우 다음과 같은 구조가 일반적이다:

```
/services
  /around-api
  /privacy-api
  /batch
/deployments
  /dev
  /prod
```

이때 각 서비스 디렉토리를 기준으로 Application을 자동 생성하는 ApplicationSet을 활용한다.\
ApplicationSet은 Monorepo의 디렉토리 구조를 기반으로 서비스 수만큼의 Application을 자동 생성할 수 있어, 대규모 조직에서 효율적인 GitOps 구조를 만들 수 있다.

***

## GitOps 환경에서의 배포 흐름 이해

ArgoCD를 제대로 이해하려면 GitOps 기반 배포가 어떤 흐름으로 이루어지는지 관점이 매우 중요하다.\
특히 다음 두 가지가 중심이다:

1. _이미지 태그(image tag) 변경 → 배포_
2. _Helm chart/values.yaml 변경 → 배포_

***

### master 변경 → 이미지 빌드 → tag 업데이트 → ArgoCD Sync

서비스의 코드가 변경되면 CI(Jenkins, GitHub Actions 등)가 다음을 수행한다:

1. 코드를 빌드하여 Docker 이미지를 생성한다
2. 새로운 image tag를 부여한다 (예: 1.4.3, git hash 기반 등)
3. Docker registry에 push한다
4. Helm values.yaml 또는 manifest에 새 tag로 업데이트한다
5. 이 commit이 master에 merge
6. ArgoCD가 변경된 Git 상태를 감지
7. diff → OutOfSync → Sync → 배포

***

#### _**image tag란 무엇인가**_

image tag는 실행 가능한 Docker 이미지의 버전이며 서비스의 상태를 정의하는 핵심 값이다.\
Kubernetes Deployment는 image tag가 바뀌어야 rollout을 시작하기 때문에 태그 변경은 배포의 트리거 역할을 한다.

***

#### _**tag는 언제 어떻게 변경되는가**_

* 코드가 변경되어 새 build가 발생할 때
* values.yaml이 변경되어 환경 설정이 바뀔 때
* 서비스 재배포가 필요할 때\
  대부분의 팀은 자동 빌드 시스템을 통해 tag를 생성하고 업데이트한다.

***

#### _**CI와 CD의 역할 분리**_

* CI: Docker 이미지 생성, push, values.yaml 업데이트
* CD(ArgoCD): 값이 업데이트된 Git을 읽고 배포 수행

GitOps에서는 배포의 최종 결정권이 CI가 아니라 Git에 있다.



## Helm Chart & values.yaml 변경에 따른 동작

Helm Chart와 values.yaml은 쿠버네티스 환경에서 애플리케이션 구성을 정의하는 중요한 파일이다.\
여기에는 환경변수, 포트, 이미지 정보, 리소스 설정 등 서비스 실행에 필수적인 설정이 포함된다.\
GitOps 환경에서 Helm Chart 변경이 발생하면 Jenkins나 ArgoCD를 수동으로 수정하지 않아도 재배포가 일어나는 이유는 바로 ArgoCD의 렌더링 및 Diff 감지 메커니즘 덕분이다.

***

### Helm chart 구조 변경 시 ArgoCD가 감지하는 방식

Helm chart를 수정하면 두 가지 변화가 생긴다.

1. **템플릿(template) 구조 변화**
   * deployment.yaml, service.yaml 등 템플릿 구조가 바뀌면 ArgoCD는 렌더링된 결과물의 차이를 감지한다.
2. **values.yaml 변화**
   * replica 수, 환경 변수 등 값이 변경되면 템플릿에 적용된 실제 manifest가 달라지므로 diff가 발생한다.

ArgoCD는 이러한 변화를 다음 단계로 감지한다:

1. repo-server가 Git의 변경된 commit을 자동 감지
2. 변경된 Helm chart를 다시 렌더링하여 manifest 생성
3. 기존 클러스터 상태와 비교
4. 차이가 있으면 OutOfSync 상태 표시
5. Sync 정책에 따라 자동 또는 수동으로 배포

따라서 Helm chart 변경은 Jenkins 개입 없이도 배포 트리거가 될 수 있다.

***

### values.yaml 변경이 자동으로 재배포되는 이유

values.yaml은 서비스 동작에 필요한 실제 구성 값을 정의한다.\
예를 들어 다음과 같다:

```
image:
  repository: around-api
  tag: "1.3.2"

env:
  MODE: "production"
  API_ENDPOINT: "https://api.example.com"
```

이 값이 변경되면:

* Helm chart 렌더링 결과물이 변경되고
* ArgoCD는 diff를 감지하여 OutOfSync로 표시한다
* 자동 Sync인 경우 즉시 재배포
* 수동 Sync인 경우 관리자가 버튼을 눌러 배포

즉, 개발자가 Jenkins나 kubectl을 직접 실행하지 않아도 **Git에 변경이 있으면 ArgoCD가 이를 자동으로 감지하고 배포 프로세스를 시작한다.**\
이것이 GitOps 운영의 가장 큰 장점이다.

***

### 개발자가 Jenkins/ArgoCD를 건드리지 않아도 배포가 되는 원리

핵심 원리는 다음과 같다:

1. **Git이 단일 진실 소스이다**\
   ArgoCD는 Git 변경을 기준으로 클러스터 상태를 조정한다.
2. **repo-server가 지속적으로 Git을 모니터링한다**\
   변경이 발생하면 자동으로 manifest를 재생성한다.
3. **application-controller가 diff를 감지한다**\
   값이나 템플릿이 조금이라도 다르면 OutOfSync 상태가 된다.
4. **자동 Sync 정책이 있는 경우 즉시 배포가 이루어진다**

이 원리 덕분에 개발자는 배포 툴을 조작할 필요 없이 Git commit만 하면 된다.\
특히 Dev 환경에서는 이 자동화가 개발 속도를 크게 높여준다.

***

## ArgoCD Sync 메커니즘의 깊은 이해

ArgoCD의 Sync(Synchronization) 기능은 GitOps 운영의 핵심이다.\
Sync는 Git에 선언된 상태(Desired State)를 클러스터의 실제 상태(Actual State)와 맞추는 과정이다.

***

### 자동 Sync vs 수동 Sync

#### _**자동 Sync (Automatic Sync)**_

Git에 변경이 생기면 ArgoCD가 자동으로 Sync를 실행한다.\
특징:

* 개발 속도 빠름
* 자동으로 배포가 이루어짐
* Self Heal, Prune 옵션과 함께 사용하면 완전한 GitOps 구현 가능\
  실무에서는 dev, stage 환경에서 많이 사용된다.

#### _**수동 Sync (Manual Sync)**_

Git 변경은 감지하지만 Sync는 바로 실행하지 않는다.\
특징:

* 운영자가 직접 Sync 버튼을 눌러야 배포가 진행
* prod 환경에서 주로 사용
* 실수로 잘못된 설정이 배포되는 위험을 방지

환경별 추천 Sync 정책은 다음과 같다:

| 환경    | 권장 Sync 방식                             |
| ----- | -------------------------------------- |
| dev   | automatic sync + prune + self-heal     |
| stage | automatic 또는 manual (팀 정책에 따라)         |
| prod  | manual sync + prune off + self-heal on |

***

### Prune, Self Heal, Diff 계산

#### _**Prune이 필요한 이유**_

Prune은 Git에는 존재하지 않는 리소스를 클러스터에서 삭제하는 기능이다.\
예를 들어 Git에서 ConfigMap을 제거하면 실제 클러스터에서도 삭제해야 한다.\
Prune이 없으면 “좀비 리소스”가 남아 오작동을 일으킬 수 있다.

다만 prod 환경에서 Prune은 신중해야 한다.\
Git commit 실수로 리소스를 지웠을 때 실제 리소스가 삭제되는 사고가 발생할 수 있기 때문이다.

***

#### _**Self Heal의 강력함과 주의점**_

Self Heal은 클러스터에서 사람이 kubectl로 수동 수정한 값을 ArgoCD가 즉시 감지하고 Git 상태로 되돌리는 기능이다.\
실수나 임시 조작을 방지하기 때문에 강력하지만 다음 문제도 있다:

* 운영자가 수동 조작한 내용을 되돌려버릴 수 있음
* 긴급 패치가 필요한 상황에서는 임시로 기능을 끌 필요 있음

신입 개발자가 실수로 helm upgrade나 kubectl apply를 실행하더라도 Self Heal이 있으면 Git 기준으로 복원되기 때문에 조직 내 운영 규칙을 강하게 enforcing할 수 있다.

***

#### _**ArgoCD가 Diff를 계산하는 방식**_

ArgoCD는 다음 단계를 거쳐 diff를 계산한다:

1. repo-server에서 Git의 manifest 렌더링
2. Kubernetes API를 통해 실제 리소스 YAML 조회
3. 운영에 필요 없는 필드(status, annotations 등) 제외
4. 순수 값 차이만 비교
5. 차이가 있으면 OutOfSync 발생

Diff 기능 덕분에 GitOps 운영에서 모든 변경이 투명해지고, 실수나 설정 누락이 줄어든다.

***

## OutOfSync, 롤백 동작 원리

Sync와 함께 가장 많이 마주치는 상태가 바로 OutOfSync이다.

***

#### _**OutOfSync는 무엇을 의미하는가**_

OutOfSync는 다음을 의미한다:

* Git 선언 값과 클러스터의 실제 값이 다르다
* 배포가 필요하다
* 또는 사람이 클러스터에 수동 수정했다

대표적인 OutOfSync 원인은 다음과 같다:

* image tag 변경
* values.yaml 변경
* helm chart 템플릿 변경
* 사람이 kubectl edit, apply 등으로 수동 수정
* Git에 리소스가 삭제되었는데 prune되지 않은 경우

***

#### _**Sync를 통해 어떤 일이 일어나는가**_

Sync가 실행되면:

* Git에서 manifest 재생성
* Diff 비교
* 변경된 부분만 apply
* 필요 시 신규 리소스 생성, 수정
* prune 옵션 시 삭제까지 진행

Sync가 성공하면 상태는 Healthy / Synced로 표시된다.

***

#### _**Git 롤백과 ArgoCD 롤백의 차이**_

**Git 롤백**

* Git commit을 revert
* ArgoCD는 새로운 Git 상태를 읽고 Sync
* 가장 안전하고 GitOps 원칙에 맞음

**ArgoCD UI 롤백**

* Application history에서 이전 revision 선택
* 해당 리소스를 직접 복원하여 Sync
* Git 기록과 달라질 수 있어 주의 필요

실무에서는 Git 롤백 방식이 권장된다.



## 실무 적용 사례와 고급 기능

ArgoCD는 단순히 GitOps 배포 자동화 도구에 그치지 않는다.\
규모가 있는 조직에서는 여러 클러스터를 운영하고, 안전한 비밀 관리(Secret), 대규모 서비스 운영, 카나리·블루그린 배포 등 고급 기능까지 필요하다.\
이 Part에서는 실제 운영 환경에서 ArgoCD가 어떻게 활용되는지, 그리고 GitOps 관점에서 어떤 확장 기능을 사용해야 하는지를 다룬다.

***

### 여러 클러스터를 관리하는 구조

ArgoCD는 하나의 인스턴스만으로 여러 클러스터를 관리할 수 있다.\
이것은 GitOps 환경에서 매우 중요한 기능이다.\
관리되는 클러스터가 늘어나도 운영 복잡도를 크게 줄여준다.

***

#### _**관리 클러스터 vs 타깃 클러스터 구조**_

일반적으로 다음과 같은 구조를 사용한다:

```
[ Management Cluster (ArgoCD 설치) ]
    ├─ Dev Cluster
    ├─ Stage Cluster
    └─ Prod Cluster
```

관리 클러스터(Management Cluster)에 ArgoCD를 설치하고, Dev/Stage/Prod 같은 타깃 클러스터(Target Cluster)를 연결한다.\
이 방식의 장점은 다음과 같다:

* ArgoCD가 장애나 업데이트로 중단되더라도 Prod 클러스터는 영향을 받지 않는다
* 중앙에서 모든 클러스터 상태를 관찰하고 관리할 수 있다
* RBAC와 보안 정책을 일원화하여 운영 가능

특히 Prod 클러스터에 ArgoCD를 직접 설치하지 않는 구조가 안전하게 운영되는 일반적인 패턴이다.

***

#### _**클러스터 추가 방법과 운영 전략**_

클러스터를 ArgoCD에 추가하려면 간단히 다음 명령을 수행한다:

```
argocd cluster add <context-name>
```

ArgoCD는 해당 클러스터에 접근하기 위한 ServiceAccount, Role, RoleBinding을 자동 생성한다.\
이 방식은 Multi-Cluster 환경에서 GitOps를 확장하는 데 매우 적합하다.

운영 전략은 다음과 같이 구분할 수 있다:

* Dev/Stage: 자동 Sync + prune + self-heal
* Prod: 수동 Sync + self-heal on + prune off
* 민감한 네임스페이스는 read-only 권한으로 제한

***

## Secret을 안전하게 관리하는 방법

GitOps 방식에서는 모든 설정이 Git에 저장되지만, Secret만큼은 예외이다.\
민감 정보를 Git에 그대로 올리는 것은 절대 금지이기 때문에 여러 암호화/외부 연동 전략이 필요하다.

***

### _**Sealed Secrets**_

Sealed Secrets는 Secret을 공개키로 암호화해 Git에 저장할 수 있게 해준다.\
복호화는 클러스터 내부 컨트롤러가 수행하며 외부에서는 원문을 확인할 수 없다.

장점:

* GitOps 원칙 준수
* 설치와 사용이 간단

단점:

* 복호화 컨트롤러가 동작하지 않으면 Secret 생성 불가

***

### _**External Secrets Operator (ESO)**_

요즘 가장 많이 사용되는 방식이다.\
Secret을 Git에 암호화된 형태로 저장하는 대신, AWS Secrets Manager, GCP Secret Manager 등 외부 저장소를 사용한다.

예시:

```
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
spec:
  secretStoreRef:
    name: aws-secret-store
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: prod/db_password
```

특징:

* Git에는 Secret 정보가 저장되지 않음
* 보안팀 정책을 완벽하게 준수
* Secret 회전(rotation)이 쉬움
* ArgoCD는 ExternalSecret CRD만 관리

***

### _**SOPS**_

SOPS는 특정 YAML 파일의 민감 정보 부분만 선택적으로 암호화할 수 있다.\
AWS KMS, GPG Key 등 다양한 키 관리 시스템과 연동된다.

장점:

* 세밀한 암호화 가능
* GitOps와 완벽하게 통합

***

## ApplicationSet과 Argo Rollouts

규모가 커질수록, ArgoCD Application을 하나씩 사람이 만들어주는 것은 불가능하다.\
또한 배포 전략도 롤링 업데이트만으로는 충분하지 않을 수 있다.\
이때 ApplicationSet과 Argo Rollouts가 필수 요소가 된다.

***

### ApplicationSet의 필요성과 구조

ApplicationSet은 다음과 같은 환경에 특히 적합하다:

* Monorepo에서 수십 개 서비스가 존재하는 경우
* 여러 지역/여러 클러스터에 동일 앱을 배포해야 하는 경우
* 매번 application yaml을 생성하는 대신 자동화가 필요한 경우

ApplicationSet은 “generator”를 기반으로 Application을 자동 생성한다.\
예시:

```
generators:
  - git:
      directories:
        - path: services/*
template:
  spec:
    project: default
    source:
      repoURL: <repo>
      path: "{{path}}"
```

이 설정은 services/\* 안에 있는 모든 디렉토리를 Application으로 변환한다.\
서비스가 추가되면 Application도 자동으로 생성된다.

***

### Argo Rollouts로 카나리/블루그린 구현하기

ArgoCD는 **배포** 자체를 담당하지만, 롤링 업데이트 이상의 전략은 제공하지 않는다.\
고급 배포 전략이 필요하다면 Argo Rollouts를 사용해야 한다.

Rollouts는 Deployment 대신 Rollout 리소스를 사용하며, 다음 기능을 제공한다:

* 카나리 배포
* 블루그린 배포
* 트래픽 분할 (Istio, NGINX, SMI 등)
* 메트릭 기반 자동 롤백
* Pause/Resume 단계적 배포

예시 (카나리 전략):

```
strategy:
  canary:
    steps:
      - setWeight: 20
      - pause: {}
      - setWeight: 50
      - pause: {}
```

실무에서는 다음과 같이 조합된다:

* ArgoCD → GitOps 배포 자동화
* Argo Rollouts → 배포 전략/트래픽 제어

***

## Monorepo에서 ArgoCD 운영하기

Monorepo 구조는 코드가 한 레포에 모여 있어 관리가 명확하지만, 서비스별 배포 단위 분리가 필요하다.\
ArgoCD는 Monorepo 환경을 다루기에 매우 적합하며, ApplicationSet과 함께 사용하면 대규모 서비스도 쉽게 관리할 수 있다.

***

### _**서비스 디렉토리 구조 설계**_

예시:

```
/services
  /around-api
  /privacy-api
  /batch
/deployments
  /dev
  /prod
```

서비스별 소스와 배포 선언을 분리해두면 diff 계산 및 배포 단위가 명확해진다.

***

### _**ApplicationSet 자동 생성 전략**_

ApplicationSet을 사용하면 디렉토리 기반 자동 Application 생성이 가능하다:

* services/\* → 각 서비스별 Application 생성
* deployments/dev/\* → dev 환경 Application 자동 생성
* deployments/prod/\* → prod 환경 Application 자동 생성

이 구조는 서비스가 10개든 100개든 관리 비용이 동일하다.

***

### _**서비스 변경만 배포하도록 구성하는 방법**_

CI 단계에서 “어떤 서비스가 변경되었는가?”를 분석하고\
해당 서비스의 values.yaml만 업데이트하여 commit하면 된다.

ArgoCD는 그 commit 변화만 감지해 필요한 서비스만 배포한다.\
즉, Monorepo지만 불필요한 모든 서비스를 재배포하지 않는다.





## ArgoCD를 적용했을 때의 전체 배포 흐름 정리

ArgoCD 기반 GitOps 배포는 다음 흐름으로 구성된다:

1. 개발자가 코드를 수정하고 PR을 생성한다
2. master에 merge되면 CI(Jenkins/GitHub Actions)가 Docker 이미지를 빌드한다
3. 새로운 image tag를 생성해 레지스트리에 push한다
4. CI 파이프라인이 Helm values.yaml(또는 manifest)에 새 tag를 업데이트한다
5. Git 변경 사항이 commit → master merge
6. ArgoCD repo-server가 Git 변경을 감지한다
7. Helm 템플릿을 렌더링하고 application-controller가 Diff를 계산한다
8. OutOfSync 발생
9. 자동 또는 수동 Sync 실행
10. 쿠버네티스에 새로운 배포가 적용되며 ArgoCD UI에서 Healthy/Synced 상태 확인

ArgoCD는 Git 변경을 기준으로 클러스터의 상태를 자동 조정하며, 오류 또는 수동 변경에 대해서도 즉시 감지하거나 자동 복구(Self Heal)한다.

***

## 신입 개발자가 반드시 알아야 할 핵심 개념

ArgoCD를 도입한 팀에서 신입 개발자가 빠르게 적응하려면 다음 개념을 반드시 이해해야 한다.

***

### _**Git이 진실의 원천이다**_

ArgoCD 환경에서는 배포는 Git에서 결정된다.\
kubectl로 임의로 수정하는 것은 원칙적으로 금지되며, 모든 변경은 Git commit을 통해 이루어져야 한다.

이 구조를 이해하면 ArgoCD가 왜 Diff를 감지하고, 왜 Self Heal로 되돌리는지 명확해진다.

***

### _**image tag는 배포 트리거이다**_

코드 변경이 실제 배포에 반영되려면 반드시 image tag가 변경되어야 한다.\
CI에서 tag를 생성하는 규칙을 조직마다 다르게 정의할 수 있으나 다음 원칙은 공통이다:

* tag는 자동으로 증가해야 한다
* tag는 한 번 생성되면 변경되지 않아야 한다 (immutable)
* latest 사용 금지

ArgoCD는 이미지 빌드가 아니라 “tag 변경”을 기준으로 배포를 판단한다.

***

### _**Helm chart 변경도 배포 트리거이다**_

Helm 템플릿이나 values.yaml이 바뀌면 그것 자체가 서비스 구성의 변화이기 때문에 ArgoCD는 diff를 감지하고 배포를 진행한다.\
이는 Jenkins 개입 없이도 이루어지며, 개발자는 Git commit만 하면 된다.

***

### _**환경별 Application 분리**_

개발/스테이지/운영 환경은 Sync 정책과 위험도가 완전히 다르다.\
따라서 ArgoCD는 Application을 환경별로 분리해야 한다.

일반적인 구성은 다음과 같다:

* around-api-dev
* around-api-stage
* around-api-prod

이렇게 구성하면 환경별 정책(자동 sync/mannual sync, prune 여부 등)을 명확하게 나눌 수 있다.

***

### _**Secret은 Git에 올리지 않는다**_

GitOps에서 Secret은 반드시 암호화하거나 외부 비밀 저장소로 관리해야 한다.

신입 개발자는 다음 두 가지 방식 중 하나를 팀 정책에 따라 학습해야 한다:

1. Sealed Secrets
2. External Secrets Operator

둘 중 하나만 익혀도 GitOps 환경에서 비밀을 안전하게 유지할 수 있다.

***

## 실무에서 흔히 겪는 문제와 해결 방법

ArgoCD를 막 도입했거나 신입 개발자가 자주 겪는 문제들을 정리하면 다음과 같다.

***

### _**개발자가 kubectl로 수정했는데 왜 원래대로 돌아오나요?**_

ArgoCD Self Heal 기능이 동작한 결과이다.\
Git과 클러스터 상태가 달라지면 ArgoCD는 자동으로 Git 상태로 복원한다.

해결 방법:

* 실무에서는 “kubectl 패치를 하지 않는다”는 운용 규칙을 명확히 한다.
* 긴급 패치가 필요한 경우 Self Heal 옵션을 일시적으로 끈 뒤 작업한다.

***

### _**ArgoCD가 OutOfSync라고 뜨는데 왜 배포가 안 되나요?**_

가능한 원인은 다음과 같다:

* Sync 정책이 manual이다
* values.yaml만 변경되고 image tag는 변경되지 않았다
* prune이 꺼져 있어서 리소스 삭제가 반영되지 않는다
* helm chart 구조 변경으로 orphan 리소스가 남아있다

해결 방법:

* UI에서 Sync 버튼을 눌러 배포 진행
* 또는 Git 상태를 다시 확인하여 commit 정상 여부 확인

***

### _**prod에 위험한 변경이 들어가면 어떻게 하나요?**_

ArgoCD는 GitOps 기반이므로 Git 변경 자체가 배포의 기준이 된다.\
따라서 prod에는 다음 정책을 반드시 둬야 한다:

* prod branch는 보호(branched protected)
* deploy 승인 절차 필수
* manual sync + prune off
* image tag는 immutable
* 운영자만 sync 권한 보유

이 구성대로라면 prod에서 실수로 큰 사고가 날 가능성이 줄어든다.

***

## ArgoCD와 Monorepo 운영의 결합

Monorepo는 하나의 레포 안에 여러 서비스가 있는 구조로 대규모 조직에서 흔히 사용된다.\
ArgoCD는 다음 방식으로 Monorepo와 결합하면 효율이 매우 높아진다.

***

### _**서비스 디렉토리 기준 Application 분리**_

예:

```
/services
  /around-api
  /privacy-api
  /batch-service
```

각 디렉토리가 하나의 Application이 된다.

이렇게 하면:

* 서비스별 배포가 독립된다
* diff 범위가 명확하다
* CI에서 어떤 서비스가 수정되었는지 쉽게 판단 가능

***

### _**ApplicationSet을 사용한 자동 Application 생성**_

ApplicationSet의 git-directory generator를 사용하면 services/\* 디렉토리의 개수만큼 자동으로 Application이 생성된다.

이 방식은 다음과 같은 이점이 있다:

* 서비스 추가 시 Application yaml 파일을 만들 필요가 없다
* 삭제하면 자동 제거
* 디렉토리 구조를 그대로 배포 단위로 활용

대규모 조직에서는 사실상 필수적인 기능이다.

***

### _**변경된 서비스만 배포하는 구조 만들기**_

CI 단계에서 diff를 분석해 특정 서비스만 수정되었는지 판단한 다음, 해당 서비스의 Helm values.yaml만 업데이트하면 된다.\
ArgoCD는 Git 변경을 감지하여 해당 서비스만 OutOfSync 처리한다.

이런 방식 덕분에 Monorepo에서도 불필요한 재배포가 일어나지 않는다.



***
