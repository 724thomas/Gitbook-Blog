---
description: 시스템 콜의 유형
---

# 유형

시스템콜은 크게 6가지로 분류할 수 있다.

1. **프로세스 제어 (Process Control)**
   * 끝내기(exit), 중지 (abort)
   * 적재(load), 실행(execute)
   * 프로세스 생성(create process) - fork
   * 프로세스 속성 획득과 속성 설정
   * 시간 대기 (wait time)
   * 사건 대기 (wait event)
   * 사건을 알림 (signal event)
   * 메모리 할당 및 해제
2. **파일 조작 (File Manipulation)**
   * 파일 생성 / 삭제 (create, delete)
   * 열기 / 닫기 / 읽기 / 쓰기 (open, close, read, wirte)
   * 위치 변경 (reposition)
   * 파일 속성 획득 및 설정 (get file attribute, set file attribute)
3. **장치 관리 (Device Manipulation)**
   * 하드웨어의 제어와 상태 정보를 얻음 (ioctl)
   * 장치를 요구(request device), 장치를 방출 (relese device)
   * 읽기 (read), 쓰기(write), 위치 변경
   * 장치 속성 획득 및 설정
   * 장치의 논리적 부착 및 분리
4. **정보 유지 (Information Maintenance)**
   * getpid(), alarm(), sleep()
   * 시간과 날짜의 설정과 획득 (time)
   * 시스템 데이터의 설정과 획득 (date)
   * 프로세스 파일, 장치 속성의 획득 및 설정
5. **통신 (Communication)**
   * pipe(), shm\_open(), mmap()
   * 통신 연결의 생성, 제거
   * 메시지의 송신, 수신
   * 상태 정보 전달
   * 원격 장치의 부착 및 분리
6. **보호 (Protection)**
   * chmod()
   * umask()
   * chown()
