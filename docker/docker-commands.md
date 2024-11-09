---
description: 도커 명령어 정리
---

# Docker commands

## **1. 이미지 관련 명령어**

* `docker image ls`: 다운로드한 이미지 목록 조회
* `docker image rm redis`: `redis` 이미지를 삭제
* `docker image rm f5da`: `f5da`로 시작하는 이미지 삭제 (단, 컨테이너에서 사용 중인 이미지는 삭제되지 않음)
* `docker image rm -f f5da`: `f5da`로 시작하는 이미지를 강제로 삭제 (단, 실행 중인 컨테이너에서 사용 중인 이미지는 제외)
* `docker image rm $(docker images -q)`: 사용되지 않는 모든 이미지 삭제
* `docker image rm -f $(docker images -q)`: 실행 중인 컨테이너의 이미지를 제외한 모든 이미지 삭제

## **2. 컨테이너 관련 명령어**

### **컨테이너 생성 및 실행**

* `docker create nginx`: `nginx` 컨테이너 생성 (실행하지 않음)
* `docker run redis`: 이미지가 없으면 자동으로 다운로드 후, `redis` 이미지를 기반으로 컨테이너 생성 및 실행 (포그라운드 실행)
* `docker run -d redis`: `redis` 컨테이너를 백그라운드에서 실행
* `docker run -d --name my-redis-server redis`: `my-redis-server`라는 이름으로 백그라운드에서 `redis` 컨테이너 실행
* `docker run -d -p [호스트포트]:[컨테이너 포트] nginx`: 호스트의 특정 포트를 컨테이너의 지정 포트와 연결하여 `nginx` 컨테이너 실행

### **컨테이너 상태 확인 및 관리**

* `docker ps`: 현재 실행 중인 컨테이너 목록 조회
* `docker ps -a`: 모든 컨테이너 목록 조회 (실행 중, 중지된 상태 포함)
* `docker start defc`: `CONTAINER ID`가 `defc`로 시작하는 컨테이너 실행
* `docker stop defc`: `CONTAINER ID`가 `defc`로 시작하는 실행 중인 컨테이너 중지
* `docker restart defc`: `CONTAINER ID`가 `defc`로 시작하는 컨테이너를 중지 후 재시작
* `docker kill defc`: `CONTAINER ID`가 `defc`로 시작하는 실행 중인 컨테이너 강제 중지

### **컨테이너 삭제**

* `docker rm defc`: `CONTAINER ID`가 `defc`로 시작하는 중지된 컨테이너 삭제
* `docker rm $(docker ps -qa)`: 모든 중지된 컨테이너를 일괄 삭제
* `docker rm -f 868`: `868`로 시작하는 컨테이너를 강제로 삭제 (실행 중인 경우 중지 후 삭제)

### **컨테이너 로그 관리**

* `docker logs ee6`: `ee6`로 시작하는 컨테이너의 전체 로그 출력
* `docker logs --tail 10 ee6`: 가장 최근 10줄의 로그만 출력
* `docker logs -f ee6`: 실시간으로 로그를 스트리밍
* `docker logs --tail 0 -f 6c3`: 가장 최근 로그부터 실시간으로 새로운 로그를 스트리밍 (기존 로그는 표시하지 않고 새로운 로그부터 확인)

### **컨테이너 모니터링 및 디버깅**

* `docker exec -it [컨테이너 ID 또는 이름] /bin/bash`: 실행 중인 컨테이너에 접속하여 터미널을 엽니다. (컨테이너 내부 상태 확인이나 문제 해결에 유용)
* `docker inspect [컨테이너 ID 또는 이름]`: 특정 컨테이너의 상세 정보를 JSON 형식으로 출력 (네트워크 설정, IP 주소 등 포함)
* `docker stats [컨테이너 ID 또는 이름]`: 특정 컨테이너의 실시간 리소스 사용량(메모리, CPU, 네트워크 등)을 확인
* `docker top [컨테이너 ID 또는 이름]`: 컨테이너 내에서 실행 중인 프로세스 목록 확인

## **3. 기타 자주 쓰이는 명령어**

* `docker network ls`: 생성된 네트워크 목록 조회
* `docker volume ls`: 생성된 볼륨 목록 조회
* `docker system prune`: 사용하지 않는 모든 컨테이너, 이미지, 네트워크, 볼륨을 삭제하여 디스크 공간 확보
* `docker-compose up`: `docker-compose.yml` 파일을 사용하여 여러 개의 컨테이너를 동시에 시작 (Docker Compose 필요)
* `docker-compose down`: `docker-compose.yml`로 실행 중인 모든 컨테이너를 중지하고 삭제 (네트워크와 볼륨도 포함)
