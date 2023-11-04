---
description: 컨텍스트 스위칭할 때 저장되는 레지스터
---

# Registers saved during context switching

1. **일반 레지스터 (General Purpose Registers):** 이들은 일반적인 연산에 사용되는 레지스터입니다.
2. **프로그램 카운터 (Program Counter, PC):** 다음에 실행될 명령어의 주소를 저장하고 있습니다.
3. **스택 포인터 (Stack Pointer):** 현재 스택의 최상위를 가리키는 포인터입니다.
4. **프로세스 상태 레지스터 (Process Status Register, PSR):** 현재 프로세스의 상태 정보(예: 실행 모드, 인터럽트 허용 상태)를 저장하고 있습니다.
5. **메모리 관리 레지스터 (Memory Management Registers):** 페이지 테이블 기반의 메모리 관리 시스템에서 사용됩니다. 이는 페이징 정보, 세그먼트 정보 등을 포함할 수 있습니다.
