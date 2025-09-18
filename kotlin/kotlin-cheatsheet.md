---
description: 코틀린 치트시트
---

# Kotlin CheatSheet

<details>

<summary>Summary</summary>

### Lec 00. 코틀린 34가지 사실

* 간결성, null safety, 함수형, 자바 100% 호환
* `?.`, `?:`, 스마트 캐스트, 데이터 클래스 등 핵심 기능

***

### Lec 01. 변수

* `val` (불변), `var` (가변)
* 타입 추론 지원

***

### Lec 02. Null

* `?`: nullable 타입
* `?.`: safe call
* `?:`: elvis 연산자
* `!!`: 강제 not-null

***

### Lec 03. Type

* 모든 타입은 객체, JVM 최적화 시 primitive 변환
* 특별한 타입: `Any`, `Unit`, `Nothing`
* 스마트 캐스트(`is` → 자동 변환)

***

### Lec 04. 연산자

* `==`: 값 비교 / `===`: 참조 비교
* `in`, `!in`: 포함 여부
* `..`, `until`, `downTo`, `step`: 범위 연산자

***

### Lec 05. 조건문

* `if`는 표현식
* `when`: switch 대체, 값/타입/범위 검사 가능

***

### Lec 06. 반복문

* `for (i in 0 until n)`
* `withIndex()` → 인덱스 + 값
* `while`, `do-while` 자바와 동일

***

### Lec 07. 예외

* Checked Exception 없음 → `throws` 불필요
* `try/catch/finally`는 표현식
* 함수형 스타일: `runCatching`, `Result<T>`

***

### Lec 08. 함수

* `fun add(a: Int, b: Int) = a + b`
* 기본값, named parameter, vararg 지원
* 최상위 함수 가능 → static 대체

***

### Lec 09. 클래스

* `class User(val id: Int, var name: String)`
* `init` 블록으로 초기화 로직
* getter/setter 자동 생성, 필요 시 커스터마이즈 가능

***

### Lec 10. 상속

* 기본적으로 `final`, 상속하려면 `open`
* `abstract`, `interface` 지원
* `override` 필수

***

### Lec 11. 접근 제어

* `public` (기본), `private`, `protected`, `internal`
* getter/setter에 다른 접근 제어 가능
* `internal`: 모듈 경계 단위 제어

***

### Lec 12. object

* `object` → 싱글톤
* `companion object` → static 대체
* `object : 인터페이스 {}` → 익명 객체

***

### Lec 13. 중첩 클래스

* `class Nested`: static nested
* `inner class Inner`: 외부 참조 가능 (`this@Outer`)

***

### Lec 14. 다양한 클래스

* `data class`: equals, hashCode, toString, copy 자동 생성
* `enum class`: 상수 집합
* `sealed class`: 제한된 클래스 계층 → when과 함께 exhaustiveness 체크

***

### Lec 15. 배열과 컬렉션

* 배열: `arrayOf`, `intArrayOf`
* 컬렉션: `listOf`(불변), `mutableListOf`(가변)
* 함수형 API: `map`, `filter`, `reduce`

***

### Lec 16. 고급 함수 문법

* `infix`: DSL 스타일 중위 호출
* `inline`: 성능 최적화, 람다 인라인 처리
* 지역 함수(local function): 함수 내부에 함수 정의

***

### Lec 17. 람다

* `{ 파라미터 -> 본문 }`
* 마지막 파라미터는 트레일링 람다 가능
* 클로저 지원 (non-final 변수 사용 가능)

***

### Lec 18. 컬렉션 함수형 처리

* `filter`, `map`, `mapNotNull`
* 조건: `all`, `any`, `none`
* 집계/정렬: `count`, `sortedBy`, `distinctBy`
* 변환: `groupBy`, `associateBy`, `flatMap`, `flatten`

***

### Lec 19. 이모저모

* `typealias`: 타입 별칭
* `as import`: import 충돌 방지
* 구조분해 선언: `(a, b) = pair`
* `takeIf`, `takeUnless`: 조건부 반환
* Jump+Label: `return@label`, `break@label` (지양)

***

### Lec 20. Scope Function

* `let`: null-safe, 결과 반환 (`it`)
* `run`: 초기화 + 결과 반환 (`this`)
* `apply`: 설정, 객체 반환 (`this`)
* `also`: 부수 효과, 객체 반환 (`it`)
* `with`: 컨텍스트 실행 (`this`)

</details>



## 변수

개요

* `val`: 불변(immutable), 재할당 불가 (Java의 `final`에 해당)
* `var`: 가변(mutable), 재할당 가능
* 타입 추론 지원: 명시하지 않아도 컴파일러가 타입을 결정

***

예시1 (기본 문법)

#### (1) Java → Kotlin 전환

```java
// Java
int count = 10;
final String greeting = "Hello";
```

```kotlin
// Kotlin
var count = 10        // 가변
val greeting = "Hello" // 불변
```

#### (2) 타입 추론

```java
// Java
String name = "Joon";
int age = 28;
```

```kotlin
// Kotlin
val name = "Joon"   // String으로 추론
val age = 28        // Int로 추론
```

***

예시2 (실무 활용)

#### (1) 서버 설정값 정의

```kotlin
val MAX_RETRY = 3
var currentRetry = 0

fun retryConnection() {
    if (currentRetry < MAX_RETRY) {
        currentRetry++
        println("Retry $currentRetry...")
    }
}
```

→ `MAX_RETRY`는 상수, `currentRetry`는 실행 중 변경되는 값.

#### (2) Spring Boot 환경설정

```kotlin
@Component
class CacheConfig(
    @Value("\${cache.ttl}") val ttl: Long,  // 환경설정값 불변
) {
    var hitCount: Int = 0   // 런타임 통계 (가변)
}
```

→ 설정값은 `val`로 불변, 실행 중 수집되는 통계는 `var`로 관리.

***

## Null

개요

* 코틀린은 **Null Safety**를 언어 차원에서 제공
* `?` : nullable 타입
* `?.` : Safe call
* `?:` : Elvis 연산자
* `!!` : Not-null 단언 (주의 필요, 실무에서는 최소화)

***

예시1 (기본 문법)

#### (1) Java → Kotlin 전환

```java
// Java
String name = null;
if (name != null) {
    System.out.println(name.length());
} else {
    System.out.println(0);
}
```

```kotlin
// Kotlin
val name: String? = null
println(name?.length ?: 0)
```

***

#### (2) 안전하지 않은 코드 vs 안전한 코드

```kotlin
// 위험한 방법
val len1 = name!!.length   // null이면 NPE 발생

// 안전한 방법
val len2 = name?.length ?: 0
```

***

예시2 (실무 활용)

#### (1) DB 조회 결과 처리

```kotlin
fun findUser(id: Long): User? = repo.findById(id)

val userName = findUser(1)?.name ?: "Guest"
```

→ DB에서 유저가 없을 경우 `Guest` 반환.

***

#### (2) API 요청 파라미터 처리

```kotlin
data class CreateUserRequest(
    val name: String?,
    val email: String?
)

fun toEntity(req: CreateUserRequest): User {
    return User(
        name = req.name ?: throw IllegalArgumentException("Name is required"),
        email = req.email ?: "unknown@example.com"
    )
}
```

→ 필수 값은 `?: throw`, 선택 값은 기본값으로 처리.



## Type

개요

* 코틀린은 \*\*기본 타입도 객체(Reference Type)\*\*로 다룸
* 컴파일러가 최적화 시 JVM의 primitive로 변환 (성능 동일)
* 스마트 캐스트(smart cast) 지원 → `is`로 타입 검사 후 안전하게 변환
* Any (최상위 타입), Unit (Java의 void), Nothing (절대 리턴 불가 타입) 존재

***

예시1 (기본 문법)

#### (1) Java Primitive vs Kotlin

```java
// Java
int a = 1;
Integer b = 2;
```

```kotlin
// Kotlin
val a: Int = 1     // JVM에서 primitive int로 컴파일
val b: Int? = 2    // nullable → boxed Integer로 처리
```

***

#### (2) 스마트 캐스트

```java
// Java
Object obj = "hello";
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}
```

```kotlin
// Kotlin
val obj: Any = "hello"
if (obj is String) {
    println(obj.length)  // 자동 캐스트
}
```

***

예시2 (실무 활용)

#### (1) API 응답 처리

```kotlin
fun parse(data: Any): String =
    if (data is String) data
    else data.toString()

val result = parse(123)   // "123"
```

→ JSON 파싱, DB 결과 변환 등에서 다양한 타입을 안전하게 처리 가능.

***

#### (2) Nothing 타입 활용

```kotlin
fun fail(msg: String): Nothing {
    throw IllegalArgumentException(msg)
}

fun getUser(id: Long): User =
    repo.findById(id) ?: fail("User not found")
```

→ `Nothing`은 함수가 정상적으로 리턴하지 않음을 표현 → null-safe 코드 작성 가능.



## 연산자

개요

* 코틀린의 연산자는 사실상 **함수 호출로 해석**됨 (`a + b` → `a.plus(b)`)
* `==` vs `===`: 값 비교와 참조 비교 구분
* `is`, `!is`: 타입 검사
* `in`, `!in`: 포함 여부 검사
* 범위 연산자: `..`, `until`, `downTo`, `step`
* if/when도 **표현식(Expression)** 으로 값 할당 가능

***

예시1 (기본 문법)

#### (1) 값 비교 vs 참조 비교

```java
// Java
String s1 = new String("hi");
String s2 = new String("hi");
System.out.println(s1.equals(s2)); // true
System.out.println(s1 == s2);      // false
```

```kotlin
// Kotlin
val s1 = String(charArrayOf('h','i'))
val s2 = String(charArrayOf('h','i'))
println(s1 == s2)   // true (equals)
println(s1 === s2)  // false (reference)
```

***

#### (2) 범위 연산자

```java
// Java
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}
```

```kotlin
// Kotlin
for (i in 1..5) println(i)     // 1,2,3,4,5
for (i in 1 until 5) println(i)// 1,2,3,4
```

***

예시2 (실무 활용)

#### (1) 권한 검사

```kotlin
enum class Role { ADMIN, USER, GUEST }

fun authorize(role: Role) =
    if (role == Role.ADMIN) "Full Access" else "Limited"
```

→ API 보안 로직에서 값 비교(`==`) 활용.

***

#### (2) 리스트 필터링

```kotlin
val emails = listOf("a@test.com", "b@test.com", "c@test.com")
if ("b@test.com" in emails) {
    println("User exists")
}
```

→ 서버에서 특정 값이 컬렉션에 포함되는지 `in`으로 직관적으로 확인.



## 조건문

개요

* `if`는 문(statement)뿐 아니라 **표현식(expression)** → 값 할당 가능
* `when`은 자바의 `switch`를 확장한 문법
  * 값 매칭, 타입 검사, 범위 검사, 여러 조건 지원
  * `else`는 default와 동일한 역할
* 실무에서는 권한 처리, 입력 검증, 상태 전이 로직에 자주 활용

***

예시1 (기본 문법)

#### (1) if 표현식

```java
// Java
int max;
if (a > b) {
    max = a;
} else {
    max = b;
}
```

```kotlin
// Kotlin
val max = if (a > b) a else b
```

***

#### (2) switch vs when

```java
// Java
switch (x) {
    case 1: System.out.println("one"); break;
    case 2: 
    case 3: System.out.println("two or three"); break;
    default: System.out.println("other");
}
```

```kotlin
// Kotlin
when (x) {
    1 -> println("one")
    2, 3 -> println("two or three")
    else -> println("other")
}
```

***

예시2 (실무 활용)

#### (1) API 권한 처리

```kotlin
fun authorize(role: Role) = when(role) {
    Role.ADMIN -> "Full Access"
    Role.USER -> "Limited"
    Role.GUEST -> "Read Only"
}
```

→ API 보안 로직에서 enum 기반 권한 분기.

***

#### (2) 요청 파라미터 검증

```kotlin
fun validateAge(age: Int) {
    if (age !in 0..120) {
        throw IllegalArgumentException("Invalid age")
    }
}
```

→ 서버 입력값 검증 로직에서 범위 조건 활용.



## 반복문

핵심 요약

* `for (i in 0 until n)` → 범위 기반 반복
* `for (item in list)` → Iterable 반복
* `withIndex()` → 인덱스 + 값 동시 사용
* `while`, `do-while` → 자바와 동일
* 실무: 배치, 로그 출력, 컬렉션 처리에서 활용

***

예시1 (기본 문법)

#### (1) Java for vs Kotlin for-in

```java
// Java
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}
```

```kotlin
// Kotlin
for (i in 0 until 5) println(i)   // 0,1,2,3,4
```

***

#### (2) 리스트 순회

```java
// Java
List<String> list = Arrays.asList("A","B","C");
for (String item : list) {
    System.out.println(item);
}
```

```kotlin
// Kotlin
val list = listOf("A","B","C")
for (item in list) println(item)
```

***

예시2 (실무 활용)

#### (1) 인덱스와 값 동시 사용

```kotlin
val names = listOf("Joon", "Tom", "Amy")
for ((index, name) in names.withIndex()) {
    println("$index: $name")
}
```

→ 서버 로그 출력 시 유용.

***

#### (2) 데이터 처리 로직

```kotlin
val orders = repo.findAll()
for (order in orders) {
    if (order.isExpired()) {
        order.cancel()
    }
}
```

→ 배치 작업에서 주문 만료 처리 로직.



## 예외

개요

* 코틀린은 **Checked Exception 없음** → 모든 예외는 Unchecked
* `try`/`catch`/`finally` 문법은 자바와 동일, 하지만 **표현식**으로도 사용 가능
* 함수형 스타일로 `runCatching`, `Result` 타입 자주 활용
* 사용자 정의 예외는 `RuntimeException` 상속

핵심 요약

* Checked Exception 없음 → `throws` 불필요
* `try`/`catch`/`finally`는 표현식으로도 사용 가능
* 함수형 스타일: `runCatching`, `Result<T>`
* 실무 활용: API 응답 Wrapping, 트랜잭션 처리

***

1\. Checked Exception 없음

#### 기본 문법 예시

```kotlin
// Kotlin - throws 선언 필요 없음
fun readFile(path: String): String {
    throw IOException("File not found")
}
```

```kotlin
// Kotlin - try 블록에서 바로 예외 발생 가능
val text = try {
    readFile("data.txt")
} catch (e: IOException) {
    "default"
}
```

#### 실무 활용 예시

```kotlin
// JPA save에서 SQLException 발생 가능
fun saveUser(user: User): User {
    return repo.save(user) // throws 선언 필요 없음
}
```

```kotlin
// 외부 API 호출
fun callApi(): String {
    val response = httpClient.get("https://api.test.com") 
    if (!response.isSuccessful) throw RuntimeException("API Error")
    return response.body
}
```

***

2\. try/catch/finally (표현식)

#### 기본 문법 예시

```kotlin
val number = try {
    "123".toInt()
} catch (e: NumberFormatException) {
    0
}
```

```kotlin
val result = try {
    riskyOperation()
} catch (e: Exception) {
    -1
} finally {
    println("always runs")
}
```

#### 실무 활용 예시

```kotlin
val fileContent = try {
    Files.readString(Paths.get("config.yaml"))
} catch (e: IOException) {
    logger.error("Failed to load config", e)
    "{}"
}
```

```kotlin
val json = try {
    objectMapper.writeValueAsString(dto)
} catch (e: Exception) {
    "{}"
}
```

***

3\. runCatching

#### 기본 문법 예시

```kotlin
val result = runCatching { "123".toInt() }
println(result.getOrDefault(0))
```

```kotlin
val res = runCatching { riskyOperation() }
    .onSuccess { println("ok: $it") }
    .onFailure { println("error: ${it.message}") }
```

#### 실무 활용 예시

```kotlin
fun <T> safe(block: () -> T): Result<T> =
    runCatching { block() }

val res = safe { service.getUser(1) }
```

```kotlin
val user = runCatching { repo.findById(1) }
    .getOrElse { throw BusinessException("User not found") }
```

***

4\. Result\<T>

#### 기본 문법 예시

```kotlin
val success: Result<Int> = Result.success(42)
val failure: Result<Int> = Result.failure(Exception("Oops"))
```

```kotlin
val res = Result.runCatching { "abc".toInt() }
println(res.getOrElse { -1 })
```

#### 실무 활용 예시

```kotlin
fun loadUser(id: Long): Result<User> =
    Result.runCatching { repo.findById(id)!! }
```

```kotlin
val output = service.loadConfig()
    .fold(
        onSuccess = { "Config loaded: $it" },
        onFailure = { "Fallback config" }
    )
```

***

5\. 사용자 정의 예외 (RuntimeException 상속)

#### 기본 문법 예시

```kotlin
class InvalidAgeException(message: String) : RuntimeException(message)

fun validate(age: Int) {
    if (age < 0) throw InvalidAgeException("Age cannot be negative")
}
```

```kotlin
class AuthenticationException: RuntimeException("Invalid credentials")
```

#### 실무 활용 예시

```kotlin
class BusinessException(val code: String, msg: String) : RuntimeException(msg)

fun findUser(id: Long): User =
    repo.findById(id) ?: throw BusinessException("USER_NOT_FOUND", "User $id not found")
```

```kotlin
class TokenExpiredException: RuntimeException("JWT expired")

fun validateToken(token: String) {
    if (jwt.isExpired(token)) throw TokenExpiredException()
}
```



## 함수

개요

* `fun` 키워드로 함수 선언
* 반환 타입은 `: Type`, 추론 가능 시 생략
* 블록 본문 vs 식(Expression) 본문 (`=` 사용)
* 기본값 파라미터, 이름 붙은 인자, 가변 인자 지원
* 최상위 함수 가능 (클래스 밖 함수)
* 자바의 static 유틸 메소드보다 간결

핵심 요약

* `fun name(param: Type): Return`
* 블록 본문 `{}` vs 식 본문 `=`
* 기본값 파라미터 + 이름 붙은 인자 → 오버로딩 대체
* 최상위 함수 지원 → 유틸/헬퍼 함수 작성 용이
* 실무 활용: 컨트롤러 파라미터 처리, 유틸 메소드, DTO 변환 등

***

예시1 (기본 문법)

#### (1) Java → Kotlin 변환

```java
// Java
public int add(int a, int b) {
    return a + b;
}
```

```kotlin
// Kotlin
fun add(a: Int, b: Int): Int = a + b
```

***

#### (2) 기본값 + 이름 붙은 인자

```java
// Java - 오버로딩 필요
public String greet(String name) { return greet(name, 1); }
public String greet(String name, int times) {
    return "Hello " + name.repeat(times);
}
```

```kotlin
// Kotlin
fun greet(name: String, times: Int = 1) =
    "Hello ".repeat(times) + name

println(greet("Joon"))          // 기본값 사용
println(greet("Joon", times=3)) // named argument
```

***

예시2 (실무 활용)

#### (1) Spring Boot Controller

```kotlin
@RestController
class HelloController {
    @GetMapping("/hello")
    fun hello(@RequestParam name: String = "World"): String =
        "Hello, $name"
}
```

→ 기본값 파라미터로 오버로딩 줄이고 코드 간결화.

***

#### (2) 유틸 함수 (최상위 함수)

```kotlin
// Utils.kt
fun now(): Long = System.currentTimeMillis()

// 사용
val timestamp = now()
```

→ 자바처럼 static 유틸 클래스 필요 없음, 최상위 함수로 해결.



## 클래스

개요 & 핵심 요약

* `class` 키워드로 클래스 선언
* 생성자: 주 생성자(primary) + 보조 생성자(secondary)
* 프로퍼티(Property): `val` (읽기 전용), `var` (읽기/쓰기)
* 기본 접근 제어자는 `public`
* `init` 블록으로 초기화 로직 실행
* getter/setter는 자동 생성되며, 필요 시 오버라이딩 가능
* **핵심 요약**
  * `class User(val id: Int, var name: String)` → 프로퍼티 + 생성자 한 줄로
  * `init` 블록 → 생성 시 로직 실행
  * `val` 불변, `var` 가변
  * 자바보다 보일러플레이트가 대폭 감소

***

예시1 (기본 문법)

#### (1) Java → Kotlin 전환

```java
// Java
public class User {
    private final int id;
    private String name;

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

```kotlin
// Kotlin
class User(val id: Int, var name: String)
```

***

#### (2) init 블록

```kotlin
class Order(val id: Long, val amount: Int) {
    init {
        require(amount > 0) { "Amount must be positive" }
    }
}
```

→ 객체 생성 시 검증 로직 수행.

***

예시2 (실무 활용)

#### (1) JPA Entity 정의

```kotlin
@Entity
class User(
    @Id @GeneratedValue
    val id: Long? = null,
    var email: String,
    var name: String
)
```

→ 불변 id + 가변 필드 조합으로 DB 엔티티 표현.

***

#### (2) DTO 클래스 정의

```kotlin
data class UserDto(
    val id: Long,
    val name: String
)

fun User.toDto() = UserDto(id!!, name)
```

→ Entity와 API 응답 DTO를 구분해서 사용하는 패턴.



## 상속

개요 & 핵심 요약

* 코틀린 클래스는 기본적으로 `final` → 상속 불가
* 상속을 허용하려면 `open` 키워드 필요
* 메소드 오버라이드는 `override` 키워드 필수
* 추상 클래스(`abstract`)는 인스턴스화 불가, 하위 클래스에서 구현 강제
* 인터페이스도 구현 가능하며, default 구현 지원
* **핵심 요약**
  * `open class` → 상속 가능
  * `abstract class` → 미구현 메소드 강제
  * `interface` → 다중 구현 가능, 디폴트 메소드 지원
  * 오버라이드 시 `override` 반드시 명시

***

1️⃣ `open`

#### 🔹 기본 문법 예시

**(1) 클래스 상속 허용**

```kotlin
open class Animal {
    open fun sound() = println("...")
}
class Dog: Animal() {
    override fun sound() = println("Woof")
}
```

**(2) 메서드 재정의 허용**

```kotlin
open class Vehicle {
    open fun move() = println("Moving...")
}
class Car: Vehicle() {
    override fun move() = println("Car moving fast")
}
```

#### 🔹 실무 활용 예시

**(1) 확장 가능한 서비스 클래스**

```kotlin
open class BaseService {
    open fun execute() = println("Base logic")
}
class UserService: BaseService() {
    override fun execute() = println("User logic")
}
```

**(2) 테스트에서 Mocking 용이**

```kotlin
open class EmailSender {
    open fun send(to: String) = println("Sending email to $to")
}
class FakeEmailSender: EmailSender() {
    override fun send(to: String) = println("Mock send to $to")
}
```

***

2️⃣ `abstract`

#### 🔹 기본 문법 예시

**(1) 추상 클래스**

```kotlin
abstract class Shape {
    abstract fun area(): Double
}
class Circle(val r: Double): Shape() {
    override fun area() = Math.PI * r * r
}
```

**(2) 추상 메서드 + 구현 메서드 혼합**

```kotlin
abstract class Animal {
    abstract fun sound(): String
    fun info() = "I am an animal"
}
class Dog: Animal() {
    override fun sound() = "Woof"
}
```

#### 🔹 실무 활용 예시

**(1) 서비스 템플릿**

```kotlin
abstract class BaseService {
    abstract fun execute()
}
class OrderService: BaseService() {
    override fun execute() = println("Order executed")
}
```

**(2) UI 컴포넌트 추상화**

```kotlin
abstract class View {
    abstract fun render()
}
class Button: View() {
    override fun render() = println("Render Button")
}
```

***

3️⃣ `interface`

#### 🔹 기본 문법 예시

**(1) 단순 인터페이스**

```kotlin
interface Walkable {
    fun walk()
}
class Person: Walkable {
    override fun walk() = println("Walking...")
}
```

**(2) default 메서드 제공**

```kotlin
interface Runnable {
    fun run() = println("Default running...")
}
class Robot: Runnable
```

#### 🔹 실무 활용 예시

**(1) 전략 패턴**

```kotlin
interface Payment {
    fun pay(amount: Int)
}
class CardPayment: Payment {
    override fun pay(amount: Int) = println("Pay $amount by card")
}
class KakaoPay: Payment {
    override fun pay(amount: Int) = println("Pay $amount by KakaoPay")
}
```

**(2) Enum + 인터페이스 조합**

```kotlin
interface Privilege { fun check(): Boolean }

enum class Role: Privilege {
    ADMIN { override fun check() = true },
    USER { override fun check() = false }
}
```

***

4️⃣ `override`

#### 🔹 기본 문법 예시

**(1) 메서드 오버라이드**

```kotlin
open class Parent {
    open fun greet() = println("Hello from Parent")
}
class Child: Parent() {
    override fun greet() = println("Hello from Child")
}
```

**(2) 프로퍼티 오버라이드**

```kotlin
open class Animal {
    open val sound: String = "..."
}
class Dog: Animal() {
    override val sound: String = "Woof"
}
```

#### 🔹 실무 활용 예시

**(1) JPA Entity 재정의**

```kotlin
import jakarta.persistence.*

@MappedSuperclass
open class BaseEntity {
    open val createdAt: Long = System.currentTimeMillis()
}

@Entity
class User(
    @Id @GeneratedValue val id: Long? = null,
    override val createdAt: Long = System.currentTimeMillis()
): BaseEntity()
```

**(2) 스프링 시큐리티 UserDetails 구현**

```kotlin
import org.springframework.security.core.userdetails.UserDetails

class CustomUser(
    private val username: String,
    private val password: String
): UserDetails {
    override fun getUsername() = username
    override fun getPassword() = password
    override fun isEnabled() = true
    override fun isAccountNonExpired() = true
    override fun isAccountNonLocked() = true
    override fun isCredentialsNonExpired() = true
    override fun getAuthorities() = emptyList<GrantedAuthority>()
}
```



## 접근 제어

개요 & 핵심 요약

* 코틀린 접근 제어자: `public`, `private`, `protected`, `internal`
* 기본값은 `public` (Java의 default와 다름)
* `protected`: 상속 관계에서만 접근 가능 (Java처럼 패키지 단위 X)
* `internal`: 같은 모듈(module) 내에서만 접근 가능
* 파일 최상위 함수/프로퍼티에도 접근 제어자 사용 가능
* 프로퍼티 getter/setter에 서로 다른 접근 제어자 부여 가능
* **핵심 요약**
  * 기본은 `public`
  * `internal`: 모듈 경계 제어
  * `protected`: 패키지 무시, 상속만 허용
  * getter/setter 가시성 다르게 설정 가능

***

예시1 (기본 문법)

#### (1) Java vs Kotlin 기본값 차이

```java
// Java
class User {   // default → 같은 패키지에서만 접근
    String name;
}
```

```kotlin
// Kotlin
class User(   // 기본은 public
    val name: String
)
```

***

#### (2) 프로퍼티 가시성 제어

```kotlin
class Account {
    var balance: Int = 0
        private set   // 외부에서 읽기는 가능, 쓰기는 불가
}
```

***

예시2 (실무 활용)

#### (1) 모듈 경계 제어 (internal)

```kotlin
// module A
internal class InternalService {
    fun run() = println("only inside module A")
}
```

→ 다른 모듈에서 접근 불가, 라이브러리 내부 구현 숨김.

***

#### (2) 생성자 제어

```kotlin
class User private constructor(val id: Long) {
    companion object {
        fun create(id: Long) = User(id)
    }
}
```

→ 팩토리 메소드 패턴, 외부에서 직접 생성 제한.



## object 키워드

개요 & 핵심 요약

* `object` 키워드는 세 가지 주요 용도
  1. **싱글톤 객체 선언**: 클래스 정의 + 인스턴스 생성 동시에 수행
  2. **companion object**: 클래스와 함께 존재하는 유일 객체, Java의 static 대체
  3. **익명 객체(Anonymous object)**: 일회성 구현체 생성
* 멀티스레드 환경에서도 안전하게 초기화됨
* **핵심 요약**
  * `object` → 싱글톤
  * `companion object` → static 대체, 팩토리/유틸리티 메소드 제공
  * `object : 인터페이스 {}` → 익명 객체

***

예시1 (기본 문법)

#### (1) 싱글톤 객체

```java
// Java (싱글톤)
public class Logger {
    private static final Logger INSTANCE = new Logger();
    private Logger() {}
    public static Logger getInstance() { return INSTANCE; }
}
```

```kotlin
// Kotlin
object Logger {
    fun log(msg: String) = println("[LOG] $msg")
}
Logger.log("Hello")
```

***

#### (2) companion object

```kotlin
class User private constructor(val id: Long) {
    companion object {
        fun create(id: Long) = User(id)
    }
}

val user = User.create(1)
```

***

예시2 (실무 활용)

#### (1) Repository 싱글톤

```kotlin
object UserRepository {
    private val users = mutableListOf<User>()
    fun findAll(): List<User> = users
}
```

→ 간단한 캐싱/저장소 패턴 구현.

***

#### (2) Spring Bean Factory 메소드

```kotlin
class JwtTokenProvider private constructor(val secret: String) {
    companion object {
        fun fromEnv() = JwtTokenProvider(System.getenv("JWT_SECRET"))
    }
}
```

→ companion object로 환경 변수 기반 Factory 메소드 제공.



## 중첩 클래스

개요 & 핵심 요약

* 코틀린에서 클래스 내부에 선언된 클래스는 기본적으로 **static nested class**처럼 동작 (외부 클래스 참조 X)
* 바깥 클래스 참조가 필요할 경우 `inner` 키워드 명시 → 바깥 인스턴스에 접근 가능
* 바깥 클래스 참조는 `this@Outer` 문법 사용
* 자바는 기본이 non-static inner class, 코틀린은 반대 (안전성이 우선)
* **핵심 요약**
  * `class Nested` → static nested class
  * `inner class Inner` → 외부 클래스 참조
  * 외부 참조 문법: `this@Outer`
  * 기본적으로 static nested → 메모리 누수 방지

***

예시1 (기본 문법)

#### (1) Java vs Kotlin 기본 차이

```java
// Java
class Outer {
    class Inner {} // 기본적으로 Outer 참조 보유
}
```

```kotlin
// Kotlin
class Outer {
    class Nested    // 바깥 참조 없음
    inner class Inner  // 바깥 참조 있음
}
```

***

#### (2) 외부 참조 접근

```kotlin
class Outer(val name: String) {
    inner class Inner {
        fun printOuter() = println(this@Outer.name)
    }
}

Outer("Joon").Inner().printOuter() // Joon
```

***

예시2 (실무 활용)

#### (1) Helper 클래스 (Nested)

```kotlin
class JsonParser {
    class Tokenizer {
        fun tokenize(s: String) = s.split(" ")
    }
}
```

→ 외부 상태 필요 없음 → Nested class로 안전하게 설계.

***

#### (2) Validator (Inner)

```kotlin
class Order(val id: Long, val items: List<String>) {
    inner class Validator {
        fun validate() {
            if (items.isEmpty()) throw IllegalArgumentException("Empty order")
        }
    }
}

Order(1, listOf()).Validator().validate()
```

→ Order의 상태(`items`)를 참조해야 하므로 inner class로 구현.



## 다양한 클래스 (Data, Enum, Sealed)

개요 & 핵심 요약

* **Data class**
  * `data` 키워드 → equals, hashCode, toString, copy, componentN 자동 생성
  * 주로 DTO, 값 객체(Value Object)에 사용
* **Enum class**
  * 제한된 상수 집합 표현
  * 각 상수에 필드/메소드 정의 가능
* **Sealed class**
  * 제한된 클래스 계층
  * `when`과 함께 사용 시 컴파일러가 모든 경우 체크
  * 상태(State)와 결과(Result) 표현에 적합
* **핵심 요약**
  * Data → DTO/VO, 불변 객체
  * Enum → 권한, 상태, 타입 구분
  * Sealed → API 응답, 상태 전이, 오류 처리

***

📌 Data Class

#### 🔹 기본 문법 예시

**(1) DTO 정의**

```kotlin
data class UserDto(val id: Long, val name: String)
```

**(2) copy & toString**

```kotlin
data class Product(val id: Long, val name: String, val price: Int)

val p1 = Product(1, "Keyboard", 20000)
val p2 = p1.copy(price = 25000)

println(p1) // Product(id=1, name=Keyboard, price=20000)
println(p2) // Product(id=1, name=Keyboard, price=25000)
```

***

#### 🔹 실무 활용 예시

**(1) 엔티티 → DTO 변환**

```kotlin
data class UserResponse(val id: Long, val name: String)

fun User.toResponse() = UserResponse(id!!, name)
```

→ 컨트롤러에서 Entity → DTO 변환을 간결하게 처리.

**(2) API 요청/응답 직렬화**

```kotlin
data class LoginRequest(val username: String, val password: String)
data class LoginResponse(val token: String, val expiresAt: Long)
```

→ REST API 요청/응답 모델을 직렬화/역직렬화에 활용.

***

📌 Enum Class

#### 🔹 기본 문법 예시

**(1) 상수 집합**

```kotlin
enum class Role { ADMIN, USER, GUEST }
```

**(2) 값 포함**

```kotlin
enum class Direction(val symbol: String) {
    NORTH("↑"), SOUTH("↓"), EAST("→"), WEST("←")
}
```

***

#### 🔹 실무 활용 예시

**(1) 권한 체크**

```kotlin
enum class Role { ADMIN, USER, GUEST }

fun hasAccess(role: Role) = role == Role.ADMIN
```

**(2) 주문 상태 관리**

```kotlin
enum class OrderStatus(val isFinal: Boolean) {
    PENDING(false), SHIPPED(false), DELIVERED(true), CANCELED(true);

    fun canTransitionTo(next: OrderStatus): Boolean =
        !this.isFinal && this != next
}
```

***

📌 Sealed Class

#### 🔹 기본 문법 예시

**(1) 결과 표현**

```kotlin
sealed class Result {
    data class Success(val data: String): Result()
    data class Error(val message: String): Result()
}
```

**(2) 상태 표현**

```kotlin
sealed class UiState {
    object Loading : UiState()
    data class Content(val items: List<String>) : UiState()
    data class Error(val cause: Throwable) : UiState()
}
```

***

#### 🔹 실무 활용 예시

**(1) API 응답 Wrapping**

```kotlin
sealed class ServiceResult<out T> {
    data class Ok<T>(val value: T): ServiceResult<T>()
    data class Fail(val reason: String): ServiceResult<Nothing>()
}

fun findUser(id: Long): ServiceResult<UserDto> =
    repo.find(id)?.let { ServiceResult.Ok(it) }
        ?: ServiceResult.Fail("Not found")
```

**(2) UI 상태 전이**

```kotlin
fun render(state: UiState) = when (state) {
    UiState.Loading -> println("Loading...")
    is UiState.Content -> println("Items: ${state.items}")
    is UiState.Error -> println("Error: ${state.cause.message}")
}
```



## 배열과 컬렉션

개요 & 핵심 요약

* 배열: `arrayOf`, `intArrayOf`, `Array(size) { init }` 등으로 생성
* 하지만 코틀린에서는 배열보다 **컬렉션(List, Set, Map)** 사용이 권장됨
* 컬렉션은 **불변(immutable)** 과 **가변(mutable)** 버전이 구분됨
* 함수형 API (`filter`, `map`, `reduce`, `sortedBy` 등) 제공
* **핵심 요약**
  * 배열보단 컬렉션 활용
  * 불변: `listOf`, `setOf`, `mapOf`
  * 가변: `mutableListOf`, `mutableSetOf`, `mutableMapOf`
  * 컬렉션 처리 → 함수형 API 적극 활용

***

예시1 (기본 문법)

#### (1) 배열 생성

```java
// Java
int[] arr = {1, 2, 3};
```

```kotlin
// Kotlin
val arr = intArrayOf(1, 2, 3)
val arr2 = Array(3) { i -> i * 2 }  // [0, 2, 4]
```

***

#### (2) 불변/가변 컬렉션

```kotlin
val nums = listOf(1, 2, 3)          // 불변
val mnums = mutableListOf(1, 2, 3)  // 가변
mnums.add(4)
```

***

예시2 (실무 활용)

#### (1) DTO 변환

```kotlin
val users = rows.map { UserDto(it.id, it.name) }
```

→ DB 조회 결과를 API DTO 리스트로 변환.

***

#### (2) 캐싱된 데이터 필터링

```kotlin
val activeTokens = tokens.filter { it.expiresAt > now() }
```

→ 만료되지 않은 토큰만 남기기.



## 고급 함수 문법 (infix, inline, local function)

개요 & 핵심 요약

* **infix 함수**
  * 중위 호출 지원 → `a to b` 처럼 더 읽기 좋은 DSL 스타일 문법
  * 조건: 멤버 함수/확장 함수, 단일 파라미터만 허용
* **inline 함수**
  * 함수 호출 대신 함수 본문을 호출 지점에 삽입 → 람다 객체 생성/호출 오버헤드 제거
  * 고차 함수 성능 최적화에 사용
* **지역 함수(local function)**
  * 함수 내부에 함수 정의 가능
  * 중복 제거 및 캡슐화 강화, 외부로 노출 방지
* **핵심 요약**
  * infix → DSL/가독성
  * inline → 성능 최적화
  * local function → 코드 중복 제거 + 스코프 제한

***

📌 Infix 함수

* **중위 호출 지원** → `a to b` 처럼 DSL 스타일 문법 가능
* **조건**: 멤버 함수/확장 함수여야 하고, **단일 파라미터만 허용**

#### 🔹 기본 문법 예시

**(1) Int 확장**

```kotlin
infix fun Int.times(str: String) = str.repeat(this)

val result = 3 times "Hi "   // "Hi Hi Hi "
```

**(2) Pair 생성기**

```kotlin
infix fun <A, B> A.pairWith(that: B) = Pair(this, that)

val pair = "Alice" pairWith 20   // Pair("Alice", 20)
```

***

#### 🔹 실무 활용 예시

**(1) DSL 스타일 API (권한 체크)**

```kotlin
infix fun User.hasRole(role: Role): Boolean = role in this.roles

if (user hasRole Role.ADMIN) {
    println("관리자 접근 허용")
}
```

**(2) 컬렉션 빌더 DSL**

```kotlin
infix fun MutableList<String>.addItem(item: String) { this.add(item) }

val items = mutableListOf<String>()
items addItem "Apple"
items addItem "Banana"
println(items) // [Apple, Banana]
```

***

📌 Inline 함수

* **함수 호출 대신 본문을 호출 지점에 삽입**
* **람다 객체 생성/호출 오버헤드 제거** → 고차 함수 성능 최적화

#### 🔹 기본 문법 예시

**(1) 실행 시간 측정**

```kotlin
inline fun measureTime(block: () -> Unit) {
    val start = System.currentTimeMillis()
    block()
    println("Took ${System.currentTimeMillis() - start}ms")
}
```

**(2) repeat 실행**

```kotlin
inline fun repeatAction(n: Int, action: () -> Unit) {
    for (i in 1..n) action()
}

repeatAction(3) { println("Hello") }
```

***

#### 🔹 실무 활용 예시

**(1) 안전 실행 블록**

```kotlin
inline fun <T> runCatchingOrDefault(default: T, block: () -> T): T =
    try { block() } catch (e: Exception) { default }

val result = runCatchingOrDefault(0) { "abc".toInt() }  // 0
```

**(2) 트랜잭션 처리**

```kotlin
inline fun <T> transaction(block: () -> T): T {
    println("Begin Transaction")
    val result = block()
    println("Commit Transaction")
    return result
}

transaction {
    println("DB 작업 수행")
}
```

***

📌 Local Function

* **함수 내부에 함수 정의 가능**
* **중복 제거 및 캡슐화 강화**, 외부로 노출되지 않음

#### 🔹 기본 문법 예시

**(1) 간단한 중복 제거**

```kotlin
fun printUserInfo(name: String, age: Int) {
    fun validate(value: Int, field: String) {
        require(value > 0) { "$field must be positive" }
    }
    validate(age, "age")
    println("User: $name, $age")
}
```

**(2) 내부 전용 유틸**

```kotlin
fun calculateScore(scores: List<Int>): Int {
    fun normalize(score: Int) = score.coerceIn(0, 100)
    return scores.sumOf { normalize(it) }
}
```

***

#### 🔹 실무 활용 예시

**(1) 검증 로직 캡슐화**

```kotlin
fun createOrder(req: OrderRequest) {
    fun checkPositive(value: Int, field: String) {
        require(value > 0) { "$field must be positive" }
    }
    checkPositive(req.qty, "qty")
    checkPositive(req.price, "price")
}
```

**(2) 파싱 로직 분리**

```kotlin
fun parseCsvLine(line: String): List<String> {
    fun clean(field: String) = field.trim().removeSurrounding("\"")
    return line.split(",").map { clean(it) }
}
```



## 람다

개요 & 핵심 요약

* **람다(lambda)**: 함수 자체를 값처럼 다루는 문법 (`{ 파라미터 -> 본문 }`)
* 함수 타입: `(매개변수 타입, ...) -> 반환 타입`
* 마지막 파라미터가 함수라면, 소괄호 밖에 작성 가능 (트레일링 람다)
* 클로저(closure): 외부 변수를 캡처 가능 (자바와 달리 final 필요 없음)
* **핵심 요약**
  * `val f: (Int) -> Int = { x -> x * 2 }`
  * `list.forEach { println(it) }`
  * `runCatching { ... }` 처럼 함수형 스타일 활용
  * 실무에서는 컬렉션 처리, 고차 함수, DSL에 자주 쓰임

***

예시1 (기본 문법 전/후)

#### (1) Java Stream vs Kotlin 람다

```java
// Java
List<Integer> nums = Arrays.asList(1, 2, 3);
nums.stream().map(n -> n * 2).forEach(System.out::println);
```

```kotlin
// Kotlin
val nums = listOf(1, 2, 3)
nums.map { it * 2 }.forEach { println(it) }
```

***

#### (2) 클로저 활용

```java
// Java (final 필요)
int[] counter = {0};
Runnable r = () -> counter[0]++;
```

```kotlin
// Kotlin (자유롭게 캡처)
var counter = 0
val inc = { counter++ }
inc(); println(counter)  // 1
```

***

예시2 (실무 활용)

#### (1) 공통 로직 유틸

```kotlin
fun <T> measureTime(block: () -> T): T {
    val start = System.currentTimeMillis()
    return block().also { println("Took ${System.currentTimeMillis()-start}ms") }
}

measureTime { service.callExternalApi() }
```

→ 고차 함수로 성능 로깅을 간결하게 처리.

***

#### (2) DSL 스타일 API

```kotlin
fun route(path: String, handler: (Request) -> Response) { ... }

route("/health") { req -> Response(200, "OK") }
```

→ Ktor 같은 서버 프레임워크에서 라우팅 DSL을 직관적으로 정의.



## 컬렉션을 함수형으로

개요 & 핵심 요약

* 코틀린 컬렉션은 **함수형 스타일 API**를 기본 지원
* `filter`, `map`, `reduce`, `groupBy`, `flatMap` 등 풍부한 연산 제공
* null-safe 버전(`mapNotNull`, `firstOrNull`)도 함께 제공
* 컬렉션 조합으로 **데이터 변환, 집계, 필터링, 매핑**을 간단히 처리 가능
* **핵심 요약**
  * 필터링: `filter`, `filterIndexed`
  * 매핑: `map`, `mapIndexed`, `mapNotNull`
  * 조건: `all`, `any`, `none`
  * 집계/정렬: `count`, `sortedBy`, `distinctBy`
  * 변환: `groupBy`, `associateBy`, `flatMap`, `flatten`

***

🔹 필터링

***

#### 1. `filter`

**기본 문법**

```kotlin
val nums = listOf(1,2,3,4,5)
val evens = nums.filter { it % 2 == 0 }     // [2,4]
```

```kotlin
val words = listOf("apple","banana","pear")
val aWords = words.filter { it.startsWith("a") } // ["apple"]
```

**실무 활용**

```kotlin
val users = listOf(User("A",true), User("B",false))
val activeUsers = users.filter { it.active } // 활성 사용자만
```

```kotlin
val orders = orderRepo.findAll()
val expensive = orders.filter { it.amount > 10000 }
```

***

#### 2. `filterIndexed`

**기본 문법**

```kotlin
val letters = listOf("a","b","c","d")
val evenIndex = letters.filterIndexed { i, _ -> i % 2 == 0 } // ["a","c"]
```

```kotlin
val nums = listOf(10,20,30)
val gtIndex = nums.filterIndexed { i, n -> n > i*10 } // [30]
```

**실무 활용**

```kotlin
val logs = listOf("INFO","WARN","ERROR")
val oddIndexed = logs.filterIndexed { i, _ -> i % 2 == 1 }
```

```kotlin
val batch = items.filterIndexed { i, _ -> i < 100 } // 첫 100개 페이징
```

***

🔹 매핑

***

#### 3. `map`

**기본 문법**

```kotlin
val nums = listOf(1,2,3)
val squares = nums.map { it*it } // [1,4,9]
```

```kotlin
val names = listOf("a","b")
val upper = names.map { it.uppercase() } // ["A","B"]
```

**실무 활용**

```kotlin
val entities = repo.findAll()
val dtos = entities.map { it.toDto() }
```

```kotlin
val files = listOf("a.txt","b.txt")
val sizes = files.map { File(it).length() }
```

***

#### 4. `mapIndexed`

**기본 문법**

```kotlin
val words = listOf("a","b")
val mapped = words.mapIndexed { i, s -> "$i:$s" } // ["0:a","1:b"]
```

```kotlin
val nums = listOf(10,20)
val doubled = nums.mapIndexed { i,n -> i+n } // [10,21]
```

**실무 활용**

```kotlin
val items = cart.mapIndexed { i, item -> OrderLine(i+1, item) }
```

```kotlin
val cols = header.mapIndexed { i, col -> i to col }
```

***

#### 5. `mapNotNull`

**기본 문법**

```kotlin
val strs = listOf("1","a","2")
val ints = strs.mapNotNull { it.toIntOrNull() } // [1,2]
```

```kotlin
val maybe = listOf(null,"x",null)
val notNull = maybe.mapNotNull { it } // ["x"]
```

**실무 활용**

```kotlin
val prices = listOf("100","200",null,"abc")
val valid = prices.mapNotNull { it?.toIntOrNull() }
```

```kotlin
val tokens = input.split(" ").mapNotNull { parseToken(it) }
```

***

🔹 조건

***

#### 6. `all`

**기본 문법**

```kotlin
val nums = listOf(2,4,6)
nums.all { it % 2 == 0 } // true
```

```kotlin
val names = listOf("Ann","Amy")
names.all { it.startsWith("A") } // true
```

**실무 활용**

```kotlin
val users = listOf(User("A",true), User("B",true))
val allActive = users.all { it.active }
```

```kotlin
val passwords = inputs.all { it.length >= 8 }
```

***

#### 7. `any`

**기본 문법**

```kotlin
val nums = listOf(1,2,3)
nums.any { it > 2 } // true
```

```kotlin
val empty = emptyList<Int>()
empty.any() // false
```

**실무 활용**

```kotlin
val orders = orders.any { it.status == "CANCELLED" }
```

```kotlin
val roles = user.roles.any { it == "ADMIN" }
```

***

#### 8. `none`

**기본 문법**

```kotlin
val nums = listOf(1,2,3)
nums.none { it < 0 } // true
```

```kotlin
val words = listOf("apple","pear")
words.none { it.length > 10 } // true
```

**실무 활용**

```kotlin
val users = users.none { it.banned }
```

```kotlin
val jobs = queue.none { it.isRunning }
```

***

🔹 집계/정렬

***

#### 9. `count`

**기본 문법**

```kotlin
val nums = listOf(1,2,2,3)
nums.count { it==2 } // 2
```

```kotlin
val empty = emptyList<String>()
empty.count() // 0
```

**실무 활용**

```kotlin
val errors = logs.count { it.level=="ERROR" }
```

```kotlin
val duplicates = data.count { it==target }
```

***

#### 10. `sortedBy`

**기본 문법**

```kotlin
val words = listOf("ccc","a","bb")
words.sortedBy { it.length } // ["a","bb","ccc"]
```

```kotlin
val nums = listOf(3,1,2)
nums.sortedBy { it } // [1,2,3]
```

**실무 활용**

```kotlin
val players = players.sortedByDescending { it.score }
```

```kotlin
val events = events.sortedBy { it.timestamp }
```

***

#### 11. `distinctBy`

**기본 문법**

```kotlin
val nums = listOf(1,1,2,2,3)
nums.distinctBy { it } // [1,2,3]
```

```kotlin
val users = listOf(User(1,"A"), User(2,"A"))
users.distinctBy { it.name } // User(1,"A")
```

**실무 활용**

```kotlin
val emails = listOf("a@a.com","a@a.com","b@b.com")
emails.distinctBy { it } // 중복 제거
```

```kotlin
val books = repo.findAll().distinctBy { it.isbn }
```

***

🔹 변환

***

#### 12. `groupBy`

**기본 문법**

```kotlin
val words = listOf("a","bb","ccc","dd")
words.groupBy { it.length }
// {1=["a"],2=["bb","dd"],3=["ccc"]}
```

```kotlin
val nums = listOf(1,2,3,4)
nums.groupBy { it%2 } // {1=[1,3],0=[2,4]}
```

**실무 활용**

```kotlin
val users = users.groupBy { it.department }
```

```kotlin
val orders = orders.groupBy { it.customerId }
```

***

#### 13. `associateBy`

**기본 문법**

```kotlin
val users = listOf(User(1,"A"), User(2,"B"))
val map = users.associateBy { it.id } 
// {1=User(1,"A"),2=User(2,"B")}
```

```kotlin
val strs = listOf("apple","banana")
val map = strs.associateBy { it.first() }
// {a="apple", b="banana"}
```

**실무 활용**

```kotlin
val books = books.associateBy { it.isbn }
```

```kotlin
val sessions = sessions.associateBy { it.token }
```

***

#### 14. `flatMap`

**기본 문법**

```kotlin
val nested = listOf(listOf(1,2), listOf(3))
nested.flatMap { it } // [1,2,3]
```

```kotlin
val strs = listOf("ab","cd")
val chars = strs.flatMap { it.toList() } // [a,b,c,d]
```

**실무 활용**

```kotlin
val companies = companies.flatMap { it.employees }
```

```kotlin
val orders = orders.flatMap { it.items }
```

***

#### 15. `flatten`

**기본 문법**

```kotlin
val nested = listOf(listOf(1,2), listOf(3,4))
nested.flatten() // [1,2,3,4]
```

```kotlin
val nestedStr = listOf(listOf("a","b"), listOf("c"))
nestedStr.flatten() // ["a","b","c"]
```

**실무 활용**

```kotlin
val menu = listOf(category1.items, category2.items).flatten()
```

```kotlin
val permissions = roles.map { it.permissions }.flatten()
```



## 이모저모

개요 & 핵심 요약

* **typealias**: 타입에 별칭(alias)을 붙여 코드 가독성 향상자바 개발자를 위한 코틀린 …
* **as import**: import 시 이름 변경 가능 → 충돌 방지자바 개발자를 위한 코틀린 …
* **구조분해 선언**: `componentN` 함수를 기반으로 객체를 여러 변수로 분해PPT
* **Jump와 Label**: `return`, `break`, `continue`를 라벨과 함께 사용 가능 (권장X)PPT
* **takeIf / takeUnless**: 조건 만족 여부에 따라 객체를 반환하거나 null 반환 → 메서드 체이닝 가능PPT
* **핵심 요약**
  * typealias: 복잡한 제네릭 단순화
  * as import: 동일 클래스명 충돌 해결
  * 구조분해: DTO/Pair/Triple 분해
  * Jump+Label: break/continue 확장 (지양)
  * takeIf/takeUnless: 조건부 반환으로 코드 간결화

***

예시1 (기본 문법 전/후)

#### (1) typealias & as import

```java
// Java
Map<Long, List<String>> userMap = new HashMap<>();
```

```kotlin
// Kotlin
typealias UserMap = Map<Long, List<String>>
val userMap: UserMap = mapOf()
```

```kotlin
import com.example.service.User as ServiceUser
import com.example.model.User as ModelUser
```

***

#### (2) 구조분해 선언

```kotlin
data class User(val id: Long, val name: String)

val user = User(1, "Joon")
val (id, name) = user   // 구조분해
```

***

예시2 (실무 활용)

#### (1) 조건부 실행 (takeIf)

```kotlin
val token = request.token.takeIf { it.isNotBlank() }
    ?: throw IllegalArgumentException("Token missing")
```

→ 유효성 검증을 한 줄로 처리.

***

#### (2) Map 처리 with destructuring

```kotlin
val configs = mapOf("host" to "localhost", "port" to "8080")
for ((key, value) in configs) {
    println("$key = $value")
}
```

→ 설정값을 구조분해로 직관적으로 순회.



## Scope Function

개요 & 핵심 요약

* Scope Function: 객체에 대해 \*\*일시적인 영역(scope)\*\*을 만들어 코드를 간결하게 하거나 메서드 체이닝에 활용자바 개발자를 위한 코틀린 …
* 종류: `let`, `run`, `also`, `apply`, `with`자바 개발자를 위한 코틀린 …
* 구분 기준:
  * 결과 반환: `let`, `run` → 람다 결과 반환
  * 객체 반환: `also`, `apply`, `with`
  * 람다 내부 접근: `it` (let/also), `this` (run/apply/with)PPT
* **핵심 요약**
  * `let`: null-safe 호출, 임시 변수 활용
  * `run`: 객체 초기화 + 반환 값 계산
  * `apply`: 객체 설정(빌더 스타일)
  * `also`: 부수효과(side-effect) 로깅, 디버깅
  * `with`: 객체 컨텍스트에서 여러 연산 수행

***

1\) let

#### 기본 문법 예시

```kotlin
val name: String? = "Joon"
name?.let { println("Length = ${it.length}") }
```

```kotlin
val numbers = listOf(1, 2, 3)
numbers.map { it * 2 }.let { println("Result = $it") }
```

#### 실무 활용 예시

```kotlin
val userName = request.user?.let { it.trim() } ?: "Guest"
```

```kotlin
val conn = dbConnection?.let { it.query("SELECT * FROM user") }
```

***

2\) run

#### 기본 문법 예시

```kotlin
val strLength = "Hello".run {
    println("Original: $this")
    length
}
```

```kotlin
val result = run {
    val x = 10
    val y = 20
    x + y
}
```

#### 실무 활용 예시

```kotlin
val response = connection.run {
    connect()
    send("ping")
    receive()
}
```

```kotlin
val session = run {
    val s = Session()
    s.open()
    s
}
```

***

3\) apply

#### 기본 문법 예시

```kotlin
val u = User().apply {
    id = 1L
    name = "Joon"
}
```

```kotlin
val list = mutableListOf<Int>().apply {
    add(1)
    add(2)
    add(3)
}
```

#### 실무 활용 예시

```kotlin
val request = HttpRequest().apply {
    method = "POST"
    url = "/login"
    headers["Content-Type"] = "application/json"
}
```

```kotlin
val builder = StringBuilder().apply {
    append("Hello, ")
    append("World")
}
```

***

4\) also

#### 기본 문법 예시

```kotlin
val list = mutableListOf(1, 2, 3).also { println("Init size = ${it.size}") }
```

```kotlin
val text = "hello".also { println("Before upper = $it") }.uppercase()
```

#### 실무 활용 예시

```kotlin
val token = generateToken()
    .also { log.info("Generated token = $it") }
```

```kotlin
val user = User("Joon").also { auditService.recordCreation(it) }
```

***

5\) with

#### 기본 문법 예시

```kotlin
val str = with(StringBuilder()) {
    append("Hello, ")
    append("World")
    toString()
}
```

```kotlin
val person = Person("Joon", 30)
val description = with(person) { "$name is $age years old" }
```

#### 실무 활용 예시

```kotlin
fun toDto(user: User) = with(user) {
    UserDto(id!!, name)
}
```

```kotlin
val properties = with(envConfig) {
    "DB: $dbUrl, Redis: $redisUrl, Kafka: $kafkaUrl"
}
```
