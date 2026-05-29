# Day 46 — Java 8+ Features Review

**Topics:** Lambdas · Streams · Optional · java.time · Records · Sealed classes · Interview Q&A

---

# 1. Java Version Timeline (Interview Quick Reference)

| Version | Key Features |
|---------|--------------|
| Java 8 | Lambdas, Streams, Optional, java.time, default methods |
| Java 9 | Modules (JPMS), `List.of()`, private interface methods |
| Java 10 | `var` local inference |
| Java 11 | LTS, `String.isBlank()`, HTTP Client |
| Java 14 | Records (preview), switch expressions |
| Java 15 | Text blocks `"""` |
| Java 17 | LTS, sealed classes, pattern matching for instanceof |
| Java 21 | LTS, virtual threads, sequenced collections |

---

# 2. Lambda Expressions & Functional Interfaces

```java
// Before Java 8
Collections.sort(list, new Comparator<String>() {
    public int compare(String a, String b) { return a.compareTo(b); }
});

// Lambda
list.sort((a, b) -> a.compareTo(b));

// Method reference
list.sort(String::compareTo);
```

## Built-in Functional Interfaces

| Interface | Method | Use |
|-----------|--------|-----|
| `Predicate<T>` | `test(T)` | Filter |
| `Function<T,R>` | `apply(T)` | Map |
| `Consumer<T>` | `accept(T)` | forEach |
| `Supplier<T>` | `get()` | Lazy creation |
| `BiFunction<T,U,R>` | `apply(T,U)` | Reduce/combine |

---

# 3. Streams API

```java
List<Order> recentPaid = orders.stream()
    .filter(o -> o.getStatus() == Status.PAID)
    .filter(o -> o.getAmount().compareTo(BigDecimal.valueOf(100)) > 0)
    .sorted(Comparator.comparing(Order::getCreatedAt).reversed())
    .limit(10)
    .collect(Collectors.toList());

Map<Long, List<Order>> byUser = orders.stream()
    .collect(Collectors.groupingBy(Order::getUserId));

double avgAmount = orders.stream()
    .mapToDouble(o -> o.getAmount().doubleValue())
    .average()
    .orElse(0.0);
```

## Intermediate vs Terminal

```text
Intermediate (lazy): filter, map, flatMap, sorted, distinct, peek
Terminal (eager):    collect, forEach, reduce, count, findFirst, anyMatch
```

## Parallel Streams — Caution

```java
long count = largeList.parallelStream()
    .filter(x -> expensiveCheck(x))
    .count();
// Only when: large data, CPU-bound, no shared mutable state, no order dependency
```

---

# 4. Optional

```java
// Anti-pattern: Optional as field or parameter (usually)
// Good: return type for "may be absent"

public Optional<User> findByEmail(String email) {
    return userRepo.findByEmail(email); // returns Optional<User>
}

User user = findByEmail("a@b.com")
    .orElseThrow(() -> new UserNotFoundException("a@b.com"));

String name = findByEmail("x@y.com")
    .map(User::getName)
    .orElse("Guest");

// Java 9+
optional.ifPresentOrElse(
    u -> sendEmail(u),
    () -> log.warn("User not found")
);
```

---

# 5. java.time (Java 8+)

```java
LocalDate today = LocalDate.now();
LocalDateTime meeting = LocalDateTime.of(2026, 5, 29, 14, 30);
ZonedDateTime utc = ZonedDateTime.now(ZoneOffset.UTC);
Instant instant = Instant.now();

Duration between = Duration.between(start, end);       // time-based
Period period = Period.between(birthDate, today);        // date-based

LocalDate nextWeek = today.plusWeeks(1);
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd-MM-yyyy");
String formatted = today.format(fmt);
```

**Never use `java.util.Date` / `Calendar` in new code.**

---

# 6. Default & Static Interface Methods

```java
public interface PaymentGateway {
    PaymentResult charge(Money amount);

    default PaymentResult chargeWithRetry(Money amount) {
        for (int i = 0; i < 3; i++) {
            try { return charge(amount); }
            catch (GatewayException e) { /* retry */ }
        }
        throw new GatewayException("Max retries");
    }

    static PaymentGateway noop() {
        return amount -> PaymentResult.ok("noop");
    }
}
```

Enables interface evolution without breaking implementors.

---

# 7. Records (Java 16+)

```java
public record OrderResponse(Long id, String status, BigDecimal amount) {
    // Compact constructor for validation
    public OrderResponse {
        Objects.requireNonNull(status);
    }
}

// Auto: constructor, getters, equals, hashCode, toString
OrderResponse r = new OrderResponse(1L, "PAID", BigDecimal.TEN);
Long id = r.id();  // not getId()
```

Use for DTOs, value objects — immutable by default.

---

# 8. Sealed Classes (Java 17)

```java
public sealed interface PaymentResult
    permits Success, Failure, Pending {

    record Success(String txnId) implements PaymentResult {}
    record Failure(String reason) implements PaymentResult {}
    record Pending(String ref) implements PaymentResult {}
}

// Exhaustive switch (Java 21+)
String msg = switch (result) {
    case Success s  -> "Paid: " + s.txnId();
    case Failure f  -> "Failed: " + f.reason();
    case Pending p  -> "Pending: " + p.ref();
};
```

---

# 9. var (Java 10+)

```java
var list = new ArrayList<String>();  // inferred ArrayList<String>
var map = Map.of("a", 1, "b", 2);

// NOT for fields, method params, or when type unclear
var x = getResult();  // OK if return type obvious
```

---

# 10. Immutable Collections (Java 9+)

```java
List<String> list = List.of("a", "b");       // immutable
Set<Integer> set = Set.of(1, 2, 3);
Map<String, Integer> map = Map.of("x", 1, "y", 2);
// map.put("z", 3); → UnsupportedOperationException
```

---

# 11. Virtual Threads (Java 21) — Bonus

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i ->
        executor.submit(() -> handleRequest(i)));
}
// Millions of lightweight threads — great for I/O-bound workloads
```

---

# Interview Questions

## Q1. Stream vs loop?

Streams: declarative, composable, parallel option. Loops: simpler, mutable state OK, often faster for small collections.

## Q2. When NOT to use Optional?

As method parameter (use overloads), as class field (use nullable or sentinel), in collections.

## Q3. Record vs Lombok @Value?

Record is language-native, zero dependency, guaranteed semantics. Lombok generates bytecode — records preferred for DTOs in Java 17+.

## Q4. Difference Duration vs Period?

Duration = time amount (hours, minutes). Period = calendar amount (days, months, years).

## Q5. Sealed vs final?

Sealed restricts **who can extend** (known hierarchy). Final prevents all extension.

---

# One-Line Revision

```text
Java 8+ = lambdas/streams/optional/time; 10+ var; 16+ records; 17+ sealed; 21+ virtual threads — know one example each.
```

---

*End of Day 46 Java*
