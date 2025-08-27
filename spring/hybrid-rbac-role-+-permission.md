---
description: Hybrid RBAC (Role + Permission) 접근 제어
---

# Hybrid RBAC (Role + Permission)

[https://github.com/implement-study-25/ticket-reserve-service/pull/14](https://github.com/implement-study-25/ticket-reserve-service/pull/14)

#### 티켓 예매 서비스에서의 Role & Privilege 기반 접근 제어(R\&PBAC) 설계와 구현



## **요약**

* **핵심**: JWT로 사용자 식별 → 별도 필터가 DB에서 `roles + privileges` 로딩 → `@PreAuthorize("hasAuthority(..)")`로 인가.
* **장점**: 역할 폭발 방지, 무배포 정책 변경, 셀프서비스 운영, 감사 용이, 기간제 권한, 캐시 확장.
* **구현**: `SecurityConfig`에 `@EnableMethodSecurity`, `GrantedAuthorityDefaults("")`, 필터 체인. `JwtAuthenticationFilter`는 식별만, `RbacAuthoritiesFilter`는 권한 주입.



## **왜 R\&PBAC인가? (RBAC과의 차이 및 장점)**

* **RBAC(Role-Based Access Control)**: 사용자는 하나 이상의 역할(Role)을 갖고, 리소스 접근은 역할 단위로 제어한다. 단순하고 관리가 쉽지만, 역할이 세분화될수록 역할 폭발(Role Explosion) 문제가 발생한다.
* **R\&PBAC(Role & Privilege-Based Access Control)**: 역할과 함께 보다 세밀한 **권한(Privilege)** 을 별도로 정의하고, 역할과 권한을 매핑한다. 결과적으로 사용자의 권한은 `역할 + 권한`의 합으로 구성된다.
  * **장점**
    * **세분화된 권한 설계**: `ADMIN`, `USER` 같은 거친 역할과 `EVENT_CREATE`, `EVENT_UPDATE` 같은 미세 권한을 분리해 관리.
    * **역할 폭발 방지**: 역할 수는 적정 수준으로 유지하고, 신규 기능은 권한만 추가해 역할-권한 매핑으로 유연하게 확장.
    * **정책 변경 용이성**: 비즈니스 변화(새 기능, 한시적 오퍼레이션)에 맞춰 권한만 수정/추가 가능.
  * **RBAC 대비 차이점**
    * RBAC은 역할만으로 접근 제어. R\&PBAC은 역할과 권한을 동시에 사용하며, 최종 인가 판단 시 두 집합을 모두 `GrantedAuthority`로 주입해 `hasAuthority`로 일관되게 검증.



## **RBAC을 도입하지 않았을 때의 문제와, R\&PBAC 도입 후 해결**

* **미도입 시 발생 문제**
  * 코드 곳곳에서 임시 플래그, 하드코딩된 이메일/사용자 체크 등으로 접근 제어 로직이 분산·중복.
  * 권한 변경 요구 시 코드 수정 범위가 광범위하고 테스트 비용 증가.
  * 기능 단위 권한 위임(예: 관리자 중 특정 작업만 가능한 운영자) 구현 난이도 급상승.
* **R\&PBAC 도입 후 개선**
  * 인증과 인가를 분리: JWT 기반 인증 필터는 사용자 식별까지만, 이후 **R\&PBAC 필터**가 역할·권한을 로딩해 `SecurityContext`에 주입.
  * 컨트롤러/서비스 단에는 `@PreAuthorize("hasAuthority('...')")`만 배치 → 단일 진입점으로 가독성과 변경 용이성 향상.
  * 역할·권한 모델이 DB로 관리되어 운영 환경에서 정책 변경이 코드 배포 없이 가능(매핑 변경만으로 반영).

***

## 구현 구조 개요

* 위치: `com.study.ticketservice.common.security`, `com.study.ticketservice.domain.auth`, `com.study.ticketservice.utils`
* 핵심 흐름
  1. `JwtAuthenticationFilter`가 쿠키의 Access Token을 검증하고 `principal`에 `userId`만 설정
  2. `RbacAuthoritiesFilter`가 DB에서 `roles + privileges`를 로딩해 `GrantedAuthority`로 주입
  3. `@PreAuthorize("hasAuthority('...')")`로 엔드포인트/메서드 보호
  4. `GrantedAuthorityDefaults("")`로 ROLE\_ 프리픽스 제거 → `ADMIN`, `USER`, `EVENT_*` 그대로 사용

```java
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
@RequiredArgsConstructor
public class SecurityConfig {
```

```java
@Bean
public GrantedAuthorityDefaults grantedAuthorityDefaults() {
    return new GrantedAuthorityDefaults("");
}

@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .cors(Customizer.withDefaults())
        .csrf(csrf -> csrf.disable())
        .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/", "/actuator/**",
                    "/api/v1/auth/login",
                    "/api/v1/auth/refresh",
                    "/api/v1/auth/logout").permitAll()
            .anyRequest().authenticated()
        )
        .addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
        .addFilterAfter(rbacAuthoritiesFilter, JwtAuthenticationFilter.class);

    return http.build();
}
```

```java
@Override
protected boolean shouldNotFilter(HttpServletRequest request) {
    String uri = request.getRequestURI();
    return uri.equals("/api/v1/auth/login") || uri.equals("/api/v1/auth/logout");
}
```

```java
@Override
protected boolean shouldNotFilter(HttpServletRequest request) {
    String uri = request.getRequestURI();
    return uri.equals("/api/v1/auth/login")
        || uri.equals("/api/v1/auth/logout")
        || uri.equals("/api/v1/auth/refresh");
}
```

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                FilterChain filterChain) throws ServletException, IOException {
    if (request.getRequestURI().equals("/api/v1/auth/refresh")) {
        filterChain.doFilter(request, response);
        return;
    }

    String token = jwtUtil.extractAccessTokenFromCookie(request);
    if (!StringUtils.hasText(token)) {
        filterChain.doFilter(request, response);
        return;
    }

    try {
        Claims claims = jwtUtil.extractAccessClaims(token);
        String type = claims.get("type", String.class);
        if (!"access".equals(type)) throw new BadCredentialsException("INVALID_TOKEN");
        Number userIdNum = claims.get("userId", Number.class);
        if (userIdNum == null) throw new BadCredentialsException("MISSING_USER_ID");
        Long userId = userIdNum.longValue();

        // 사용자 식별만 설정, 권한은 R&PBAC 필터에서 주입
        UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                userId, null, List.of());
        SecurityContextHolder.getContext().setAuthentication(authentication);

    } catch (ExpiredJwtException e) {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Token has expired");
        return;
    } catch (Exception e) {
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Invalid token");
        return;
    }

    filterChain.doFilter(request, response);
}
```

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {
    var authentication = SecurityContextHolder.getContext().getAuthentication();
    if (authentication == null || authentication.getPrincipal() == null) {
        filterChain.doFilter(request, response);
        return;
    }
    Object principal = authentication.getPrincipal();
    if (!(principal instanceof Long userId)) {
        filterChain.doFilter(request, response);
        return;
    }

    Set<String> authorities = getAuthorities(userId); // Role & Privilege 모두 추가
    List<SimpleGrantedAuthority> granted = authorities.stream().map(SimpleGrantedAuthority::new).toList();
    UsernamePasswordAuthenticationToken newAuth = new UsernamePasswordAuthenticationToken(
            userId, authentication.getCredentials(), granted);
    SecurityContextHolder.getContext().setAuthentication(newAuth);
    filterChain.doFilter(request, response);
}

private Set<String> getAuthorities(Long userId) {
    List<String> roleNames = userRoleMapRepository.findRoleNamesByUserId(userId);
    Set<String> authorities = new HashSet<>(roleNames);
    if (!roleNames.isEmpty()) {
        List<UserRoleMap> userRoles = userRoleMapRepository.findAllByUserUserId(userId);
        List<Long> roleIds = userRoles.stream().map(urm -> urm.getRole().getRoleId()).toList();
        if (!roleIds.isEmpty()) {
            List<String> privilegeNames = rolePrivilegeMapRepository.findPrivilegeNamesByRoleIds(roleIds);
            authorities.addAll(privilegeNames);
        }
    }
    return authorities;
}
```

***



## 도메인 모델링 (엔티티 & 매핑)

* 사용자-역할 다대다: `users` ↔ `roles` 를 `user_roles`로 매핑
* 역할-권한 다대다: `roles` ↔ `privileges` 를 `role_privileges`로 매핑
* 엔티티
  * `User`, `Role`, `Privilege`, `UserRoleMap`, `RolePrivilegeMap`

```java
@Entity
@Table(name = "user_roles")
public class UserRoleMap {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_role_id")
    private Long userRoleId;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "role_id", nullable = false)
    private Role role;
}
```

```java
@Entity
@Table(name = "role_privileges")
public class RolePrivilegeMap {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "role_privilege_id")
    private Long rolePrivilegeId;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "role_id", nullable = false)
    private Role role;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "privilege_id", nullable = false)
    private Privilege privilege;
}
```

* 조회 쿼리

```java
@Query("select r.name from UserRoleMap urm join urm.role r where urm.user.userId = :userId")
List<String> findRoleNamesByUserId(@Param("userId") Long userId);
```

```java
@Query("select distinct p.name from RolePrivilegeMap map join map.privilege p where map.role.roleId in :roleIds")
List<String> findPrivilegeNamesByRoleIds(@Param("roleIds") Collection<Long> roleIds);
```

***

## 컨트롤러 인가 적용 예시

* **관리자 전용 이벤트 관리**

```42:66:src/main/java/com/study/ticketservice/domain/event/controller/eventapicontroller.java
@PostMapping("/v1/admin/events")
@PreAuthorize("hasAuthority('EVENT_CREATE')")
public ResponseEntity<ApiResponse<EventDetailResponse>> createEvent(@Valid @RequestBody EventCreateRequest request) {
    EventDetailResponse response = eventService.createEvent(request);
    return ApiResponse.success(201, response);
}
```

* **사용자 좌석 예약**

```21:30:src/main/java/com/study/ticketservice/domain/reservation/controller/reservationapicontroller.java
@PostMapping("/v1/reservations")
@PreAuthorize("hasAuthority('EVENT_SEAT_RESERVE')")
public ResponseEntity<ApiResponse<ReservationCreateResponse>> reserveEventSeats(
        @RequestBody ReservationCreateRequest request,
        @AuthenticationPrincipal Long userId,
        @RequestHeader(value = "X-Idempotency-Key") String idempotencyKey) {
    ReservationCreateResponse result = reservationService.reservation(request, userId, idempotencyKey);
    return ApiResponse.success(result);
}
```

* **통합 데모(RBAC vs Privilege 비교)**

```13:55:src/main/java/com/study/ticketservice/test/presentation/rbacmockcontroller.java
@GetMapping("/admin/event/create")
@PreAuthorize("hasAuthority('ADMIN')")
public ResponseEntity<String> adminCreateByRole() { return ResponseEntity.ok("EVENT_CREATE_OK"); }

@GetMapping("/admin/event/create/privilege")
@PreAuthorize("hasAuthority('EVENT_CREATE')")
public ResponseEntity<String> adminCreateByPrivilege() { return ResponseEntity.ok("EVENT_CREATE_OK"); }
```

***

## 인증(JWT)과 쿠키 전략

* `JwtUtil`
  * Access/Refresh 분리, 각기 다른 시크릿/만료
  * `accessToken`, `refreshToken`를 `HttpOnly`, `Secure` 쿠키로 설정 (프론트는 `credentials: 'include'` 필요)
  * 토큰 페이로드: `userId`, `roles`, `type`

```37:49:src/main/java/com/study/ticketservice/utils/jwtutil.java
public String createAccessToken(Long userId, List<String> roles) {
    return Jwts.builder()
            .setSubject("AccessToken")
            .claim("userId", userId)
            .claim("roles", roles)
            .claim("type", "access")
            .setIssuedAt(now)
            .setExpiration(expireDate)
            .signWith(getSecretKey(ACCESS_TOKEN_SECRET))
            .compact();
}
```

* 리프레시 플로우

```50:75:src/main/java/com/study/ticketservice/domain/auth/service/authservice.java
public String refresh(HttpServletRequest request, HttpServletResponse response) {
    String refreshToken = jwtUtil.extractRefreshTokenFromCookie(request);
    if (!StringUtils.hasText(refreshToken) || !jwtUtil.validateRefreshToken(refreshToken)) {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        return null;
    }
    var claims = jwtUtil.extractRefreshClaims(refreshToken);
    Long userId = ((Number) claims.get("userId")).longValue();
    List<String> roles = ((List<?>) claims.get("roles")).stream().map(String::valueOf).toList();
    String newAccess = jwtUtil.createAccessToken(userId, roles);
    String newRefresh = jwtUtil.createRefreshToken(userId, roles);
    jwtUtil.setTokenCookies(response, newAccess, newRefresh);
    return "ACCESS_TOKEN_REFRESHED";
}
```

***

## 예외 처리(인가/인증/공통)

* **인가 실패**: 403 `Access Denied` 통일
* **인증 실패**: 401 `Unauthorized` 통일
* **도메인 예외**: `ApiException` → 일관 응답 포맷

```16:27:src/main/java/com/study/ticketservice/common/exception/globalexceptionhandler.java
@ExceptionHandler({AccessDeniedException.class, AuthorizationDeniedException.class})
public ResponseEntity<ErrorResponse> handleAccessDenied(Exception e) {
    return ErrorResponse.error(403, "Access Denied");
}
```

```29:33:src/main/java/com/study/ticketservice/common/exception/globalexceptionhandler.java
@ExceptionHandler(AuthenticationException.class)
public ResponseEntity<ErrorResponse> handleAuthenticationException(AuthenticationException e) {
    return ErrorResponse.error(401, "Unauthorized");
}
```

```13:18:src/main/java/com/study/ticketservice/common/exception/apiexceptionhandler.java
@ExceptionHandler(ApiException.class)
public ResponseEntity<ErrorResponse> handleApiException(ApiException e) {
    return ErrorResponse.error(e);
}
```

***



## 주요 포인트:

* **권한 주입 이원화**: 인증 필터에서 사용자 식별만, 별도 필터에서 `roles + privileges`를 로딩 후 주입 → 성능/책임 분리 명확.
* **ROLE\_ 프리픽스 제거**: `GrantedAuthorityDefaults("")`로 `ADMIN`, `USER`, `EVENT_*` 그대로 사용 → 팀/프론트와의 커뮤니케이션 비용 절감.
* **DB 기반 동적 인가**: `UserRoleMapRepository`, `RolePrivilegeMapRepository` 쿼리로 인가 정책을 DB만 바꿔도 즉시 반영 가능.
* **쿠키 기반 JWT**: `HttpOnly + Secure` 쿠키로 XSS 취약점 완화, `SameSite=None`(운영 환경) 적용 전제.
* **테스트 픽스처 제공**: `rbac-test-data.sql`로 역할/권한/매핑을 한눈에 확인 가능 → 데모/테스트 재현 용이.

```1:35:src/test/resources/rbac-test-data.sql
-- roles
INSERT INTO roles (role_id, name) VALUES (1, 'ADMIN'), (2, 'USER');
-- privileges
INSERT INTO privileges (privilege_id, name) VALUES
  (1, 'EVENT_CREATE'),
  (2, 'EVENT_UPDATE'),
  (3, 'EVENT_CHANGE_STATUS'),
  (4, 'EVENT_SEAT_RESERVE'),
  (5, 'EVENT_SEAT_CANCEL');
-- mappings
INSERT INTO role_privileges (role_privilege_id, role_id, privilege_id) VALUES
  (1, 1, 1), (2, 1, 2), (3, 1, 3), (4, 2, 4), (5, 2, 5);
```

***

## 운영  효과: 개발자 개입 최소화(셀프서비스 권한 관리)

* **무배포 정책 변경**: 인가 정책을 코드가 아닌 데이터(`user_roles`, `role_privileges`)로 관리하므로, 운영/보안 담당자가 UI나 승인된 배치로 매핑만 변경하면 즉시 반영된다. 개발자가 데이터를 직접 조작하거나 코드를 고쳐 배포할 필요가 없다.
* **셀프서비스 권한 관리**: 어드민 콘솔에서 사용자-역할, 역할-권한 매핑을 관리하도록 만들기 쉽다. 온보딩/조직 변경 때 운영자가 스스로 권한을 부여/회수 가능.
* **정책 거버넌스와 감사(Audit)**: 매핑 변경 이력을 별도 테이블로 남기고(예: `auth_policy_change_log`), 승인 워크플로를 붙이면 누가 언제 어떤 권한을 변경했는지 추적 가능.
* **임시/시간제한 권한 부여**: 매핑 테이블에 `granted_until` 같은 유효기간 컬럼을 확장하고, `RbacAuthoritiesFilter` 조회 시 현재 시각 기준으로 필터링하면 기간 한정 권한을 안전하게 운용 가능.
* **롤백과 실험(A/B)**: 새로운 기능 권한을 역할에 점진적으로 붙였다 떼는 방식으로 실험과 롤백이 쉽다. 실패 시 매핑만 되돌리면 된다.
* **멀티테넌시/팀별 정책 응용**: 권한 네임스페이스(예: `ORG_A:EVENT_CREATE`)를 도입하면 테넌트/조직별로 다른 정책을 같은 코드로 운용 가능.

> 한 줄 요약: R\&PBAC은 접근 정책을 “코드”가 아니라 “데이터”로 다루게 하므로, 운영이 개발자에게 의존하지 않고 자율·민첩하게 권한을 관리할 수 있게 만든다.

***

## 마무리

* 본 프로젝트는 역할과 권한을 분리해 유연성과 유지보수성을 확보했고, 스프링 시큐리티 필터 체인을 활용해 인증과 인가의 책임을 명확히 분리했습니다.&#x20;
* 컨트롤러 단의 인가는 `@PreAuthorize("hasAuthority('...')")`로 일관되게 표현되어 가독성이 높고, 운영 중 정책 변경에도 신속히 대응 가능합니다.
