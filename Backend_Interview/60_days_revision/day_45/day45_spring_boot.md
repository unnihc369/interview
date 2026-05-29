# Day 45 — Unit vs Integration Test Slices

**Topics:** Test pyramid · @WebMvcTest · @DataJpaTest · @SpringBootTest · Interview Q&A

---

# 1. Test Types Overview

```text
Unit Test        → single class, mocked deps, no Spring (or minimal)
Slice Test       → partial Spring context (@WebMvcTest, @DataJpaTest)
Integration Test → @SpringBootTest, multiple layers wired
E2E Test         → full app + real HTTP + external services (or Testcontainers)
```

---

# 2. Pure Unit Test (No Spring)

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock OrderRepository orderRepo;
    @Mock PaymentGateway paymentGateway;
    @InjectMocks OrderService orderService;

    @Test
    void createOrder_success() {
        when(orderRepo.save(any())).thenAnswer(i -> i.getArgument(0));
        when(paymentGateway.charge(any())).thenReturn(PaymentResult.ok("txn-1"));

        Order order = orderService.create(new CreateOrderRequest(1L, 99.0));

        assertThat(order.getStatus()).isEqualTo(OrderStatus.PAID);
        verify(paymentGateway).charge(any());
    }
}
```

**When:** Business logic, edge cases, fast feedback (milliseconds).

---

# 3. @WebMvcTest — Web Layer Slice

Loads: controllers, `@ControllerAdvice`, Jackson, MockMvc.
Does NOT load: `@Service`, `@Repository` (unless `@Import`).

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired MockMvc mockMvc;
    @MockBean OrderService orderService;

    @Test
    void createOrder_returns201() throws Exception {
        when(orderService.create(any())).thenReturn(new OrderResponse(1L, "PAID"));

        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"productId\":1,\"amount\":99}"))
               .andExpect(status().isCreated())
               .andExpect(jsonPath("$.status").value("PAID"));
    }
}
```

**When:** HTTP contract, validation, status codes, JSON shape.

---

# 4. @DataJpaTest — Persistence Slice

Loads: JPA, repositories, embedded DB (H2 by default).
Auto-rollback after each test.
Does NOT load: web layer, `@Service`.

```java
@DataJpaTest
class OrderRepositoryTest {

    @Autowired OrderRepository orderRepo;
    @Autowired TestEntityManager em;

    @Test
    void findByUserId_returnsOrders() {
        Order order = new Order(1L, BigDecimal.TEN);
        em.persist(order);

        List<Order> found = orderRepo.findByUserId(1L);
        assertThat(found).hasSize(1);
    }
}
```

**When:** Custom queries, `@Query`, entity mappings, constraints.

---

# 5. @JsonTest — Serialization Slice

```java
@JsonTest
class OrderResponseJsonTest {

    @Autowired JacksonTester<OrderResponse> json;

    @Test
    void serialize() throws Exception {
        assertThat(json.write(new OrderResponse(1L, "PAID")))
            .isEqualToJson("{\"id\":1,\"status\":\"PAID\"}");
    }
}
```

---

# 6. @SpringBootTest — Full Integration

```java
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
class OrderFlowIntegrationTest {

    @Autowired MockMvc mockMvc;
    @Autowired OrderRepository orderRepo;
    @MockBean PaymentGateway paymentGateway;  // stub external

    @Test
    void fullCreateOrderFlow() throws Exception {
        when(paymentGateway.charge(any())).thenReturn(PaymentResult.ok("x"));

        mockMvc.perform(post("/api/v1/orders")...)
               .andExpect(status().isCreated());

        assertThat(orderRepo.count()).isEqualTo(1);
    }
}
```

**When:** Cross-layer bugs, security filters, transaction boundaries.

---

# 7. Comparison Table

| Annotation | Loads | Speed | MockMvc | Real DB |
|------------|-------|-------|---------|---------|
| (none) + Mockito | Class only | Fastest | No | No |
| @WebMvcTest | Controller | Fast | Yes | No |
| @DataJpaTest | JPA + Repo | Fast | No | Embedded |
| @JsonTest | Jackson | Fastest | No | No |
| @SpringBootTest | Everything | Slow | Optional | Configurable |

---

# 8. @Import and @TestConfiguration

```java
@WebMvcTest(OrderController.class)
@Import(SecurityConfig.class)  // need security rules in slice
class SecuredOrderControllerTest { ... }

@SpringBootTest
@TestConfiguration
static class TestConfig {
    @Bean @Primary
    PaymentGateway mockPayment() { return mock(PaymentGateway.class); }
}
```

---

# 9. Testcontainers for Integration

```java
@SpringBootTest
@Testcontainers
class OrderPostgresIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
    }
}
```

Real Postgres in Docker — closer to production than H2.

---

# 10. Choosing the Right Slice

```text
Validation logic bug?           → Unit test (OrderService)
Wrong HTTP status code?         → @WebMvcTest
JPQL query wrong?               → @DataJpaTest
Security blocks valid user?     → @SpringBootTest + MockMvc
JSON field naming?              → @JsonTest
Payment + DB + controller?      → @SpringBootTest, @MockBean external API
```

---

# Interview Questions

## Q1. Why not only @SpringBootTest?

Slow — loads entire context per class. CI with 500 tests becomes minutes. Slices keep feedback loop fast.

## Q2. @MockBean vs @Mock in slice tests?

`@MockBean` replaces Spring context bean. `@Mock` only works with `@ExtendWith(MockitoExtension)` without Spring.

## Q3. Does @DataJpaTest roll back?

Yes — `@Transactional` on test by default; changes not committed.

## Q4. Test `@Scheduled` or `@Async`?

`@SpringBootTest` + `@EnableScheduling` or `@Async` with `Awaitility` to wait for async completion.

## Q5. Ideal test ratio?

~70% unit, ~20% slice/integration, ~10% E2E — adjust per team.

---

# One-Line Revision

```text
Unit = Mockito no Spring; @WebMvcTest = controller; @DataJpaTest = repo; @SpringBootTest = full stack — pick narrowest slice that proves the bug fixed.
```

---

*End of Day 45 Spring Boot*
