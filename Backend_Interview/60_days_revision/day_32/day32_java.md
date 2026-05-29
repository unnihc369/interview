# Day 32 — CompletableFuture

**Topics:** Async Composition · thenApply · thenCombine · Exception Handling · Interview Questions

---

# 1. Basics

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    Thread.sleep(1000);
    return "Hello";
});

String result = future.get(); // blocking
// future.get(2, TimeUnit.SECONDS); // timeout
```

`supplyAsync` — returns value  
`runAsync` — void, no return value

Default executor: `ForkJoinPool.commonPool()`. Pass custom executor for I/O-bound work.

---

# 2. Chaining Transformations

```java
CompletableFuture<Integer> future = CompletableFuture
    .supplyAsync(() -> fetchUserId("alice"))
    .thenApply(id -> fetchOrders(id))           // sync transform
    .thenApplyAsync(orders -> summarize(orders)); // async on pool
```

| Method | When |
|--------|------|
| `thenApply` | Map result (sync, same thread) |
| `thenApplyAsync` | Map result (async) |
| `thenAccept` | Consume result (void) |
| `thenRun` | Run action, ignore result |

---

# 3. Combining Multiple Futures

```java
CompletableFuture<String> user = CompletableFuture.supplyAsync(() -> fetchUser());
CompletableFuture<String> orders = CompletableFuture.supplyAsync(() -> fetchOrders());

CompletableFuture<String> combined = user.thenCombine(orders,
    (u, o) -> u + " has " + o);

// Wait for all
CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2, f3);
all.join(); // all complete

// First to complete
CompletableFuture<Object> any = CompletableFuture.anyOf(f1, f2);
```

---

# 4. Exception Handling

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> riskyCall())
    .exceptionally(ex -> {
        log.error("Failed", ex);
        return "default";
    });

// handle — both result and exception
CompletableFuture<String> handled = future.handle((result, ex) -> {
    if (ex != null) return "fallback";
    return result;
});

// whenComplete — side effect, doesn't change result
future.whenComplete((result, ex) -> audit.log(result, ex));
```

---

# 5. Manual Completion

```java
CompletableFuture<String> future = new CompletableFuture<>();

// Another thread completes later
executor.submit(() -> {
    try {
        String data = pollExternalService();
        future.complete(data);
    } catch (Exception e) {
        future.completeExceptionally(e);
    }
});

return future; // caller chains thenApply etc.
```

Use for callbacks, WebSocket events, Kafka consumers.

---

# 6. Real-World: Parallel API Aggregation

```java
public OrderDetails getOrderDetails(String orderId) {
    CompletableFuture<Order> orderF =
        CompletableFuture.supplyAsync(() -> orderRepo.find(orderId), ioPool);
    CompletableFuture<List<Item>> itemsF =
        CompletableFuture.supplyAsync(() -> itemRepo.findByOrder(orderId), ioPool);
    CompletableFuture<User> userF = orderF.thenComposeAsync(
        order -> CompletableFuture.supplyAsync(
            () -> userRepo.find(order.getUserId()), ioPool));

    return CompletableFuture.allOf(orderF, itemsF, userF)
        .thenApply(v -> new OrderDetails(
            orderF.join(), itemsF.join(), userF.join()))
        .join();
}
```

`thenCompose` — flatten nested `CompletableFuture<CompletableFuture<T>>`.

---

# 7. CompletableFuture vs Future

| Future | CompletableFuture |
|--------|-------------------|
| `get()` blocking only | Non-blocking composition |
| No chaining | thenApply, thenCombine |
| No manual complete | complete(), completeExceptionally |
| cancel | cancel (mayInterruptIfRunning) |

---

# 8. Pitfalls

```text
1. Blocking common pool with I/O → starvation; use dedicated executor
2. Forgetting exception handling → CompletionException wrapped
3. fire-and-forget without join → silent failures
4. ThreadLocal (e.g., MDC) lost across async → use TaskDecorator
5. Long chains hard to debug → structured concurrency (Java 21+)
```

---

# Interview Questions

## `thenApply` vs `thenCompose`?

`thenApply`: `T → U`, returns `CompletableFuture<U>`.  
`thenCompose`: `T → CompletableFuture<U>`, flattens nested future.

## Difference `join()` vs `get()`?

`join()` throws unchecked `CompletionException`. `get()` throws checked `ExecutionException`, `InterruptedException`.

## Run async on same executor for order service?

Use single-thread executor if ordering required; otherwise parallel pool.

## CompletableFuture vs @Async Spring?

`@Async` AOP-based, returns void/Future/CompletableFuture. CF is programmatic, finer control in plain Java.

## Java 21 StructuredTaskScope?

Scoped parallelism with cancellation policy — successor pattern for structured concurrency.

---

# Quick Revision

```text
supplyAsync / runAsync
thenApply / thenApplyAsync / thenCompose
thenCombine / allOf / anyOf
exceptionally / handle / whenComplete
Always use custom Executor for I/O
```

---

*End of Day 32 CompletableFuture*
