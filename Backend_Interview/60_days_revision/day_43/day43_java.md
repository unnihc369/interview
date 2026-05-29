# Day 43 — Deadlock, Livelock & Starvation

**Topics:** Concurrency failure modes · Detection · Prevention · Interview Q&A

---

# 1. Deadlock

## Definition

Two or more threads are **blocked forever**, each waiting for a resource held by another.

```text
Thread A: holds Lock-1, waits for Lock-2
Thread B: holds Lock-2, waits for Lock-1
→ Circular wait → neither progresses
```

---

# Classic Example

```java
Object lock1 = new Object();
Object lock2 = new Object();

// Thread 1
synchronized (lock1) {
    Thread.sleep(100);
    synchronized (lock2) { /* work */ }
}

// Thread 2
synchronized (lock2) {
    Thread.sleep(100);
    synchronized (lock1) { /* work */ }
}
```

---

# Coffman Conditions (All 4 required for deadlock)

| Condition | Meaning |
|-----------|---------|
| Mutual Exclusion | Resource used by one thread at a time |
| Hold and Wait | Thread holds resource while waiting for another |
| No Preemption | Resources can't be forcibly taken |
| Circular Wait | Cycle in wait-for graph |

**Prevention:** break at least one condition.

---

# Prevention Strategies

| Strategy | How |
|----------|-----|
| Lock ordering | Always acquire locks in same global order (lock1 → lock2) |
| tryLock with timeout | `ReentrantLock.tryLock(timeout)` — back off and retry |
| Reduce lock scope | Hold locks for minimum time |
| Lock-free structures | `ConcurrentHashMap`, atomics |

```java
// Lock ordering fix
synchronized (lock1) {
    synchronized (lock2) { /* safe */ }
}
// Both threads acquire lock1 first, then lock2
```

---

# Detection

- **Thread dump:** `jstack <pid>` or `jcmd <pid> Thread.print`
- Look for: `Found one Java-level deadlock`
- **JMC / VisualVM:** Threads tab shows blocked threads and lock owners

---

# 2. Livelock

## Definition

Threads are **not blocked** but continuously change state in response to each other — **no forward progress**, like two people in a hallway stepping aside the same way repeatedly.

```text
Thread A: "You go first" → steps aside
Thread B: "You go first" → steps aside
→ Both keep yielding, neither enters critical section
```

---

# Example

```java
class LivelockDemo {
    static class Spoon {
        private Diner owner;
        synchronized void use() { owner.eat(); }
        synchronized void setOwner(Diner d) { owner = d; }
    }

    static class Diner implements Runnable {
        private Spoon spoon;
        public void run() {
            while (true) {
                if (spoon.owner != this) {
                    spoon.use(); // got it
                    break;
                }
                // Polite yield — causes livelock if both diners do this
                Diner other = (spoon.owner == dinerA) ? dinerB : dinerA;
                spoon.setOwner(other); // "After you!"
            }
        }
    }
}
```

---

# Livelock vs Deadlock

| | Deadlock | Livelock |
|---|----------|----------|
| Thread state | BLOCKED | RUNNABLE (active) |
| CPU usage | Low (waiting) | Can be high (busy yielding) |
| Progress | None | None |
| Fix | Lock ordering, timeouts | Randomized backoff, priority |

---

# Fix: Randomized Backoff

```java
if (spoon.owner != this) {
    Thread.sleep(ThreadLocalRandom.current().nextInt(1, 10));
    continue;
}
```

---

# 3. Starvation

## Definition

A thread is **perpetually denied** access to a resource because other threads monopolize it.

```text
Low-priority thread never gets CPU
Non-fair lock → one thread always wins tryLock
```

---

# Causes

- **Non-fair `ReentrantLock`** — barging allowed
- **Thread priority misuse** — high-priority threads dominate
- **Unbounded thread pool** — some tasks wait forever in queue

---

# Prevention

```java
// Fair lock — FIFO acquisition order
ReentrantLock fairLock = new ReentrantLock(true);

// Fair thread pool
new ThreadPoolExecutor(
    core, max, keepAlive, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(),  // unbounded queue can starve if producers flood
    new ThreadPoolExecutor.CallerRunsPolicy()  // backpressure
);
```

- Avoid relying on thread priorities in application code
- Use fair locks when starvation is a concern (slight throughput cost)

---

# 4. Comparison Table

| Issue | Threads blocked? | CPU active? | Typical fix |
|-------|------------------|-------------|-------------|
| Deadlock | Yes | No | Lock ordering, tryLock timeout |
| Livelock | No | Yes | Random backoff, state machine |
| Starvation | Sometimes (waiting) | Others active | Fair locks, fair scheduling |

---

# 5. Production Debugging Checklist

1. Capture thread dump under load
2. Identify lock holders and waiters
3. Check lock acquisition order in code review
4. Prefer `java.util.concurrent` over raw `synchronized` where appropriate
5. Load test with realistic concurrency

---

# Interview Questions

## Q1. How do you prevent deadlock in a transfer service (account A → B)?

Always lock accounts in **consistent order** (lower account ID first). Or use database row-level locks with `SELECT FOR UPDATE` in sorted ID order.

## Q2. Deadlock vs livelock in one sentence?

Deadlock: threads blocked waiting for locks. Livelock: threads actively responding but making no progress.

## Q3. Does `synchronized` cause deadlock?

It can, if two threads acquire the same locks in opposite order. The keyword itself doesn't prevent circular wait.

## Q4. Fair vs non-fair ReentrantLock?

Non-fair (default): higher throughput, possible starvation. Fair: FIFO queue for lock, prevents starvation, lower throughput.

## Q5. Can deadlock happen without locks?

Yes — e.g. `Thread.join()` cycles, or waiting on futures that depend on each other.

---

# One-Line Revision

```text
Deadlock = circular lock wait (fix: ordering/tryLock); Livelock = busy but no progress (fix: backoff); Starvation = never scheduled (fix: fair locks/queues).
```

---

*End of Day 43 Java*
