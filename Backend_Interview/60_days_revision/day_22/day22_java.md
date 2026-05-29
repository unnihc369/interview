# Day 22 - CountDownLatch & CyclicBarrier

---

# 1. CountDownLatch

---

# Definition

`CountDownLatch` is a synchronization aid that allows one or more threads to **wait until a set of operations in other threads completes**.

```text
CountDown from N → 0, then waiting threads proceed.
```

One-time use only — **cannot be reset**.

---

# Key Methods

| Method | Purpose |
|--------|---------|
| `CountDownLatch(int count)` | Initialize counter |
| `await()` | Block until count reaches 0 |
| `await(timeout, unit)` | Block with timeout |
| `countDown()` | Decrement count by 1 |

---

# Use Case

**Main thread waits for worker threads to finish startup before serving traffic.**

```text
Service starts
   ↓
5 worker threads initialize DB/cache
   ↓
Each calls countDown()
   ↓
Main thread await() unblocks
   ↓
Application ready
```

---

# Example — Wait for Parallel Tasks

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        int workers = 3;
        CountDownLatch latch = new CountDownLatch(workers);

        for (int i = 1; i <= workers; i++) {
            int id = i;
            new Thread(() -> {
                try {
                    System.out.println("Worker " + id + " started");
                    Thread.sleep(1000 * id);
                    System.out.println("Worker " + id + " done");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    latch.countDown();
                }
            }).start();
        }

        latch.await(); // main waits here
        System.out.println("All workers finished. Proceed.");
    }
}
```

---

# Real-World Backend Example

Parallel health checks before accepting requests:

```java
CountDownLatch ready = new CountDownLatch(3);

executor.submit(() -> { checkDatabase(); ready.countDown(); });
executor.submit(() -> { checkRedis(); ready.countDown(); });
executor.submit(() -> { checkKafka(); ready.countDown(); });

ready.await(10, TimeUnit.SECONDS);
// start HTTP server
```

---

# 2. CyclicBarrier

---

# Definition

`CyclicBarrier` lets a set of threads **wait for each other to reach a common barrier point** before any can proceed.

```text
Reusable — can be reset and used again (cyclic).
```

---

# Key Methods

| Method | Purpose |
|--------|---------|
| `CyclicBarrier(int parties)` | Number of threads that must wait |
| `CyclicBarrier(int parties, Runnable barrierAction)` | Optional action when barrier tripped |
| `await()` | Wait at barrier |
| `reset()` | Break barrier and reset count |

---

# Use Case

**Parallel computation with phases** — all threads finish phase 1, then start phase 2 together.

```text
Thread1 ──┐
Thread2 ──┼── Barrier (wait for all)
Thread3 ──┘
          ↓
      Phase 2 starts together
```

---

# Example

```java
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        int threads = 3;
        CyclicBarrier barrier = new CyclicBarrier(threads, () ->
            System.out.println("All threads reached barrier. Next phase.")
        );

        for (int i = 1; i <= threads; i++) {
            int id = i;
            new Thread(() -> {
                try {
                    System.out.println("Thread " + id + " phase 1 done");
                    barrier.await(); // wait for others
                    System.out.println("Thread " + id + " phase 2 started");
                } catch (Exception e) {
                    Thread.currentThread().interrupt();
                }
            }).start();
        }
    }
}
```

---

# Real-World Backend Example

MapReduce-style batch processing:

```text
Workers load partition → barrier → aggregate → barrier → write output
```

Each worker processes its shard; barrier ensures all shards complete before aggregation.

---

# 3. CountDownLatch vs CyclicBarrier

| Feature | CountDownLatch | CyclicBarrier |
|---------|----------------|---------------|
| Purpose | One thread waits for others to finish | Threads wait for each other |
| Reusable | No | Yes |
| Count direction | Count down to 0 | Reset at barrier |
| Who waits | Usually 1 waiter, N workers | N threads wait for N |
| Action on trip | None built-in | Optional `barrierAction` |

---

# Memory Model Note (Advanced)

Both use `AbstractQueuedSynchronizer` (AQS) internally.

- `await()` releases lock and parks thread
- `countDown()` / barrier trip unparks waiting threads
- Happens-before relationship: actions before `countDown()` visible to thread after `await()`

---

# Common Interview Questions

## Q1. When to use CountDownLatch?

When one (or more) threads must wait for a fixed number of events to complete — startup, batch job completion, test coordination.

## Q2. When to use CyclicBarrier?

When multiple threads must synchronize at a checkpoint before continuing — phased parallel work, simulation steps.

## Q3. Can CountDownLatch be reused?

No. Once count hits 0, it stays 0. Create new instance for next use.

## Q4. What happens if a thread is interrupted during await()?

`InterruptedException` is thrown; latch state unchanged unless count already 0.

## Q5. Difference from Semaphore?

Semaphore controls **access to a resource pool** (permits). Latch/Barrier coordinate **completion/synchronization events**.

## Q6. CyclicBarrier vs Phaser?

`Phaser` is more flexible (dynamic party count, multiple phases, deregistration). Use Phaser for complex pipelines.

---

# One-Line Revision

```text
CountDownLatch = one-time wait for N events; CyclicBarrier = reusable sync point for N threads.
```

---

*End of Day 22 Java*
