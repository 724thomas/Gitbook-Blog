# Kotlin Test Tutorial

## 📘 1. 데이터베이스에 저장하고, 해당 값을 불러와서 조회해야할 때

#### ✅ 기본 예시 1: save + findById

```kotlin
@Test
fun `기본 - 저장 후 조회`() {
    val repo = mockk<UserRepository>()
    val user = User(id = 1L, name = "홍길동")

    every { repo.save(any()) } returns user
    every { repo.findById(1L) } returns user

    val saved = repo.save(user)
    val found = repo.findById(1L)

    assertEquals("홍길동", found.name) // 저장된 값 검증
}
```

#### ✅ 기본 예시 2: saveAll + findAll

```kotlin
@Test
fun `기본 - 여러건 저장 후 전체 조회`() {
    val repo = mockk<UserRepository>()
    val users = listOf(User(1, "A"), User(2, "B"))

    every { repo.saveAll(any()) } returns users
    every { repo.findAll() } returns users

    repo.saveAll(users)
    val found = repo.findAll()

    assertEquals(2, found.size)
}
```

#### 💡 실전 예시 1: IdentifyHistory 저장 후 조회

```kotlin
@Test
fun `실전 - 본인인증 저장 후 결과 조회`() {
    val history = IdentifyHistory(...).apply { id = 1L }

    every { identifyHistoryRepository.saveAndFlush(any()) } returns history
    every { identifyHistoryRepository.getResult(1L) } returns IdentifyResultResponse(1L, "0000", "성공")

    val id = identifyService.saveIdentify(request, "127.0.0.1")
    val result = identifyService.getResult(id)

    assertEquals("0000", result.resultCode)
}
```

#### 💡 실전 예시 2: save 후 count 조회

```kotlin
@Test
fun `실전 - 요청 저장 후 count 조회`() {
    val history = IdentifyHistory(...).apply { id = 10L }

    every { identifyHistoryRepository.save(any()) } returns history
    every {
        identifyHistoryRepository.countByMobileAndTypeAndCreatedAtBetween(any(), any(), any(), any())
    } returns 1

    identifyService.savePreIdentify(request, "127.0.0.1")
    val count = identifyService.getRequestCount(countRequest)

    assertEquals(1, count)
}
```

***

## 📘 2. @Transactional 어노테이션이 붙어있을 때

> 단위 테스트에서는 트랜잭션이 크게 의미가 없지만, **통합 테스트**에서는 실제 DB 롤백/커밋 여부가 중요합니다.

#### ✅ 기본 예시 1: 단위 테스트에서 무시

```kotlin
@Service
class MyService(val repo: UserRepository) {
    @Transactional
    fun saveUser(user: User) = repo.save(user)
}

@Test
fun `기본 - Transactional은 단위테스트에 영향 없음`() {
    val repo = mockk<UserRepository>()
    val service = MyService(repo)
    every { repo.save(any()) } returns User(1, "홍길동")

    val result = service.saveUser(User(0, "홍길동"))
    assertEquals(1, result.id)
}
```

#### ✅ 기본 예시 2: 통합 테스트에서 롤백 확인

```kotlin
@SpringBootTest
@Transactional
class TransactionTest(@Autowired val repo: UserRepository) {

    @Test
    fun `기본 - 저장 후 롤백`() {
        repo.save(User(0, "홍길동"))
        val count = repo.count()
        assertEquals(1, count)
        // 테스트 끝나면 롤백 -> DB에 데이터 없음
    }
}
```

#### 💡 실전 예시 1: IdentifyService saveIdentify

```kotlin
@Test
fun `실전 - saveIdentify는 트랜잭션 내에서 저장`() {
    val history = IdentifyHistory(...).apply { id = 1L }
    every { identifyHistoryRepository.saveAndFlush(any()) } returns history

    val id = identifyService.saveIdentify(request, "127.0.0.1")

    assertEquals(1L, id)
}
```

#### 💡 실전 예시 2: savePreIdentify (트랜잭션 읽기 전용 아님)

```kotlin
@Test
fun `실전 - savePreIdentify도 트랜잭션 내 저장`() {
    every { identifyHistoryRepository.save(any()) } returns IdentifyHistory(...)

    identifyService.savePreIdentify(request, "127.0.0.1")

    verify { identifyHistoryRepository.save(any()) }
}
```

***

## 📘 3. when CALLED -> return X

(MockK의 `every { ... } returns ...` 구문)

#### ✅ 기본 예시 1

```kotlin
every { repo.findById(1L) } returns User(1, "홍길동")
```

#### ✅ 기본 예시 2

```kotlin
every { repo.count() } returns 5
```

#### 💡 실전 예시 1

```kotlin
every { identifyHistoryRepository.getResult(1L) } returns IdentifyResultResponse(1L, "0000", "성공")
```

#### 💡 실전 예시 2

```kotlin
every { identifyHistoryRepository.findVerifyValidateInfo("tx123") } returns IdentifyVerifyValidationInfoDto("ci", "홍길동", "19900101")
```

***

## 📘 4. Slot

#### ✅ 기본 예시 1

```kotlin
val slot = slot<User>()
every { repo.save(capture(slot)) } returns User(1, "홍길동")

repo.save(User(0, "홍길동"))
assertEquals("홍길동", slot.captured.name)
```

#### ✅ 기본 예시 2

```kotlin
val slot = slot<String>()
every { repo.findByName(capture(slot)) } returns User(1, "A")

repo.findByName("AAA")
assertEquals("AAA", slot.captured)
```

#### 💡 실전 예시 1

```kotlin
val slot = slot<IdentifyHistory>()
every { identifyHistoryRepository.save(capture(slot)) } returns IdentifyHistory(...)

identifyService.savePreIdentify(request, "127.0.0.1")

assertEquals("홍길동", slot.captured.name)
```

#### 💡 실전 예시 2

```kotlin
val slot = slot<IdentifyHistory>()
every { identifyHistoryRepository.saveAndFlush(capture(slot)) } returns IdentifyHistory(...).apply { id = 1L }

identifyService.saveIdentify(request, "127.0.0.1")

assertEquals(IdentifyType.VERIFY, slot.captured.type)
```

***

## 📘 5. mockk()

#### ✅ 기본 예시 1

```kotlin
val repo = mockk<UserRepository>()
```

#### ✅ 기본 예시 2

```kotlin
val user = mockk<User>()
every { user.name } returns "홍길동"
```

#### 💡 실전 예시 1

```kotlin
every { identifyHistoryRepository.save(any()) } returns mockk()
```

#### 💡 실전 예시 2

```kotlin
val mockHistory: IdentifyHistory = mockk()
every { mockHistory.id } returns 1L
every { identifyHistoryRepository.saveAndFlush(any()) } returns mockHistory
```

***

## 📘 6. verify

#### ✅ 기본 예시 1

```kotlin
every { repo.save(any()) } returns User(1, "홍길동")

repo.save(User(0, "홍길동"))

verify { repo.save(any()) }
```

#### ✅ 기본 예시 2

```kotlin
every { repo.deleteById(1L) } returns Unit

repo.deleteById(1L)

verify { repo.deleteById(1L) }
```

#### 💡 실전 예시 1

```kotlin
every { identifyHistoryRepository.save(any()) } returns IdentifyHistory(...)

identifyService.savePreIdentify(request, "127.0.0.1")

verify { identifyHistoryRepository.save(any()) }
```

#### 💡 실전 예시 2

```kotlin
every { identifyHistoryRepository.countByMobileAndTypeAndCreatedAtBetween(any(), any(), any(), any()) } returns 3

identifyService.getRequestCount(countRequest)

verify {
    identifyHistoryRepository.countByMobileAndTypeAndCreatedAtBetween(
        countRequest.mobile,
        IdentifyType.REQUEST,
        countRequest.from,
        countRequest.to
    )
}
```

***

## 📘 7. @BeforeEach / @AfterEach

테스트 실행 전/후에 반복적으로 실행해야 하는 코드를 넣을 때 사용합니다.

#### ✅ 기본 예시 1

```kotlin
@BeforeEach
fun setup() {
    println("테스트 시작 전 실행")
}
```

#### ✅ 기본 예시 2

```kotlin
@AfterEach
fun cleanup() {
    println("테스트 끝난 후 실행")
}
```

#### 💡 실전 예시 1

```kotlin
@BeforeEach
fun initMocks() {
    clearAllMocks() // MockK에서 이전 테스트 설정 초기화
}
```

#### 💡 실전 예시 2

```kotlin
@AfterEach
fun clear() {
    database.cleanTables() // 실제 통합 테스트에서 테이블 초기화
}
```

***

## 📘 8. assert 계열 (검증)

테스트의 **기대값 vs 실제값**을 비교할 때 사용합니다.

#### ✅ 기본 예시 1

```kotlin
assertEquals(5, 2 + 3)
```

#### ✅ 기본 예시 2

```kotlin
assertTrue("kotlin".startsWith("kot"))
```

#### 💡 실전 예시 1

```kotlin
val result = identifyService.getResult(1L)
assertNotNull(result)
```

#### 💡 실전 예시 2

```kotlin
val count = identifyService.getRequestCount(countRequest)
assertEquals(3, count, "요청 횟수는 3이어야 한다")
```

***

## 📘 9. assertThrows (예외 검증)

예외가 발생하는 상황을 테스트할 때 사용합니다.

#### ✅ 기본 예시 1

```kotlin
assertThrows<IllegalArgumentException> {
    throw IllegalArgumentException("잘못된 값")
}
```

#### ✅ 기본 예시 2

```kotlin
assertThrows<NullPointerException> {
    val str: String? = null
    str!!.length
}
```

#### 💡 실전 예시 1

```kotlin
assertThrows<InternalServerException> {
    identifyService.saveIdentify(request, "127.0.0.1") // id=null 반환되면 예외 발생
}
```

#### 💡 실전 예시 2

```kotlin
assertThrows<NoSuchElementException> {
    identifyHistoryRepository.findById(999L).get()
}
```

***

## 📘 10. any()

MockK에서 **모든 값 허용 매처**입니다.

#### ✅ 기본 예시 1

```kotlin
every { repo.findById(any()) } returns User(1, "홍길동")
```

#### ✅ 기본 예시 2

```kotlin
every { repo.save(any()) } returns User(2, "이몽룡")
```

#### 💡 실전 예시 1

```kotlin
every { identifyHistoryRepository.saveAndFlush(any()) } returns IdentifyHistory(...).apply { id = 1L }
```

#### 💡 실전 예시 2

```kotlin
verify { identifyHistoryRepository.countByMobileAndTypeAndCreatedAtBetween(any(), any(), any(), any()) }
```

***

## 📘 11. coEvery / coVerify

코루틴 `suspend` 함수를 테스트할 때 사용합니다.

#### ✅ 기본 예시 1

```kotlin
coEvery { repo.findUser(any()) } returns User(1, "홍길동")
```

#### ✅ 기본 예시 2

```kotlin
coVerify { repo.saveUser(any()) }
```

#### 💡 실전 예시 1

```kotlin
coEvery { identifyAsyncClient.verify(any()) } returns true
```

#### 💡 실전 예시 2

```kotlin
coVerify { identifyAsyncClient.verify(request) }
```

***

## 📘 12. parameterizedTest (파라미터화 테스트)

여러 입력값으로 반복 테스트할 때 유용합니다.

#### ✅ 기본 예시 1

```kotlin
@ParameterizedTest
@ValueSource(strings = ["A", "B", "C"])
fun `문자열 길이 테스트`(input: String) {
    assertTrue(input.length == 1)
}
```

#### ✅ 기본 예시 2

```kotlin
@ParameterizedTest
@ValueSource(ints = [1, 2, 3])
fun `정수 양수 테스트`(num: Int) {
    assertTrue(num > 0)
}
```

#### 💡 실전 예시 1

```kotlin
@ParameterizedTest
@CsvSource(
    "01012345678,true",
    "010, false"
)
fun `전화번호 유효성 검사`(mobile: String, expected: Boolean) {
    assertEquals(expected, mobile.length == 11)
}
```

#### 💡 실전 예시 2

```kotlin
@ParameterizedTest
@CsvSource(
    "홍길동, 19900101",
    "이몽룡, 19950505"
)
fun `이름과 생년월일 매핑 테스트`(name: String, birth: String) {
    val dto = IdentifyVerifyValidationInfoDto("ci", name, birth)
    assertEquals(birth, dto.birth)
}
```

***

## 📘 13. @Mockk, @InjectMockKs

Mock 객체와 주입 객체를 간단히 설정할 수 있습니다.

#### ✅ 기본 예시 1

```kotlin
@Mockk
lateinit var repo: UserRepository
```

#### ✅ 기본 예시 2

```kotlin
@InjectMockKs
lateinit var service: UserService
```

#### 💡 실전 예시 1

```kotlin
@Mockk
lateinit var identifyHistoryRepository: IdentifyHistoryRepository

@InjectMockKs
lateinit var identifyService: IdentifyService
```

#### 💡 실전 예시 2

```kotlin
@BeforeEach
fun setUp() = MockKAnnotations.init(this)
```

***

## 📘 14. @SpringBootTest / @DataJpaTest

스프링 통합 테스트용 어노테이션입니다.

#### ✅ 기본 예시 1

```kotlin
@SpringBootTest
class IntegrationTest
```

#### ✅ 기본 예시 2

```kotlin
@DataJpaTest
class RepositoryTest
```

#### 💡 실전 예시 1

```kotlin
@SpringBootTest
@Transactional
class IdentifyServiceIntegrationTest(
    @Autowired val identifyService: IdentifyService
)
```

#### 💡 실전 예시 2

```kotlin
@DataJpaTest
class IdentifyHistoryRepositoryTest(
    @Autowired val identifyHistoryRepository: IdentifyHistoryRepository
)
```

***

## 📘 15. Capturing multiple values

Slot은 1개만 저장되지만, `mutableListOf<>()`로 여러 값도 캡처 가능합니다.

#### ✅ 기본 예시 1

```kotlin
val list = mutableListOf<User>()
every { repo.save(capture(list)) } answers { list.last() }
```

#### ✅ 기본 예시 2

```kotlin
repo.save(User(1, "A"))
repo.save(User(2, "B"))
assertEquals(2, list.size)
```

#### 💡 실전 예시 1

```kotlin
val captured = mutableListOf<IdentifyHistory>()
every { identifyHistoryRepository.save(capture(captured)) } answers { captured.last() }

identifyService.savePreIdentify(request1, "ip")
identifyService.savePreIdentify(request2, "ip")

assertEquals(2, captured.size)
```

#### 💡 실전 예시 2

```kotlin
val captured = mutableListOf<IdentifyHistory>()
every { identifyHistoryRepository.saveAndFlush(capture(captured)) } answers { captured.last() }

identifyService.saveIdentify(request, "ip")

assertEquals(IdentifyType.VERIFY, captured.first().type)
```

***

## 📘 16. answers (동적으로 리턴값 만들기)

정적인 `returns` 대신, 호출될 때 전달된 인자에 따라 동적으로 결과를 반환.

#### ✅ 기본 예시 1

```kotlin
every { repo.findById(any()) } answers { User(firstArg(), "이름") }
```

#### ✅ 기본 예시 2

```kotlin
every { repo.count() } answers { 10 + 5 }
```

#### 💡 실전 예시 1

```kotlin
every { identifyHistoryRepository.save(any()) } answers {
    val history = firstArg<IdentifyHistory>()
    history.apply { id = 99L }
}
```

#### 💡 실전 예시 2

```kotlin
every { identifyHistoryRepository.findVerifyValidateInfo(any()) } answers {
    IdentifyVerifyValidationInfoDto("ci", "홍길동", "19900101")
}
```

***

## 📘 17. relaxed mock (필요한 부분만 스텁)

`mockk(relaxed = true)` → 기본 리턴값을 자동으로 만들어줌.

#### ✅ 기본 예시 1

```kotlin
val repo = mockk<UserRepository>(relaxed = true)
```

#### ✅ 기본 예시 2

```kotlin
val service = mockk<MyService>(relaxed = true)
```

#### 💡 실전 예시 1

```kotlin
val repo = mockk<IdentifyHistoryRepository>(relaxed = true)
identifyService = IdentifyService(repo)
```

#### 💡 실전 예시 2

```kotlin
val mockHistory = mockk<IdentifyHistory>(relaxed = true)
every { mockHistory.id } returns 1L
```

***

## 📘 18. clearMocks / clearAllMocks

테스트 간 **mock 설정 초기화**.

#### ✅ 기본 예시 1

```kotlin
clearAllMocks()
```

#### ✅ 기본 예시 2

```kotlin
clearMocks(repo)
```

#### 💡 실전 예시 1

```kotlin
@BeforeEach
fun setup() {
    clearAllMocks()
}
```

#### 💡 실전 예시 2

```kotlin
@AfterEach
fun cleanup() {
    clearMocks(identifyHistoryRepository)
}
```

***

## 📘 19. confirmVerified / excludeRecords

모든 mock이 기대대로 호출됐는지 확인.

#### ✅ 기본 예시 1

```kotlin
confirmVerified(repo)
```

#### ✅ 기본 예시 2

```kotlin
excludeRecords { repo.toString() }
```

#### 💡 실전 예시 1

```kotlin
verify { identifyHistoryRepository.save(any()) }
confirmVerified(identifyHistoryRepository)
```

#### 💡 실전 예시 2

```kotlin
excludeRecords { identifyHistoryRepository.toString() }
```

***

## 📘 20. order / sequence 검증

메서드 호출 순서를 검증.

#### ✅ 기본 예시 1

```kotlin
verifyOrder {
    repo.save(any())
    repo.findAll()
}
```

#### ✅ 기본 예시 2

```kotlin
verifySequence {
    repo.save(any())
    repo.delete(any())
}
```

#### 💡 실전 예시 1

```kotlin
verifyOrder {
    identifyHistoryRepository.save(any())
    identifyHistoryRepository.saveAndFlush(any())
}
```

#### 💡 실전 예시 2

```kotlin
verifySequence {
    identifyHistoryRepository.save(any())
    identifyHistoryRepository.getResult(any())
}
```

***

## 📘 21. timeout (비동기 동작 검증)

호출이 일정 시간 내 일어났는지 검증.

#### ✅ 기본 예시 1

```kotlin
verify(timeout = 1000) { repo.save(any()) }
```

#### ✅ 기본 예시 2

```kotlin
verify(atLeast = 1, timeout = 500) { repo.findAll() }
```

#### 💡 실전 예시 1

```kotlin
verify(timeout = 2000) { identifyHistoryRepository.save(any()) }
```

#### 💡 실전 예시 2

```kotlin
verify(atLeast = 1, timeout = 1000) { identifyHistoryRepository.countByMobileAndTypeAndCreatedAtBetween(any(), any(), any(), any()) }
```

***

## 📘 22. spyK (부분 모킹)

기존 객체 일부만 모킹.

#### ✅ 기본 예시 1

```kotlin
val list = spyk(mutableListOf<String>())
list.add("A")
verify { list.add("A") }
```

#### ✅ 기본 예시 2

```kotlin
val user = spyk(User(1, "홍길동"))
every { user.name } returns "이몽룡"
```

#### 💡 실전 예시 1

```kotlin
val service = spyk(IdentifyService(identifyHistoryRepository))
```

#### 💡 실전 예시 2

```kotlin
val history = spyk(IdentifyHistory(...))
every { history.name } returns "SpyTest"
```

***

## 📘 23. @TestInstance(TestInstance.Lifecycle.PER\_CLASS)

테스트 인스턴스를 클래스 단위로 공유.

#### ✅ 기본 예시 1

```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest
```

#### ✅ 기본 예시 2

```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class RepoTest
```

#### 💡 실전 예시 1

```kotlin
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class IdentifyServiceTest {
    // beforeAll, afterAll 가능
}
```

#### 💡 실전 예시 2

```kotlin
@BeforeAll
fun initAll() { println("모든 테스트 시작 전 1회 실행") }
```

***

## 📘 24. SpringExtension (JUnit5 + Spring 연동)

JUnit5에서 스프링 컨텍스트 사용.

#### ✅ 기본 예시 1

```kotlin
@ExtendWith(SpringExtension::class)
class SpringTest
```

#### ✅ 기본 예시 2

```kotlin
@ExtendWith(SpringExtension::class)
@SpringBootTest
class IntegrationTest
```

#### 💡 실전 예시 1

```kotlin
@ExtendWith(SpringExtension::class)
@DataJpaTest
class IdentifyRepositoryTest
```

#### 💡 실전 예시 2

```kotlin
@ExtendWith(SpringExtension::class)
@SpringBootTest
class IdentifyServiceIntegrationTest
```

***

## 📘 25. Testcontainers (실제 DB 띄워서 통합 테스트)

실제 DB 환경을 도커로 실행.

#### ✅ 기본 예시 1

```kotlin
@Testcontainers
class ContainerTest {
    @Container
    val postgres = PostgreSQLContainer("postgres:15")
}
```

#### ✅ 기본 예시 2

```kotlin
@Testcontainers
class RedisTest {
    @Container
    val redis = GenericContainer("redis:7").withExposedPorts(6379)
}
```

#### 💡 실전 예시 1

```kotlin
@Testcontainers
class IdentifyRepositoryTest {
    @Container
    val mysql = MySQLContainer("mysql:8")
}
```

#### 💡 실전 예시 2

```kotlin
@Testcontainers
class IdentifyServiceIntegrationTest {
    @Container
    val cassandra = CassandraContainer("cassandra:4")
}
```

***

## 📘 26. Tagging / Filtering (테스트 그룹핑)

#### ✅ 기본 예시 1

```kotlin
@Tag("fast")
@Test
fun fastTest() {}
```

#### ✅ 기본 예시 2

```kotlin
@Tag("slow")
@Test
fun slowTest() {}
```

#### 💡 실전 예시 1

```kotlin
@Tag("repository")
@Test
fun repoTest() {}
```

#### 💡 실전 예시 2

```kotlin
@Tag("integration")
@Test
fun integrationTest() {}
```

***

## 📘 27. Nested Tests (중첩 구조 테스트)

#### ✅ 기본 예시 1

```kotlin
@Nested
inner class SaveTests {
    @Test fun saveSuccess() {}
}
```

#### ✅ 기본 예시 2

```kotlin
@Nested
inner class FindTests {
    @Test fun findSuccess() {}
}
```

#### 💡 실전 예시 1

```kotlin
@Nested
inner class IdentifySaveTests {
    @Test fun saveIdentifySuccess() {}
}
```

#### 💡 실전 예시 2

```kotlin
@Nested
inner class IdentifyQueryTests {
    @Test fun getRequestCountTest() {}
}
```
