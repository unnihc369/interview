# Day 22 - LLD Pattern Review & UML

---

# 1. Why Pattern Review?

Week 4 LLD starts with consolidating design patterns learned so far. Interviewers expect you to:

- Name the right pattern for a problem
- Draw UML quickly
- Explain trade-offs (coupling, extensibility, testability)

---

# 2. Creational Patterns

| Pattern | Intent | When to Use | Example |
|---------|--------|-------------|---------|
| **Singleton** | One instance globally | Logger, config, connection pool manager | `DatabaseConnectionPool.getInstance()` |
| **Factory** | Delegate object creation | Client shouldn't know concrete class | `NotificationFactory.create("EMAIL")` |
| **Builder** | Step-by-step complex object build | Many optional fields | `Order.builder().items().address().build()` |

---

# Factory vs Builder (Interview)

```text
Factory   → WHICH object to create
Builder   → HOW to configure one object
```

---

# 3. Structural Patterns

| Pattern | Intent | When to Use | Example |
|---------|--------|-------------|---------|
| **Adapter** | Convert incompatible interface | Integrate third-party SDK | `PaymentGatewayAdapter` wraps Stripe API |
| **Decorator** | Add behavior dynamically | Cross-cutting without subclass explosion | `BufferedInputStream` wraps `FileInputStream` |
| **Proxy** | Control access to real object | Lazy load, auth, caching | `UserServiceProxy` checks `@PreAuthorize` |
| **Facade** | Simplified interface to subsystem | Hide complexity | `OrderFacade` orchestrates payment + inventory |

---

# 4. Behavioral Patterns

| Pattern | Intent | When to Use | Example |
|---------|--------|-------------|---------|
| **Strategy** | Swap algorithms at runtime | Multiple pricing/shipping rules | `DiscountStrategy` |
| **Observer** | Notify dependents on change | Event-driven updates | `OrderStatusPublisher` → email/SMS listeners |
| **Command** | Encapsulate request as object | Undo, queue, audit | `PlaceOrderCommand` |
| **State** | Behavior changes with internal state | Order lifecycle | `Pending → Shipped → Delivered` |
| **Template Method** | Skeleton algorithm, steps overridden | Shared workflow, varying steps | `PaymentProcessor.process()` |
| **Chain of Responsibility** | Pass request along handler chain | Validation pipeline, logging filters | Servlet filters, approval workflow |

---

# 5. SOLID Quick Map to Patterns

| Principle | Pattern Connection |
|-----------|-------------------|
| SRP | Each handler/class one reason to change |
| OCP | Strategy, Decorator extend without modifying |
| LSP | Subtypes replaceable in polymorphic use |
| ISP | Small focused interfaces (not fat god interfaces) |
| DIP | Depend on abstractions (Factory, Strategy interfaces) |

---

# 6. UML Class Diagram — Pattern Example

## Strategy Pattern

```text
+------------------+       +-------------------+
|  CheckoutService |------>| PricingStrategy   |<<interface>>
+------------------+       +-------------------+
                                    ^
                    +---------------+---------------+
                    |               |               |
            +---------------+ +---------------+ +---------------+
            | RegularPrice  | | FestivePrice  | | MemberPrice   |
            +---------------+ +---------------+ +---------------+
```

---

# 7. UML Sequence Diagram — Pattern Example

## Observer Pattern — Order Placed

```text
Client -> OrderService: placeOrder(orderId)
OrderService -> OrderRepository: save(order)
OrderService -> EventPublisher: publish(OrderPlacedEvent)
EventPublisher -> EmailListener: onEvent(event)
EventPublisher -> SMSListener: onEvent(event)
EmailListener --> EventPublisher: done
SMSListener --> EventPublisher: done
OrderService --> Client: OrderResponse
```

---

# 8. Pattern Selection Cheat Sheet

| Requirement | Pattern |
|-------------|---------|
| Multiple notification channels | Observer or Strategy |
| Different payment providers | Strategy + Factory |
| Undo/redo actions | Command |
| Object creation based on type string | Factory |
| Complex object with many fields | Builder |
| Wrap legacy API | Adapter |
| Add logging/caching around service | Decorator or Proxy |
| Multi-step approval | Chain of Responsibility |
| State-dependent behavior | State |

---

# 9. LLD Interview Workflow (Pattern-Aware)

1. **Clarify** functional + non-functional requirements
2. **Identify nouns** → entities (User, Order, Room)
3. **Identify verbs** → services/use cases
4. **Spot variation points** → apply Strategy/Factory/State
5. **Draw class diagram** with interfaces at extension points
6. **Draw sequence diagram** for 1–2 critical flows
7. **Discuss concurrency** (locks, idempotency) if booking/payment
8. **Mention trade-offs** (in-memory vs DB, sync vs async)

---

# 10. Anti-Patterns to Avoid in Interviews

| Anti-Pattern | Better Approach |
|--------------|-----------------|
| God class doing everything | Split into services by SRP |
| Concrete class dependencies everywhere | Inject interfaces |
| Pattern name-dropping without need | Use simplest design that fits |
| Over-engineering for 3 classes | Start simple, extend with OCP |

---

# 11. Common Interview Questions

## Q1. Factory vs Abstract Factory?

Factory creates one product type. Abstract Factory creates **families** of related products (UI theme: Button + Checkbox).

## Q2. Decorator vs Proxy?

Both wrap objects. Proxy controls **access**; Decorator adds **behavior**.

## Q3. Strategy vs State?

Strategy: client chooses algorithm. State: object **transitions** between behaviors internally.

## Q4. When NOT to use Singleton?

When testability suffers, when you need multiple instances, or when DI container already manages lifecycle.

## Q5. How to show patterns in UML?

Use `<<interface>>`, inheritance arrows, composition for has-a, and label roles (e.g., `strategy: PricingStrategy`).

## Q6. Most asked LLD patterns at Amazon/Flipkart?

Strategy, Observer, Factory, State, Singleton (careful), and basic SOLID layering.

---

# One-Line Revision

```text
Match variation to Strategy/Factory, structure to Adapter/Decorator, flow to Observer/Command/State — always draw UML.
```

---

*End of Day 22 LLD*
