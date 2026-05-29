# Day 10 — Vending Machine LLD

**Topics:** State machine · Inventory · Payment · Class diagram · Sequence flow · Interview points

---

# 1. Requirements (Clarify in Interview)

- Machine has slots with products (code, price, quantity).
- User inserts coins/cash (or UPI in extension).
- Select product → validate stock & balance → dispense → return change.
- States: IDLE, HAS_MONEY, DISPENSING, OUT_OF_STOCK handling.
- Admin can restock (optional extension).

---

# 2. Class Diagram (Text)

```text
+------------------+
| VendingMachine   |
+------------------+
| - state          |
| - inventory      |
| - paymentSlot    |
+------------------+
| + insertMoney()  |
| + selectProduct()|
| + dispense()     |
| + cancel()       |
+------------------+
        | uses
        v
+------------------+       +----------------+
| Inventory        | 1---* | Slot           |
+------------------+       +----------------+
| + getSlot(code)  |       | - product      |
| + restock()      |       | - quantity     |
+------------------+       +----------------+

+------------------+
| Product          |
+------------------+
| - code           |
| - name           |
| - price          |
+------------------+

+------------------+<<interface>>
| VendingState     |
+------------------+
| + insertMoney()  |
| + selectProduct()|
| + dispense()     |
+------------------+
        △
   +----+----+---------+
   |         |         |
+------+ +--------+ +----------+
|Idle  | |HasMoney| |Dispensing|
+------+ +--------+ +----------+
```

---

# 3. Core Entities (Java Sketch)

```java
class Product {
    private final String code;
    private final String name;
    private final int price; // paise/cents
}

class Slot {
    private final Product product;
    private int quantity;

    boolean isAvailable() { return quantity > 0; }
    void dispense() { if (quantity > 0) quantity--; }
}

class Inventory {
    private final Map<String, Slot> slots = new HashMap<>();

    Optional<Slot> getSlot(String code) {
        return Optional.ofNullable(slots.get(code));
    }
}
```

---

# 4. State Pattern Implementation

```java
interface VendingState {
    void insertMoney(VendingMachine vm, int amount);
    void selectProduct(VendingMachine vm, String code);
    void dispense(VendingMachine vm);
}

class IdleState implements VendingState {
    public void insertMoney(VendingMachine vm, int amount) {
        vm.addBalance(amount);
        vm.setState(new HasMoneyState());
    }
    public void selectProduct(VendingMachine vm, String code) {
        throw new IllegalStateException("Insert money first");
    }
    public void dispense(VendingMachine vm) {}
}

class HasMoneyState implements VendingState {
    public void selectProduct(VendingMachine vm, String code) {
        Slot slot = vm.getInventory().getSlot(code)
            .orElseThrow(() -> new IllegalArgumentException("Invalid code"));
        if (!slot.isAvailable()) throw new IllegalStateException("Out of stock");
        if (vm.getBalance() < slot.getProduct().getPrice())
            throw new IllegalStateException("Insufficient balance");
        vm.setSelectedSlot(slot);
        vm.setState(new DispensingState());
        vm.dispense();
    }
    // insertMoney, dispense ...
}
```

---

# 5. VendingMachine Facade

```java
class VendingMachine {
    private VendingState state = new IdleState();
    private final Inventory inventory;
    private int balance = 0;
    private Slot selectedSlot;

    public void insertMoney(int amount) {
        state.insertMoney(this, amount);
    }

    public void selectProduct(String code) {
        state.selectProduct(this, code);
    }

    void dispense() {
        int price = selectedSlot.getProduct().getPrice();
        selectedSlot.dispense();
        int change = balance - price;
        balance = 0;
        selectedSlot = null;
        state = new IdleState();
        System.out.println("Dispensed. Change: " + change);
    }

    void addBalance(int amount) { balance += amount; }
    void setState(VendingState state) { this.state = state; }
    Inventory getInventory() { return inventory; }
    int getBalance() { return balance; }
    void setSelectedSlot(Slot slot) { this.selectedSlot = slot; }
}
```

---

# 6. Sequence Flow — Buy Product

```text
User -> VendingMachine: insertMoney(50)
VendingMachine -> IdleState: insertMoney
IdleState -> VendingMachine: balance=50, state=HasMoney

User -> VendingMachine: selectProduct("A1")
VendingMachine -> HasMoneyState: selectProduct
HasMoneyState -> Inventory: getSlot("A1")
Inventory --> HasMoneyState: Slot(price=40, qty>0)
HasMoneyState -> VendingMachine: state=Dispensing, dispense()
VendingMachine -> Slot: dispense()  // qty--
VendingMachine --> User: product + change=10
```

---

# 7. Design Patterns Used

| Pattern | Where |
|---------|-------|
| State | IDLE / HAS_MONEY / DISPENSING |
| Singleton | One machine instance (optional) |
| Strategy | Payment method (coins vs card) extension |

---

# 8. Edge Cases (Interview)

1. Cancel → refund balance, return to IDLE.
2. Concurrent users — single-threaded machine (lock) or queue requests.
3. Exact change only mode — reject if cannot return change.
4. Admin restock — separate role/API, not exposed to buyer flow.
5. Power failure — persist inventory/balance (extension).

---

# 9. Interview Points

1. Start with requirements and actors (User, Admin).
2. Separate **inventory**, **payment**, **state** — SRP.
3. State pattern avoids giant if-else on machine status.
4. Discuss extensibility: electronic payment, multiple currency.
5. Scale: this is single-machine LLD, not distributed vending network (HLD).

---

# Quick Revision

```text
Vending Machine = Inventory + Balance + State pattern + Dispense/Change logic.
```

---

*End of Day 10 LLD*
