# Day 16 — Custom Annotations: @Retention & @Target

**Topics:** Meta-annotations · Creating Custom Annotations · Runtime Processing · Interview Questions

---

# 1. What is a Custom Annotation?

Annotations are metadata attached to code. Custom annotations let you define domain-specific markers processed at compile time or runtime.

Common in Spring:
- `@Transactional`
- `@Cacheable`
- `@Valid`

---

# 2. Creating a Custom Annotation

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimited {
    int requestsPerMinute() default 60;
    String key() default "";
}
```

---

# 3. @Target — Where Can Annotation Be Applied?

`@Target` restricts annotation placement.

```java
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface SecuredEndpoint { }
```

---

# ElementType Values (Interview Table)

| ElementType | Applies To |
|-------------|------------|
| TYPE | Class, interface, enum |
| FIELD | Field/variable |
| METHOD | Method |
| PARAMETER | Method parameter |
| CONSTRUCTOR | Constructor |
| LOCAL_VARIABLE | Local variable |
| PACKAGE | Package declaration |
| ANNOTATION_TYPE | Another annotation |
| TYPE_PARAMETER | Generic type parameter (Java 8+) |
| TYPE_USE | Any type use (Java 8+) |

---

# 4. @Retention — How Long Annotation Persists?

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Audited { }
```

---

# RetentionPolicy Values

| Policy | Available At | Use Case |
|--------|-------------|----------|
| SOURCE | Compile time only | `@Override`, `@SuppressWarnings` |
| CLASS | Bytecode, not runtime | Bytecode tools, rarely used directly |
| RUNTIME | Runtime via Reflection | Spring AOP, validation frameworks |

**Interview default for custom framework annotations:** `RUNTIME`.

---

# 5. Other Meta-Annotations

```java
@Documented          // Include in Javadoc
@Inherited           // Subclasses inherit annotation
@Repeatable          // Same annotation multiple times (Java 8+)
```

Example:

```java
@Documented
@Inherited
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Entity {
    String tableName();
}
```

---

# 6. Annotation Elements (Attributes)

Annotations can have elements (like methods):

```java
public @interface CacheConfig {
    String cacheName();
    int ttlSeconds() default 300;
    TimeUnit unit() default TimeUnit.SECONDS;
}
```

Usage:

```java
@CacheConfig(cacheName = "users", ttlSeconds = 600)
public User getUser(Long id) { ... }
```

Rules:
- Element types: primitives, String, Class, enum, annotation, arrays of these
- No `null` default values
- Elements must have defaults OR be provided at usage

---

# 7. Processing Annotations at Runtime

```java
@Component
public class RateLimitAspect {

    @Around("@annotation(rateLimited)")
    public Object enforce(ProceedingJoinPoint pjp, RateLimited rateLimited)
            throws Throwable {
        String key = rateLimited.key().isEmpty()
                ? pjp.getSignature().getName()
                : rateLimited.key();
        if (!rateLimiter.allow(key, rateLimited.requestsPerMinute())) {
            throw new TooManyRequestsException();
        }
        return pjp.proceed();
    }
}
```

---

# 8. Reflection-Based Processing

```java
Method method = clazz.getMethod("processOrder");
if (method.isAnnotationPresent(Audited.class)) {
    Audited audited = method.getAnnotation(Audited.class);
    log.info("Auditing action: {}", audited.action());
}
```

Requires `@Retention(RetentionPolicy.RUNTIME)`.

---

# 9. Real-World Example — Validation Annotation

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PhoneNumberValidator.class)
public @interface ValidPhone {
    String message() default "Invalid phone number";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

Validator:

```java
public class PhoneNumberValidator implements ConstraintValidator<ValidPhone, String> {
    @Override
    public boolean isValid(String value, ConstraintValidatorContext ctx) {
        return value != null && value.matches("^\\+?[1-9]\\d{9,14}$");
    }
}
```

---

# 10. @Repeatable Example

```java
@Repeatable(Roles.class)
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Role {
    String value();
}

public @interface Roles {
    Role[] value();
}
```

Usage:

```java
@Role("ADMIN")
@Role("MANAGER")
public void approveExpense() { }
```

---

# Interview Questions

## Difference between @Target and @Retention?

`@Target` defines **where** annotation can be placed. `@Retention` defines **how long** it survives (source, class file, or runtime).

---

## Why RUNTIME for Spring annotations?

Spring uses reflection and AOP at runtime to detect annotations and apply behavior (transactions, caching, security).

---

## Can annotation extend another annotation?

No. Annotations are not inheritable by default unless marked `@Inherited` (only for class-level).

---

## SOURCE vs RUNTIME retention example?

`@Override` is SOURCE — compiler checks, then discarded. `@Autowired` is RUNTIME — Spring reads it when creating beans.

---

## How does @Transactional work internally?

Spring scans beans at startup. Methods/classes with `@Transactional` get proxied. Interceptor reads annotation at runtime and manages transaction boundaries.

---

# Quick Revision

```text
@Target     → WHERE annotation applies (METHOD, FIELD, TYPE...)
@Retention  → HOW LONG it persists (SOURCE, CLASS, RUNTIME)
Meta-annotations → annotations on annotations
RUNTIME     → required for Spring/Reflection processing
Custom annotations → declare + process via AOP or Reflection
```

---

*End of Day 16 Custom Annotations*
