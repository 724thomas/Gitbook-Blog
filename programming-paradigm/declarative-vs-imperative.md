---
description: 선언형, 명령형의 차이
---

# Declarative vs Imperative

<div data-full-width="true"><figure><img src="/broken/files/uWZP9rng4rQqfcnNGiJh" alt=""><figcaption></figcaption></figure></div>

요약:

* **선언형 프로그래밍**: "무엇을(What)"에 집중하여, 결과를 기술하는 방식. 내부 동작에 대한 관심이 줄어듦.
* **명령형 프로그래밍**: "어떻게(How)"에 집중하여, 절차적으로 문제를 해결하는 방식. 상태와 제어 흐름을 직접 관리.



## 1. 선언형 프로그래밍

선언형 프로그래밍은 **무엇을(What)** 할 것인지에 중점을 두며, **프로그램이 수행해야 할 작업의 결과를 설명**합니다. 이 패러다임에서는 **어떻게** 그것을 해야 하는지에 대한 명령보다는, 목표가 무엇인지를 정의합니다. 로직이 더 추상화되어 있으며, 상태 변화나 세부적인 작업 절차를 명시하지 않아도 됩니다.

### 1.1. 특징

* 상태 변화와 제어 흐름에 대한 관심이 줄어듦.
* 목표 또는 해결해야 할 문제의 결과만 명시하고, 그 과정은 시스템에 맡김.
* 함수형 프로그래밍이 대표적인 선언형 프로그래밍 방식에 속함.



## 2. 명령형 프로그래밍

명령형 프로그래밍은 **프로그램의 동작을 단계별로 설명**하며, **어떻게(How)** 할 것인지에 중점을 둡니다. 즉, 프로그래머가 프로그램이 수행해야 할 작업을 순차적으로 명시하는 방식입니다. 이 방식은 컴퓨터의 CPU 동작과 유사하게 작동하며, 상태 변화를 통해 목표를 달성합니다.

### 2.1. 특징

* 상태 변화 (변수 값의 변경)와 제어 흐름(조건문, 반복문 등)을 통해 로직을 구성.
* 특정 작업을 수행하기 위한 절차를 세부적으로 작성.
* 메모리 관리, 변수 사용, 루프 등의 직접적인 제어가 가능.



## 3. 예시

가정: (0, 0)에 있는 A점에서 (2, 2)에 있는 B점까지 가야하는 상황입니다.\
A: (0, 0), B: (2, 2)

### 3.1. 명령형 프로그래밍 (어떻게)

"(0, 0)에서 위로 2칸, 오른쪽으로 2칸 가줘" 처럼 어떻게 동작하는지 선언을 해주는 것입니다. 선언형 프로그래밍을 사용하게되면 **어떻게**를 통해 동작하는 방식이 딱 1개만 존재합니다.

### 3.2. 명령형 프로그래밍 예시

```java
public getTotal(int count){
    int sum = 0;
    for (int i = 0; i <= count; i++) { // <-- 어떻게 부분. 직접 어떻게 해야할지 정의
        sum += i;
    return sum;
}
```

```java
public List<Integer> squareNumbers(List<Integer> numbers) {
    List<Integer> squaredNumbers = new ArrayList<>();
    for (Integer number : numbers) {
        squaredNumbers.add(number * number); // <-- 어떻게 해야할지 정의가 되어있음
    }
    return squaredNumbers;
}

```

### 3.3. 선언형 프로그래밍 (무엇)

"A와 B를 만나게 해줘" 라고 하는 것입니다.

내부에서는 이를 어떻게 정의하는지 잘 모릅니다. 내부에서는 위로 2칸, 오른쪽으로 2칸을 갈 수도 있지만, 오른쪽으로 2칸, 위로 2칸을 가는 방법도 있습니다. 또한, 최적의 경로가 아닌, 위로 3칸, 오른쪽으로 2칸, 아래로 1칸을 갈 수도 있습니다.

### 3.4. 선언형 프로그래밍&#x20;

```java
public int getTotal(int count) {
    int[] sum = {0}; // 값을 참조형으로 유지하기 위해 배열 사용
    IntStream.iterate(0, i -> i + 1)
             .limit(count + 1)
             .forEach(i -> sum[0] += i); 
    // iterate, limit, foreach가 각각 어떻게 동작하는지 모르지만 무엇을 무엇을 할지 표현이
    // 가능하다는 점에서 선언형
    return sum[0];
}
```

```java
public List<Integer> squareNumbers(List<Integer> numbers) {
    return numbers.stream()
                  .map(number -> number * number)
                  .collect(Collectors.toList());
// stream, map, collect 모두 내부 동작을 모르기때문에 선언형입니다.
// 그래서추가로 Collectors.toList()도 선언형이 됩니다.
}
```
