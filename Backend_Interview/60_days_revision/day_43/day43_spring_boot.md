# Day 43 — @SpringBootTest, MockMvc & TestRestTemplate

**Topics:** Integration testing · Full context vs sliced · HTTP testing · Interview Q&A

---

# 1. Testing Pyramid in Spring Boot

```text
        /\
       /  \     E2E (@SpringBootTest + TestRestTemplate)
      /----\
     /      \   Integration (@DataJpaTest, @WebMvcTest)
    /--------\
   /          \ Unit (JUnit + Mockito)
  /------------\
```

---

# 2. @SpringBootTest — Full Application Context

Loads **entire** Spring Boot application (all beans, auto-config, embedded server optional).

```java
@SpringBootTest
class OrderIntegrationTest {

    @Autowired
    private OrderService orderService;

    @Test
    void createOrder_persistsToDb() {
        Order order = orderService.create(new CreateOrderRequest(1L, 100.0));
        assertNotNull(order.getId());
    }
}
```

---

# Key Properties

| Property | Purpose |
|----------|---------|
| `webEnvironment` | `MOCK` (default), `RANDOM_PORT`, `DEFINED_PORT`, `NONE` |
| `@ActiveProfiles("test")` | Use test profile / H2 DB |
| `@Transactional` | Roll back DB changes after each test |

---

# @SpringBootTest with MockMvc (MOCK web env)

```java
@SpringBootTest
@AutoConfigureMockMvc
class UserApiIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void getUser_returns200() throws Exception {
        mockMvc.perform(get("/api/v1/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").exists());
    }
}
```

- No real HTTP server — MockMvc simulates servlet container
- Faster than RANDOM_PORT
- Good for controller + service + security integration

---

# 3. MockMvc Deep Dive

## Setup Options

```java
// Option A: @SpringBootTest + @AutoConfigureMockMvc
@SpringBootTest
@AutoConfigureMockMvc

// Option B: @WebMvcTest (controller slice only)
@WebMvcTest(UserController.class)

// Option C: Standalone setup (no Spring context)
MockMvc mockMvc = MockMvcBuilders
    .standaloneSetup(new UserController(userService))
    .build();
```

---

# Common MockMvc Patterns

```java
mockMvc.perform(post("/api/v1/orders")
        .contentType(MediaType.APPLICATION_JSON)
        .header("Authorization", "Bearer " + token)
        .content("""
            {"productId":1,"quantity":2}
            """))
    .andExpect(status().isCreated())
    .andExpect(jsonPath("$.orderId").isNumber())
    .andExpect(header().exists("Location"))
    .andDo(print());  // debug
```

---

# Security Testing with MockMvc

```java
@SpringBootTest
@AutoConfigureMockMvc
class SecuredApiTest {

    @Autowired MockMvc mockMvc;

    @Test
    @WithMockUser(username = "admin", roles = "ADMIN")
    void adminCanDelete() throws Exception {
        mockMvc.perform(delete("/api/v1/users/1"))
               .andExpect(status().isNoContent());
    }

    @Test
    void unauthenticated_returns401() throws Exception {
        mockMvc.perform(get("/api/v1/users/me"))
               .andExpect(status().isUnauthorized());
    }
}
```

---

# 4. TestRestTemplate — Real HTTP Client

Use when `webEnvironment = RANDOM_PORT` or `DEFINED_PORT` — tests hit **actual embedded server**.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class OrderE2ETest {

    @Autowired
    private TestRestTemplate restTemplate;

    @LocalServerPort
    private int port;

    @Test
    void createOrder_viaHttp() {
        CreateOrderRequest req = new CreateOrderRequest(1L, 2);
        ResponseEntity<OrderResponse> response = restTemplate.postForEntity(
            "/api/v1/orders", req, OrderResponse.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody().getOrderId());
    }
}
```

---

# MockMvc vs TestRestTemplate

| | MockMvc | TestRestTemplate |
|---|---------|------------------|
| HTTP server | Simulated (servlet) | Real embedded Tomcat/Netty |
| Speed | Faster | Slower |
| Full filter chain | Yes (with full context) | Yes |
| External client simulation | No | Yes |
| JSON helpers | jsonPath | RestTemplate + ObjectMapper |
| Use when | Most integration tests | E2E, testing actual HTTP semantics |

---

# 5. Test Configuration Best Practices

```java
@SpringBootTest
@ActiveProfiles("test")
@Transactional  // rollback after each test method
class BaseIntegrationTest {

    @Autowired
    protected MockMvc mockMvc;

    @MockBean  // replace external payment gateway
    private PaymentGatewayClient paymentClient;
}
```

```yaml
# application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
  jpa:
    hibernate:
      ddl-auto: create-drop
```

---

# 6. @MockBean vs @SpyBean in Integration Tests

```java
@MockBean
private EmailService emailService;  // full mock, no real emails

@SpyBean
private OrderService orderService;  // real logic, stub specific methods
```

Use `@MockBean` for external dependencies (payment, email, S3). Keep real beans for code under test.

---

# Interview Questions

## Q1. When @SpringBootTest vs @WebMvcTest?

`@WebMvcTest` — test controller layer only, mock services, fast.
`@SpringBootTest` — test full stack (service + repo + security), slower but catches wiring bugs.

## Q2. Why @AutoConfigureMockMvc with @SpringBootTest?

Enables MockMvc bean injection without starting a real server (`webEnvironment.MOCK`).

## Q3. When TestRestTemplate over MockMvc?

When you need real HTTP (port binding, actual serialization, client-side behavior) or testing `@RestClient`/Feign against local server.

## Q4. How to speed up @SpringBootTest?

Use test slices (`@WebMvcTest`, `@DataJpaTest`), `@MockBean` external services, shared `@TestConfiguration`, or `@Import` only needed configs.

## Q5. @Transactional on tests — caveat?

Only rolls back **same-thread** transactions. Async/`@Async` methods may commit outside test transaction.

---

# One-Line Revision

```text
@SpringBootTest = full context; MockMvc = simulated HTTP (fast); TestRestTemplate = real embedded server (E2E); pick slice vs full based on scope.
```

---

*End of Day 43 Spring Boot*
