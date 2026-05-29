# Day 43 — LLD Pattern Review: State, Strategy, Observer

**Topics:** Behavioral patterns · When to use · Code sketches · Interview flow

---

# 1. Why These Three Patterns?

Amazon/senior LLD rounds frequently use:

| Pattern | Solves |
|---------|--------|
| **State** | Object behavior changes with internal state |
| **Strategy** | Swap algorithms at runtime |
| **Observer** | One-to-many notification on events |

All three reduce `if-else` / `switch` explosion and follow **Open/Closed Principle**.

---

# 2. State Pattern

## Intent

Allow an object to **alter its behavior** when its internal state changes. Object appears to change class.

## When to Use

- Multiple states with different behavior (order lifecycle, vending machine, document workflow)
- State transitions are well-defined

---

# Example: Vending Machine

```text
States: Idle → HasMoney → Dispensing → OutOfStock
```

```java
public interface VendingState {
    void insertCoin(VendingMachine vm, int amount);
    void selectProduct(VendingMachine vm, String productId);
    void dispense(VendingMachine vm);
}

public class IdleState implements VendingState {
    public void insertCoin(VendingMachine vm, int amount) {
        vm.addBalance(amount);
        vm.setState(new HasMoneyState());
    }
    public void selectProduct(VendingMachine vm, String id) {
        throw new IllegalStateException("Insert coin first");
    }
    public void dispense(VendingMachine vm) {
        throw new IllegalStateException("No product selected");
    }
}

public class HasMoneyState implements VendingState {
    public void selectProduct(VendingMachine vm, String id) {
        Product p = vm.getProduct(id);
        if (p.getStock() == 0) vm.setState(new OutOfStockState());
        else if (vm.getBalance() >= p.getPrice()) vm.setState(new DispensingState());
        else throw new IllegalStateException("Insufficient balance");
    }
    // ...
}

public class VendingMachine {
    private VendingState state = new IdleState();
    private int balance;

    public void setState(VendingState state) { this.state = state; }
    public void insertCoin(int amount) { state.insertCoin(this, amount); }
    public void selectProduct(String id) { state.selectProduct(this, id); }
    public void dispense() { state.dispense(this); }
}
```

---

# State vs Strategy

| State | Strategy |
|-------|----------|
| States know transitions | Strategies are independent |
| Context delegates to current state | Client picks strategy |
| State objects may switch themselves | Strategy usually set once per operation |

---

# 3. Strategy Pattern

## Intent

Define family of algorithms, encapsulate each, make them **interchangeable**.

## When to Use

- Multiple ways to compute something (pricing, routing, splitting bills, compression)
- Avoid growing conditional logic

---

# Example: Splitwise Split Strategies

```java
public interface SplitStrategy {
    List<Split> split(BigDecimal total, List<SplitRequest> requests);
}

public class EqualSplitStrategy implements SplitStrategy {
    public List<Split> split(BigDecimal total, List<SplitRequest> req) {
        BigDecimal share = total.divide(
            BigDecimal.valueOf(req.size()), 2, RoundingMode.HALF_UP);
        return req.stream()
            .map(r -> new Split(r.getUserId(), share))
            .collect(Collectors.toList());
    }
}

public class PercentageSplitStrategy implements SplitStrategy {
    public List<Split> split(BigDecimal total, List<SplitRequest> req) {
        return req.stream()
            .map(r -> new Split(r.getUserId(),
                total.multiply(r.getPercentage()).divide(BigDecimal.valueOf(100))))
            .collect(Collectors.toList());
    }
}

public class ExpenseService {
    private final Map<SplitType, SplitStrategy> strategies = Map.of(
        SplitType.EQUAL, new EqualSplitStrategy(),
        SplitType.PERCENTAGE, new PercentageSplitStrategy()
    );

    public Expense addExpense(CreateExpenseRequest req) {
        SplitStrategy strategy = strategies.get(req.getSplitType());
        List<Split> splits = strategy.split(req.getAmount(), req.getParticipants());
        return expenseRepo.save(new Expense(req, splits));
    }
}
```

---

# Strategy + Factory (Interview Plus)

```java
@Component
public class SplitStrategyFactory {
    public SplitStrategy get(SplitType type) {
        return switch (type) {
            case EQUAL -> new EqualSplitStrategy();
            case EXACT -> new ExactSplitStrategy();
            case PERCENTAGE -> new PercentageSplitStrategy();
        };
    }
}
```

---

# 4. Observer Pattern

## Intent

Define **one-to-many** dependency: when subject changes state, all observers are notified automatically.

## When to Use

- Event-driven updates (order status → email + SMS + analytics)
- Decouple publisher from subscribers

---

# Example: Order Status Notifications

```java
public interface OrderObserver {
    void onOrderStatusChanged(Order order, OrderStatus oldStatus, OrderStatus newStatus);
}

@Component
public class EmailNotificationObserver implements OrderObserver {
    public void onOrderStatusChanged(Order order, OrderStatus old, OrderStatus neu) {
        if (neu == OrderStatus.SHIPPED) {
            emailService.send(order.getUserEmail(), "Your order shipped!");
        }
    }
}

@Component
public class InventoryObserver implements OrderObserver {
    public void onOrderStatusChanged(Order order, OrderStatus old, OrderStatus neu) {
        if (neu == OrderStatus.CANCELLED) {
            inventoryService.restoreStock(order.getItems());
        }
    }
}

@Service
public class OrderService {
    private final List<OrderObserver> observers;

    public OrderService(List<OrderObserver> observers) { this.observers = observers; }

    public Order updateStatus(Long orderId, OrderStatus newStatus) {
        Order order = orderRepo.findById(orderId).orElseThrow();
        OrderStatus old = order.getStatus();
        order.setStatus(newStatus);
        orderRepo.save(order);
        observers.forEach(o -> o.onOrderStatusChanged(order, old, newStatus));
        return order;
    }
}
```

---

# Observer in Spring

Spring's **`ApplicationEventPublisher`** is built-in Observer:

```java
public record OrderShippedEvent(Long orderId) {}

@Component
public class OrderEventListener {
    @EventListener
    public void handleShipped(OrderShippedEvent event) {
        // notify, analytics, etc.
    }
}

@Service
public class OrderService {
    @Autowired ApplicationEventPublisher publisher;

    public void ship(Order order) {
        order.setStatus(SHIPPED);
        publisher.publishEvent(new OrderShippedEvent(order.getId()));
    }
}
```

Prefer Spring events for decoupling in real apps; classic Observer interface for LLD whiteboard.

---

# 5. Pattern Selection Guide

```text
Behavior changes with STATE of object     → State
Multiple ALGORITHMS for same operation    → Strategy
Notify MULTIPLE listeners on event        → Observer
```

---

# 6. Mock LLD Interview Flow (5 Steps)

1. **Clarify requirements** — entities, operations, edge cases
2. **Identify patterns** — "Order has states → State pattern"
3. **Draw class diagram** — interfaces + context + concrete implementations
4. **Code core path** — one state transition or one strategy + one observer
5. **Discuss trade-offs** — extensibility, testing, thread safety

---

# Interview Questions

## Q1. State vs if-else on enum?

Enum + switch works for 2–3 states. State pattern scales when each state has **multiple methods** with different behavior and transitions.

## Q2. Strategy vs Template Method?

Template Method: fixed skeleton in base class, subclasses override steps (inheritance).
Strategy: composes algorithm object (composition, runtime swap).

## Q3. Observer vs Pub/Sub (Kafka)?

Observer: in-process, synchronous/async in same JVM. Pub/Sub: distributed, durable, multiple services — use for microservices.

## Q4. Thread-safe Observer?

Copy observer list before iteration; use `ConcurrentHashMap.newKeySet()`; or event bus with async dispatch.

## Q5. How to add new split type without modifying ExpenseService?

Register new `SplitStrategy` in factory/map — OCP. Interviewer wants to hear "Open for extension, closed for modification."

---

# One-Line Revision

```text
State = behavior varies by internal state; Strategy = pluggable algorithms; Observer = notify dependents on change — all replace conditionals with polymorphism.
```

---

*End of Day 43 LLD*
