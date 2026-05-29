# Day 39 — Spring REST Docs vs Swagger (OpenAPI)

**Topics:** API Documentation Strategies · springdoc · Spring REST Docs · Interview Comparison

---

# 1. Why API Documentation?

- Contract between frontend and backend
- Onboarding and integration testing
- Interview: shows production maturity

Two mainstream approaches in Spring:

| Tool | Style |
|------|-------|
| **Swagger / OpenAPI** (springdoc) | Annotation-driven, live UI |
| **Spring REST Docs** | Test-driven, AsciiDoc/HTML |

---

# 2. Swagger / OpenAPI with springdoc

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

```java
@OpenAPIDefinition(info = @Info(title = "Order API", version = "v1"))
@SpringBootApplication
public class Application {}
```

```java
@Operation(summary = "Create order")
@ApiResponses({
    @ApiResponse(responseCode = "201", description = "Created"),
    @ApiResponse(responseCode = "400", description = "Invalid input")
})
@PostMapping("/orders")
public ResponseEntity<OrderResponse> create(@Valid @RequestBody OrderRequest req) {
    return ResponseEntity.status(HttpStatus.CREATED).body(service.create(req));
}
```

Access: `http://localhost:8080/swagger-ui.html`

---

# 3. OpenAPI Spec Generation

```properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
```

Export JSON/YAML for Postman, codegen, API gateways.

---

# 4. Spring REST Docs — Test-Driven

```xml
<dependency>
    <groupId>org.springframework.restdocs</groupId>
    <artifactId>spring-restdocs-mockmvc</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@WebMvcTest(OrderController.class)
@AutoConfigureRestDocs
class OrderControllerTest {

    @Autowired MockMvc mockMvc;

    @Test
    void createOrder() throws Exception {
        mockMvc.perform(post("/api/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"productId":1,"quantity":2}
                    """))
            .andExpect(status().isCreated())
            .andDo(document("create-order",
                requestFields(
                    fieldWithPath("productId").description("Product ID"),
                    fieldWithPath("quantity").description("Quantity")
                ),
                responseFields(
                    fieldWithPath("id").description("Order ID"),
                    fieldWithPath("status").description("Order status")
                )
            ));
    }
}
```

AsciiDoc snippets assembled into polished PDF/HTML — **always accurate** because generated from passing tests.

---

# 5. Comparison Table (Interview Gold)

| Criteria | Swagger/OpenAPI | Spring REST Docs |
|----------|-----------------|------------------|
| Source of truth | Annotations | Integration tests |
| Drift risk | Docs can lie if annotations stale | Fails build if API changes |
| Live try-it UI | Yes (Swagger UI) | No (static docs) |
| Setup effort | Low | Higher (write tests + snippets) |
| Non-Spring consumers | OpenAPI standard | Custom pipeline |
| CI integration | Publish spec artifact | Maven asciidoctor plugin |

---

# 6. Hybrid Approach (Production)

```text
Spring REST Docs → human-readable guide for devs
springdoc OpenAPI  → machine-readable spec + Swagger UI for QA
Contract tests     → Pact / OpenAPI validator
```

---

# 7. Documenting Security in OpenAPI

```java
@SecurityScheme(name = "bearerAuth", type = SecuritySchemeType.HTTP,
    scheme = "bearer", bearerFormat = "JWT")
@SecurityRequirement(name = "bearerAuth")
@GetMapping("/orders")
public List<Order> list() { ... }
```

---

# 8. Versioning in Docs

```java
@Tag(name = "Orders V2")
@RestController
@RequestMapping("/api/v2/orders")
public class OrderV2Controller {}
```

---

# Interview Questions

## Q1. Which do you prefer and why?

REST Docs when correctness critical (finance, public API). Swagger when rapid iteration and frontend needs playground.

## Q2. How prevent doc drift?

REST Docs enforces via tests; for Swagger add OpenAPI contract tests in CI comparing spec to responses.

## Q3. springfox vs springdoc?

springfox unmaintained for Spring Boot 3; use **springdoc-openapi**.

## Q4. Can REST Docs generate OpenAPI?

Third-party bridges exist; common pattern is both tools for different audiences.

## Q5. Document error responses?

REST Docs: `responseFields` for 4xx bodies. OpenAPI: `@ApiResponse` per status code.

---

# One-Line Revision

```text
Swagger/springdoc = fast UI + OpenAPI spec; REST Docs = test-generated accurate docs; use both in enterprise.
```

---

*End of Day 39 Spring Boot*
