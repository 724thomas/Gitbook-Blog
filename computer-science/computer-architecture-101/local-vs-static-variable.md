---
description: 지역변수 vs Static 변수
---

# Local vs Static Variable

#### 지역 변수 (Local Variable)

* **정의**: 함수 내부에서 선언되며, 함수가 호출될 때 생성되고 함수가 종료될 때 소멸하는 변수입니다.
* **메모리 할당**: 스택 메모리에 할당됩니다.
* **수명**: 함수가 실행되는 동안에만 존재합니다. 함수가 반환되면, 해당 지역 변수는 스택에서 제거됩니다.
* **초기화**: 일반적으로 자동으로 초기화되지 않으며, 프로그래머가 명시적으로 초기화해야 사용할 수 있습니다 (언어에 따라 다를 수 있습니다).\
  \


**Static 변수**

* **정의**: 함수 내부 또는 클래스 내부에서 선언되지만, `static` 키워드를 사용하여 프로그램의 실행이 시작될 때 한 번만 초기화되고 프로그램이 종료될 때까지 메모리에 남아 있는 변수입니다.
* **메모리 할당**: 데이터 영역의 정적 영역에 할당되거나, 경우에 따라 특수한 `bss` 세그먼트에 할당됩니다.
* **수명**: 프로그램의 실행이 시작될 때부터 종료될 때까지 유지됩니다.
* **초기화**: 자동으로 0 또는 null로 초기화되는 경우가 많습니다 (언어에 따라 다를 수 있습니다).



```c
#include <stdio.h>

void function() {
    int local_variable = 5; // 지역 변수, 함수 호출 시마다 스택에 할당
    static int static_variable = 5; // static 변수, 프로그램 시작 시 데이터 영역에 할당

    printf("Local: %d, Static: %d\n", local_variable, static_variable);

    local_variable++;
    static_variable++;
}

int main() {
    function(); // 첫 호출: Local: 5, Static: 5
    function(); // 두 번째 호출: Local: 5, Static: 6
    return 0;
}

```

위 코드에서 `local_variable`은 함수가 호출될 때마다 새로운 값으로 초기화되는 지역 변수입니다. 반면, `static_variable`은 프로그램 시작 시에 한 번만 초기화되고, 함수가 호출될 때마다 그 값이 유지되어 증가합니다.
