---
description: 고정소수점 vs 부동소수점
---

# Fixed-point vs Floating-point

1.  고정소수점 (Fixed-Point)\
    ![](<../../../.gitbook/assets/image (110).png>)\


    고정소수점 방식에서는 소수점의 위치가 고정되어 있습니다.

    예를 들어, 16비트에서 소수점 앞에 8비트, 뒤에 8비트를 할당한다고 하면, 소수점 위치는 항상 같은 위치에 있습니다.

    * **장점**
      * 연산이 단순하고 빠르다.
      * 하드웨어 구현이 간단하다.
    * **단점**
      * 표현 범위가 제한적이다.
      * 큰 수와 작은 수의 연산에서 정밀도 손실이 발생할 수 있다.
2.  부동소수점 (Floating-Point)\
    ![](<../../../.gitbook/assets/image (111).png>)\


    부동소수점 방식에서는 수의 실제 값을 가수(Mantissa)와 지수(Exponent)의 조합으로 표현합니다.

    소수점의 위치가 "부동"하기 때문에 이러한 이름이 붙었습니다.

    * **장점**
      * 큰 범위의 숫자를 표현할 수 있다.
      * 정밀도가 높다 (특히 큰 범위의 숫자 연산에서).
    * **단점**
      * 연산이 복잡하고 상대적으로 느리다.
      * 하드웨어 구현이 복잡하다.

실제로, 부동소수점 연산은 특수한 하드웨어 유닛, 즉 FPU (Floating Point Unit)에서 처리되곤 합니다.

고정소수점은 종종 임베디드 시스템과 같은 자원이 제한된 환경에서 선택되는 반면, 부동소수점은 과학적 연산이나 그래픽스 처리와 같은 곳에서 선호됩니다.

