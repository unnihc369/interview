# Day 39 — Java 9+ Features: var, List.of, Modules

**Topics:** Local Variable Type Inference · Immutable Collections · Optional Enhancements · Interview Questions

---

# 1. `var` — Local Variable Type Inference (Java 10)

Compiler infers type from initializer.

```java
var list = new ArrayList<String>();  // ArrayList<String>
var map = Map.of("a", 1);            // Map<String, Integer>
var stream = list.stream();            // Stream<String>
```

**Rules:**

- Only for **local variables** with initializer
- Cannot use for fields, method params, or without initializer
- Cannot be `null` without explicit type context issues

```java
// INVALID
var x;           // no initializer
var[] arr = {1}; // no arrays with var
```

---

# 2. When to Use `var` (Interview)

| Good | Bad |
|------|-----|
| Long generic chains obvious from RHS | Obscures important type |
| `var entries = map.entrySet()` | `var data = getData()` — unclear return type |
| try-with-resources | Public API signatures |

---

# 3. `List.of`, `Set.of`, `Map.of` (Java 9)

Immutable collections — no add/remove/set.

```java
List<String> list = List.of("a", "b", "c");
Set<Integer> set = Set.of(1, 2, 3);
Map<String, Integer> map = Map.of("x", 1, "y", 2);

// List.of allows duplicates? NO for List.of — null not allowed
List<Integer> nums = List.of(1, 2, 2); // OK distinct elements

// Map.of — max 10 key-value pairs; use Map.ofEntries for more
Map<String, Integer> big = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2)
);
```

**Null forbidden** — throws NPE on creation.

**Mutating throws** `UnsupportedOperationException`.

---

# 4. Copy Factory Methods

```java
List<String> copy = List.copyOf(mutableList); // unmodifiable snapshot
```

If source is already unmodifiable, may return same instance.

---

# 5. `Optional` Enhancements (Java 9+)

```java
optional.ifPresentOrElse(v -> log(v), () -> log("empty"));
optional.stream(); // Java 9 — stream of 0 or 1
optional.or(() -> Optional.of("default")); // Java 9
optional.isEmpty(); // Java 11 — opposite of isPresent
```

---

# 6. `takeWhile`, `dropWhile` on Streams (Java 9)

```java
List<Integer> nums = List.of(1, 2, 3, 4, 5, 2);
nums.stream().takeWhile(n -> n < 4).forEach(System.out::println);
// 1, 2, 3 — stops at first false (not filter!)
```

---

# 7. Private Interface Methods (Java 9)

```java
public interface MyService {
    default void execute() {
        validate();
        doWork();
    }
    private void validate() { ... }
    void doWork();
}
```

---

# 8. Module System JPMS (Java 9) — Brief

```java
// module-info.java
module com.example.app {
    requires java.sql;
    requires spring.boot;
    exports com.example.api;
}
```

Strong encapsulation — internal packages hidden unless exported.

Spring Boot 3+ modular adoption still often classpath-based; know concept for interviews.

---

# 9. Other Java 9–11 Quick Hits

| Feature | Version |
|---------|---------|
| `String.isBlank()`, `lines()`, `strip()` | 11 |
| `HttpClient` (java.net.http) | 11 |
| `var` in lambda params | 11 |
| Text blocks `"""` | 15 |
| `record` | 16 |
| `sealed` classes | 17 |

---

# Interview Questions

## Q1. `List.of` vs `Arrays.asList`?

`Arrays.asList` — fixed size, backed by array, allows set. `List.of` — truly immutable, null-safe.

## Q2. Can `var` replace generics declaration?

No for method signatures. Only local inference.

## Q3. Why immutable collections in Java 9?

Safer defaults, less defensive copying, clearer intent.

## Q4. Module system benefit?

Strong boundaries, smaller custom JRE (jlink), explicit dependencies.

## Q5. `takeWhile` vs `filter`?

`takeWhile` stops at first failure; `filter` tests all elements.

---

# One-Line Revision

```text
var = local inference only; List.of/Map.of = immutable; copyOf for snapshots; Optional.ifPresentOrElse in Java 9.
```

---

*End of Day 39 Java*
