# Day 24 - @Scheduled & @Async

---

# 1. @Scheduled — Cron & Fixed-Rate Tasks

---

# Enable Scheduling

```java
@SpringBootApplication
@EnableScheduling
public class Application { }
```

---

# Fixed Rate vs Fixed Delay

```java
@Component
public class ReportScheduler {

    // Every 5 seconds from START of previous run
    @Scheduled(fixedRate = 5000)
    public void fixedRateTask() {
        System.out.println("Fixed rate: " + Instant.now());
    }

    // 5 seconds AFTER previous run COMPLETES
    @Scheduled(fixedDelay = 5000)
    public void fixedDelayTask() {
        heavyProcessing();
    }

    // Initial delay 10s, then every 5s
    @Scheduled(initialDelay = 10000, fixedRate = 5000)
    public void delayedStart() { }
}
```

---

# Cron Expressions

```java
// Every day at 2:30 AM
@Scheduled(cron = "0 30 2 * * *")
public void dailyCleanup() {
    expiredSessionRepository.deleteAllExpired();
}

// Every Monday at 9 AM
@Scheduled(cron = "0 0 9 * * MON")
public void weeklyReport() { }
```

Cron format (Spring 6-field):

```text
second minute hour day month weekday
```

---

# Externalize Cron

```properties
app.cron.cleanup=0 0 3 * * *
```

```java
@Scheduled(cron = "${app.cron.cleanup}")
public void cleanup() { }
```

---

# 2. @Async — Non-Blocking Method Execution

---

# Enable Async

```java
@SpringBootApplication
@EnableAsync
public class Application { }
```

---

# Basic Usage

```java
@Service
public class NotificationService {

    @Async
    public void sendEmailAsync(String to, String body) {
        // runs in separate thread pool
        emailClient.send(to, body);
    }

    @Async
    public CompletableFuture<String> sendWithResult(String to) {
        emailClient.send(to, "Hello");
        return CompletableFuture.completedFuture("SENT");
    }
}
```

Caller returns immediately; work happens in background thread.

---

# Custom Thread Pool

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

```java
@Async("taskExecutor")
public void processOrder(Order order) { ... }
```

---

# 3. @Scheduled vs @Async

| | @Scheduled | @Async |
|---|------------|--------|
| Trigger | Time-based (cron/fixed) | Method call |
| Use case | Periodic jobs | Offload slow work from request thread |
| Return | void typically | void or `Future`/`CompletableFuture` |
| Example | Nightly DB cleanup | Send email after order placed |

---

# 4. Real-World Backend Patterns

## Scheduled Jobs

| Job | Cron Example |
|-----|--------------|
| Cache warm-up | `@Scheduled(fixedRate = 3600000)` |
| Session cleanup | Daily 3 AM |
| Report generation | Weekly |
| Health metric push | Every minute |

## Async Operations

| Operation | Why Async |
|-----------|-----------|
| Email/SMS notification | Don't block HTTP response |
| Audit log write | Fire-and-forget |
| Image thumbnail generation | CPU-intensive |
| External API call (non-critical) | Improve latency |

---

# 5. Important Caveats

## Self-Invocation

```java
@Service
public class OrderService {
    public void placeOrder() {
        this.sendEmailAsync(); // NOT async — no proxy!
    }

    @Async
    public void sendEmailAsync() { }
}
```

Fix: inject self or separate `@Service`.

## Exception Handling

Uncaught async exceptions may be swallowed. Implement `AsyncUncaughtExceptionHandler`:

```java
@Override
public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return (ex, method, params) ->
        log.error("Async error in {}: {}", method.getName(), ex.getMessage());
}
```

## Scheduled Task Overlap

`fixedRate` can overlap if task runs longer than interval. Use `@Scheduled` with `ScheduledExecutorService` or distributed lock (ShedLock) in clustered deployments.

---

# 6. Distributed Scheduling (Interview)

Single instance: `@Scheduled` is fine.

Multiple instances: **only one should run** the job.

Solutions:

- **ShedLock** — DB/Redis lock around scheduled method
- **Quartz cluster**
- **Kubernetes CronJob** — external scheduler

```java
@Scheduled(cron = "0 0 2 * * *")
@SchedulerLock(name = "dailyCleanup", lockAtMostFor = "30m")
public void cleanup() { }
```

---

# 7. Testing

```java
@SpringBootTest
class SchedulerTest {
    @Autowired
    ReportScheduler scheduler;

    @Test
    void testCleanup() {
        scheduler.dailyCleanup(); // call directly, no wait
    }
}
```

For async:

```java
@Async
public CompletableFuture<Integer> compute() {
    return CompletableFuture.completedFuture(42);
}

@Test
void testAsync() throws Exception {
    assertEquals(42, service.compute().get());
}
```

---

# Common Interview Questions

## Q1. fixedRate vs fixedDelay?

fixedRate: interval from start. fixedDelay: interval from end of previous execution.

## Q2. Default async executor?

`SimpleAsyncTaskExecutor` — creates new thread per task. Always configure pool in production.

## Q3. Can @Scheduled method return value?

No meaningful consumer — use void. For results, publish to queue or DB.

## Q4. How to run job only on one pod in K8s?

ShedLock, leader election, or external CronJob.

## Q5. @Async with @Transactional?

Async method runs in different thread — transaction from caller does NOT propagate. Start new transaction inside async method if needed.

## Q6. Spring Batch vs @Scheduled?

Batch: large chunked ETL jobs with restart. Scheduled: lightweight periodic tasks.

---

# One-Line Revision

```text
@EnableScheduling + cron for periodic jobs; @EnableAsync + thread pool to offload slow work from request path.
```

---

*End of Day 24 Spring Boot*
