# Day 17 — Method Security: @PreAuthorize & @Secured

**Topics:** Method-Level Authorization · SpEL · Role-Based Access · Interview Questions

---

# 1. Method-Level Security

After authentication (JWT/session), **authorization** controls what authenticated users can do.

Spring provides declarative method security via annotations.

---

# Enable Method Security

```java
@Configuration
@EnableMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class MethodSecurityConfig {
}
```

Requires `spring-boot-starter-security`.

---

# 2. @Secured

Simple role-based check using Spring Security roles.

```java
@Service
public class AdminService {

    @Secured("ROLE_ADMIN")
    public void deleteUser(Long userId) {
        userRepository.deleteById(userId);
    }

    @Secured({"ROLE_ADMIN", "ROLE_MANAGER"})
    public Report generateReport() {
        return reportService.build();
    }
}
```

---

# @Secured Rules

- Must include `ROLE_` prefix (Spring convention)
- No SpEL expressions — static role names only
- Multiple roles = OR logic (any role grants access)

---

# 3. @PreAuthorize (Preferred)

Uses **SpEL** (Spring Expression Language) for flexible authorization.

```java
@Service
public class OrderService {

    @PreAuthorize("hasRole('USER')")
    public Order createOrder(OrderRequest req) {
        return orderRepository.save(toEntity(req));
    }

    @PreAuthorize("hasRole('ADMIN')")
    public void cancelAnyOrder(Long orderId) {
        orderRepository.deleteById(orderId);
    }

    @PreAuthorize("hasRole('USER') and #userId == authentication.principal.id")
    public List<Order> getOrders(Long userId) {
        return orderRepository.findByUserId(userId);
    }
}
```

---

# 4. @PostAuthorize

Checks authorization **after** method executes (can filter return value).

```java
@PostAuthorize("returnObject.ownerId == authentication.principal.id")
public Order getOrder(Long orderId) {
    return orderRepository.findById(orderId).orElseThrow();
}
```

Use when authorization depends on returned data.

---

# 5. Common SpEL Expressions

| Expression | Meaning |
|------------|---------|
| `hasRole('ADMIN')` | User has ROLE_ADMIN |
| `hasAuthority('READ')` | User has READ authority |
| `hasAnyRole('ADMIN','MANAGER')` | Any of listed roles |
| `#userId == authentication.principal.id` | Parameter matches logged-in user |
| `@authService.canAccess(#orderId)` | Custom bean method check |

---

# 6. Custom Authorization Bean

```java
@Component("authService")
public class AuthorizationService {

    public boolean canAccessOrder(Long orderId, Authentication auth) {
        Order order = orderRepository.findById(orderId).orElse(null);
        if (order == null) return false;
        UserDetails user = (UserDetails) auth.getPrincipal();
        return order.getOwnerEmail().equals(user.getUsername())
                || user.getAuthorities().stream()
                    .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
    }
}
```

Usage:

```java
@PreAuthorize("@authService.canAccessOrder(#orderId, authentication)")
public Order getOrder(Long orderId) { ... }
```

---

# 7. @Secured vs @PreAuthorize

| Feature | @Secured | @PreAuthorize |
|---------|----------|---------------|
| Expression language | No (static roles) | Yes (SpEL) |
| Parameter access | No | Yes (`#paramName`) |
| Custom beans | No | Yes (`@beanName.method()`) |
| Flexibility | Low | High |
| Interview preference | Legacy/simple apps | Modern Spring apps |

---

# 8. Controller vs Service Layer

**Best practice:** Apply security at **service layer**, not just controllers.

Why?
- Controllers aren't the only entry point (schedulers, message listeners)
- Defense in depth

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @GetMapping("/{id}")
    public Order getOrder(@PathVariable Long id) {
        return orderService.getOrder(id); // @PreAuthorize on service method
    }
}
```

---

# 9. Handling Access Denied

```java
@RestControllerAdvice
public class SecurityExceptionHandler {

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex) {
        return ResponseEntity
                .status(HttpStatus.FORBIDDEN)
                .body(new ErrorResponse("ACCESS_DENIED", "Insufficient permissions"));
    }
}
```

Returns `403 Forbidden` (not `401 Unauthorized`).

---

# 10. JWT + Method Security Integration

With JWT filter setting `SecurityContext`, method security works automatically:

```text
JWT Filter → sets Authentication in SecurityContext
                    ↓
@PreAuthorize → evaluates SpEL against Authentication
                    ↓
Allow or throw AccessDeniedException
```

Ensure JWT includes roles/authorities in token claims and maps them to `GrantedAuthority` in filter.

---

# Interview Questions

## 401 vs 403?

| Status | Meaning |
|--------|---------|
| 401 Unauthorized | Not authenticated (no/invalid token) |
| 403 Forbidden | Authenticated but lacks permission |

---

## hasRole vs hasAuthority?

`hasRole('ADMIN')` automatically prepends `ROLE_` → checks `ROLE_ADMIN`.

`hasAuthority('ROLE_ADMIN')` checks exact authority string.

---

## Can @PreAuthorize work without JWT?

Yes. Works with any authentication mechanism that populates `SecurityContext` (session, OAuth2, HTTP Basic).

---

## Performance impact of method security?

Spring creates proxy around secured beans. SpEL parsed once and cached. Negligible for most apps.

---

## How to test @PreAuthorize?

```java
@WebMvcTest(OrderController.class)
@Import(SecurityConfig.class)
class OrderControllerTest {

    @Test
    @WithMockUser(roles = "USER")
    void userCanGetOwnOrders() { ... }

    @Test
    @WithMockUser(roles = "ADMIN")
    void adminCanDeleteOrder() { ... }
}
```

---

# Quick Revision

```text
@EnableMethodSecurity → activate method-level security
@Secured              → simple role check (ROLE_ prefix)
@PreAuthorize         → SpEL-based, flexible (preferred)
@PostAuthorize        → check after method returns
403 Forbidden         → authenticated but not authorized
Apply on service layer → defense in depth
```

---

*End of Day 17 @PreAuthorize & @Secured*
