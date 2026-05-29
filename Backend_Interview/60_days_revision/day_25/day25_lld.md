# Day 25 - LLD: Chain of Responsibility & Interpreter

---

# 1. Chain of Responsibility Pattern

---

# Definition

Pass a request along a chain of handlers. Each handler decides to process or forward to next.

```text
Request → Handler1 → Handler2 → Handler3 → ...
```

Decouples sender from receivers.

---

# Use Cases

| Domain | Chain |
|--------|-------|
| Servlet filters | Auth → Logging → CORS |
| Approval workflow | Manager → Director → VP |
| Support tickets | L1 → L2 → L3 |
| Validation pipeline | Format → Business → Fraud |

---

# Structure

```java
abstract class Handler {
    protected Handler next;

    public Handler setNext(Handler next) {
        this.next = next;
        return next; // fluent chaining
    }

    public abstract void handle(Request request);
}

class AuthHandler extends Handler {
    @Override
    public void handle(Request request) {
        if (!request.isAuthenticated()) {
            throw new UnauthorizedException();
        }
        if (next != null) next.handle(request);
    }
}

class RateLimitHandler extends Handler {
    @Override
    public void handle(Request request) {
        if (rateLimiter.exceeded(request.getUserId())) {
            throw new RateLimitException();
        }
        if (next != null) next.handle(request);
    }
}

class BusinessHandler extends Handler {
    @Override
    public void handle(Request request) {
        // process order
        if (next != null) next.handle(request);
    }
}
```

---

# Building the Chain

```java
Handler chain = new AuthHandler();
chain.setNext(new RateLimitHandler())
     .setNext(new ValidationHandler())
     .setNext(new BusinessHandler());

chain.handle(incomingRequest);
```

---

# Spring Filter Chain (Real Example)

```text
HTTP Request
  → SecurityFilterChain
    → JwtAuthenticationFilter
    → AuthorizationFilter
    → DispatcherServlet
```

Same pattern — each filter calls `chain.doFilter(request, response)`.

---

# 2. Interpreter Pattern

---

# Definition

Define a grammar for language/expressions and interpret sentences using an AST (Abstract Syntax Tree).

```text
Expression → Terminal / Non-terminal rules
Context    → variables and state
```

---

# Classic Example — Roman Numerals (Simple)

```java
interface Expression {
    int interpret(Map<String, Integer> context);
}

class NumberExpression implements Expression {
    private final int value;
    NumberExpression(int value) { this.value = value; }
    public int interpret(Map<String, Integer> ctx) { return value; }
}

class AddExpression implements Expression {
    private final Expression left, right;
    AddExpression(Expression l, Expression r) { left = l; right = r; }
    public int interpret(Map<String, Integer> ctx) {
        return left.interpret(ctx) + right.interpret(ctx);
    }
}

// Usage: (3 + 5) → new AddExpression(new NumberExpression(3), new NumberExpression(5))
```

---

# Rule Engine Example (Interview-Friendly)

Parse discount rules:

```text
"SUMMER AND cartTotal > 1000" → 10% off
"VIP OR orderCount > 50"      → free shipping
```

```java
interface Rule {
    boolean evaluate(OrderContext ctx);
}

class AndRule implements Rule {
    private final Rule left, right;
    public boolean evaluate(OrderContext ctx) {
        return left.evaluate(ctx) && right.evaluate(ctx);
    }
}

class CartTotalRule implements Rule {
    private final BigDecimal threshold;
    public boolean evaluate(OrderContext ctx) {
        return ctx.getCartTotal().compareTo(threshold) > 0;
    }
}
```

---

# 3. Chain vs Interpreter

| | Chain of Responsibility | Interpreter |
|---|-------------------------|-------------|
| Purpose | Process/validate request sequentially | Evaluate expressions/rules |
| Structure | Linked handlers | Composite expression tree |
| Order matters | Yes (pipeline) | Tree traversal |
| Example | Filters, approvals | Rule engine, query DSL |

---

# 4. Combined LLD — Order Validation Pipeline

## Requirement

Validate order through: auth → inventory → payment → fraud check.

## Class Diagram

```text
OrderRequest
    ↓
ValidationHandler (abstract)
    ↑
AuthHandler → InventoryHandler → PaymentHandler → FraudHandler

RuleExpression (interface)
    ↑
AndRule, OrRule, StockAvailableRule, MinAmountRule
```

## Sequence

```text
Client -> OrderController: POST /orders
OrderController -> ValidationChain: handle(order)
ValidationChain -> AuthHandler: handle
AuthHandler -> InventoryHandler: handle
InventoryHandler -> PaymentHandler: handle
PaymentHandler -> FraudHandler: handle
FraudHandler --> OrderController: OK
OrderController -> OrderService: create()
```

---

# 5. Servlet Filter as Chain (Code Sketch)

```java
@Component
@Order(1)
public class LoggingFilter extends OncePerRequestFilter {
    protected void doFilterInternal(HttpServletRequest req,
                                    HttpServletResponse res,
                                    FilterChain chain) throws ServletException, IOException {
        log.info("{} {}", req.getMethod(), req.getRequestURI());
        chain.doFilter(req, res); // pass to next
    }
}
```

---

# 6. When to Use in Interviews

| Scenario | Pattern |
|----------|---------|
| Multi-step validation | Chain of Responsibility |
| Escalating support | Chain |
| Configurable business rules without if-else hell | Interpreter + Composite |
| SQL/DSL parsing | Interpreter (with parser) |

---

# 7. Anti-Pattern Warning

Don't use Interpreter for complex language parsing in production without a proper parser generator (ANTLR). For interviews, simplified rule trees are enough.

---

# Common Interview Questions

## Q1. Chain vs Decorator?

Chain: handler may not process — forwards. Decorator: always wraps and adds behavior.

## Q2. Can request skip handlers?

Yes — handler decides whether to call `next`.

## Q3. Spring Security filter order?

Critical — `@Order` determines chain sequence; auth before authorization.

## Q4. Interpreter real-world examples?

Hibernate HQL, Spring SpEL, regex engines, pricing rule engines.

## Q5. How to make chain configurable?

Build chain from ordered list in DB/config at startup.

---

# One-Line Revision

```text
Chain = sequential handlers forwarding request; Interpreter = composite tree evaluating rules/expressions.
```

---

*End of Day 25 LLD*
