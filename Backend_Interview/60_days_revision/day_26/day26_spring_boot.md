# Day 26 - @WebMvcTest vs @DataJpaTest

---

# 1. Spring Boot Test Slices

Spring Boot provides **test slices** — load only part of application context for faster, focused tests.

```text
@SpringBootTest     → full context (slow)
@WebMvcTest          → web layer only
@DataJpaTest         → JPA + DB only
@JsonTest            → JSON serialization
@RestClientTest      → REST client
```

---

# 2. @WebMvcTest — Controller Layer

Loads:

- `@Controller`, `@RestController`
- `@ControllerAdvice`
- Web MVC infrastructure
- Security (optional, limited)

Does **NOT** load:

- `@Service`, `@Repository` (unless `@Import`)
- Full database

---

# Example

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void getUser_returns200() throws Exception {
        UserResponse user = new UserResponse(1L, "John");
        when(userService.getById(1L)).thenReturn(user);

        mockMvc.perform(get("/api/v1/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("John"));

        verify(userService).getById(1L);
    }

    @Test
    void createUser_returns201() throws Exception {
        when(userService.create(any())).thenReturn(new UserResponse(1L, "Jane"));

        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name":"Jane","email":"j@x.com"}
                    """))
               .andExpect(status().isCreated())
               .andExpect(jsonPath("$.id").value(1));
    }
}
```

---

# MockMvc Key Methods

| Method | Purpose |
|--------|---------|
| `perform(get/post/put/delete(...))` | Simulate HTTP request |
| `andExpect(status().isOk())` | Assert HTTP status |
| `andExpect(jsonPath("$.field").value(...))` | Assert JSON body |
| `andExpect(header().exists("Location"))` | Assert headers |
| `andDo(print())` | Debug output |

---

# @WebMvcTest with Security

```java
@WebMvcTest(SecuredController.class)
@Import(SecurityConfig.class)
class SecuredControllerTest {

    @Autowired MockMvc mockMvc;

    @Test
    @WithMockUser(roles = "ADMIN")
    void adminEndpoint() throws Exception {
        mockMvc.perform(get("/admin/users"))
               .andExpect(status().isOk());
    }

    @Test
    void unauthorizedWithoutAuth() throws Exception {
        mockMvc.perform(get("/admin/users"))
               .andExpect(status().isUnauthorized());
    }
}
```

---

# 3. @DataJpaTest — Repository Layer

Loads:

- JPA entities
- Spring Data repositories
- `TestEntityManager`
- Embedded DB (H2) by default

Does **NOT** load:

- Controllers
- Services (unless `@Import`)

---

# Example

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void findByEmail_returnsUser() {
        User user = new User("John", "john@x.com");
        entityManager.persist(user);
        entityManager.flush();

        Optional<User> found = userRepository.findByEmail("john@x.com");

        assertTrue(found.isPresent());
        assertEquals("John", found.get().getName());
    }

    @Test
    void save_persistsEntity() {
        User saved = userRepository.save(new User("Jane", "j@x.com"));
        assertNotNull(saved.getId());
    }
}
```

---

# @DataJpaTest Properties

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = Replace.NONE) // use real DB from properties
class UserRepositoryIntegrationTest { }
```

Default: replaces configured datasource with embedded H2.

---

# 4. Comparison Table

| | @WebMvcTest | @DataJpaTest |
|---|-------------|--------------|
| Tests | Controllers, HTTP | Repositories, queries |
| Mocks | `@MockBean` services | Real JPA (embedded DB) |
| Speed | Fast | Medium |
| MockMvc | Yes | No |
| DB | No (mocked services) | Yes (H2) |
| Transaction | No default rollback | `@Transactional` rollback per test |

---

# 5. @SpringBootTest — Full Integration

When you need everything:

```java
@SpringBootTest
@AutoConfigureMockMvc
class FullIntegrationTest {

    @Autowired MockMvc mockMvc;

    @Test
    void endToEnd() throws Exception {
        mockMvc.perform(post("/api/v1/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{...}"))
               .andExpect(status().isCreated());
    }
}
```

Slower — boots entire application context.

---

# 6. Test Slice Decision Guide

```text
Testing controller mapping/validation/status codes?
  → @WebMvcTest + @MockBean service

Testing custom @Query / entity mapping / constraints?
  → @DataJpaTest

Testing service business logic?
  → @ExtendWith(MockitoExtension) unit test

Testing full flow controller → service → DB?
  → @SpringBootTest + @AutoConfigureMockMvc
```

---

# 7. @Import and @ContextConfiguration

Add beans not auto-included in slice:

```java
@WebMvcTest(OrderController.class)
@Import(OrderMapper.class)
class OrderControllerTest { }
```

---

# 8. Testcontainers (Advanced Interview)

For real MySQL/PostgreSQL in tests:

```java
@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = Replace.NONE)
class UserRepositoryIT {

    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
    }
}
```

---

# Common Interview Questions

## Q1. Why @MockBean in @WebMvcTest?

Controller depends on service — service not loaded in slice, must mock.

## Q2. Does @DataJpaTest roll back transactions?

Yes — each test runs in transaction rolled back by default.

## Q3. @WebMvcTest vs @SpringBootTest for controller?

WebMvcTest is faster and focused; BootTest for integration across layers.

## Q4. How to test validation errors?

```java
mockMvc.perform(post("/users").content("{}"))
       .andExpect(status().isBadRequest());
```

## Q5. @Sql annotation?

Load test data: `@Sql("/data/users.sql")` on `@DataJpaTest` methods.

## Q6. Difference @Mock vs @MockBean?

@Mock = Mockito only. @MockBean = replaces/adds bean in Spring test context.

---

# One-Line Revision

```text
@WebMvcTest = controller + MockMvc + mock services; @DataJpaTest = repository + embedded DB; @SpringBootTest = full stack.
```

---

*End of Day 26 Spring Boot*
