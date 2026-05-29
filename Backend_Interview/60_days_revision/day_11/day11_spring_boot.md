# Day 11 — @Transactional (Propagation & Isolation)

**Topics:** ACID · Transaction boundaries · Propagation levels · Isolation levels · Pitfalls · Interview Q&A

---

# 1. What is @Transactional?

Spring wraps method in a database transaction — commit on success, rollback on runtime exception (by default).

```java
@Service
public class OrderService {

    @Transactional
    public Order placeOrder(OrderRequest req) {
        Order order = orderRepository.save(new Order(req));
        inventoryService.reduceStock(req.getItems());
        paymentService.charge(order.getTotal());
        return order;
    }
}
```

If any step throws, entire transaction rolls back (atomicity).

---

# 2. How It Works (Proxy)

```text
Client → OrderServiceProxy → begin TX → real OrderService → commit/rollback
```

`@Transactional` only works on **public methods** called through Spring proxy (not self-invocation within same class).

---

# 3. Propagation

Defines behavior when a transactional method calls another transactional method.

```java
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() { methodB(); }

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() { }
```

| Propagation | Behavior |
|-------------|----------|
| **REQUIRED** (default) | Join existing TX or create new |
| **REQUIRES_NEW** | Always new TX; suspend current |
| **NESTED** | Nested TX with savepoint (if supported) |
| **SUPPORTS** | Use TX if exists, else non-transactional |
| **NOT_SUPPORTED** | Run non-transactional, suspend TX |
| **MANDATORY** | Must run in existing TX, else exception |
| **NEVER** | Must not run in TX, else exception |

---

# Interview Examples

## REQUIRED

```java
@Transactional
public void transfer() {
    debit();
    credit(); // same transaction
}
```

## REQUIRES_NEW — Audit log even if main fails

```java
@Transactional
public void placeOrder() {
    try {
        saveOrder();
    } finally {
        auditService.log(); // REQUIRES_NEW — commits independently
    }
}
```

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void log() { auditRepository.save(...); }
```

---

# 4. Isolation

Controls visibility of concurrent transactions.

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public Order getOrder(Long id) { ... }
```

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|-------|------------|---------------------|--------------|
| READ_UNCOMMITTED | Possible | Possible | Possible |
| READ_COMMITTED | No | Possible | Possible |
| REPEATABLE_READ | No | No | Possible |
| SERIALIZABLE | No | No | No |

Default depends on DB (MySQL InnoDB default: REPEATABLE_READ).

---

# Phenomena Explained

- **Dirty read:** Read uncommitted data from another TX.
- **Non-repeatable read:** Same row read twice, different values.
- **Phantom read:** Range query returns different row count.

---

# 5. rollbackFor / noRollbackFor

```java
@Transactional(rollbackFor = Exception.class) // include checked exceptions
public void importData() throws Exception { ... }

@Transactional(noRollbackFor = BusinessWarningException.class)
public void process() { ... }
```

Default: rollback on `RuntimeException` and `Error` only.

---

# 6. readOnly = true

```java
@Transactional(readOnly = true)
public List<Order> listOrders() {
    return orderRepository.findAll();
}
```

Hints Hibernate/JPA to skip dirty checking — performance optimization for queries.

---

# 7. Common Pitfalls

## Self-invocation bypasses proxy

```java
@Transactional
public void outer() {
    inner(); // NOT transactional — same object, no proxy
}

@Transactional
public void inner() { }
```

Fix: inject self, move to another bean, or use `AopContext`.

## Exception swallowed

```java
@Transactional
public void save() {
    try {
        repo.save(entity);
    } catch (Exception e) {
        log.error("fail"); // TX still commits!
    }
}
```

## Long transactions

Holding TX during external API calls → locks, timeouts. Keep TX short.

---

# Common Interview Questions

## Q1. Why @Transactional on service not controller?

Service layer = business boundary; controller should stay thin.

---

## Q2. Does @Transactional work on private methods?

No — proxy cannot intercept private methods.

---

## Q3. REQUIRED vs REQUIRES_NEW?

REQUIRED joins one TX (all-or-nothing). REQUIRES_NEW is independent commit/rollback.

---

## Q4. Isolation in real apps?

Often rely on DB default + optimistic locking (`@Version`) instead of SERIALIZABLE.

---

## Q5. Transaction and JPA LazyInitializationException?

Access lazy collections inside `@Transactional` service method or use `JOIN FETCH`.

---

# One-Line Revision

```text
@Transactional = proxy-managed DB transaction.
Propagation = how TX composes across methods; Isolation = concurrency visibility.
```

---

*End of Day 11 Spring Boot*
