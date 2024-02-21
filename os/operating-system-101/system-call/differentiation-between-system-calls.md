---
description: 서로 다른 시스템 콜의 구분
---

# Differentiation between system calls

각각의 시스템 콜은 숫자로 구별된다.

\*\*시스템 콜 인터페이스(System Call Interface)\*\*가 이 숫자에 따라 매핑된 테이블을 유지한다. \*\*시스템 콜 테이블(System Call Table)\*\*은 메모리 주소의 모음인데, 각 메모리 주소는 호출한 시스템 콜에 맞는 기능(함수)를 가리키고 있다.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 시스템 콜 호출 시, 시스템 콜의 고유번호를 eax 32bit(rax 64bit) 레지스터에 저장
* system\_call() 함수에는 호출된 시스템 콜 번호와 레지스터들을 스택에 저장하고 올바른 시스템 콜 번호인지 검사 후 sys\_call\_table에서 시스템 콜 번호에 해당하는 함수를 호출한다
