# Day 31 — Collectors (Advanced)

**Topics:** Stream Collectors · groupingBy · teeing · Custom Collectors · Interview Questions

---

# 1. Basic Collectors Recap

```java
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

Map<Long, Person> byId = people.stream()
    .collect(Collectors.toMap(Person::getId, p -> p));

String joined = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));
```

---

# 2. groupingBy — Multi-level Aggregation

```java
// Group employees by department
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Group + count
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()));

// Group + average salary
Map<String, Double> avgSalary = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)));
```

---

# 3. partitioningBy

Split into two buckets (Predicate-based):

```java
Map<Boolean, List<Employee>> activeSplit = employees.stream()
    .collect(Collectors.partitioningBy(Employee::isActive));

// Partition + downstream
Map<Boolean, Long> countActive = employees.stream()
    .collect(Collectors.partitioningBy(
        Employee::isActive,
        Collectors.counting()));
```

Always returns map with `true` and `false` keys.

---

# 4. mapping & flatMapping (Nested Groups)

```java
// Dept → list of employee names only
Map<String, List<String>> deptToNames = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(Employee::getName, Collectors.toList())));

// Dept → distinct skills (flatten projects)
Map<String, Set<String>> deptSkills = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.flatMapping(
            e -> e.getSkills().stream(),
            Collectors.toSet())));
```

---

# 5. reducing as Downstream

```java
Map<String, Optional<Employee>> topEarnerPerDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.reducing(
            BinaryOperator.maxBy(Comparator.comparing(Employee::getSalary)))));
```

---

# 6. collectingAndThen — Finishing Transform

```java
// Unmodifiable list
List<String> immutable = stream.collect(
    Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList));

// Truncate top 10 after sort
List<Employee> top10 = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        list -> list.subList(0, Math.min(10, list.size()))));
```

---

# 7. teeing (Java 12+) — Two Collectors in Parallel

```java
record Stats(long count, double avg) {}

Stats stats = numbers.stream().collect(
    Collectors.teeing(
        Collectors.counting(),
        Collectors.averagingInt(Integer::intValue),
        Stats::new));

// Order summary: total + max item
record OrderSummary(BigDecimal total, OrderItem maxItem) {}

OrderSummary summary = order.getItems().stream().collect(
    Collectors.teeing(
        Collectors.reducing(BigDecimal.ZERO,
            OrderItem::getPrice,
            BigDecimal::add),
        Collectors.maxBy(Comparator.comparing(OrderItem::getPrice)),
        (total, max) -> new OrderSummary(total, max.orElseThrow())));
```

---

# 8. toMap — Merge & Uniqueness

```java
// Handle duplicate keys
Map<String, Employee> map = employees.stream()
    .collect(Collectors.toMap(
        Employee::getEmail,
        Function.identity(),
        (e1, e2) -> e1)); // keep first on collision

// EnumMap / TreeMap supplier
Map<Department, List<Employee>> sorted = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        () -> new TreeMap<>(),
        Collectors.toList()));
```

**Avoid:** `Collectors.toMap` without merge function when duplicates possible → `IllegalStateException`.

---

# 9. Custom Collector

```java
public static Collector<Order, ?, BigDecimal> summingOrderTotal() {
    return Collector.of(
        () -> new BigDecimal[]{BigDecimal.ZERO},
        (acc, order) -> acc[0] = acc[0].add(order.getTotal()),
        (a, b) -> { a[0] = a[0].add(b[0]); return a; },
        acc -> acc[0]
    );
}
```

Use when built-in collectors don't fit (complex accumulation).

---

# Interview Questions

## groupingBy vs partitioningBy?

`groupingBy` — arbitrary key function, many groups. `partitioningBy` — boolean predicate, exactly two groups.

## Concurrent collectors?

`groupingByConcurrent`, `toConcurrentMap` — thread-safe for parallel streams.

## Why not mutate list inside forEach?

Side effects break parallelization and are error-prone. Prefer `collect`.

## `Optional` from `maxBy` in grouping?

Downstream returns `Optional<T>` per group when no elements — handle with `orElse`.

## Performance of complex collectors?

Single pass O(n). Nested grouping still one traversal; downstream collectors run per element.

---

# Quick Revision

```text
groupingBy(classifier, downstream)
partitioningBy(predicate, downstream)
mapping / flatMapping → transform group values
teeing(c1, c2, merger) → two stats in one pass
collectingAndThen → post-process result
toMap(key, val, mergeFn) → duplicate keys
```

---

*End of Day 31 Advanced Collectors*
