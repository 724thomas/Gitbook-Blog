---
description: 이상현상, 함수적 종속성, 정규화
---

# Week4 Anomaly, Functional Dependency, Normalization

## 이상 현상이 뭘까요?

* 데이터베이스에서 테이블 설계가 비효율적이거나 잘못되었을 때 발생하는 데이터 무결성 문제. 보통 **중복된 데이터** 때문에 발생
* 삽입 이상: 데이터를 삽입할때 불필요하거나 잘못된 데이터를 추가해야 하는 상황.\
  예를 들어, 테이블이 너무 많은 정보를 가지고 있어서 필요한 정보를 추가하려면 다른 불필요한 정보도 함께 추가해야 하는 경우
* 갱신 이상: 데이터를 수정할 때 중복된 여러 항목에서 모두 수정해야 하며, 일부만 수정될 경우 데이터 불일치가 발생하는 문제\
  예를 들어, 한 테이블에 동일한 정보가 여러 군데 저장되어 있고, 한 곳만 수정하면 일관성이 깨집니다.
* 삭제 이상: 데이터를 삭제할 때 의도치 않게 다른 중요한 데이터까지 함께 삭제되는 문제\
  특정 데이터를 삭제하면 그와 관련된 다른 중요한 데이터도 함께 삭제 또는 삭제되야하는 데이터가 남아있는 상황

정규화 과정을 통해 최소화하거나 해결할 수 있습니다.



## 삽입 이상에 대해서 설명해주세요.

* 제대로 정규화가 이루어지지않은 테이블에 데이터를 삽입하게 될때, 불완전한 정보를 삽입하거나 NULL 값을 넣어야하는 상황.
* 부서를 만들면 강제로 부서에 있는 직원 데이터도 같이 넣어야하는 문제



## 갱신 이상에 대해서 설명해주세요.

* 여러 곳에 중복된 데이터가 저장되어 있을때, 그 데이터를 수정하면 모든 위치의 데이터도 갱신해야합니다.
* 일부 데이터만 수정하게되면 데이터 일관성이 깨져 서로 다른 정보가 공존하는 문제 발생. (데이터 불일치)



## 삭제 이상에 대해서 설명해주세요.

* 정규화가 제대로 이루어지지 않아서, 데이터를 삭제할때 삭제되지 말아야하는 데이터도 같이 삭제되는 문제.
* 또는 중복된 데이터가 존재하여 삭제 되어야하는 데이터가 같이 삭제가 되지 않는 문제



## 함수 종속성(Functional Dependency)가 뭔가요?

* 데이터베이스에서 **어떤 속성의 값이 다른 속성의 값을 고유하게 결정**하는 관계를 의미.\
  즉, 하나의 속성 값이 주어지면, 다른 속성 값이 반드시 하나로 결정되는 경우
* **학번 → 이름**: 학번이 주어지면, 그 학번에 해당하는 학생의 이름이 결정됩니다. 이때 **학번**은 **결정자(Determinant)**이고, **이름**은 **종속자(Dependent)**
* **완전 함수적 종속, 부분 함수적 종속**



## 완전 함수적 종속(Full Functional Dependency)가 뭔가요?

* **완전 함수적 종속**이란, **어떤 속성(B)**이 **기본키(A)** 전체에 대해 완전히 종속되어 있는 경우를 의미.\
  즉, **기본키의 일부가 아닌 전체**에 의해 다른 속성이 결정될 때

#### 예시: 학생 수강 테이블

| **학생 ID** | **수업 ID** | **교수명** | **점수** |
| --------- | --------- | ------- | ------ |
| 1001      | C101      | 김교수     | 90     |
| 1001      | C102      | 이교수     | 85     |
| 1002      | C101      | 김교수     | 95     |

**테이블의 기본키:**

**학생 ID**와 **수업 ID**를 **복합 기본키**로 사용합니다. 이 두 속성이 함께 있어야 학생이 어떤 수업을 듣고 있는지 고유하게 식별할 수 있습니다.

**1. 완전 함수적 종속:**

**점수**는 **학생 ID**와 **수업 ID**가 둘 다 주어져야만 결정됩니다.

* 예를 들어, 학생 ID `1001`과 수업 ID `C101`을 함께 사용해야 그 학생이 김교수의 수업을 듣고, **점수가 90점**이라는 것을 알 수 있습니다.
* **학생 ID**나 **수업 ID** 중 하나만으로는 점수가 무엇인지 알 수 없습니다. **두 개의 속성이 모두 있어야 점수가 결정되기 때문에, 점수는 완전 함수적 종속**입니다.

**2. 부분 함수적 종속과의 차이:**

만약 **교수명**이 **수업 ID**에만 종속되어 있다면, 이것은 **부분 함수적 종속**에 해당합니다. 왜냐하면, 수업 ID만 주어져도 교수명을 알 수 있기 때문입니다.

* 예를 들어, 수업 ID `C101`은 언제나 **김교수**가 담당합니다. 즉, **수업 ID** 하나만으로도 교수명을 결정할 수 있으므로 교수명은 수업 ID에 대해 **부분 함수적 종속**입니다.



## 부분 함수적 종속이 뭔가요?

* 어떤 속성(B)이 **기본키 전체가 아닌 일부**에만 종속되는 경우\
  이때, 속성 B는 기본키 일부에만 종속되므로, **기본키의 나머지 부분이 없어도** B의 값이 결정될 수 있습니다.

#### 예시:

| 학생 ID (A1) | 수업 ID (A2) | 교수명 (B) |
| ---------- | ---------- | ------- |
| 1001       | C101       | 김교수     |
| 1002       | C101       | 김교수     |
| 1001       | C102       | 이교수     |

* 여기서 **학생 ID**와 **수업 ID**가 **복합 기본키**입니다.
* 그러나 **교수명**(B)은 **수업 ID(A2)** 하나만으로 결정됩니다. 즉, **학생 ID**는 필요 없이 **수업 ID**만 있으면 교수명이 결정됩니다. 이는 **부분 함수적 종속**의 예입니다.
  * **교수명**은 기본키 전체(학생 ID + 수업 ID)에 종속되지 않고, **수업 ID**에만 종속됩니다.



## 이행적 함수적 종속(Transitive Functional Dependency)가 뭔가요?

* **이행적 함수적 종속**이란, **A → B**와 **B → C**라는 두 개의 함수적 종속이 있을 때, **A → C**의 종속 관계가 성립하는 것\
  즉, **속성 A**가 **속성 B**를 결정하고, **속성 B**가 **속성 C**를 결정할 때, 결과적으로 **속성 A**가 **속성 C**를 결정하는 관계
* **제3정규형(3NF)**에서는 이행적 종속을 제거하는 것이 목표

#### 예시:

**직원 정보 테이블**

| 직원 ID (A) | 부서 ID (B) | 부서장 이름 (C) |
| --------- | --------- | ---------- |
| 1001      | D01       | 김부장        |
| 1002      | D02       | 이부장        |
| 1003      | D01       | 김부장        |

* **직원 ID**가 **부서 ID**를 결정할 수 있습니다. 즉, **A → B**.
* **부서 ID**가 **부서장 이름**을 결정할 수 있습니다. 즉, **B → C**.
* 결과적으로 **직원 ID**는 **부서장 이름**을 간접적으로 결정할 수 있습니다. 즉, **A → C**.
  * 이는 **이행적 함수적 종속**의 예입니다. 직원 ID는 부서 ID를 통해 부서장 이름을 간접적으로 결정하고 있습니다.

#### 요약:

* **이행적 함수적 종속**은 간접적인 종속 관계를 의미합니다.
* A → B, B → C일 때, A → C도 성립합니다.



## 정규화가 뭔가요?

* 설계 과정에서 **데이터 중복을 줄이고, 이상 현상(삽입, 갱신, 삭제 이상)을 방지**하기 위해 테이블을 **최적화**하는 과정

**1. 제1정규형(1NF):**

* 테이블의 모든 속성이 **원자값**(더 이상 나눌 수 없는 값)으로 이루어져 있어야 합니다. 즉, **중첩된 데이터나 중복된 값**이 없도록 데이터를 정리하는 단계입니다.

**2. 제2정규형(2NF):**

* **제1정규형을 만족**하고, **기본키에 대해 완전 함수적 종속**을 만족해야 합니다. 즉, 기본키의 일부에만 종속되는 속성(부분 함수적 종속)을 제거합니다.

**3. 제3정규형(3NF):**

* **제2정규형을 만족**하고, **이행적 함수적 종속**을 제거합니다. 즉, 기본키가 아닌 속성들 간의 간접적인 종속 관계를 없애는 단계입니다.

**4. BCNF(Boyce-Codd Normal Form):**

* **제3정규형을 강화**한 형태로, 테이블의 모든 **결정자**가 반드시 **후보키**여야 한다는 규칙을 따릅니다. 복잡한 종속성을 제거하여 더 높은 무결성을 확보합니다.



## 반정규화가 뭔가요?

* **정규화된 데이터베이스**에서 **성능 향상**을 위해 **일부 정규화를 되돌리는 과정**
* **정규화로 인해 쪼개진 테이블들을 다시 결합**하거나, **중복 데이터를 일부 허용**하여 쿼리 성능을 최적화하는 방법
* **JOIN 연산**이 많아져 성능이 저하