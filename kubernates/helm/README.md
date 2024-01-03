---
description: 쿠버네티스 패키지 매니저
---

# Helm

helm은yum, apt 같은 패키지매니저이다.



Helm의 기능

* 새로운 차트 생성
* chart로 chart archive files로 패키지화
* chart가 저장되는 chart 저장소와 상호작용
* Kubernates cluster에 chart의 설치 및 제거, 릴리즈 주기 관리

Helm의 구성

* CHart : helm 패키지. k8s application, tool, service를 구동하는데 필요한 resource의 집합(mariadb, kafka, redis 등)
* Repoositoory : helm chart를 모아두고 공유하는 저장소
  * 원격 저장소 - bitnami, Artifact Hub
  * 로컬 저장소 - Chart Museum
