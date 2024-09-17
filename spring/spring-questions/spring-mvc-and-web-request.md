---
description: 스프링 MVC와 웹 요청 처리
---

# Spring MVC & Web Request

**Spring MVC와 요청 처리 과정**

## Spring MVC란 무엇이며, 그 동작 원리를 설명해주세요.

* **Model-View-Controller (MVC) 패턴**을 기반으로 동작하는 웹 프레임워크
* Model: 애플리케이션의 데이터를 표현하고, 비즈니스 로직을 처리하는 역할을 합니다. 데이터베이스와의 상호작용 및 서비스 로직이 포함
* View: 주로 HTML, JSP, Thymeleaf와 같은 뷰 기술을 사용
* Controller: 클라이언트의 요청을 받아 적절한 비즈니스 로직을 호출하고, 그 결과를 **Model**에 저장한 뒤 **View**로 전달

동작 원리:

1. 클라이언트가 요청을 보낼 때, **DispatcherServlet**이 모든 요청을 수신
2. **DispatcherServlet**은 **HandlerMapping**을 이용해 적절한 컨트롤러를 찾습니다.
3. 컨트롤러는 비즈니스 로직을 처리하고, **Model** 객체에 데이터를 담습니다.
4. **ViewResolver**가 요청에 맞는 뷰(View)를 찾아 데이터를 넘겨줍니다.
5. **View**는 클라이언트에게 데이터를 포함한 화면을 응답으로 전송



## MVC 패턴에 대해 설명하고, Spring MVC에서의 구현 방법을 설명해주세요.

* **Model**, **View**, **Controller** 세 가지 요소로 구성된 소프트웨어 설계 패턴

클라이언트-> DispatcherServlet -> HandlerMapping -> HandlerAdapter -> Controller -> ModelAndView->

&#x20;\-> DispatcherServlet -> ViewResolver -> View ->

\-> DispatcherServlet -> 클라이언트



## 하나의 컨트롤러로 여러 요청을 받을 수 있는 방법은 무엇인가요?

* @RequestMapping 어노테이션을 사용하여 URL 패턴, HTTP 메서드, 매개변수에 따라 하나의 컨트롤러에서 다양한 요청을 처리할 수 있습니다.

<details>

<summary> 매개변수에 따라</summary>

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    // 요청: /products/100
    @GetMapping("/{id}")
    public String getProductById(@PathVariable("id") Long id) {
        return "Product ID: " + id;
    }

    // 요청: /products/category/electronics
    @GetMapping("/category/{categoryName}")
    public String getProductByCategory(@PathVariable("categoryName") String categoryName) {
        return "Category: " + categoryName;
    }
    
    // 요청: /users?age=25
    @GetMapping
    public String getUsersByAge(@RequestParam("age") int age) {
        return "Users with age: " + age;
    }
    
    // POST 요청으로 JSON 데이터를 전달받아 처리
    @PostMapping
    public String createUser(@RequestBody User user) {
        return "User created: " + user.getName();
    }
}
```



</details>



## @RequestParam, @RequestBody, @ModelAttribute의 차이에 대해 설명해주세요.

* RequestParam: URL의 **쿼리 파라미터** 또는 **폼 데이터**에서 값을 추출할 때 사용
* RequestBody: **HTTP Body**에 포함된 데이터를 직접 읽어서 처리. 데이터를 자바 객체로 변환해줍니다.
* ModelAttribute: **폼 데이터**나 URL 파라미터를 **객체**에 바인딩하는 데 사용

<details>

<summary>RequestBody &#x26; ModelAttribute</summary>

**@RequestBody 사용 예 (JSON 데이터 처리):**

* **상황**: 클라이언트가 서버에 JSON 형식의 데이터를 전송하여 새로운 사용자를 등록하려고 할 때.

```json
// 클라이언트 요청 Body 예시 (JSON):
{
  "name": "John",
  "age": 25
}
```

* **Controller 메서드**:

```java
@PostMapping("/user")
public String createUser(@RequestBody User user) {
    return "User created: " + user.getName();
}
```

* **설명**:
  * 클라이언트는 HTTP Body에 JSON 데이터를 담아서 서버로 보냅니다.
  * **@RequestBody**는 이 JSON 데이터를 자바 객체(User)로 **변환**합니다.
  * 주로 **REST API**에서 JSON이나 XML 데이터를 처리할 때 사용됩니다.

**@ModelAttribute 사용 예 (폼 데이터 처리):**

* **상황**: 사용자가 웹 폼을 통해 데이터를 제출할 때, 서버가 폼 필드 데이터를 자바 객체로 받아 처리하는 경우.

```html
<!-- 클라이언트의 HTML 폼 예시 -->
<form action="/user" method="POST">
  <input type="text" name="name" value="John"/>
  <input type="number" name="age" value="25"/>
  <button type="submit">Submit</button>
</form>
```

* **Controller 메서드**:

```java
@PostMapping("/user")
public String createUser(@ModelAttribute User user) {
    return "User created: " + user.getName();
}
```

</details>



## 요청마다 새로운 스레드가 생성되는데, 어떻게 하나의 Controller 인스턴스로 처리할 수 있나요?

* **Controller**는 기본적으로 **싱글톤 스코프**로 관리
* **하나의 인스턴스**만 생성되고, 여러 요청에서 재사용
* 동기화 문제가 될만한 상태를 저장하지 않는 이상 동기화같은 문제가 발생하지 않습니다.





**DispatcherServlet과 프론트 컨트롤러 패턴**

## DispatcherServlet의 역할과 동작 원리에 대해 설명해주세요.

* **DispatcherServlet**은 Spring MVC에서 **프론트 컨트롤러** 역할. 모든 클라이언트 요청을 중앙에서 받아 처리
* **HandlerMapping**을 통해 요청 URL에 매핑된 적절한 **Controller**를 찾습니다.
* **Controller**를 찾은 후, DispatcherServlet은 **HandlerAdapter**를 이용해 해당 컨트롤러의 메서드를 실행
* Controller에서 Model을 거치고 ModelAndView를 반환하고, DispatcherServlet은 ViewResolver를 통해 View를 결정합니다.
* View는 HTML페이지를 생성하고 반환합니다. DispatcherServlet은 이 페이지를 클라이언트로 반환

클라이언트 -> DispatcherServlet -> HandlerMapping -> HandlerAdapter -> Controller -> ModelAndView -> DispatcherServlet -> ViewResolver -> View -> DispatcherServlet -> 클라이언트



## 프론트 컨트롤러 패턴이란 무엇인가요?

* 모든 클라이언트 요청을 하나의 진입점(컨트롤러)에서 받아 처리하는 소프트웨어 설계 패턴
* 모든 요청을 하나의 컨트롤러가 수신하고 적절한 처리기로 전달하는 역할





**Filter와 Interceptor**

## Filter와 Interceptor의 차이점을 설명해주세요.

Filter

* **Filter**는 **Servlet API**의 일부이며, **서블릿 컨테이너** 수준에서 동작
* 주로 요청을 **가로채어** HTTP 요청과 응답을 변경하거나, 특정 조건에 맞는 요청만을 처리하는 데 사용
* **요청 전처리**와 **응답 후처리** 모두 가능하며, 요청이 **DispatcherServlet**에 도달하기 전과 후에 동작
* 인증, 로깅, CORS 처리 등

Intercepter

* **Interceptor**는 **Spring MVC**에서 제공하는 기능으로, **DispatcherServlet**에 의해 처리되며, Spring의 **핸들러 계층**에서 동작
* **Controller**에 대한 요청을 가로채어 추가 작업을 수행하는 데 사용
* **HandlerMapping**과 **Controller** 사이에서 요청을 가로채고, **비즈니스 로직 전후**에 특정 작업을 처리
* 로그인 체크, 요청 전후 데이터 가공, 권한 확인 등



## Filter와 Interceptor의 동작 방식과 적용 사례를 설명해주세요.

Filter: **요청 전처리**나 **응답 후처리**를 담당

* 클라이언트 요청 → Filter가 요청 가로채기.
* Filter에서 전처리 작업 수행 (예: 인증 검사).
* 필터 체인을 통해 **DispatcherServlet**에 요청 전달.
* 응답이 돌아오면 Filter에서 후처리 작업을 수행 (예: 응답 데이터 변경).
* 최종적으로 응답이 클라이언트로 반환됨.

Intercepter: 컨트롤러에 대한 **전처리, 후처리** 및 **요청 완료 후 처리**를 수행

* 클라이언트 요청 → **preHandle()**에서 전처리 (예: 로그인 여부 체크).
* 컨트롤러가 요청 처리.
* **postHandle()**에서 응답 전처리 (예: 데이터 포맷 변경).
* 뷰가 렌더링된 후, **afterCompletion()**에서 후처리 (예: 로그 기록, 예외 처리).



## 애플리케이션에 필터를 추가하는 방법은 무엇인가요?

* Spring Boot에서 @Bean을 사용해 필터 등록
* 서블릿 컨테이너에서 필터 등록
* FIlterChain을 통한 필터 체이닝



## Filter와 Interceptor를 이용하여 예외를 처리하는 방법은 무엇인가요?

Filter

* **서블릿 컨테이너 수준**에서 동작하며, 요청과 응답을 처리하는 중 발생한 예외를 잡아냄
* 일반적으로 **try-catch** 블록을 사용
* 예외 발생시 HTTP 응답코드 또는 사용자 정의 에러 메세지를 클라이언트에게 반환

<details>

<summary>Example</summary>

```java
@WebFilter(urlPatterns = "/*")
public class CustomFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        try {
            // 요청을 필터 체인으로 넘김
            chain.doFilter(request, response);
        } catch (Exception e) {
            // 예외 처리
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR); // 500 에러 반환
            httpResponse.getWriter().write("An error occurred: " + e.getMessage());
        }
    }

```

</details>

Interceptor

* **핸들러 계층**에서 동작하며, **preHandle()**, **postHandle()**, **afterCompletion()** 메서드를 통해 요청을 처리하는 동안 예외를 잡아낼 수 있습니다.

<details>

<summary>Example</summary>

```java
public class CustomInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        // 요청 전에 동작
        return true; // 요청을 계속 처리하도록 허용
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
                           ModelAndView modelAndView) throws Exception {
        // 컨트롤러 실행 후 동작
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        // 요청이 완료된 후(뷰가 렌더링된 후) 예외 처리
        if (ex != null) {
            // 예외가 발생한 경우 로그 처리 및 사용자 정의 응답
            response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR); // 500 에러 설정
            response.getWriter().write("Error occurred: " + ex.getMessage());
        }
    }
}
```

</details>



## Spring Boot에서 예외를 처리하는 방법을 설명해주세요.

* Filter 및 interceptor를 통한 에외 처리
* ExceptionHandler를 사용한 예외 처리
* ControllerAdvice를 사용한 전역 예외 처리
* ResponseEntityExceptionHandler 확장





**서블릿과 서블릿 컨테이너**

## 서블릿과 서블릿 컨테이너에 대해 설명해주세요.

서블릿

**HTTP 요청을 처리**하고 **응답을 생성**하는 서버 측 컴포넌트.&#x20;

서블릿은 자바 클래스의 일종으로, 클라이언트(주로 웹 브라우저)로부터 들어오는 요청을 처리하고 그 결과를 반환하는 역할

1. **요청 수신**: 클라이언트가 HTTP 요청을 보내면, 서블릿이 이 요청을 수신합니다.
2. **비즈니스 로직 처리**: 서블릿은 요청 데이터를 분석하고, 필요한 비즈니스 로직을 처리합니다. 예를 들어, 데이터베이스에 접근하여 데이터를 조회하거나, 파일을 읽고 클라이언트에 응답을 보냅니다.
3. **응답 생성**: 서블릿은 클라이언트에게 보내질 **HTML, JSON, XML** 등의 응답을 생성합니다.
4. **응답 반환**: 마지막으로, 서블릿은 응답을 클라이언트에게 반환합니다.

서블릿 컨테이너

**서블릿 컨테이너**는 **서블릿을 실행하고 관리하는 환경**을 제공.&#x20;

**서블릿 컨테이너**가 서블릿의 생명주기를 관리하며, 클라이언트의 요청을 서블릿에 전달하고 응답을 다시 클라이언트에게 전송하는 역할.&#x20;

1. **서블릿 생명주기 관리**:
   * 서블릿의 **생성**, **초기화**(`init()` 메서드), **요청 처리**(`service()` 메서드), **종료**(`destroy()` 메서드) 등의 생명주기를 관리합니다.
2. **스레드 관리**:
   * 서블릿 컨테이너는 **멀티스레드** 환경을 지원합니다. 각 요청마다 별도의 스레드를 생성하여 동시에 여러 요청을 처리할 수 있도록 관리합니다. 이로 인해 하나의 서블릿 인스턴스가 여러 요청을 처리할 수 있습니다.
3. **요청과 응답 처리**:
   * 서블릿 컨테이너는 HTTP 요청을 서블릿에 전달하고, 서블릿이 생성한 응답을 클라이언트에게 반환하는 **전달자 역할**을 합니다. 클라이언트와 서블릿 사이에서 중재자 역할을 수행합니다.

Apache Tomcat, Jetty 등이 있습니다.
