# Day 50 — LLD Redo: Vending Machine + Elevator

**Week 8 · Monday · ~2 hours**  
**Goal:** Whiteboard both designs in 35 min each with checklist verification

---

# How to Use This File

1. **Round 1 (45 min):** Vending Machine — requirements → classes → one sequence → code sketch  
2. **Break 5 min**  
3. **Round 2 (45 min):** Elevator — same flow  
4. **Round 3 (30 min):** Compare against checklists below; note gaps  

*Original deep dives: `day_10/day10_lld.md` (Vending), `day_13/day13_lld.md` (Elevator)*

---

# Part A — Vending Machine Redo Guide

---

## 1. Clarifying Questions (2 min)

| Question | Default assumption |
|----------|-------------------|
| Payment types? | Coins only (extensible to interface) |
| Change? | Exact change or return coins |
| Concurrent users? | Single user (no locking) |
| Admin restock? | Mention extension, not required in v1 |
| Failure modes? | Out of stock, insufficient funds |

---

## 2. Requirements Summary

**Functional**

- Display products (code → name, price, qty)  
- Accept money incrementally  
- Select product → validate → dispense → return change  
- Cancel → refund inserted money  

**Non-functional**

- Extensible payment (Strategy)  
- Clear state machine (State pattern)  

---

## 3. Class Diagram (Must Draw)

```text
VendingMachine
  ├── Inventory (Map<code, Slot>)
  ├── PaymentProcessor / CoinSlot
  ├── currentBalance
  └── VendingState (State pattern)

Slot → Product (code, name, priceCents), quantity

States: IdleState | HasMoneyState | DispensingState
```

---

## 4. State Transitions

```text
IDLE
  insertMoney → HAS_MONEY

HAS_MONEY
  insertMoney → HAS_MONEY (balance +=)
  selectProduct(valid) → DISPENSING → dispense → HAS_MONEY or IDLE
  cancel → refund → IDLE

DISPENSING
  (internal) dispense product, deduct balance, return change
```

---

## 5. Core Operations (Pseudo)

```java
void selectProduct(String code) {
    Slot slot = inventory.getSlot(code).orElseThrow();
    if (!slot.isAvailable()) throw new OutOfStockException();
    if (balance < slot.getProduct().getPrice()) throw new InsufficientFundsException();
    balance -= price;
    slot.dispense();
    returnChange(balance);
    balance = 0;
    setState(new IdleState());
}
```

---

## 6. Sequence: Buy Snack B7 for ₹30, paid ₹50

```text
User → VM: insertMoney(50)
VM: balance=50, state=HAS_MONEY
User → VM: selectProduct("B7")
VM → Inventory: getSlot("B7")
Inventory → VM: price=30, qty>0
VM → Slot: dispense()
VM → User: product + change(20)
VM: balance=0, state=IDLE
```

---

## 7. Design Patterns to Mention

| Pattern | Where |
|---------|-------|
| State | Idle / HasMoney / Dispensing |
| Strategy | Coin vs Card payment (extension) |
| Singleton | Not needed — one VM instance per machine |

---

## 8. Vending Machine Checklist

| # | Item | ✓ |
|---|------|---|
| 1 | Requirements stated (buy, pay, change, cancel) | |
| 2 | `Product`, `Slot`, `Inventory` separated | |
| 3 | State pattern for mode (not giant if-else) | |
| 4 | `selectProduct` validates stock AND balance | |
| 5 | Change calculation explicit | |
| 6 | `cancel` refunds full balance | |
| 7 | Extension: admin restock API | |
| 8 | Extension: multiple coin denominations | |
| 9 | No God class — VM orchestrates, doesn't implement coin logic | |
| 10 | Enums for states or dedicated state classes | |

---

# Part B — Elevator Redo Guide

---

## 1. Clarifying Questions (2 min)

| Question | Default |
|----------|---------|
| Floors? | 0..N-1 or 1..N (state clearly) |
| Elevators? | 1 elevator first; extend to M |
| Algorithm? | SCAN / LOOK or simple direction sweep |
| Door timing? | Open at floor, then close, then move |
| Hall vs cabin? | Both request types |

---

## 2. Requirements Summary

- Hall button: (floor, UP/DOWN)  
- Cabin panel: destination floor  
- Elevator moves, stops, opens door  
- Multiple pending stops — efficient ordering  

---

## 3. Class Diagram (Must Draw)

```text
ElevatorController
  ├── List<Elevator>
  └── dispatch(Request)

Elevator
  ├── currentFloor, direction, state
  ├── TreeSet<Integer> upStops
  ├── TreeSet<Integer> downStops (reverse order)
  └── Door

Request: sourceFloor, destFloor, direction, type(HALL|CABIN)
Enums: Direction, ElevatorState
```

---

## 4. SCAN / Direction-Serving Logic

**While MOVING_UP:** serve `upStops` ascending until empty, then reverse.  
**While MOVING_DOWN:** serve `downStops` descending.  
**IDLE:** pick nearest request or priority queue policy.

```java
void move() {
    if (direction == UP) {
        if (upStops.isEmpty()) direction = downStops.isEmpty() ? IDLE : DOWN;
        else { currentFloor = upStops.pollFirst(); openDoor(); }
    } else if (direction == DOWN) {
        if (downStops.isEmpty()) direction = upStops.isEmpty() ? IDLE : UP;
        else { currentFloor = downStops.pollFirst(); openDoor(); }
    }
}
```

---

## 5. Multi-Elevator Dispatch (Extension)

| Strategy | Idea |
|----------|------|
| Nearest car | Min distance + same direction bonus |
| Zone-based | Elevators serve floor ranges |
| Load balance | Queue depth per elevator |

---

## 6. Sequence: Hall UP at 3, going to 7

```text
User floor 3 → Controller: requestElevator(3, UP)
Controller → Elevator1: addRequest(source=3, dest=7)
Elevator1: upStops.add(3), upStops.add(7)
Elevator1: move → stop 3 → openDoor → close
Elevator1: move → stop 7 → openDoor
```

---

## 7. Elevator Checklist

| # | Item | ✓ |
|---|------|---|
| 1 | Separate `Request` from `Elevator` | |
| 2 | `upStops` / `downStops` (TreeSet) for ordering | |
| 3 | Direction enum: UP, DOWN, IDLE | |
| 4 | State: IDLE, MOVING, DOOR_OPEN | |
| 5 | Stop at floor when in active stop set | |
| 6 | Hall request adds source floor to correct set | |
| 7 | Cabin request adds destination only | |
| 8 | Don't pass through stops — check current floor in set | |
| 9 | Multi-elevator: controller picks elevator | |
| 10 | Discuss SCAN vs FCFS tradeoff | |

---

# Part C — Side-by-Side Comparison

| Aspect | Vending Machine | Elevator |
|--------|-----------------|----------|
| Primary pattern | State (user session) | Scheduler + state |
| Data structures | Map slots | TreeSet stops |
| Concurrency | Usually single-user | Multi-request queue |
| Extension | Payment strategy | Multi-car dispatch |
| Common mistake | Forgetting change | Wrong direction after idle |

---

# Mock Timing (70 min total)

| Segment | Minutes |
|---------|---------|
| Vending: req + diagram + 2 methods | 35 |
| Elevator: req + diagram + move() logic | 35 |

**Bar raiser question:** “How would you unit test the elevator without real time?”  
→ Inject `Clock` / tick-based `step()` simulation; assert floor sequence.

---

# Quick Reference Code — State Interface (Vending)

```java
interface VendingState {
    void insertMoney(VendingMachine vm, int amount);
    void selectProduct(VendingMachine vm, String code);
    void cancel(VendingMachine vm);
}
```

---

*End of Day 50 — LLD*
