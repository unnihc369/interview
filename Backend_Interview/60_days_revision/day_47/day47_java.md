# Day 47 — Concurrency Collections Review

**Topics:** ConcurrentHashMap · CopyOnWriteArrayList · BlockingQueue · ConcurrentLinkedQueue · Interview Q&A

---

# 1. Why Concurrent Collections?

```text
HashMap + synchronized     → coarse lock, poor read scalability
Collections.synchronized*  → single lock on entire collection
java.util.concurrent       → fine-grained locking / lock-free reads
```

Use when multiple threads read/write shared data structures.

---

# 2. ConcurrentHashMap

## Key Properties (Java 8+)

- **No global lock** for reads
- Writes lock single bin (bucket)
- `size()` is approximate under concurrency
- **No null keys or values** (unlike HashMap)

```java
ConcurrentHashMap<String, Order> cache = new ConcurrentHashMap<>();

cache.put("order-1", order);
cache.putIfAbsent("order-1", other);  // atomic
cache.compute("order-1", (k, v) -> v.withStatus(PAID));

// Thread-safe iteration — weakly consistent (no ConcurrentModificationException)
for (Map.Entry<String, Order> e : cache.entrySet()) {
    process(e.getValue());
}
```

---

# Atomic Compound Operations

```java
// Increment counter per key
cache.merge("visits", 1L, Long::sum);

// Read-modify-write atomically
cache.computeIfPresent("stock", (k, v) -> v - 1);

// Bulk parallel operation (Java 8+)
cache.forEach(4, (k, v) -> process(k, v));  // parallelism threshold 4
```

---

# 3. CopyOnWriteArrayList / CopyOnWriteArraySet

## Pattern

Copy entire backing array on **every write**. Reads are lock-free and fast.

```java
CopyOnWriteArrayList<Listener> listeners = new CopyOnWriteArrayList<>();

listeners.add(new EmailListener());   // copies array
for (Listener l : listeners) {        // safe iteration, no lock
    l.onEvent(event);
}
```

## When to Use

- **Read-heavy**, write-rare (listener lists, config snapshots)
- Iteration must not throw CME
- **Avoid** frequent writes — O(n) copy per add/remove

---

# 4. BlockingQueue Family

## Implementations

| Class | Backing | Bounded | Notes |
|-------|---------|---------|-------|
| `ArrayBlockingQueue` | Array | Yes | Fixed capacity, optional fairness |
| `LinkedBlockingQueue` | Linked nodes | Optional | Default unbounded (Integer.MAX) |
| `PriorityBlockingQueue` | Heap | No | Ordered by priority |
| `DelayQueue` | Priority heap | No | Elements available after delay |
| `SynchronousQueue` | None | Zero capacity | Handoff producer→consumer |

```java
BlockingQueue<Task> queue = new ArrayBlockingQueue<>(1000);

// Producer
queue.put(task);           // blocks if full
queue.offer(task, 1, SECONDS);  // timeout

// Consumer
Task t = queue.take();     // blocks if empty
Task t2 = queue.poll(500, MILLISECONDS);
```

---

# 5. Producer-Consumer Pattern

```java
public class OrderProcessor {
    private final BlockingQueue<Order> queue = new LinkedBlockingQueue<>(500);
    private volatile boolean running = true;

    public void submit(Order order) throws InterruptedException {
        queue.put(order);
    }

    public void startWorkers(int n) {
        for (int i = 0; i < n; i++) {
            Thread.ofVirtual().start(() -> {
                while (running || !queue.isEmpty()) {
                    try {
                        Order order = queue.poll(1, TimeUnit.SECONDS);
                        if (order != null) process(order);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            });
        }
    }
}
```

---

# 6. ConcurrentLinkedQueue & ConcurrentLinkedDeque

**Lock-free** (CAS-based) unbounded queue/deque.

```java
ConcurrentLinkedQueue<Event> events = new ConcurrentLinkedQueue<>();
events.offer(event);   // non-blocking
Event e = events.poll(); // non-blocking

// No size() in O(1) — size() is O(n)
```

Use for high-throughput, non-blocking event pipelines.

---

# 7. ConcurrentSkipListMap / ConcurrentSkipListSet

Sorted concurrent map/set — `O(log n)` operations.

```java
ConcurrentNavigableMap<Long, Metric> metrics = new ConcurrentSkipListMap<>();
metrics.subMap(startTime, true, endTime, true).forEach((k, v) -> aggregate(v));
```

Use when you need sorted order under concurrency (time-series windows).

---

# 8. Thread-Safe Counters

```java
LongAdder adder = new LongAdder();  // preferred over AtomicLong under contention
adder.increment();
long sum = adder.sum();

AtomicInteger counter = new AtomicInteger();
counter.incrementAndGet();
```

---

# 9. Choosing the Right Collection

```text
Shared cache (K→V)           → ConcurrentHashMap
Listener list (read-heavy)   → CopyOnWriteArrayList
Task queue (producer-consumer)→ ArrayBlockingQueue / LinkedBlockingQueue
Lock-free event bus          → ConcurrentLinkedQueue
Sorted concurrent map        → ConcurrentSkipListMap
High-contention counter      → LongAdder
```

---

# Interview Questions

## Q1. ConcurrentHashMap vs synchronized HashMap?

CHM: segmented/bin locking, concurrent reads. Sync HashMap: one lock, all threads serialize.

## Q2. Why no null in ConcurrentHashMap?

Ambiguity: `get(key)` returns null — is key missing or value null? Avoid NPE in concurrent context.

## Q3. CopyOnWrite — when NOT to use?

Frequent writes (large lists) — copying becomes expensive.

## Q4. ArrayBlockingQueue vs LinkedBlockingQueue?

Array: fixed size, one lock (or two in some impl). Linked: optionally unbounded, separate put/take locks — higher throughput under mixed producers/consumers.

## Q5. BlockingQueue in ThreadPoolExecutor?

Worker threads `take()` from work queue. `SynchronousQueue` for cached pool (direct handoff). `LinkedBlockingQueue` for fixed pool.

---

# One-Line Revision

```text
ConcurrentHashMap = fine-grained map; CopyOnWrite = read-heavy lists; BlockingQueue = producer-consumer; ConcurrentLinkedQueue = lock-free FIFO; LongAdder = hot counters.
```

---

*End of Day 47 Java*
