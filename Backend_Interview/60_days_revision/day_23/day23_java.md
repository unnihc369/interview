# Day 23 - WeakHashMap & Heap vs Stack

---

# 1. WeakHashMap

---

# Definition

`WeakHashMap` is a `Map` implementation where keys are held by **weak references**. When a key object has no strong references, it can be **garbage collected**, and the entry is removed automatically.

```text
Key dies → entry removed from map (eventually)
```

---

# Weak Reference Recap

| Reference Type | GC Behavior |
|----------------|-------------|
| Strong | Never collected while reachable |
| Soft | Collected when memory pressure (caches) |
| Weak | Collected at next GC if only weak refs |
| Phantom | After finalization, for cleanup queues |

---

# Example

```java
import java.util.WeakHashMap;

public class WeakHashMapDemo {
    public static void main(String[] args) throws InterruptedException {
        WeakHashMap<Object, String> map = new WeakHashMap<>();
        Object key = new Object();
        map.put(key, "value");

        System.out.println(map.size()); // 1

        key = null; // remove strong reference
        System.gc();
        Thread.sleep(100);

        System.out.println(map.size()); // likely 0
    }
}
```

---

# Internal Behavior

- Uses `WeakReference` wrappers for keys
- Values are held by **strong references** until key is collected
- When key is GC'd, entry becomes "expunged" and removed on next map operation
- **Not thread-safe** — use `Collections.synchronizedMap` or `ConcurrentHashMap` if needed

---

# Use Cases

| Use Case | Why WeakHashMap |
|----------|-----------------|
| **Canonical mapping cache** | Don't prevent GC of unused keys |
| **Listener registries** | Auto-remove listeners when owner GC'd |
| **Metadata attached to objects** | Avoid memory leaks in frameworks |
| **ClassLoader caches** | Old class loaders can be collected |

---

# Real-World Example — Metadata Cache

```java
public class ObjectMetadataCache {
    private final WeakHashMap<Object, Map<String, Object>> cache = new WeakHashMap<>();

    public void putMetadata(Object obj, String key, Object value) {
        cache.computeIfAbsent(obj, k -> new HashMap<>()).put(key, value);
    }

    public Object getMetadata(Object obj, String key) {
        Map<String, Object> meta = cache.get(obj);
        return meta != null ? meta.get(key) : null;
    }
}
```

When `obj` is no longer used elsewhere, metadata disappears — no leak.

---

# WeakHashMap vs HashMap vs ConcurrentHashMap

| Map | Key retention | Thread-safe | Use when |
|-----|---------------|-------------|----------|
| HashMap | Strong | No | General purpose |
| WeakHashMap | Weak keys | No | Cache keyed by object identity |
| ConcurrentHashMap | Strong | Yes | High concurrency |

---

# Interview Trap

```java
String key = new String("temp");
map.put(key, "data");
key = null;
```

String may still live in **string pool** if interned — weak key might not be collected. Use `new Object()` as key in demos.

---

# 2. Heap vs Stack (JVM Memory)

---

# Overview

```text
┌─────────────────────────────────────┐
│              HEAP                    │
│  Objects, arrays, static/class data  │
│  Shared across threads               │
│  GC managed                          │
├─────────────────────────────────────┤
│         STACK (per thread)         │
│  Method frames, local primitives     │
│  References to heap objects          │
│  LIFO, fast allocation               │
└─────────────────────────────────────┘
```

---

# Stack

| Property | Detail |
|----------|--------|
| Scope | Per thread |
| Stores | Local variables, method parameters, return addresses |
| Lifetime | Method call duration |
| Size | Smaller, fixed (e.g. `-Xss1m`) |
| Overflow | `StackOverflowError` (deep recursion) |
| GC | Not garbage collected |

```java
void method() {
    int x = 10;           // primitive on stack
    User u = new User();  // reference on stack, object on heap
}
```

---

# Heap

| Property | Detail |
|----------|--------|
| Scope | All threads |
| Stores | `new` objects, arrays, instance fields |
| Lifetime | Until no references (GC) |
| Size | Larger (`-Xmx`, `-Xms`) |
| Overflow | `OutOfMemoryError` |
| GC | Generational GC (Young/Old) |

---

# Where Things Live

```java
class Demo {
    static int staticVar = 1;     // Metaspace (class metadata area)
    int instanceVar = 2;        // Heap (inside object)

    void run() {
        int local = 3;            // Stack
        String s = new String();  // ref stack, object heap (or pool)
    }
}
```

---

# Stack Frame Contents

Each method invocation creates a frame:

```text
- Local variable table
- Operand stack (bytecode execution)
- Reference to constant pool
- Return address
```

---

# Heap Generations (G1GC era)

```text
Young Gen: Eden + Survivor (S0, S1) — short-lived objects
Old Gen: long-lived objects promoted after GC cycles
```

Most objects die young → optimized for fast minor GC.

---

# Stack vs Heap — Interview Comparison

| | Stack | Heap |
|---|-------|------|
| Speed | Faster allocation | Slower (GC overhead) |
| Size | Limited | Large |
| Thread safety | Thread-local | Shared — needs sync |
| Failure | StackOverflowError | OutOfMemoryError |
| Contains | Primitives + refs | Actual objects |

---

# Common Interview Questions

## Q1. Are objects stored on stack in Java?

No. Objects are always on heap (except JVM optimizations like escape analysis / scalar replacement — mention as advanced).

## Q2. What is escape analysis?

If compiler proves object doesn't escape method, it may allocate on stack or eliminate allocation entirely.

## Q3. Why WeakHashMap for caches?

Prevents cache from keeping unused objects alive — avoids memory leaks.

## Q4. Difference between WeakHashMap and IdentityHashMap?

IdentityHashMap uses `==` for keys; WeakHashMap uses weak equality and weak key references.

## Q5. What causes StackOverflowError?

Infinite or very deep recursion, excessive stack frame size.

## Q6. heap vs stack overflow in production?

Heap OOM: large collections, memory leaks. Stack overflow: buggy recursive algorithms.

---

# One-Line Revision

```text
WeakHashMap = weak keys auto-GC'd; Stack = per-thread frames; Heap = shared objects under GC.
```

---

*End of Day 23 Java*
