---
description: 권한
---

# Role

## 1. 개요

개발에서 사용자에게 여러 역할을 부여하고, 그 역할에 따라 권한을 다르게 설정하는 경우가 많습니다. 유저와 관리자 등의 사용자는 모두 역할이 다르며, 가능한 행동과 접근 권한 역시 다릅니다.

역할 관리를 단순히 Enum 형태로 하드코딩하는 방식을 택하는 경우가 많은데, if (role == Role.ADMIN) {..} 같은 분기로 간단히 사용할 수 있습니다. 다만, 실무에서 유연성과 확장성을 심각하게 저해하게 됩니다. 예를 들어, 새로운 역할을 추가하거나 세부 권한을 변경해야 할 때마다 소스 코드를 직접 수정해야 하고, 배포도 다시 해야합니다.

Role이 있지만 권한이 없는 상태에서 시작해, 어떻게 하면 동적인 권한 관리 체계를 구축할 수 있는지, 하드코딩 없이 확장성 있는 구조를 만들 수 있는지 알아보려고합니다. RBAC(Role-Based Access Control), ACL(Access Control List), 혹은 정책 기반 권한 모델 등 다양한 관리 기법을 폭넓게 살펴보려고합니다.



## 2. Enum 하드코딩의 한계

### 2.1. 초기 Enum 설계

처음에는 다음과 같은 단순한 Enum을 만들 수 있습니다.

```java
public enum Role {
    USER,
    SELLER,
    ADMIN
}
```

그리고, 사용자의 `role` 필드는 DB에서 `VARCHAR` 또는 `INT` 값으로 저장하든지, 애플리케이션 레벨에서 직접 `Role` Enum 값을 매핑해 사용하곤 합니다. 예를 들어, `User` 엔터티에 `Role` 필드를 둡니다:

```java
@Entity
public class User {
    @Id
    @GeneratedValue
    private Long id;

    private String email;
    private String password;

    @Enumerated(EnumType.STRING)
    private Role role;

    // ... getters, setters ...
}
```

이 방식은 **초기 프로토타입**이나 **단순 MVP**에서는 빠르게 구현 가능하다는 장점이 있습니다. 그러나 실무에서 “어떤 API는 SELLER만 접근 가능하고, 어떤 페이지는 ADMIN만 편집할 수 있음” 같은 세부 권한 조건을 구현하려면, 결국 코드에서 **if (role == ADMIN)** 식의 분기가 곳곳에서 등장합니다.

* 새로운 역할, 예컨대 `SUPER_ADMIN`, `MARKETING_MANAGER` 등이 추가되면?
* `ADMIN`도 세분화해야 해서, “상품 관리만 가능한 ADMIN”과 “회원 관리도 가능한 ADMIN”이 따로 필요하다면?

결국 Enum 하드코딩으로는 대응하기 쉽지 않습니다. 게다가 역할이 늘어날 때마다 **배포**를 해야 하고, 이미 배포된 시스템에서는 곤란해지는 경우가 많습니다.



### 2.2. 코드 중복과 분기 난립

아래와 같은 코드를 곳곳에서 보게 됩니다:

```java
if (currentUser.getRole() == Role.ADMIN) {
    // 관리자 전용 로직
} else if (currentUser.getRole() == Role.SELLER) {
    // 판매자 전용 로직
} else {
    // 일반 사용자 로직
}
```

이런 로직이 모든 서비스, 컨트롤러, UI 단에서 중복된다면, 프로젝트가 점점 복잡해지는 건 시간문제입니다. 또한, “상품 관리 권한”만 있는 ADMIN과, “회원 관리 권한”도 있는 ADMIN을 구분해야 하면, 또 다른 Enum 상수나 조건이 추가될 것입니다.

결국 Enum 하드코딩 모델은 **확장성**과 **유연성**이 부족합니다. 이를 해결하기 위해서는 **권한(Permission)** 을 별도로 정의하고, **역할(Role)과 권한을 매핑**하는 구조가 필요합니다.



## 3. 권한 관리 체계의 필요성

## 3.1. 역할(Role) vs 권한 (Permission)

* **역할(Role)**: “사용자가 어떤 그룹에 속하는가?”를 나타내는 상위 개념. 예) Admin, User, Moderator 등.
* **권한(Permission)**: “이 사용자가 어떤 행동을 할 수 있는가?”를 나타내는 하위 개념. 예) 상품 등록, 상품 수정, 회원 차단, 매출 통계 보기 등.

둘을 분리하면, **역할**은 단순히 “관리자, 회원, 게스트” 같은 범주이고, **권한**은 실제 기능에 대한 허용 여부를 세분화합니다. 이때, **RBAC(Role-Based Access Control)** 모델에서는 특정 Role이 여러 개의 Permission을 갖게 설계합니다. 예를 들어, `ADMIN` Role은 “상품 등록, 상품 수정, 회원 차단” 권한을 모두 갖고, `SELLER` Role은 “상품 등록, 상품 수정” 권한만 갖게 됩니다.



## 4. RBAC(Role-Based Access Control) 기초

RBAC는 역할 기반 접근 제어입니다. 사용자는 하나 이상의 역할을 갖고, 각 역할은 여러 권한을 가집니다.

1. 사용자가 로그인
2. 서버가 사용자의 역할(들)을 식별
3. 역할들은 어떤 권한을 가지는지 조회
4. 클라이언트가 특정 요청을 할 때, 서버에서, 이 권한이 필요한데, 사용자 역할에 이 권한이 있는지 확인.

하나 이상의 역할의 예: 사용자가 마케팅 업무와 회계 업무를 같이 맡은 경우.



RBAC의 장점은 아래와 같습니다.

* 유연성: 새로운 사용자에게 복잡한 권한을 일일이 부여하지 않아도 됨.
* 확장성: Permission이 새로 생겨도 DB에 추가하면 되고, 특정 Role에 매핑만 하면 됨.
* 정책성: 보안 정책을 역할 단위로 정의할 수 있음

## 5. ACL(Access Control List), 정책 기반

ACL은 주로 “리소스 관점”에서 접근 권한을 정의합니다. 예컨대 “문서 123번은 Alice는 읽기/쓰기 가능, Bob은 읽기만 가능” 이런 식이죠. 웹 애플리케이션에서 “게시글, 댓글, 파일” 등 개별 리소스에 대해 세밀하게 권한을 제어하려면 ACL이 필요할 수 있습니다. \
(하지만 이커머스 프로젝트처럼 **기능 단위**로 권한을 관리할 때는, RBAC가 더 단순 명료)



## 6. 실제 DB 설계 예시

이제 **하드코딩(enum) 없이** DB에 Role, Permission을 정의하는 스키마 예시를 보겠습니다.

(유저 N : N 역할. 역할 N : N 권한)

```sql
-- roles 테이블
CREATE TABLE roles (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- permissions 테이블
CREATE TABLE permissions (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- role_permissions (N:M 매핑)
CREATE TABLE role_permissions (
    role_id BIGINT NOT NULL,
    permission_id BIGINT NOT NULL,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id),
    FOREIGN KEY (permission_id) REFERENCES permissions(id)
);

-- users 테이블 (role_id 저장 여부는 선택)
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role_id BIGINT, --또는 user_roles_id
    FOREIGN KEY (role_id) REFERENCES roles(id),
    ...
);
```

#### 6.1. 여러 역할을 가질 수 있는 구조

만약 한 사용자가 여러 Role을 가질 수 있어야 한다면, `user_roles`라는 N:M 매핑 테이블을 별도로 둡니다:

```sql
-- roles 테이블
CREATE TABLE roles (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- permissions 테이블
CREATE TABLE permissions (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL
);

-- role_permissions 테이블 (Role : Permission = N:M)
CREATE TABLE role_permissions (
    role_id BIGINT NOT NULL,
    permission_id BIGINT NOT NULL,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id,
    FOREIGN KEY (permission_id) REFERENCES permissions(id)
);

-- users 테이블
-- (role_id 컬럼 제거 -> 다중 role은 아래 user_roles 테이블로 관리)
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- user_roles 테이블 (User : Role = N:M)
CREATE TABLE user_roles (
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id,
    FOREIGN KEY (role_id) REFERENCES roles(id)
);

```

이렇게 하면 사용자는 원하는 만큼 Role을 할당받을 수 있습니다.

#### 6.2. Permission 예시

`permissions` 테이블에는 다음과 같은 Permission들이 들어갈 수 있습니다:

* `MANAGE_PRODUCTS`
* `MANAGE_ORDERS`
* `MANAGE_USERS`
* `VIEW_REPORTS`
* `ISSUE_COUPONS`
* ...

Role마다 어떤 권한이 필요한지 `role_permissions` 테이블에 매핑하면 됩니다. (ADMIN Role에는 모든 권한을, `USER` Role에는 기본 조회 권한만.)



## 7. Java 코드 예시: 동적 RBAC

### 7.1. 엔티티 설계

스프링 부트 + JPA 예시를 들어보겠습니다.

```java
@Entity
@Table(name = "roles")
public class RoleEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String name;

    // N:M 관계
    @ManyToMany
    @JoinTable(
        name = "role_permissions",
        joinColumns = @JoinColumn(name = "role_id"),
        inverseJoinColumns = @JoinColumn(name = "permission_id")
    )
    private Set<PermissionEntity> permissions = new HashSet<>();

    // getters, setters ...
}

@Entity
@Table(name = "permissions")
public class PermissionEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String name;

    // Role의 ManyToMany에서 mappedBy를 설정할 수도 있으나,
    // 편의상 단방향 예시
}
```

`UserEntity`에서 `role_id`를 가질 수도 있고, 혹은 N:M 관계로 `Set<RoleEntity> roles`를 가질 수도 있습니다.

```java
@Entity
@Table(name = "users")
public class UserEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;
    private String password;

    // 예시) 다중 Role 가능
    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<RoleEntity> roles = new HashSet<>();

    // getters, setters ...
}
```

### 7.2. 서비스 계층: 권한 검사

사용자가 어떤 API를 요청했을 때, 어떤 Permission이 필요한지 정하고, 아래 로직으로 검증합니다.

```java
public boolean hasPermission(UserEntity user, String permissionName) {
    // 1) user의 모든 role을 순회
    for (RoleEntity role : user.getRoles()) {
        // 2) role이 갖고 있는 permissions 탐색
        for (PermissionEntity perm : role.getPermissions()) {
            if (perm.getName().equals(permissionName)) {
                return true;
            }
        }
    }
    return false;
}
```

이것을 **AOP**나 **스프링 시큐리티** 필터 등으로 추상화할 수도 있습니다. 예를 들어, API 컨트롤러 메서드에 `@RequiresPermission("MANAGE_PRODUCTS")` 같은 어노테이션을 달면, 인터셉터가 위 로직을 자동으로 실행해주도록 만들 수 있습니다.

### 7.3. 커스텀 어노테이션 정의

먼저, `@RequiresPermission`이라는 어노테이션을 정의합니다.

```java
package com.example.demo.security;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
@Documented
public @interface RequiresPermission {
    String value(); // 예: "MANAGE_PRODUCTS" 등
}
```

이 어노테이션은 **메서드**(또는 클래스) 단위로 붙여서, 해당 메서드가 실행되기 전에 특정 **Permission**을 요구함을 나타냅니다.

***

### 7.4. AOP를 이용한 권한 검사 로직

어노테이션을 인식하려면, **스프링 AOP**를 사용하여 특정 메서드가 호출될 때 “어노테이션 값”을 가져오고, DB에서 현재 사용자 권한을 체크하는 로직을 수행하면 됩니다. 이를 위해 **Aspect**를 하나 정의합시다.

```java
package com.example.demo.security;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.http.HttpStatus;

import java.lang.reflect.Method;

@Slf4j
@Aspect
@Component
public class PermissionCheckAspect {

    // 1) @RequiresPermission 어노테이션이 붙은 모든 메서드에 대해 포인트컷 설정
    @Pointcut("@annotation(com.example.demo.security.RequiresPermission)")
    public void requiresPermissionAnnotationPointcut() { }

    // 2) 해당 메서드가 실행되기 *직전*에 권한 검사를 수행
    @Before("requiresPermissionAnnotationPointcut()")
    public void checkPermission(JoinPoint joinPoint) {
        // 2.1) 실행 대상 메서드 정보를 가져옴
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        // 2.2) 메서드에 달린 @RequiresPermission 어노테이션의 value(권한명) 읽기
        RequiresPermission requiresPermission = method.getAnnotation(RequiresPermission.class);
        String requiredPermission = requiresPermission.value();

        // 2.3) 현재 인증된 사용자 정보 가져오기 (Session, SecurityContext 등)
        // 여기서는 예시로 간단하게 SecurityContextHolder에서 UserEntity를 받아온다고 가정
        UserEntity currentUser = SecurityUtil.getCurrentUser(); 
        if (currentUser == null) {
            // 비로그인 상태라면 권한 부족 → 401 Unauthorized
            throw new ResponseStatusException(HttpStatus.UNAUTHORIZED, "Please log in first.");
        }

        // 2.4) 실제 권한 검사 로직 (hasPermission)
        boolean hasPerm = hasPermission(currentUser, requiredPermission);
        if (!hasPerm) {
            // 권한이 없으면 403 Forbidden
            throw new ResponseStatusException(HttpStatus.FORBIDDEN, 
                "You do not have permission: " + requiredPermission);
        }

        // 2.5) 권한이 있다면 그냥 통과 (Proceed)
        log.debug("User '{}' has permission '{}', proceeding.", currentUser.getEmail(), requiredPermission);
    }

    // 예시용: 사용자 권한 검사
    private boolean hasPermission(UserEntity user, String permissionName) {
        // 사용자 -> 여러 Role -> Role마다 여러 Permission
        for (RoleEntity role : user.getRoles()) {
            for (PermissionEntity perm : role.getPermissions()) {
                if (perm.getName().equals(permissionName)) {
                    return true;
                }
            }
        }
        return false;
    }
}
```

#### 설명

1. **@Aspect**와 **@Component** 어노테이션을 통해, 이 클래스가 AOP에 의해 프록시로 관리됨을 선언.
2. `@Pointcut("@annotation(...)")` 구문은 `@RequiresPermission`을 달고 있는 메서드를 포인트컷으로 지정.
3. `@Before("requiresPermissionAnnotationPointcut()")`를 통해, 해당 메서드가 **실행되기 전**에 `checkPermission` 메서드를 호출.
4. `checkPermission` 내부에서:
   * 실행 대상 메서드의 `Method` 객체를 가져오고, `@RequiresPermission` 어노테이션 값(`requiredPermission`)을 읽음.
   * 현재 로그인 사용자(UserEntity)를 가져와 권한 검사.
   * 권한이 없으면 예외(403) 발생, 있으면 정상 진행.

이렇게 하면, **컨트롤러**나 **서비스** 메서드에 `@RequiresPermission("MANAGE_PRODUCTS")`만 달아주면 자동으로 권한 검사가 이뤄집니다.

***

### 7.5. SecurityUtil 예시

위 예시 중 `SecurityUtil.getCurrentUser()` 부분은, **스프링 시큐리티** 또는 **세션** 기반 인증에서 현재 사용자 정보를 가져오는 로직입니다. 간단히 예시를 들면:

```java
public class SecurityUtil {
    public static UserEntity getCurrentUser() {
        // 스프링 시큐리티의 SecurityContextHolder에서 인증 정보 꺼내기
        Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        if (principal instanceof CustomUserDetails) {
            return ((CustomUserDetails) principal).getUser();
        }
        return null;
    }
}
```

* `CustomUserDetails`는 `UserDetails`를 구현한 클래스이며, 내부에 `UserEntity`를 보관해둔다고 가정.

혹은 세션 기반이라면 `HttpSession`에서 `session.getAttribute("USER")` 식으로 가져올 수도 있습니다.

***

### 7.6. 컨트롤러 사용 예시

이제 **컨트롤러**에 `@RequiresPermission("...")`을 달아봅시다:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    // 예: 상품 등록
    @RequiresPermission("MANAGE_PRODUCTS")
    @PostMapping
    public ResponseEntity<?> createProduct(@RequestBody ProductDto dto) {
        // 실제 상품 등록 로직
        // ...
        return ResponseEntity.ok("Product created!");
    }

    // 예: 상품 조회(권한 불필요)
    @GetMapping("/{productId}")
    public ResponseEntity<ProductDto> getProduct(@PathVariable Long productId) {
        // ...
        return ResponseEntity.ok(productService.findById(productId));
    }
}
```

* `@RequiresPermission("MANAGE_PRODUCTS")`가 붙은 `createProduct()` 메서드는, **AOP Aspect**에 의해 실행 전에 `checkPermission`이 호출됩니다.
* 만약 현재 유저에게 `MANAGE_PRODUCTS` 권한이 없다면, **403 Forbidden** 에러를 던지도록 처리했습니다.

***

### 7.7. 스프링 시큐리티 필터로 구현하는 방법

AOP 대신, **스프링 시큐리티**의 `HandlerInterceptor`나 `OncePerRequestFilter`를 통해 “요청 경로 → 필요한 권한”을 매핑하는 방식도 있습니다. 그러나 어노테이션 기반으로 메서드를 구분하기가 AOP보다 조금 번거롭습니다.

#### 5.1. HandlerInterceptor 예시

```java
@Component
public class PermissionInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) 
            throws Exception {
        
        // 1) Handler(Controller 메서드)에서 @RequiresPermission 파악
        if (handler instanceof HandlerMethod) {
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            RequiresPermission annotation = handlerMethod.getMethodAnnotation(RequiresPermission.class);
            if (annotation != null) {
                String requiredPermission = annotation.value();
                UserEntity currentUser = SecurityUtil.getCurrentUser();
                if (currentUser == null) {
                    response.sendError(HttpServletResponse.SC_UNAUTHORIZED);
                    return false;
                }
                // 권한 체크
                if (!hasPermission(currentUser, requiredPermission)) {
                    response.sendError(HttpServletResponse.SC_FORBIDDEN);
                    return false;
                }
            }
        }
        // 권한 문제 없으면 true
        return true;
    }

    private boolean hasPermission(UserEntity user, String permissionName) {
        // ...
    }
}
```

그리고 **WebMvcConfigurer** 또는 **SecurityConfig**에서 등록:

```java
java복사편집@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private PermissionInterceptor permissionInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(permissionInterceptor)
                .addPathPatterns("/**"); // 모든 경로에서 인터셉터 동작
    }
}
```

이렇게 하면, 요청이 들어올 때 **스프링 MVC HandlerInterceptor**가 `@RequiresPermission` 어노테이션을 스캔하고, 권한이 없으면 즉시 에러를 반환합니다. 다만, 세부적인 “메서드 호출 전/후”가 아니라 “DispatcherServlet에서 Handler를 찾은 후” 시점에 동작한다는 점이 AOP와 다릅니다.



요약

1. **커스텀 어노테이션** `@RequiresPermission("...")`을 만든다.
2. **AOP Aspect**(또는 **HandlerInterceptor**)에서 메서드 실행 전 해당 어노테이션을 확인한다.
3. 현재 사용자 정보를 가져와, DB에서 “이 권한을 갖고 있는지” 검사.
4. 권한 부족 시 **403 Forbidden**(또는 **401 Unauthorized**) 응답을 반환.

이 과정을 통해, **컨트롤러**나 **서비스** 메서드에 어노테이션 한 줄을 달아주기만 해도 자동으로 권한 검증 로직이 수행되어, **코드 중복**을 최소화할 수 있습니다. 또한, **스프링 시큐리티**의 `UserDetailsService` 등과 연계하면 권한 정보를 더욱 쉽게 로딩·검사할 수 있습니다.

추가

* **Aspect vs Interceptor vs Filter**
  * **Filter**(예: `OncePerRequestFilter`)는 서블릿 레벨, 요청 초기에 동작. URL 기반 접근 제어에 유용.
  * **HandlerInterceptor**는 스프링 MVC Handler가 결정된 뒤, 실제 컨트롤러 로직 전후에 동작. @RequiresPermission 등 “메서드 어노테이션”을 읽으려면 Interceptor 수준이 편리.
  * **Aspect**는 스프링 빈 객체의 메서드가 호출되기 전/후를 잡아낼 수 있으므로, 서비스 계층, 리포지토리 계층까지도 적용 가능.
* **Spring Security**를 사용할 경우, `@PreAuthorize("hasAuthority('MANAGE_PRODUCTS')")` 같은 어노테이션 기반 접근 제어 방식을 제공하므로, 굳이 커스텀 어노테이션을 만들지 않아도 됩니다. 다만, **프로젝트 요구사항**이나 **가독성**, **소트웨어 아키텍처**에 따라 `@RequiresPermission`처럼 별도로 만들고 싶을 수도 있습니다.



## 8. 비교

## 하드코딩(Enum) vs 동적(테이블) 요약 비교

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 9. 주의사항

* **Role-Permission 중복**: 잘못된 설계로 Role이 너무 세분화되면, Permission이랑 중복되는 경우가 있음.
* **권한 폭발**: Permission을 과도하게 세분화해 `ORDER_VIEW`, `ORDER_VIEW_DETAIL`, `ORDER_VIEW_ADDRESS` 등등 만드는 건 유지보수를 어렵게 만듦. 적절한 추상화가 필요.
* **캐싱 동기화**: DB에서 권한을 변경했는데, 캐싱된 권한 정보가 갱신되지 않아 문제가 생기는 경우가 있음.
* **로그인된 유저 세션/토큰**은 어떻게 업데이트 하나?: 권한 업데이트 후 즉시 반영하려면, 세션/토큰을 갱신하거나, 인증 과정에서 매번 Role/Permission을 확인해야 할 수도 있음.

## 10. 요약

1. **Enum 하드코딩의 문제점**: 새로운 Role/Permission 추가 시 **코드 수정** 및 **재배포** 필요, 확장성 저해.
2. **Role vs Permission**: Role은 상위 개념, Permission은 실제 행동. **RBAC**를 사용하면 유연.
3. **DB 테이블 모델링**: `roles`, `permissions`, `role_permissions`(N:M) 구조.
4. **User와 Role**: 1:N 또는 M:N 관계. 필요 시 `user_roles` 중간 테이블 사용.
5. **Java 구현**: JPA 엔티티로 만들고, `@ManyToMany` 관계 설정, `hasPermission` 로직으로 확인.
6. **관리자 페이지에서 동적 수정**: 운영 시 새로운 Role/Permission 추가, 삭제, 매핑 변경이 자유로움.
7. **스프링 시큐리티 연동**: `@PreAuthorize("hasAuthority(...)")` 등으로 접근 통제.
8. **주의사항**: 권한이 너무 세분화되지 않도록, 캐싱 동기화, 감사 로그, 세션 업데이트.

궁극적으로, **역할(Role)과 권한(Permission)을 분리**하고, **DB 기반**으로 동적으로 관리함으로써 애플리케이션이 커질 때도 쉽게 확장할 수 있습니다. 이는 어떤 웹 서비스든 “사용자 그룹”이 여러 갈래로 나뉘고, “세분화된 기능 접근”을 허용해야 하는 경우에 유용합니다. 초반에 조금 복잡해 보일 수 있지만, 유지보수와 운영 단계에서 훨씬 **유연하고 편리한** 방식을 제공해 준다는 점이 핵심
