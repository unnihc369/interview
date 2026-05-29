# Day 8 — Java Multithreading (Thread, Runnable, synchronized)

**Topics:** Thread lifecycle · Runnable vs Thread · synchronized · wait/notify · Interview Q&A

---

# 1. What is a Thread?

A thread is the smallest unit of execution within a process. Multiple threads share the same heap but have separate stack memory.

```text
Process
  ├── Thread 1 (main)
  ├── Thread 2
  └── Thread 3
```

---

# 2. Creating Threads

## Way 1: Extend `Thread`

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running: " + Thread.currentThread().getName());
    }
}

MyThread t = new MyThread();
t.start(); // do NOT call run() directly for new thread
```

## Way 2: Implement `Runnable` (Preferred)

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Task on " + Thread.currentThread().getName());
    }
}

Thread t = new Thread(new MyTask());
t.start();
```

## Way 3: Lambda

```java
Thread t = new Thread(() -> System.out.println("Lambda thread"));
t.start();
```

---

# Thread vs Runnable (Interview)

| Extend Thread | Implement Runnable |
|---------------|-------------------|
| IS-A Thread | HAS-A Thread |
| Cannot extend another class | Can extend other class |
| Less flexible | Preferred in production |

---

# 3. Thread Lifecycle

```text
NEW → RUNNABLE → RUNNING → TERMINATED
         ↓           ↓
      BLOCKED    WAITING / TIMED_WAITING
```

| State | Meaning |
|-------|---------|
| NEW | Created, not started |
| RUNNABLE | Ready or running |
| BLOCKED | Waiting for monitor lock |
| WAITING | wait(), join() without timeout |
| TIMED_WAITING | sleep(), wait(timeout) |
| TERMINATED | run() completed |

---

# 4. Race Condition

When multiple threads read/write shared mutable data without synchronization, outcome is unpredictable.

```java
class Counter {
    int count = 0;

    void increment() {
        count++; // NOT atomic: read-modify-write
    }
}
```

Two threads calling `increment()` may both read `count=5` and write `6` instead of `7`.

---

# 5. synchronized Keyword

## synchronized method

```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

Lock object = `this` (instance) or class object (static method).

## synchronized block

```java
public void transfer(Account from, Account to, int amount) {
    synchronized (from) {
        synchronized (to) {
            from.debit(amount);
            to.credit(amount);
        }
    }
}
```

Finer-grained locking; always lock in consistent order to avoid deadlock.

---

# How synchronized Works (Deep)

- Every object has an intrinsic lock (monitor).
- Only one thread can hold the monitor at a time.
- Other threads block until lock is released.
- JVM uses object header mark word for lock state (biased → lightweight → heavyweight).

---

# 6. wait(), notify(), notifyAll()

Used for thread coordination inside synchronized block.

```java
class BoundedBuffer {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int capacity = 5;

    public synchronized void produce(int item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait(); // release lock, wait until notified
        }
        queue.offer(item);
        notifyAll();
    }

    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();
        }
        int item = queue.poll();
        notifyAll();
        return item;
    }
}
```

Rules:

- Must be called inside `synchronized` method/block on same monitor.
- `wait()` releases lock; `sleep()` does not release lock.

---

# 7. sleep() vs wait()

| sleep() | wait() |
|---------|--------|
| `Thread.sleep(ms)` | `object.wait()` |
| Does not release lock | Releases lock |
| Static method on Thread | Object method |
| Timed waiting | Needs notify to wake |

---

# 8. join()

Wait for another thread to finish.

```java
Thread worker = new Thread(() -> { /* work */ });
worker.start();
worker.join(); // main waits until worker completes
System.out.println("Worker done");
```

---

# 9. volatile (Related Interview Topic)

`volatile` ensures visibility (happens-before) but does NOT make compound operations atomic.

```java
private volatile boolean running = true;
```

Use for flags; use `synchronized` or `Atomic*` for counters.

---

# 10. Practical Example — Bank Account

```java
class BankAccount {
    private int balance = 0;

    public synchronized void deposit(int amount) {
        balance += amount;
    }

    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }

    public synchronized int getBalance() {
        return balance;
    }
}
```

---

# Common Interview Questions

## Q1. Why not call `run()` directly?

`start()` creates a new OS thread and JVM calls `run()` on that thread. Calling `run()` directly runs on current thread — no parallelism.

---

## Q2. Can we start a thread twice?

No. `IllegalThreadStateException` if `start()` called twice on same `Thread` instance.

---

## Q3. Difference between process and thread?

| Process | Thread |
|---------|--------|
| Separate memory space | Shares heap |
| Heavyweight | Lightweight |
| IPC costly | Shared memory (needs sync) |

---

## Q4. What is deadlock?

Thread A holds lock1, waits for lock2. Thread B holds lock2, waits for lock1. Prevention: lock ordering, timeouts, avoid nested locks.

---

## Q5. Is `synchronized` reentrant?

Yes. Same thread can acquire the same monitor multiple times (e.g., synchronized method calling another synchronized method on same object).

---

## Q6. synchronized vs ReentrantLock?

| synchronized | ReentrantLock |
|--------------|---------------|
| Built-in keyword | Explicit lock API |
| Auto release | Manual unlock in finally |
| No tryLock/fairness options | tryLock, fair lock, conditions |

---

# 7. volatile Keyword

Guarantees **visibility** across threads — writes by one thread are immediately visible to others.

```java
private volatile boolean running = true;

public void stop() { running = false; }  // writer thread

public void run() {
    while (running) { /* work */ }       // reader thread sees update
}
```

## volatile vs synchronized

| volatile | synchronized |
|----------|--------------|
| Visibility only | Visibility + atomicity |
| No mutual exclusion | Mutual exclusion |
| Lightweight | Heavier |
| No compound ops (i++) | Safe for compound ops |

**Happens-before rule:** Write to volatile → subsequent read sees new value.

**Not enough for:** `count++` — use `AtomicInteger` or synchronized block.

---

# 8. ThreadLocal

Each thread has its **own copy** of the variable — no sharing, no locking.

```java
private static final ThreadLocal<UserContext> ctx = ThreadLocal.withInitial(UserContext::new);

// In request thread:
ctx.set(new UserContext(userId));
try {
    service.process(); // reads ctx.get() anywhere in same thread
} finally {
    ctx.remove(); // CRITICAL in thread pools — prevent memory leak
}
```

**Use cases:** User context, request ID (MDC logging), SimpleDateFormat (not thread-safe).

**Interview trap:** Forgetting `remove()` in pooled threads (Tomcat worker threads) → memory leak + wrong data on next request.

---

# One-Line Revision

```text
Runnable preferred over extending Thread.
synchronized = mutual exclusion on shared mutable state.
volatile = visibility across threads (not atomicity).
ThreadLocal = per-thread copy; always remove() in finally with thread pools.
wait/notify = producer-consumer coordination inside synchronized block.
```

---

*End of Day 8 Java*
