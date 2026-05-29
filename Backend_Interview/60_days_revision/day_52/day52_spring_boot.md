# Day 52 — 2-Hour CRUD App Build Guide

**Week 8 · Wednesday · Project: Task Manager API**  
**Stack:** Spring Boot 3 · Spring Data JPA · Spring Security · Actuator · JUnit 5 · Mockito

**Companion:** `day52_java.md` for supporting language topics

---

# Overview

Build a production-shaped mini app end-to-end:

| Feature | Technology |
|---------|------------|
| CRUD REST | `@RestController`, DTOs |
| Persistence | JPA + H2 (dev) / MySQL (prod profile) |
| Security | JWT + role `USER` / `ADMIN` |
| Observability | Actuator health, metrics |
| Tests | `@WebMvcTest`, `@DataJpaTest`, service unit tests |

**Time budget (120 min)**

| Phase | Min |
|-------|-----|
| Bootstrap + entity + repo | 20 |
| Service + REST + validation | 25 |
| Security + JWT | 35 |
| Actuator + config | 10 |
| Tests | 25 |
| Manual smoke test | 5 |

---

# Phase 1 — Project Bootstrap (10 min)

## Dependencies (`pom.xml` or start.spring.io)

- Spring Web  
- Spring Data JPA  
- Spring Security  
- Validation  
- Actuator  
- H2 (runtime)  
- MySQL driver (optional)  
- Lombok (optional)  
- spring-boot-starter-test  

```bash
# Or use Spring Initializr with same deps
```

## `application.yml`

```yaml
spring:
  application:
    name: task-manager
  datasource:
    url: jdbc:h2:mem:tasks;DB_CLOSE_DELAY=-1
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

server:
  port: 8080

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: when_authorized

app:
  jwt:
    secret: ${JWT_SECRET:change-me-in-prod-min-32-chars-long!!}
    expiration-ms: 86400000
```

---

# Phase 2 — Domain Model (15 min)

## Entity

```java
@Entity
@Table(name = "tasks")
public class Task {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    private String description;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private TaskStatus status = TaskStatus.OPEN;

    @Column(nullable = false)
    private String ownerEmail;

    private Instant createdAt = Instant.now();
    private Instant updatedAt = Instant.now();

    // getters, setters, @PreUpdate on updatedAt
}

public enum TaskStatus { OPEN, IN_PROGRESS, DONE }
```

## Repository

```java
public interface TaskRepository extends JpaRepository<Task, Long> {
    Page<Task> findByOwnerEmail(String ownerEmail, Pageable pageable);
    List<Task> findByStatusAndOwnerEmail(TaskStatus status, String ownerEmail);
}
```

---

# Phase 3 — DTOs & Service (20 min)

## Request / Response DTOs

```java
public record CreateTaskRequest(
    @NotBlank @Size(max = 200) String title,
    String description
) {}

public record TaskResponse(
    Long id, String title, String description,
    TaskStatus status, String ownerEmail,
    Instant createdAt, Instant updatedAt
) {}
```

## Service

```java
@Service
@Transactional
public class TaskService {
    private final TaskRepository repo;

    public TaskResponse create(CreateTaskRequest req, String ownerEmail) {
        Task task = new Task();
        task.setTitle(req.title());
        task.setDescription(req.description());
        task.setOwnerEmail(ownerEmail);
        return toResponse(repo.save(task));
    }

    @Transactional(readOnly = true)
    public TaskResponse getById(Long id, String ownerEmail) {
        Task task = repo.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Task not found"));
        if (!task.getOwnerEmail().equals(ownerEmail)) {
            throw new AccessDeniedException("Not your task");
        }
        return toResponse(task);
    }

    public TaskResponse updateStatus(Long id, TaskStatus status, String ownerEmail) {
        Task task = repo.findById(id).orElseThrow(...);
        authorizeOwner(task, ownerEmail);
        task.setStatus(status);
        return toResponse(task);
    }

    public void delete(Long id, String ownerEmail) {
        Task task = repo.findById(id).orElseThrow(...);
        authorizeOwner(task, ownerEmail);
        repo.delete(task);
    }
}
```

## Controller

```java
@RestController
@RequestMapping("/api/v1/tasks")
public class TaskController {
    private final TaskService taskService;

    @PostMapping
    public ResponseEntity<TaskResponse> create(
            @Valid @RequestBody CreateTaskRequest req,
            @AuthenticationPrincipal UserDetails user) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(taskService.create(req, user.getUsername()));
    }

    @GetMapping("/{id}")
    public TaskResponse get(@PathVariable Long id, @AuthenticationPrincipal UserDetails user) {
        return taskService.getById(id, user.getUsername());
    }

    @PatchMapping("/{id}/status")
    public TaskResponse patchStatus(@PathVariable Long id,
            @RequestParam TaskStatus status,
            @AuthenticationPrincipal UserDetails user) {
        return taskService.updateStatus(id, status, user.getUsername());
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable Long id, @AuthenticationPrincipal UserDetails user) {
        taskService.delete(id, user.getUsername());
    }
}
```

## Global Exception Handler

```java
@RestControllerAdvice
public class ApiExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Map<String, String>> notFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(Map.of("error", ex.getMessage()));
    }
}
```

---

# Phase 4 — Security + JWT (35 min)

## User entity + repository (simplified)

```java
@Entity
public class AppUser {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(unique = true) private String email;
    private String passwordHash;
    private String role; // ROLE_USER, ROLE_ADMIN
}
```

## Security config

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain filterChain(HttpSecurity http, JwtAuthFilter jwtFilter) throws Exception {
        http.csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/actuator/health").permitAll()
                .requestMatchers("/actuator/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }

    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

## Auth endpoints

```java
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {
    @PostMapping("/register")
    public ResponseEntity<Void> register(@Valid @RequestBody RegisterRequest req) { ... }

    @PostMapping("/login")
    public TokenResponse login(@Valid @RequestBody LoginRequest req) { ... }
}
```

## JwtService (sketch)

```java
@Service
public class JwtService {
    public String generateToken(String email, List<String> roles) {
        return Jwts.builder()
            .subject(email)
            .claim("roles", roles)
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + expirationMs))
            .signWith(secretKey)
            .compact();
    }

    public String extractEmail(String token) { ... }
}
```

## JwtAuthFilter

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain)
            throws ServletException, IOException {
        String header = req.getHeader(HttpHeaders.AUTHORIZATION);
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            String email = jwtService.extractEmail(token);
            // load UserDetails, set SecurityContextHolder
        }
        chain.doFilter(req, res);
    }
}
```

---

# Phase 5 — Actuator (10 min)

```text
GET /actuator/health        → {"status":"UP"}
GET /actuator/metrics       → JVM, HTTP metrics
GET /actuator/info          → custom build info (optional)
```

Add `info.app.version` in yaml. Secure sensitive endpoints with `ROLE_ADMIN`.

**Interview talking point:** Liveness vs readiness — use separate probes in Kubernetes (`/actuator/health/liveness`).

---

# Phase 6 — Unit & Slice Tests (25 min)

## Service test (Mockito)

```java
@ExtendWith(MockitoExtension.class)
class TaskServiceTest {
    @Mock TaskRepository repo;
    @InjectMocks TaskService service;

    @Test
    void create_savesTask() {
        when(repo.save(any())).thenAnswer(inv -> inv.getArgument(0));
        CreateTaskRequest req = new CreateTaskRequest("Learn JPA", null);
        TaskResponse resp = service.create(req, "user@x.com");
        assertEquals("Learn JPA", resp.title());
        verify(repo).save(any(Task.class));
    }
}
```

## `@WebMvcTest`

```java
@WebMvcTest(TaskController.class)
@Import(SecurityConfig.class)
class TaskControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean TaskService taskService;

    @Test
    @WithMockUser(username = "user@x.com")
    void getTask_ok() throws Exception {
        when(taskService.getById(1L, "user@x.com"))
            .thenReturn(new TaskResponse(1L, "T", null, TaskStatus.OPEN, "user@x.com", null, null));
        mockMvc.perform(get("/api/v1/tasks/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.title").value("T"));
    }
}
```

## `@DataJpaTest`

```java
@DataJpaTest
class TaskRepositoryTest {
    @Autowired TaskRepository repo;

    @Test
    void findByOwnerEmail() {
        Task t = new Task();
        t.setTitle("A");
        t.setOwnerEmail("a@b.com");
        repo.save(t);
        Page<Task> page = repo.findByOwnerEmail("a@b.com", PageRequest.of(0, 10));
        assertEquals(1, page.getTotalElements());
    }
}
```

---

# Phase 7 — Manual Smoke Test

```bash
# Register
curl -X POST http://localhost:8080/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"u@test.com","password":"secret123"}'

# Login → copy token
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"u@test.com","password":"secret123"}'

# Create task
curl -X POST http://localhost:8080/api/v1/tasks \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"title":"Interview prep"}'
```

---

# Build Checklist

| # | Done |
|---|------|
| Entity + repo + ddl-auto |
| Validation on DTOs |
| Service transactional boundaries |
| Owner-scoped authorization |
| JWT stateless security |
| Actuator health exposed |
| @WebMvcTest + @DataJpaTest + service test |
| Exception handler returns JSON errors |

---

# Common Interview Questions

**Why DTO instead of exposing entity?**  
Decouple API contract, hide fields, avoid lazy-load serialization issues.

**Where to put business rules?**  
Service layer; controllers stay thin.

**@Transactional on controller?**  
Avoid — service defines transaction boundaries.

**Test pyramid for this app?**  
Many unit tests on service, few slice tests, 1-2 `@SpringBootTest` integration tests optional.

---

*End of Day 52 — Spring Boot*
