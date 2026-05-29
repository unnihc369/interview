# Day 26 - JUnit 5 & Mockito

---

# 1. JUnit 5 Architecture

JUnit 5 = **JUnit Platform** + **Jupiter** (programming model) + **Vintage** (JUnit 4 compat)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

Includes JUnit 5, Mockito, AssertJ, Hamcrest.

---

# 2. Basic Test Structure

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {

    @BeforeAll
    static void setupAll() { }

    @BeforeEach
    void setup() { }

    @Test
    @DisplayName("Add two positive numbers")
    void shouldAdd() {
        Calculator calc = new Calculator();
        assertEquals(5, calc.add(2, 3));
    }

    @Test
    void shouldThrowOnDivideByZero() {
        Calculator calc = new Calculator();
        assertThrows(ArithmeticException.class, () -> calc.divide(10, 0));
    }

    @AfterEach
    void tearDown() { }
}
```

---

# 3. Common Assertions

```java
assertEquals(expected, actual);
assertNotEquals(unexpected, actual);
assertTrue(condition);
assertFalse(condition);
assertNull(object);
assertNotNull(object);
assertAll("user",
    () -> assertEquals("John", user.getName()),
    () -> assertEquals(25, user.getAge())
);
assertTimeout(Duration.ofSeconds(2), () -> slowOperation());
```

---

# 4. Parameterized Tests

```java
@ParameterizedTest
@ValueSource(strings = {"admin@x.com", "user@y.com"})
void validEmails(String email) {
    assertTrue(validator.isValid(email));
}

@ParameterizedTest
@CsvSource({"2, 3, 5", "0, 0, 0", "-1, 1, 0"})
void add(int a, int b, int expected) {
    assertEquals(expected, calc.add(a, b));
}
```

---

# 5. Mockito Basics

Mockito creates **test doubles** (mocks) to isolate unit under test.

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    void shouldReturnUser() {
        User user = new User(1L, "John");
        when(userRepository.findById(1L)).thenReturn(Optional.of(user));

        UserResponse result = userService.getById(1L);

        assertEquals("John", result.getName());
        verify(userRepository, times(1)).findById(1L);
    }
}
```

---

# 6. Mockito Key APIs

| API | Purpose |
|-----|---------|
| `mock(Class)` | Create mock |
| `when(...).thenReturn(...)` | Stub return value |
| `when(...).thenThrow(...)` | Stub exception |
| `verify(mock).method()` | Assert method called |
| `verify(mock, never()).method()` | Assert not called |
| `argumentCaptor.capture()` | Capture arguments |
| `@Mock` / `@InjectMocks` | JUnit 5 integration |

---

# 7. Argument Matcher

```java
when(userRepository.save(any(User.class))).thenReturn(savedUser);
verify(userRepository).save(argThat(u -> u.getName().equals("John")));
```

Use `any()`, `eq()`, `isNull()`, `contains()` from `ArgumentMatchers`.

---

# 8. Spy vs Mock

```java
List<String> spyList = spy(new ArrayList<>());
spyList.add("one");              // real method
doReturn(100).when(spyList).size(); // stub specific method

List<String> mockList = mock(ArrayList.class);
when(mockList.size()).thenReturn(100); // all methods stubbed by default
```

- **Mock**: fake object, no real behavior
- **Spy**: wraps real object, partial mocking

---

# 9. Testing Service Layer (Pattern)

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock OrderRepository orderRepo;
    @Mock PaymentGateway paymentGateway;
    @Mock InventoryService inventoryService;

    @InjectMocks OrderService orderService;

    @Test
    void placeOrder_success() {
        OrderRequest req = new OrderRequest(1L, List.of(item));
        when(inventoryService.reserve(any())).thenReturn(true);
        when(paymentGateway.charge(any())).thenReturn(PaymentResult.success());

        OrderResponse resp = orderService.placeOrder(req);

        assertEquals("CONFIRMED", resp.getStatus());
        verify(orderRepo).save(any(Order.class));
    }

    @Test
    void placeOrder_paymentFails() {
        when(paymentGateway.charge(any())).thenReturn(PaymentResult.failed());
        assertThrows(PaymentException.class, () -> orderService.placeOrder(req));
        verify(orderRepo, never()).save(any());
    }
}
```

---

# 10. Test Pyramid (Interview)

```text
        /\
       /E2E\        few — slow, brittle
      /------\
     /Integration\  some — @SpringBootTest, @DataJpaTest
    /--------------\
   /   Unit Tests   \ many — fast, mocked deps
```

Unit tests with Mockito = largest layer.

---

# 11. Best Practices

| Practice | Why |
|----------|-----|
| Test behavior, not implementation | Refactor-safe |
| One logical assertion per test | Clear failure diagnosis |
| Use `@DisplayName` | Readable reports |
| Avoid mocking value objects | Mock boundaries (repos, APIs) |
| Don't mock what you don't own | Wrap external SDK |
| AAA pattern | Arrange → Act → Assert |

---

# Common Interview Questions

## Q1. Unit vs Integration test?

Unit: isolated with mocks. Integration: real Spring context, DB, or HTTP.

## Q2. verify vs assert?

Assert checks **output/state**. Verify checks **interaction** with collaborators.

## Q3. Why @InjectMocks?

Mockito injects `@Mock` fields into `@InjectMocks` target via constructor/setter/field injection.

## Q4. Static method mocking?

Mockito 3.4+ `mockStatic(MyUtil.class)` — use sparingly.

## Q5. @MockBean vs @Mock?

`@MockBean` replaces Spring context bean (integration tests). `@Mock` pure unit test.

## Q6. How to test exceptions?

`assertThrows(ExpectedException.class, () -> service.method())`

---

# One-Line Revision

```text
JUnit 5 = @Test + assertions; Mockito = mock deps, when/thenReturn, verify interactions — keep unit tests fast and isolated.
```

---

*End of Day 26 Java*
