# Day 9 — ExecutorService, Callable & Future

**Topics:** Thread pools · Callable vs Runnable · Future · CompletableFuture intro · Interview Q&A

---

# 1. Why ExecutorService?

Creating raw threads is expensive. `ExecutorService` manages a pool of reusable worker threads.

```text
Task Queue → Thread Pool → Workers execute tasks
```

Benefits:

- Controls concurrency level
- Reuses threads
- Supports task submission APIs

---

# 2. Creating ExecutorService

```java
ExecutorService executor = Executors.newFixedThreadPool(4);

// or with explicit config (preferred in production)
ExecutorService executor = new ThreadPoolExecutor(
    4, 8,
    60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100),
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

| Factory method | Behavior |
|----------------|----------|
| `newFixedThreadPool(n)` | Fixed n threads |
| `newCachedThreadPool()` | Unbounded threads, reuses idle |
| `newSingleThreadExecutor()` | One thread, sequential |
| `newScheduledThreadPool(n)` | Delayed/periodic tasks |

---

# 3. Runnable Submission

```java
executor.submit(() -> {
    System.out.println("Task on " + Thread.currentThread().getName());
});

executor.shutdown();
executor.awaitTermination(1, TimeUnit.MINUTES);
```

`shutdown()` — no new tasks; existing tasks finish.  
`shutdownNow()` — attempts to stop all tasks immediately.

---

# 4. Callable and Future

`Callable<V>` returns a value and can throw checked exceptions.

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};

Future<Integer> future = executor.submit(task);

Integer result = future.get(); // blocks until done
// future.get(2, TimeUnit.SECONDS); // with timeout
```

| Runnable | Callable |
|----------|----------|
| `void run()` | `V call()` |
| No return value | Returns value |
| Cannot throw checked ex | Can throw checked ex |

---

# 5. Future API

```java
future.isDone();
future.cancel(true); // may interrupt if running
future.get();        // blocking
future.get(5, TimeUnit.SECONDS);
```

Limitations of `Future`:

- Blocking `get()`
- No easy chaining/combining
- No exception handling pipeline

→ Use `CompletableFuture` for async composition (Java 8+).

---

# 6. CompletableFuture (Interview Favorite)

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchUser())
    .thenApply(user -> user.getName())
    .thenCompose(name -> CompletableFuture.supplyAsync(() -> fetchOrders(name)))
    .exceptionally(ex -> "default");

String result = future.join(); // or get()
```

Combine multiple:

```java
CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2, f3);
CompletableFuture<Object> any = CompletableFuture.anyOf(f1, f2);
```

---

# 7. invokeAll / invokeAny

```java
List<Callable<String>> tasks = List.of(
    () -> "A",
    () -> "B"
);

List<Future<String>> futures = executor.invokeAll(tasks);
for (Future<String> f : futures) {
    System.out.println(f.get());
}

String first = executor.invokeAny(tasks); // first completed
```

---

# 8. Practical Example — Parallel API Calls

```java
@Service
public class DashboardService {

    private final ExecutorService executor = Executors.newFixedThreadPool(3);

    public DashboardData load(Long userId) throws Exception {
        Future<User> userFuture = executor.submit(() -> userClient.get(userId));
        Future<List<Order>> ordersFuture = executor.submit(() -> orderClient.list(userId));
        Future<Wallet> walletFuture = executor.submit(() -> walletClient.get(userId));

        return new DashboardData(
            userFuture.get(),
            ordersFuture.get(),
            walletFuture.get()
        );
    }
}
```

Production: inject managed `TaskExecutor` from Spring (`@Async`).

---

# Thread Pool Sizing (Interview)

```text
CPU-bound:  threads ≈ number of cores
IO-bound:   threads ≈ cores * (1 + wait_time / compute_time)
```

---

# Common Interview Questions

## Q1. Difference between `submit()` and `execute()`?

| execute(Runnable) | submit(Callable/Runnable) |
|-------------------|---------------------------|
| void | Returns Future |
| Uncaught ex → handler | Wrapped in ExecutionException on get() |

---

## Q2. What happens if queue is full?

Depends on `RejectedExecutionHandler`:

- `AbortPolicy` — throw exception (default)
- `CallerRunsPolicy` — caller thread runs task
- `DiscardPolicy` — silently drop

---

## Q3. Why shutdown thread pool?

Prevents JVM hanging (non-daemon worker threads). Always shutdown in `@PreDestroy` or try-finally.

---

## Q4. Future.get() vs join()?

`get()` throws checked `InterruptedException`, `ExecutionException`.  
`join()` (CompletableFuture) throws unchecked `CompletionException`.

---

## Q5. @Async in Spring?

Runs method on separate thread pool; return type can be `Future` or `CompletableFuture`.

---

# One-Line Revision

```text
ExecutorService = managed thread pool.
Callable + Future = async task with result.
CompletableFuture = non-blocking composition and chaining.
```

---

*End of Day 9 Java*
