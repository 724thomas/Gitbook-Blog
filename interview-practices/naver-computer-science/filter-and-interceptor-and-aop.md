# Filter & Interceptor & AOP

## Spring에서 Filter와 Interceptor의 차이를 설명해주세요. 언제 각각을 사용하는 것이 적절한지도 함께 설명해보세요

| 항목      | Filter                 | Interceptor          |
| ------- | ---------------------- | -------------------- |
| 컨테이너    | 서블릿 컨테이너               | 스프링 컨테이너             |
| 위치      | DispatcherServlet 이전   | DispatcherServlet 이후 |
| 적용 범위   | 요청 전체 (서블릿 단위)         | 컨트롤러 단위              |
| 주 사용 용도 | 보안, 인코딩, CORS 등        | 인증, 권한, 로깅 등         |
| API     | `javax.servlet.Filter` | `HandlerInterceptor` |



## 인터셉터에서 전처리와 후처리를 담당하는 각각의 메서드는 무엇이고, 그 메서드들이 호출되는 정확한 타이밍은 언제인가요?

#### **Interceptor의 메서드 동작 시점**

* `preHandle()` → 컨트롤러 실행 전
* `postHandle()` → 컨트롤러 실행 후, View 렌더링 전
* `afterCompletion()` → View 렌더링 완료 후



Spring의 `HandlerInterceptor` 인터페이스는 **3개의 주요 메서드**를 통해 전처리, 후처리를 담당해요.

preHandle() → Controller 실행 → postHandle() → View 렌더링 → afterCompletion()

#### `preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)`

* **타이밍**: 컨트롤러의 메서드가 호출되기 **직전**
* **반환값**: `true`면 다음 단계 진행, `false`면 요청 중단
* **용도**: 인증, 권한 확인, 로깅 시작, 요청 시간 기록 등

#### `postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)`

* **타이밍**: 컨트롤러 로직 실행 **직후**, View 렌더링 **이전**
* **용도**: Model에 공통 데이터 추가, 로깅 마무리 등

#### `afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)`

* **타이밍**: View까지 렌더링을 마친 **완료 후**
* **용도**: 리소스 정리, 예외 로깅, 트랜잭션 종료 처리 등





## `preHandle()`에서 `false`를 반환하면 어떤 일이 발생하고, 이 상황을 실무에서는 주로 어떤 목적으로 활용하나요?

#### **preHandle()에서 false 반환 시**

* 컨트롤러 호출 **중단**
* 이후의 `postHandle()` / `afterCompletion()` **호출되지 않음**
* 인증 실패 처리 등에 주로 사용됨

```java
if (!isAuthenticated(request)) {
    response.sendRedirect("/login");
    return false;
}
```





## 인터셉터와 AOP의 기능이 일부 겹치는 것처럼 보이는데, 둘의 차이점은 무엇이며, 어떤 기준으로 선택해야 할까요?

| 항목      | Interceptor           | AOP                |
| ------- | --------------------- | ------------------ |
| 적용 대상   | HTTP 요청 흐름            | 스프링 빈 메서드 (서비스 계층) |
| 주요 용도   | 인증/권한, 요청 흐름 제어       | 로깅, 트랜잭션, 공통 관심사   |
| 설정 방식   | `WebMvcConfigurer`    | `@Aspect` + 포인트컷   |
| 파라미터 접근 | HttpServletRequest 중심 | 메서드 인자/리턴값 중심      |

### 🔍 어떤 기준으로 선택할까?

| 상황                           | 적절한 선택          |
| ---------------------------- | --------------- |
| 인증/권한 체크, 요청/응답 로깅           | **Interceptor** |
| 로직 실행 전후 트랜잭션, 로깅, 예외 공통 처리  | **AOP**         |
| 컨트롤러 진입 전 요청을 걸러내야 함         | **Interceptor** |
| 서비스 계층에서 공통 처리 로직을 분리하고 싶을 때 | **AOP**         |

#### 예시로 이해해보기

* 로그인 여부 확인 → `Interceptor`에서 처리
* 모든 서비스 로직의 실행 시간 측정 → `@Around` AOP 사용

## AOP에서 @Around 어노테이션을 사용할 때, join point를 직접 제어할 수 있는데요. 이때 `ProceedingJoinPoint`는 어떤 역할을 하며, 이걸 이용해서 우리가 어떤 작업을 할 수 있을까요?

* `@Around` 어드바이스에서 핵심 로직 호출을 제어할 수 있는 객체
* `joinPoint.proceed()` 호출 전/후에 원하는 로직을 끼워넣을 수 있음
* 로깅, 성능 측정, 예외 처리 등에서 활용됨

