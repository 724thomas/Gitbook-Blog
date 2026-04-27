# Decorator Pattern in Python

## 개념

"함수를 수정하지 않고, 함수의 기능을 덧붙이는 패턴"



파이썬에서는 @ 문법으로 표현됩니다.

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("before")
        result = func(*args, **kwargs)
        print("after")
        return result
    return wrapper

@my_decorator
def hello():
    print("hello")

hello()
```

위 코드는 실제로는 아래와 같이 동작합니다.

```
hello = my_decorator(hello)
```



## FastAPI에서 데코레이터의 역할

FastAPI에서 데코레이터는 단순 기능 추가가 아니라:

#### 2가지 역할을 동시에 수행

1. **라우팅 등록**
2. **메타데이터 정의 (method, path, schema 등)**



### 2.1 기본 예시

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users")
def get_users():
    return ["user1", "user2"]
```

***

### 내부적으로 일어나는 일

이 코드는 사실상:

```python
def get_users():
    return ["user1", "user2"]

app.get("/users")(get_users)
```

***

#### `app.get()`의 정체

```python
def get(self, path: str):
    def decorator(func):
        self.router.add_api_route(
            path=path,
            endpoint=func,
            methods=["GET"]
        )
        return func
    return decorator
```

***

### 핵심 포인트

#### 1. 함수 실행이 아니라 “등록”이다

```python
@app.get("/users")
```

→ 함수 실행 X\
→ 라우터에 endpoint로 등록 O

***

#### 2. 함수 자체는 그대로 유지됨

데코레이터는 `func`를 그대로 반환하기 때문에:

```python
return func
```

→ 실제 실행은 FastAPI 내부에서 나중에 호출됨

***

## 3. FastAPI 데코레이터 구조 (요약)

```
@app.get("/path")
   ↓
decorator(func)
   ↓
router.add_api_route(...)
   ↓
ASGI 서버 요청 들어오면
   ↓
해당 func 실행
```

***

## 4. 실전: 커스텀 데코레이터 (중요)

FastAPI에서는 인증, 로깅, 트랜잭션 같은 걸 직접 데코레이터로 만들 수 있습니다.

***

### 4.1 예제: 로깅 데코레이터

```python
from functools import wraps

def log_decorator(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        print("request start")
        result = await func(*args, **kwargs)
        print("request end")
        return result
    return wrapper
```

***

### 적용

```python
@app.get("/users")
@log_decorator
async def get_users():
    return ["user1", "user2"]
```

***

### 실행 흐름

```
request → FastAPI router → wrapper → 실제 함수
```

***

## 5. 주의할 점 (실무에서 중요)

### 5.1 async 반드시 유지해야 함

```python
async def wrapper(...)
```

→ 안하면 FastAPI에서 await 깨짐

***

### 5.2 wraps 필수

```python
@wraps(func)
```

이거 없으면:

* OpenAPI 문서 깨짐
* 함수 이름/타입 힌트 손실

***

### 5.3 request 객체 접근 문제

FastAPI는 DI 기반이라:

```python
def wrapper(*args, **kwargs)
```

로는 request 접근이 애매함

→ 해결 방법:

#### 방법 1: kwargs에서 꺼내기

```python
request = kwargs.get("request")
```

#### 방법 2 (추천): Depends 사용

***

## 6. 데코레이터 vs Depends (중요한 설계 포인트)

FastAPI에서는 데코레이터보다 `Depends`를 더 많이 씀

***

### 데코레이터 방식

```python
@auth_required
async def endpoint():
```

문제:

* DI와 충돌
* request/context 접근 불편

***

### Depends 방식 (권장)

```python
from fastapi import Depends

def auth():
    print("auth check")

@app.get("/users")
async def get_users(dep=Depends(auth)):
    return []
```

***

### 차이

| 항목          | 데코레이터 | Depends |
| ----------- | ----- | ------- |
| DI 지원       | ❌ 약함  | ✅ 강력    |
| 테스트         | ❌ 어려움 | ✅ 쉬움    |
| FastAPI 친화성 | ❌     | ✅       |
| 재사용성        | 보통    | 높음      |

***

## 7. 언제 데코레이터를 써야 하나?

#### 적합한 경우

* 로깅
* 공통 전처리/후처리
* 성능 측정
* retry / circuit breaker

***

#### 비추천

* 인증/인가
* DB 세션 관리
* request 기반 로직

→ 이런건 `Depends`
