# Day 52 — Supporting Java Topics (Task Manager Project)

**Week 8 · Wednesday · Use alongside `day52_spring_boot.md`**

Topics you need fluent **while** building the 2-hour CRUD app.

---

# 1. Records vs Classes for DTOs

```java
public record CreateTaskRequest(
    @NotBlank String title,
    String description
) {}
```

- Immutable by default  
- Auto `equals`, `hashCode`, `toString`, accessors  
- Use **classes** when you need mutable builders or JPA entities (records ≠ entities in older Hibernate; OK as DTOs)

---

# 2. Optional — Don't Abuse in APIs

```java
// Good: internal lookup
Task task = repo.findById(id)
    .orElseThrow(() -> new ResourceNotFoundException("Task not found"));

// Bad: return Optional from REST (awkward JSON)
```

---

# 3. Streams for Mapping Lists

```java
List<TaskResponse> mapAll(List<Task> tasks) {
    return tasks.stream().map(this::toResponse).toList();
}
```

**Interview:** `toList()` (Java 16+) returns unmodifiable list vs `collect(Collectors.toList())`.

---

# 4. Instant & java.time

```java
Instant now = Instant.now();
task.setUpdatedAt(now);
```

Never use `java.util.Date` in new code. For user-facing dates: `ZonedDateTime` with explicit zone.

---

# 5. Validation Annotations

| Annotation | Use |
|------------|-----|
| `@NotBlank` | String not null/empty/whitespace |
| `@NotNull` | Any type |
| `@Size(min,max)` | String/collection length |
| `@Email` | Email format |
| `@Valid` | Trigger validation on nested object |

Enable with `@Valid` on controller parameter + `spring-boot-starter-validation`.

---

# 6. Exception Design

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

- **Unchecked** for business errors (404, 403) mapped by `@RestControllerAdvice`  
- **Checked** only when caller must handle (rare in Spring REST)

---

# 7. Lombok (If Used)

```java
@Entity
@Getter @Setter
@NoArgsConstructor
public class Task { ... }
```

Know manual equivalents for interviews without Lombok.

---

# 8. Generics — Repository Pattern

```java
public interface TaskRepository extends JpaRepository<Task, Long> { }
//                                              ^entity  ^id type
```

---

# 9. Mockito Essentials

```java
when(repo.findById(1L)).thenReturn(Optional.of(task));
when(repo.save(any(Task.class))).thenAnswer(inv -> inv.getArgument(0));
verify(repo, times(1)).delete(task);
assertThrows(AccessDeniedException.class, () -> service.getById(2L, "other@x.com"));
```

**`@Mock` vs `@MockBean`:** MockitoExtension uses `@Mock`; Spring tests use `@MockBean` to replace context bean.

---

# 10. JWT Claims (Conceptual)

```json
{
  "sub": "user@email.com",
  "roles": ["ROLE_USER"],
  "iat": 1710000000,
  "exp": 1710086400
}
```

Parse with library (jjwt); validate signature + expiration in filter.

---

# 11. Thread Safety Note

`TaskService` singleton bean — **no mutable instance fields**; only final dependencies. Per-request state in method locals or `SecurityContextHolder`.

---

# 12. Quick Drill (15 min)

Without IDE, write:

1. Record `LoginRequest(email, password)` with validation  
2. Method `toResponse(Task t)` returning record  
3. Mockito test: `delete` calls `repo.delete` when owner matches  

---

# Mapping to Spring Concepts

| Java topic | Spring usage |
|------------|--------------|
| Records | Request/response DTOs |
| Optional | Repository return type |
| Instant | Audit timestamps |
| Validation | `@Valid` on controller |
| Mockito | Service unit tests |
| Generics | `JpaRepository<E, ID>` |

---

*End of Day 52 — Java*
