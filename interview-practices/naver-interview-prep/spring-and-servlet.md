# Spring & Servlet

스프링에서 DispatcherServlet의 역할은 무엇인가요?\
그리고 서블릿 컨테이너와 어떤 관계를 갖고 있는지도 설명해주세요.
------------------------------------

* DispatcherServlet은 **Spring MVC의 프론트 컨트롤러** 역할을 수행
* `HttpServlet`을 상속받은 클래스이며, **서블릿 컨테이너(Tomcat 등)** 에 의해 관리됨
* 모든 요청을 받아 **핸들러 탐색, 실행, 응답 렌더링**까지 담당



DispatcherServlet이 요청을 받은 후 내부적으로 어떤 흐름으로 동작하나요?\
핵심 컴포넌트들과 함께 설명해주세요.
--------------------

Client → Filter → DispatcherServlet →\
HandlerMapping → HandlerAdapter → Controller →\
ModelAndView → ViewResolver → View → Response<br>

* Filter: 서블릿 컨테이너 레벨의 전처리
* Interceptor: Spring Handler 전/후 처리
* ModelAndView: 컨트롤러에서 로직 결과와 뷰 정보를 담는 객체
* ViewResolver: 논리 뷰 이름 → 물리 뷰 이름 매핑



Spring MVC에서 HandlerMapping과 HandlerAdapter는 각각 어떤 역할을 하나요?\
그리고 왜 이 둘이 분리되어 있는지 설명해주세요.
---------------------------

| 컴포넌트           | 역할                                         |
| -------------- | ------------------------------------------ |
| HandlerMapping | 요청 URL에 매핑되는 **Handler 객체** 찾기             |
| HandlerAdapter | 해당 Handler를 실제로 **실행할 수 있는 전략 객체** 선택 및 실행 |

* 예시:
  * `@Controller` 메서드 → `RequestMappingHandlerAdapter`
  * `HttpRequestHandler` 구현 → `HttpRequestHandlerAdapter`

→ 덕분에 Spring MVC는 다양한 방식의 Controller를 유연하게 처리할 수 있음 (Adapter Pattern)



그럼 HandlerMapping은 handler를 찾는 역할이고, HandlerAdapter는 그 handler를 실행하는 역할이라는 말이 있어.\
이게 무슨 의미인지, 구체적으로 예시를 들어 설명해줄 수 있을까? (예: 어노테이션 기반 Controller 등)
---------------------------------------------------------------

Spring MVC는 **요청 처리 과정을 유연하게 구성**하기 위해 `HandlerMapping`과 `HandlerAdapter`라는 두 컴포넌트를 분리

#### 🔹 `HandlerMapping`

* **요청 정보(URL, HTTP method 등)** 를 바탕으로 어떤 **Handler 객체**가 이 요청을 처리할지 결정해요.
* 이때의 Handler는 실제로는 **Method 객체, 또는 Bean 객체와 메서드 정보를 담은 HandlerMethod 등**입니다.

🔸 예시:

* 클라이언트가 `/user/123`에 GET 요청 → `@RequestMapping("/user/{id}")`이 있는 Controller 메서드 반환됨.

#### 🔹 `HandlerAdapter`

* `HandlerMapping`이 찾아준 **Handler 객체를 실행할 수 있는 방법을 알고 있는 실행기(Executor)** 역할이에요.
* Handler의 타입에 따라 실행 방식이 다를 수 있기 때문에 Adapter 패턴이 도입되었어요.

🔸 예시:

* 위에서 찾은 `@RequestMapping` 기반 Handler는 `HandlerMethod` 객체이고,\
  이걸 실행할 수 있는 `HandlerAdapter`는 \*\*`RequestMappingHandlerAdapter`\*\*입니다.
* 만약 요청이 `SimpleControllerHandlerAdapter`나 `HttpRequestHandlerAdapter`가 처리하는 타입이라면 해당 Adapter가 실행하게 돼요.



#### 💡 왜 이렇게 분리했을까?

→ **유연성** 때문이에요.

Spring은 다양한 종류의 핸들러를 지원하길 원했어요:

* `@Controller`의 메서드
* `HttpRequestHandler`
* `SimpleController`
* 심지어 외부 라이브러리 기반 핸들러

그래서 요청 → 핸들러 찾기(HandlerMapping), 핸들러 실행(HandlerAdapter)로 나눈 거예요.



### ✅ 예시 흐름

클라이언트가 `/hello`로 GET 요청 시:

1. **DispatcherServlet**이 요청 수신
2. `HandlerMapping`이 이 URL을 처리할 `@Controller` 클래스의 `hello()` 메서드를 찾아냄 → `HandlerMethod`로 래핑
3. `HandlerAdapter` 중 `RequestMappingHandlerAdapter`가 이 HandlerMethod를 실행할 수 있는지 판단
4. 해당 Adapter가 `hello()` 메서드 실행
5. 결과인 `ModelAndView` 반환

***

### ✅ 핵심 요약

| 컴포넌트           | 역할                    | 예시                                  |
| -------------- | --------------------- | ----------------------------------- |
| HandlerMapping | 요청 URL → Handler 결정   | `/users/{id}` → UserController의 메서드 |
| HandlerAdapter | Handler 실행 방법 결정 및 실행 | HandlerMethod → 메서드 invoke          |
