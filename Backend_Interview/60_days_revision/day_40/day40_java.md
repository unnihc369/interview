# Day 40 — Java 17: Records, Sealed Classes, Pattern Matching

**Topics:** Records · Sealed Hierarchy · Pattern Matching · Interview Questions

---

# 1. Records (Java 16+)

Immutable data carriers — compiler generates constructor, accessors, `equals`, `hashCode`, `toString`.

```java
public record User(Long id, String name, String email) {
    public User {
        if (name == null || name.isBlank()) throw new IllegalArgumentException("name required");
    }

    public String displayName() {
        return name.toUpperCase();
    }
}
```

Usage:

```java
User u = new User(1L, "Alice", "a@x.com");
System.out.println(u.name()); // accessor, not getName()
```

**Not** a JavaBean — no setters. Ideal for DTOs, events, value objects.

---

# 2. Records vs Lombok `@Value`

| Records | Lombok |
|---------|--------|
| Language feature | Annotation processor |
| Always final fields | Configurable |
| Cannot extend classes | Can extend |

Records can implement interfaces:

```java
public record OrderCreated(Long orderId, Instant at) implements DomainEvent {}
```

---

# 3. Sealed Classes (Java 17)

Restrict which classes can extend/implement.

```java
public sealed interface PaymentResult
    permits Success, Failure, Pending {}

public final record Success(String txnId) implements PaymentResult {}
public final record Failure(String reason) implements PaymentResult {}
public non-sealed class Pending implements PaymentResult {}
```

**Rules:**

- Subclasses must be `final`, `sealed`, or `non-sealed`
- All permitted subclasses in same module (or same package pre-module)

Enables exhaustive handling in pattern matching.

---

# 4. Pattern Matching for `instanceof` (Java 16)

```java
// Before
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// Java 16+
if (obj instanceof String s) {
    System.out.println(s.length());
}
```

---

# 5. Pattern Matching for `switch` (Java 17+ preview, 21 finalized)

```java
static String describe(PaymentResult result) {
    return switch (result) {
        case Success s -> "OK: " + s.txnId();
        case Failure f -> "ERR: " + f.reason();
        case Pending p -> "WAIT";
    };
}
```

Guarded patterns:

```java
case Success s when s.txnId().startsWith("TX") -> "premium success";
```

Exhaustive switch — compiler error if sealed hierarchy not fully covered.

---

# 6. Record Patterns (Java 21)

```java
static void handle(Object obj) {
    if (obj instanceof User(Long id, String name, var email)) {
        System.out.println(id + " " + name);
    }
}
```

Deconstruct record components in `instanceof` and `switch`.

---

# 7. Text Blocks (Java 15) — Quick

```java
String json = """
    {
      "id": 1,
      "name": "Alice"
    }
    """;
```

---

# 8. Practical Spring Usage

```java
public record CreateOrderRequest(
    @NotNull Long productId,
    @Min(1) int quantity
) {}

@PostMapping
public OrderResponse create(@Valid @RequestBody CreateOrderRequest req) {
    return service.create(req);
}
```

Jackson deserializes records since 2.12+.

---

# Interview Questions

## Q1. Can records extend classes?

No — implicit `extends Record`. Can implement interfaces.

## Q2. Why sealed classes?

Model closed domain hierarchies; compiler-checked exhaustiveness; safer than string enums for complex types.

## Q3. Record equals semantics?

Component-wise value equality — good for DTOs; identity not reference-based for content.

## Q4. Record serialization?

Works with Jackson; customize with `@JsonProperty` on components if needed.

## Q5. `non-sealed` purpose?

Allow unknown subclasses outside hierarchy — escape hatch for extensibility.

---

# One-Line Revision

```text
record = immutable DTO; sealed = closed hierarchy; switch pattern = exhaustive domain handling.
```

---

*End of Day 40 Java*
