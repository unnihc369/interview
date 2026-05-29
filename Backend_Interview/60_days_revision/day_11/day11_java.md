# Day 11 — Java Streams API (filter, map, collect, reduce)

**Topics:** Functional-style pipelines · Intermediate vs terminal ops · Collectors · Interview Q&A

---

# 1. What is Stream API?

Introduced in Java 8 — process collections declaratively without explicit loops.

```text
source → intermediate ops → terminal op → result
```

Streams are **not** data structures; they don't store elements.

---

# 2. Creating Streams

```java
List<String> names = List.of("Ana", "Bob", "Alex");

names.stream();
Arrays.stream(new int[]{1, 2, 3});
Stream.of(1, 2, 3);
IntStream.range(1, 10);
```

---

# 3. filter

Keeps elements matching predicate.

```java
List<String> longNames = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());
```

---

# 4. map

Transforms each element.

```java
List<String> upper = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
```

`flatMap` — flatten nested structures:

```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3));
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList()); // [1,2,3]
```

---

# 5. collect

Terminal operation — gathers into collection or summary.

```java
List<String> result = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
Map<String, Integer> map = stream.collect(
    Collectors.toMap(s -> s, String::length)
);

String joined = names.stream().collect(Collectors.joining(", "));
```

### groupingBy

```java
Map<Integer, List<String>> byLen = names.stream()
    .collect(Collectors.groupingBy(String::length));
```

### partitioningBy

```java
Map<Boolean, List<String>> parts = names.stream()
    .collect(Collectors.partitioningBy(n -> n.startsWith("A")));
```

---

# 6. reduce

Combines elements to single value.

```java
int sum = List.of(1, 2, 3, 4).stream()
    .reduce(0, Integer::sum);

Optional<String> longest = names.stream()
    .reduce((a, b) -> a.length() >= b.length() ? a : b);
```

---

# 7. Other Useful Operations

| Operation | Type | Example |
|-----------|------|---------|
| `sorted` | intermediate | `.sorted()` |
| `distinct` | intermediate | `.distinct()` |
| `limit` | intermediate | `.limit(5)` |
| `skip` | intermediate | `.skip(2)` |
| `peek` | intermediate | debug `.peek(System.out::println)` |
| `count` | terminal | `.count()` |
| `anyMatch` | terminal | `.anyMatch(n -> n.startsWith("A"))` |
| `findFirst` | terminal | `.findFirst()` |

---

# 8. Primitive Streams

Avoid boxing overhead.

```java
int total = List.of(1, 2, 3).stream()
    .mapToInt(Integer::intValue)
    .sum();

IntStream.rangeClosed(1, 5).forEach(System.out::println);
```

---

# 9. Parallel Streams

```java
long count = hugeList.parallelStream()
    .filter(x -> x % 2 == 0)
    .count();
```

Use when:

- Large data
- CPU-heavy, independent operations

Avoid when:

- Order matters
- Shared mutable state
- Small collections (overhead > benefit)

---

# 10. Interview Example

Employees with salary > 50k, sorted by name, names only:

```java
List<String> names = employees.stream()
    .filter(e -> e.getSalary() > 50_000)
    .sorted(Comparator.comparing(Employee::getName))
    .map(Employee::getName)
    .collect(Collectors.toList());
```

---

# Common Interview Questions

## Q1. Stream vs Collection?

Collection holds data; Stream is pipeline for computation on data source.

---

## Q2. Can you reuse a Stream?

No. Streams are single-use; second terminal op throws `IllegalStateException`.

---

## Q3. Lazy evaluation?

Intermediate ops are lazy; executed only when terminal op runs.

---

## Q4. map vs flatMap?

`map` = 1-to-1 transform. `flatMap` = 1-to-many then flatten.

---

## Q5. Stream vs for-loop?

Streams: readable, composable, parallelizable.  
Loops: simpler debugging, better for complex control flow.

---

# One-Line Revision

```text
filter/select → map/transform → collect/reduce terminal result.
Streams are lazy, single-use pipelines.
```

---

*End of Day 11 Java*
