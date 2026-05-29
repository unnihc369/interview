# Day 10 — ConcurrentHashMap & AtomicInteger

**Topics:** Concurrent collections · CAS · Thread-safe counters · Interview Q&A

---

# 1. Problem with HashMap in Multi-threading

`HashMap` is not thread-safe. Concurrent updates can cause infinite loops (pre-Java 8) or lost updates.

```java
Map<String, Integer> map = new HashMap<>(); // unsafe shared
```

Solutions:

- `Collections.synchronizedMap(new HashMap<>())` — coarse lock, poor throughput
- `ConcurrentHashMap` — fine-grained locking / CAS segments

---

# 2. ConcurrentHashMap

## Basic Usage

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

map.put("a", 1);
map.putIfAbsent("a", 2); // only if key missing
map.compute("a", (k, v) -> v + 1);
map.merge("b", 1, Integer::sum);
```

## Thread-safe iteration

```java
map.forEach((k, v) -> System.out.println(k + "=" + v));
```

Weakly consistent iterator — reflects state at some point, no `ConcurrentModificationException`.

---

# Internal Idea (Interview)

Java 8+:

- Array of bins (like HashMap)
- Synchronized on first node of bin OR tree bin for updates
- Reads generally lock-free
- Size computed with `baseCount` + `CounterCell[]` (striped counting)

```text
Key hash → bin index → lock only that bin → high concurrency
```

---

# Important Methods

| Method | Use |
|--------|-----|
| `putIfAbsent` | Idempotent insert |
| `compute` | Atomic read-modify-write |
| `merge` | Add/aggregate safely |
| `replace(key, old, new)` | Compare-and-swap style |

---

# 3. AtomicInteger

Part of `java.util.concurrent.atomic` — lock-free atomic operations using CAS (Compare-And-Swap).

```java
AtomicInteger counter = new AtomicInteger(0);

counter.incrementAndGet();
counter.addAndGet(5);
counter.compareAndSet(10, 20); // if current==10, set 20
int val = counter.get();
```

---

# Why AtomicInteger?

```java
// BAD
int count = 0;
count++; // not atomic

// GOOD
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();
```

Operations are atomic without `synchronized` — better under contention for simple counters.

---

# Other Atomic Classes

| Class | Use |
|-------|-----|
| `AtomicLong` | Long counters |
| `AtomicBoolean` | Flags |
| `AtomicReference<V>` | Reference CAS |
| `LongAdder` | High-contention sum (better than AtomicLong) |

```java
LongAdder adder = new LongAdder();
adder.increment();
long sum = adder.sum();
```

---

# 4. Practical Example — Rate Limiter Counter

```java
@Component
public class RequestCounter {
    private final ConcurrentHashMap<String, AtomicInteger> counts = new ConcurrentHashMap<>();

    public void record(String userId) {
        counts.computeIfAbsent(userId, k -> new AtomicInteger(0))
              .incrementAndGet();
    }

    public int getCount(String userId) {
        AtomicInteger c = counts.get(userId);
        return c == null ? 0 : c.get();
    }
}
```

---

# synchronized vs ConcurrentHashMap vs Atomic

| Tool | Best for |
|------|----------|
| synchronized | Critical sections, complex invariants |
| ConcurrentHashMap | Shared concurrent map |
| Atomic* | Single variable counters/flags |

---

# Common Interview Questions

## Q1. ConcurrentHashMap vs synchronized HashMap?

ConcurrentHashMap allows concurrent reads and limited concurrent writes; synchronized map locks entire structure.

---

## Q2. Does ConcurrentHashMap allow null keys/values?

No (unlike HashMap). Avoid ambiguity in concurrent context.

---

## Q3. What is CAS?

CPU instruction: compare memory with expected; if equal, swap. Retry on failure — optimistic concurrency.

---

## Q4. AtomicInteger vs volatile int?

`volatile` only guarantees visibility, not `i++` atomicity. Use Atomic for compound updates.

---

## Q5. When LongAdder over AtomicLong?

High write contention — LongAdder stripes across cells, reduces CAS failures.

---

# One-Line Revision

```text
ConcurrentHashMap = thread-safe map with bin-level concurrency.
AtomicInteger = lock-free atomic counter via CAS.
```

---

*End of Day 10 Java*
