# Day 37 — Java Bean Validation (JSR 380)

**Topics:** Jakarta Validation · Annotations · Custom Validators · Spring Integration · Interview Questions

---

# 1. What is Bean Validation?

Standard API for declarative validation on Java beans (JSR 303/380).

```text
Annotate fields → Validator checks constraints → ConstraintViolation set
```

Implementations: **Hibernate Validator** (reference implementation).

---

# 2. Common Built-in Annotations

| Annotation | Validates |
|------------|-----------|
| `@NotNull` | Reference not null |
| `@NotBlank` | String not null/empty/whitespace |
| `@NotEmpty` | Collection/array/string not empty |
| `@Size(min,max)` | String/collection length |
| `@Min` / `@Max` | Numeric bounds |
| `@Email` | Email format |
| `@Pattern(regexp)` | Regex match |
| `@Past` / `@Future` | Date constraints |

---

# 3. Example DTO

```java
public class CreateUserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50)
    private String name;

    @NotBlank
    @Email
    private String email;

    @NotNull
    @Min(18)
    @Max(120)
    private Integer age;

    @Pattern(regexp = "^\\+?[1-9]\\d{9,14}$", message = "Invalid phone")
    private String phone;
}
```

---

# 4. Programmatic Validation

```java
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
Validator validator = factory.getValidator();

Set<ConstraintViolation<CreateUserRequest>> violations =
    validator.validate(request);

if (!violations.isEmpty()) {
    violations.forEach(v ->
        System.out.println(v.getPropertyPath() + ": " + v.getMessage()));
}
```

---

# 5. Spring Boot Integration

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<UserResponse> create(@Valid @RequestBody CreateUserRequest req) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(userService.create(req));
    }
}
```

`@Valid` triggers validation before method body runs.

---

# 6. Method-Level Validation

```java
@Service
@Validated
public class TransferService {

    public void transfer(
            @NotNull @Positive BigDecimal amount,
            @NotBlank String fromAccount,
            @NotBlank String toAccount) {
        // ...
    }
}
```

Enable with `@Validated` on class + parameter constraints.

---

# 7. Custom Constraint

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = AdultValidator.class)
public @interface Adult {
    String message() default "Must be 18+";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class AdultValidator implements ConstraintValidator<Adult, LocalDate> {
    @Override
    public boolean isValid(LocalDate dob, ConstraintValidatorContext ctx) {
        if (dob == null) return true;
        return Period.between(dob, LocalDate.now()).getYears() >= 18;
    }
}
```

---

# 8. Validation Groups

```java
public interface OnCreate {}
public interface OnUpdate {}

@Null(groups = OnCreate.class)
@NotNull(groups = OnUpdate.class)
private Long id;
```

```java
validator.validate(user, OnUpdate.class);
```

Useful when same DTO serves create vs update.

---

# 9. Global Exception Handler

```java
@RestControllerAdvice
public class ValidationExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handle(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(fe ->
            errors.put(fe.getField(), fe.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```

---

# 10. Bean Validation vs Spring Validation

| Layer | Role |
|-------|------|
| Jakarta Bean Validation | Annotation spec + SPI |
| Hibernate Validator | Implementation |
| Spring | `@Valid`, `LocalValidatorFactoryBean`, MVC integration |

---

# Interview Questions

## Q1. `@Valid` vs `@Validated`?

`@Valid` — standard JSR; cascades to nested objects. `@Validated` — Spring-specific; supports validation **groups**.

## Q2. Where to validate — controller or service?

Both: DTO shape at boundary (`@Valid`); business rules in service (e.g. sufficient balance). Never trust client-only validation.

## Q3. Performance concern?

Validation has overhead; cache `Validator` instance; avoid validating same object repeatedly in hot loops.

## Q4. Custom message i18n?

`message = "{user.email.invalid}"` with `ValidationMessages.properties`.

## Q5. Difference from manual if-checks?

Declarative, reusable, testable constraints; keeps domain code clean.

---

# One-Line Revision

```text
@NotBlank/@Email on DTO + @Valid in controller; custom ConstraintValidator for business rules; @Validated for groups.
```

---

*End of Day 37 Java*
