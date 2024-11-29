---
description: 소수 찾기, 에라토스테네스의 체
---

# Sieve of Eratosthenes

### 에라토스테네스의 체 란? <a href="#undefined" id="undefined"></a>

소수를 판별하는 알고리즘이다.

소수들을 대량으로 빠르고 정확하게 구하는 방법이다.

#### 단일 숫자 소수 여부 확인 <a href="#undefined" id="undefined"></a>

어떤 수의 소수의 여부를 확인 할 때는, 특정한 숫자의 제곱근 까지만 약수의 여부를 검증하면 O(N^1/2)의 시간 복잡도로 빠르게 구할 수 있다.

수가 수(N이라고 가정)를 나누면 몫이 생기는데, 몫과 나누는 수 둘 중 하나는 N 제곱근 이하이기 때문이다.

만약, 대량의 소수를 한꺼번에 판별해야할 경우는 '에라토스테네스의 체'를 이용한다.

#### 에라토스테네스의 체 원리 <a href="#undefined" id="undefined"></a>

에라토스테네스의 체는 가장 먼저 소수를 판별할 범위만큼 배열을 할당하여, 해당하는 값을 넣어주고, 이후에 하나씩 지워나가는 방법을 이용한다.

1. 배열을 생성하여 초기화한다.
2. 2부터 시작해서 특정 수의 배수에 해당하는 수를 모두 지운다.(지울 때 자기자신은 지우지 않고, 이미 지워진 수는 건너뛴다.)
3. 2부터 시작하여 남아있는 수를 모두 출력한다.

#### 에라토스테네스의 체 구현하기 <a href="#undefined" id="undefined"></a>

```python
def prime_list(start, end):
    # 에라토스테네스의 체 초기화: n개 요소에 True 설정(소수로 간주)
    sieve = [True] * end
    if start <= 0:
        start = 2

    # n의 최대 약수가 sqrt(n) 이하이므로 i=sqrt(n)까지 검사
    m = int(end ** 0.5)
    for i in range(2, m + 1):
        if sieve[i]:  # i가 소수인 경우
            for j in range(i + i, end, i):  # i이후 i의 배수들을 False 판정
                sieve[j] = False

    # 소수 목록 산출
    return [i for i in range(start, end) if sieve[i]]
```

시간 복잡도는 **O(NloglogN)**&#xC774;지만, 공간복잡도가 **O(n)**&#xC774;다.
