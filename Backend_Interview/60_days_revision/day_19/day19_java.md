# Day 19 — Java Garbage Collection Basics

**Topics:** JVM Memory · GC Process · Collectors · Interview Questions

---

# 1. JVM Memory Areas

```text
JVM Memory
├── Heap (shared)
│     ├── Young Generation
│     │     ├── Eden
│     │     ├── Survivor S0
│     │     └── Survivor S1
│     └── Old Generation (Tenured)
├── Metaspace (class metadata, Java 8+)
├── Stack (per thread)
└── PC Register, Native Method Stack
```

---

# 2. Object Lifecycle

```text
new Object()
     ↓
Eden Space (Young Gen)
     ↓ Minor GC (survivors → S0/S1)
     ↓ (after ~15 cycles by default)
Old Generation
     ↓ Major/Full GC
Removed (no references)
```

---

# 3. How GC Works — Mark and Sweep

## Mark Phase

Traverse from GC Roots, mark all reachable objects.

**GC Roots:**
- Local variables on stack
- Static fields
- Active threads
- JNI references

## Sweep Phase

Remove unmarked (unreachable) objects.

## Compact Phase (optional)

Defragment heap to reduce fragmentation.

---

# 4. Generational Hypothesis

Most objects die young. GC optimizes for this:

| Generation | Collection | Frequency |
|------------|------------|-----------|
| Young (Minor GC) | Fast, frequent | Every Eden fill |
| Old (Major/Full GC) | Slow, infrequent | When Old Gen full |

---

# 5. Minor GC Flow

```text
1. New objects allocated in Eden
2. Eden fills up → Minor GC triggered
3. Live objects moved to Survivor space
4. Age counter incremented
5. Long-lived objects promoted to Old Gen
6. Eden cleared
```

---

# 6. Garbage Collectors (Interview Table)

| Collector | Type | Use Case |
|-----------|------|----------|
| Serial GC | Single thread | Small apps, client mode |
| Parallel GC | Multi-thread young + old | Throughput-focused (default Java 8) |
| CMS | Concurrent old gen | Low pause (deprecated Java 14) |
| G1 GC | Region-based, concurrent | Default Java 9+, balanced |
| ZGC | Ultra-low pause | Large heaps, Java 15+ |
| Shenandoah | Low pause concurrent | Alternative to ZGC |

---

# 7. G1 GC (Default Modern Collector)

Divides heap into equal-sized **regions**.

```text
- Young collections: stop-the-world, fast
- Mixed collections: reclaim Old Gen regions with most garbage
- Target pause time configurable (-XX:MaxGCPauseMillis=200)
```

---

# 8. When is Object Eligible for GC?

When **no chain of references** exists from any GC Root.

```java
Object obj = new Object();
obj = null;  // eligible for GC (no references)

Object a = new Object();
Object b = a;
a = null;    // NOT eligible — b still references it
```

---

# 9. finalize() Method

```java
@Override
protected void finalize() throws Throwable {
    // Called before GC (unreliable, deprecated Java 9)
}
```

**Do not rely on finalize.** Use try-with-resources, Cleaner, or explicit close methods.

---

# 10. Memory Leaks in Java

Java has GC but leaks still happen via **unintended references**:

| Leak Source | Example |
|-------------|---------|
| Static collections | Cache never cleared |
| Listeners not removed | Event listener holds reference |
| ThreadLocal not cleared | Thread pool reuses threads |
| Inner class holding outer | Non-static inner class |

```java
// Memory leak example
public class Cache {
    private static Map<String, Object> cache = new HashMap<>();
    // entries never removed → grows forever
}
```

---

# 11. Useful JVM Flags

```bash
# Heap size
-Xms512m -Xmx2g

# GC logging (Java 9+)
-Xlog:gc*

# Use G1
-XX:+UseG1GC

# Heap dump on OOM
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/heapdump.hprof
```

---

# 12. Monitoring GC

Tools:
- `jstat -gc <pid> 1000` — GC stats every second
- VisualVM / JMC (Java Mission Control)
- Micrometer + Prometheus (production)

Key metrics:
- GC pause time
- GC frequency
- Heap usage trend

---

# Interview Questions

## Stack vs Heap?

| Stack | Heap |
|-------|------|
| Method calls, local vars | Objects, arrays |
| Per thread | Shared across threads |
| LIFO, fast | GC managed, slower allocation |
| Fixed size | Configurable (-Xmx) |

---

## Minor GC vs Full GC?

| Minor GC | Full GC |
|----------|---------|
| Young generation only | Entire heap (Young + Old + Metaspace) |
| Fast (milliseconds) | Slow (can be seconds) |
| Frequent | Rare, indicates memory pressure |

---

## How to reduce GC pauses?

- Increase heap if legitimately needed
- Use G1/ZGC with pause time targets
- Reduce object allocation (object pooling, reuse)
- Fix memory leaks
- Tune `-XX:MaxGCPauseMillis`

---

## OutOfMemoryError types?

| Error | Cause |
|-------|-------|
| Java heap space | Heap exhausted |
| Metaspace | Too many classes loaded |
| GC overhead limit exceeded | GC spending >98% time, recovering <2% heap |
| Unable to create native thread | Too many threads |

---

## Strong vs Weak vs Soft vs Phantom references?

| Type | GC Behavior | Use Case |
|------|-------------|----------|
| Strong | Never collected while referenced | Normal references |
| Soft | Collected when memory needed | Caches |
| Weak | Collected at next GC | WeakHashMap |
| Phantom | After finalization, before memory reclaimed | Cleanup actions |

---

# Quick Revision

```text
Young Gen (Eden + Survivor) → Minor GC → fast
Old Gen → Major/Full GC → slow
G1 GC → default modern collector, region-based
GC Roots → stack locals, statics, threads
Memory leak → unintended strong references prevent GC
-Xms/-Xmx → heap sizing
```

---

*End of Day 19 GC Basics*
