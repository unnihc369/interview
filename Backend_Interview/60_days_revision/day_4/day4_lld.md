# Day 4 - UML Basics (Class Diagram & Sequence Diagram)

---

# 1. What is UML?

UML = **Unified Modeling Language**

Used to visualize software design before coding.

In interviews, UML helps communicate:

- Object structure
- Relationships
- Runtime flow

---

# 2. Types of UML Diagrams (Interview Quick)

| Category | Diagram | Focus |
|----------|---------|-------|
| Structural | Class Diagram | Static structure (classes, fields, methods) |
| Behavioral | Sequence Diagram | Runtime interaction (message order) |

For LLD rounds, these two are most used.

---

# 3. Class Diagram Basics

---

# Class Notation

Each class box has 3 sections:

1. Class name
2. Attributes (fields)
3. Operations (methods)

```text
+----------------------+
| User                 |
+----------------------+
| - id: Long           |
| - name: String       |
+----------------------+
| + getName(): String  |
| + setName(n): void   |
+----------------------+
```

---

# Visibility Symbols

| Symbol | Meaning |
|--------|---------|
| `+` | public |
| `-` | private |
| `#` | protected |

---

# Relationships in Class Diagram

| Relationship | Symbol | Meaning |
|--------------|--------|---------|
| Association | `A ---- B` | Uses/knows each other |
| Aggregation | `A ◇---- B` | Has-a (weak ownership) |
| Composition | `A ◆---- B` | Strong ownership (lifecycle tied) |
| Inheritance | `Child ----|> Parent` | IS-A |
| Dependency | `A - - - > B` | Temporary use |

---

# Multiplicity

| Notation | Meaning |
|----------|---------|
| `1` | exactly one |
| `0..1` | optional |
| `*` | many |
| `1..*` | one or more |

Example:

```text
Customer 1 ----- * Order
```

One customer can have many orders.

---

# 4. Class Diagram Example — E-commerce Order Flow

```text
Customer 1 -------- * Order
Order    1 -------- * OrderItem
OrderItem * ------- 1 Product
Order    1 -------- 1 Payment

class Customer {
  - id: Long
  - name: String
  + placeOrder(): Order
}

class Order {
  - id: Long
  - total: double
  + addItem(p: Product, qty: int): void
  + checkout(): void
}

class OrderItem {
  - quantity: int
  - price: double
}

class Product {
  - id: Long
  - name: String
  - price: double
}

class Payment {
  + pay(amount: double): boolean
}
```

---

# Interview Tips for Class Diagram

1. Start with nouns from requirement (User, Order, Product)
2. Add key fields only (avoid too much detail)
3. Add relationships + multiplicities
4. Add critical methods (not all methods)
5. Mention SOLID where possible (interfaces for extensibility)

---

# 5. Sequence Diagram Basics

Sequence diagram shows **how objects interact over time**.

---

# Core Elements

| Element | Meaning |
|---------|---------|
| Actor | External user/system |
| Lifeline | Object participating |
| Message arrow | Method/API call |
| Activation bar | Time object is active |
| Return message | Response back |

Time flows top → bottom.

---

# Sequence Diagram Example — Place Order API

Use case:

`POST /orders` from client.

```text
Actor(Client) -> OrderController: createOrder(req)
OrderController -> OrderService: create(req)
OrderService -> ProductService: validateStock(items)
ProductService --> OrderService: stock OK
OrderService -> PaymentService: charge(amount)
PaymentService --> OrderService: success
OrderService -> OrderRepository: save(order)
OrderRepository --> OrderService: savedOrder
OrderService --> OrderController: OrderResponse
OrderController --> Actor(Client): 201 Created + response
```

---

# Sequence Diagram for Failure Case

Payment failed scenario:

```text
Client -> OrderController: createOrder(req)
OrderController -> OrderService: create(req)
OrderService -> PaymentService: charge(amount)
PaymentService --> OrderService: FAILED
OrderService --> OrderController: throw PaymentFailedException
OrderController --> Client: 400/402 error response
```

Important in interviews: show both happy and failure path.

---

# 6. Class vs Sequence Diagram

| Class Diagram | Sequence Diagram |
|---------------|------------------|
| Static view | Runtime flow |
| Focus on structure | Focus on interaction order |
| Classes and relationships | Calls/messages between objects |
| No timing order | Has explicit timing order |

---

# 7. Practical Interview Workflow

When asked LLD:

1. Clarify requirements
2. Identify entities
3. Draw class diagram (structure)
4. Explain key APIs
5. Draw sequence diagram for one use case
6. Mention edge cases and scalability

---

# 8. Common Interview Questions

## Q1. Why UML in LLD interview?

To communicate design quickly and clearly.

## Q2. Difference between aggregation and composition?

Aggregation = weak ownership.  
Composition = strong ownership (child dies with parent).

## Q3. Which comes first: class or sequence diagram?

Usually class diagram first, then sequence for important use case.

## Q4. Is sequence diagram mandatory?

Not always, but it strongly improves discussion quality.

## Q5. What should be included in class diagram?

Core entities, relationships, multiplicity, major methods.

---

# 9. Quick Revision

```text
Class Diagram = what classes exist and how they relate.
Sequence Diagram = how objects call each other in a use case.
```

---

*End of Day 4 LLD*
