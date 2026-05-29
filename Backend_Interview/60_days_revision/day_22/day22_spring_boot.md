# Day 22 - OpenAPI / Swagger Documentation

---

# 1. Why API Documentation?

In backend interviews and production systems, APIs must be:

- Discoverable by frontend/mobile teams
- Testable without reading source code
- Versioned and contract-stable

**OpenAPI (formerly Swagger)** is the industry standard for describing REST APIs.

---

# 2. OpenAPI vs Swagger

| Term | Meaning |
|------|---------|
| OpenAPI Specification (OAS) | Standard format (YAML/JSON) describing API |
| Swagger | Tooling ecosystem (UI, Editor, Codegen) built around OAS |

In Spring Boot, we typically use **springdoc-openapi** (modern) or legacy **Springfox**.

---

# 3. springdoc-openapi Setup

## Maven Dependency

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

## Default URLs

```text
OpenAPI JSON:  /v3/api-docs
Swagger UI:    /swagger-ui/index.html
```

No extra config needed for basic setup.

---

# 4. Documenting Controllers

```java
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "User API", description = "User management endpoints")
public class UserController {

    @Operation(summary = "Get user by ID")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(
            @Parameter(description = "User ID") @PathVariable Long id) {
        return ResponseEntity.ok(userService.getById(id));
    }

    @Operation(summary = "Create new user")
    @PostMapping
    public ResponseEntity<UserResponse> createUser(
            @Valid @RequestBody UserRequest request) {
        UserResponse created = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
}
```

---

# 5. DTO Documentation

```java
@Schema(description = "Request to create a user")
public class UserRequest {

    @Schema(example = "John Doe", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank
    private String name;

    @Schema(example = "john@example.com")
    @Email
    private String email;
}
```

Swagger UI shows examples, required fields, and validation constraints.

---

# 6. Global OpenAPI Configuration

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("E-Commerce API")
                .version("v1")
                .description("Backend REST API for interview revision project")
                .contact(new Contact().name("Backend Team")))
            .addSecurityItem(new SecurityRequirement().addList("BearerAuth"))
            .components(new Components()
                .addSecuritySchemes("BearerAuth",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")));
    }
}
```

---

# 7. Grouping APIs by Version/Module

```java
@Bean
public GroupedOpenApi publicApi() {
    return GroupedOpenApi.builder()
            .group("public")
            .pathsToMatch("/api/v1/**")
            .build();
}

@Bean
public GroupedOpenApi adminApi() {
    return GroupedOpenApi.builder()
            .group("admin")
            .pathsToMatch("/api/admin/**")
            .build();
}
```

Useful for large microservices with multiple bounded contexts.

---

# 8. application.properties Tuning

```properties
springdoc.api-docs.path=/v3/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operationsSorter=method
springdoc.swagger-ui.tagsSorter=alpha
springdoc.show-actuator=false
```

Disable in production if needed:

```properties
springdoc.swagger-ui.enabled=false
springdoc.api-docs.enabled=false
```

Or protect with Spring Security.

---

# 9. Springfox (Legacy — Know for Interviews)

Older projects use Springfox 3.x:

```java
@Configuration
@EnableOpenApi
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.OAS_30)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example"))
                .paths(PathSelectors.any())
                .build();
    }
}
```

**Interview point:** Springfox is largely unmaintained; prefer **springdoc-openapi** for Spring Boot 3+.

---

# 10. Best Practices

| Practice | Why |
|----------|-----|
| Use DTOs, not entities | Avoid leaking DB schema |
| Document error responses | 400, 404, 409, 500 |
| Add examples on `@Schema` | Faster integration for clients |
| Version APIs (`/v1`, `/v2`) | Backward compatibility |
| Secure Swagger in prod | Prevent API surface exposure |
| Generate spec in CI | Contract testing with consumers |

---

# 11. Contract-First vs Code-First

| Approach | Flow |
|----------|------|
| Code-first | Write controller → annotations generate OAS |
| Contract-first | Write `openapi.yaml` → generate server/client stubs |

Microservices teams often use contract-first for consumer-driven contracts.

---

# Common Interview Questions

## Q1. What is OpenAPI?

Machine-readable specification describing REST endpoints, request/response schemas, auth, and errors.

## Q2. springdoc vs Springfox?

springdoc uses native Spring MVC integration, supports Boot 3/Jakarta; Springfox is legacy.

## Q3. How to hide internal endpoints from Swagger?

Use `@Hidden` on controller/method, or configure `pathsToExclude`.

## Q4. How to document JWT auth in Swagger UI?

Define `SecurityScheme` (HTTP bearer) and apply `SecurityRequirement` globally or per endpoint.

## Q5. Why disable Swagger in production?

Reduces attack surface — exposes all endpoints, parameters, and internal structure.

## Q6. How does frontend use OpenAPI?

Import spec into Postman, generate TypeScript client, or use Swagger UI for manual testing.

---

# One-Line Revision

```text
springdoc-openapi + @Operation/@Schema = auto-generated Swagger UI and OpenAPI contract for REST APIs.
```

---

*End of Day 22 Spring Boot*
