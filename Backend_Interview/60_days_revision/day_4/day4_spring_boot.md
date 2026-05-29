# Day 4 - REST API Design (Create, Update, Delete)

---

# 1. REST API Design Basics

---

# HTTP Method Intent

| Method | Purpose | Idempotent? |
|--------|---------|-------------|
| GET | Read resource | Yes |
| POST | Create resource | No |
| PUT | Full update / replace | Yes |
| PATCH | Partial update | Usually yes |
| DELETE | Delete resource | Yes |

---

# Resource-Oriented URI Design

Use nouns, not verbs:

```text
GOOD:  /api/v1/users
BAD:   /api/v1/getUsers
```

---

# 2. @PostMapping

---

# Use Case

Create a new resource.

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @PostMapping
    public ResponseEntity<UserResponse> createUser(@RequestBody UserRequest req) {
        UserResponse saved = userService.create(req);
        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(saved);
    }
}
```

Recommended response:

- Status: `201 Created`
- Optional `Location` header for created resource URL

```java
return ResponseEntity
    .created(URI.create("/api/v1/users/" + saved.getId()))
    .body(saved);
```

---

# 3. @PutMapping

---

# Use Case

Replace/update complete resource by ID.

```java
@PutMapping("/{id}")
public ResponseEntity<UserResponse> updateUser(
        @PathVariable Long id,
        @RequestBody UserRequest req) {
    UserResponse updated = userService.update(id, req);
    return ResponseEntity.ok(updated);
}
```

If resource not found:

```text
404 Not Found
```

---

# PUT vs PATCH (Interview)

| PUT | PATCH |
|-----|-------|
| Full object update | Partial update |
| Missing fields may overwrite defaults/null | Only sent fields modified |
| Idempotent | Often idempotent |

---

# 4. @DeleteMapping

---

# Use Case

Delete resource by ID.

```java
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build(); // 204
}
```

Common status codes:

- `204 No Content` on success
- `404 Not Found` if resource missing

---

# 5. Path Variables

---

# Definition

`@PathVariable` binds dynamic path segment to method parameter.

```java
@GetMapping("/{id}")
public UserResponse getById(@PathVariable Long id) {
    return userService.getById(id);
}
```

Named mapping:

```java
@GetMapping("/{userId}/orders/{orderId}")
public OrderResponse getOrder(
    @PathVariable("userId") Long uId,
    @PathVariable("orderId") Long oId) {
    return orderService.getOrder(uId, oId);
}
```

---

# 6. Full CRUD Controller Example

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> get(@PathVariable Long id) {
        return ResponseEntity.ok(productService.get(id));
    }

    @PostMapping
    public ResponseEntity<ProductResponse> create(@RequestBody ProductRequest req) {
        ProductResponse created = productService.create(req);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<ProductResponse> update(
            @PathVariable Long id,
            @RequestBody ProductRequest req) {
        return ResponseEntity.ok(productService.update(id, req));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

# 7. Request / Response DTOs (Recommended)

Never expose entity directly from controller.

```java
public class ProductRequest {
    @NotBlank
    private String name;
    @Positive
    private Double price;
}

public class ProductResponse {
    private Long id;
    private String name;
    private Double price;
}
```

```java
@PostMapping
public ResponseEntity<ProductResponse> create(@Valid @RequestBody ProductRequest req) { ... }
```

---

# 8. Common API Design Interview Points

## Status Codes

| Scenario | Status |
|----------|--------|
| Create success | 201 |
| Read success | 200 |
| Delete success | 204 |
| Invalid input | 400 |
| Not found | 404 |
| Duplicate/conflict | 409 |
| Server error | 500 |

---

## URI Naming Best Practices

- Use plural nouns: `/users`, `/orders`
- Keep hierarchy meaningful: `/users/{id}/orders`
- Version API: `/api/v1/...`
- Avoid verbs: `createUser`, `deleteProduct`

---

## Idempotency

| Method | Same request repeated |
|--------|------------------------|
| POST | Usually creates multiple entries (not idempotent) |
| PUT | Final state same each time (idempotent) |
| DELETE | After first delete, state remains deleted |

---

# 9. Typical Service Layer (for controller)

```java
public interface ProductService {
    ProductResponse get(Long id);
    ProductResponse create(ProductRequest req);
    ProductResponse update(Long id, ProductRequest req);
    void delete(Long id);
}
```

---

# 10. Interview Quick Questions

## Q1. Why use `ResponseEntity`?

To control status code, headers, and body.

## Q2. POST vs PUT?

POST creates new; PUT replaces existing.

## Q3. Why `@PathVariable`?

To capture dynamic IDs from URI path.

## Q4. Why DTOs in API design?

Validation, security, decoupling from DB entity.

## Q5. Why `204` for delete?

Operation succeeded, no response body needed.

---

## Q6. REST API versioning strategies?

| Strategy | Example | Pros / Cons |
|----------|---------|-------------|
| URI path | `/api/v1/users` | Simple, visible — most common |
| Header | `Accept: application/vnd.myapp.v2+json` | Clean URLs — harder to test |
| Query param | `/users?version=2` | Easy but messy |
| Content negotiation | Same URL, different Accept header | Rare in practice |

**Recommendation for SDE interviews:** URI versioning (`/api/v1/`) + backward-compatible changes when possible.

---

## Q7. HTTP status codes — full guide?

| Code | When to use |
|------|-------------|
| 200 OK | Successful GET/PUT/PATCH with body |
| 201 Created | POST created resource |
| 204 No Content | DELETE success, PUT with no body |
| 400 Bad Request | Validation failure, malformed JSON |
| 401 Unauthorized | Not authenticated (missing/invalid token) |
| 403 Forbidden | Authenticated but no permission |
| 404 Not Found | Resource doesn't exist |
| 409 Conflict | Duplicate email, version conflict |
| 422 Unprocessable Entity | Semantic validation (optional) |
| 429 Too Many Requests | Rate limited |
| 500 Internal Server Error | Unhandled exception |

```java
return ResponseEntity.status(HttpStatus.CONFLICT)
    .body(new ErrorResponse("Email already exists"));
```

---

# One-Line Revision

```text
POST creates, PUT updates/replaces, DELETE removes, PathVariable binds resource IDs from URL.
```

---

*End of Day 4 Spring Boot*
