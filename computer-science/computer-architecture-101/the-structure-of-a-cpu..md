---
description: CPU의 구조
---

# The structure of a CPU.

1. **연산장치 (ALU: Arithmetic Logic Unit)**
   * 산술 연산 (예: 덧셈, 뺄셈) 및 논리 연산 (예: AND, OR, NOT)을 수행합니다.
   * 결과 값의 상태에 따라 상태 레지스터를 설정할 수 있습니다.
2. **제어장치 (Control Unit)**
   * 전체 CPU 및 연결된 다른 하드웨어 장치의 동작을 제어합니다.
   * 메모리에서 명령어를 인출하여 해독하고, 필요한 연산을 ALU에 지시합니다.
3. **레지스터 (Registers)**
   * CPU 내에서 사용되는 작은 저장 공간으로, 매우 빠른 접근 속도를 가집니다.
   * 다양한 목적의 레지스터들이 있으며, 일부 예로는 명령어 레지스터(IR: Instruction Register), 프로그램 카운터(PC: Program Counter), 누산기(ACC: Accumulator), 상태 레지스터(Status Register) 등이 있습니다.
4. **버스 (Bus)**
   * 데이터 버스: CPU와 메모리 및 입출력 장치 간의 데이터 전송을 위한 경로
   * 주소 버스: 메모리 주소 또는 입출력 장치의 주소를 지정하기 위한 경로
   * 제어 버스: CPU와 기타 장치 간의 명령 및 상태 신호를 전송하기 위한 경로
5. **캐시 메모리 (Cache)**
   * 주기억장치와 CPU 사이에 위치하여, 자주 사용되는 데이터나 명령어를 빠르게 접근할 수 있도록 저장합니다.
   * 캐시의 효율성은 CPU의 성능에 큰 영향을 미칩니다.

참조 : [https://namu.wiki/w/CPU/구조와 원리#s-2.1.3](https://namu.wiki/w/CPU/%EA%B5%AC%EC%A1%B0%EC%99%80%20%EC%9B%90%EB%A6%AC#s-2.1.3)
