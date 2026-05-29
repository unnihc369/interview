# Day 33 — LLD: Coffee Vending Machine

**Topics:** State Machine · Inventory · Payment · Recipe · Interview Flow

---

# 1. Problem Statement

Design a coffee vending machine that:

- Displays menu (Espresso, Latte, Cappuccino)
- Accepts coins/card payment
- Dispenses drink if ingredients available
- Returns change
- Reports low inventory
- Handles concurrent users (single machine — one transaction at a time)

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Select Drink | User picks from menu |
| Insert Payment | Coins or card |
| Dispense | Brew and deliver cup |
| Change | Return excess coins |
| Cancel | Refund inserted amount |
| Inventory | Track beans, milk, water, cups |
| Maintenance | Refill ingredients |

---

# 3. State Machine

```text
IDLE → DRINK_SELECTED → COLLECTING_PAYMENT → DISPENSING → IDLE
  ↑           ↓ cancel              ↓ cancel
  └───────────┴──────────────────────┘
```

```java
public enum MachineState {
    IDLE, DRINK_SELECTED, COLLECTING_PAYMENT, DISPENSING, OUT_OF_ORDER
}
```

---

# 4. Core Entities

```text
CoffeeVendingMachine
  ├── state, selectedDrink
  ├── PaymentProcessor
  ├── Inventory
  └── Map<DrinkType, Recipe>

Recipe
  ├── beans, milk, water (amounts)

Inventory
  ├── beans, milk, water, cups
  └── canMake(Recipe), consume(Recipe)

DrinkType (enum)
  ESPRESSO, LATTE, CAPPUCCINO

PaymentSlot / CardReader
  └── insertCoin(), swipeCard(), refund()
```

---

# 5. Recipe & Inventory

```java
public record Recipe(int beans, int milk, int water) {}

public class Inventory {
    private int beans, milk, water, cups;

    public boolean canMake(Recipe recipe) {
        return beans >= recipe.beans() && milk >= recipe.milk()
            && water >= recipe.water() && cups > 0;
    }

    public void consume(Recipe recipe) {
        if (!canMake(recipe)) throw new IllegalStateException("Insufficient stock");
        beans -= recipe.beans();
        milk -= recipe.milk();
        water -= recipe.water();
        cups--;
    }
}
```

---

# 6. Drink Selection & Payment

```java
public class CoffeeVendingMachine {
    private MachineState state = MachineState.IDLE;
    private DrinkType selectedDrink;
    private int insertedAmount;
    private final Map<DrinkType, Integer> prices;
    private final Map<DrinkType, Recipe> recipes;
    private final Inventory inventory;

    public synchronized void selectDrink(DrinkType drink) {
        if (state != MachineState.IDLE) throw new IllegalStateException("Busy");
        if (!inventory.canMake(recipes.get(drink))) {
            throw new IllegalStateException("Out of stock");
        }
        selectedDrink = drink;
        insertedAmount = 0;
        state = MachineState.DRINK_SELECTED;
    }

    public synchronized void insertCoin(int cents) {
        if (state != MachineState.DRINK_SELECTED
            && state != MachineState.COLLECTING_PAYMENT) {
            throw new IllegalStateException("Select drink first");
        }
        state = MachineState.COLLECTING_PAYMENT;
        insertedAmount += cents;
        int price = prices.get(selectedDrink);
        if (insertedAmount >= price) {
            dispense();
        }
    }

    private void dispense() {
        state = MachineState.DISPENSING;
        Recipe recipe = recipes.get(selectedDrink);
        inventory.consume(recipe);
        int change = insertedAmount - prices.get(selectedDrink);
        if (change > 0) returnChange(change);
        reset();
    }

    private void reset() {
        selectedDrink = null;
        insertedAmount = 0;
        state = MachineState.IDLE;
    }
}
```

---

# 7. Decorator Pattern for Coffee (Extension)

```java
public interface Coffee {
    String getDescription();
    double getCost();
}

public class Espresso implements Coffee {
    public String getDescription() { return "Espresso"; }
    public double getCost() { return 2.0; }
}

public class MilkDecorator implements Coffee {
    private final Coffee base;
    public MilkDecorator(Coffee base) { this.base = base; }
    public String getDescription() { return base.getDescription() + ", Milk"; }
    public double getCost() { return base.getCost() + 0.5; }
}
```

Interview bonus: link decorator to customizable drinks.

---

# 8. Low Stock Observer

```java
public interface InventoryObserver {
    void onLowStock(String ingredient, int remaining);
}

public class Inventory {
    private final List<InventoryObserver> observers = new ArrayList<>();

    private void checkThresholds() {
        if (beans < 10) notify("beans", beans);
        if (cups < 5) notify("cups", cups);
    }
}
```

---

# 9. Design Patterns

| Pattern | Use |
|---------|-----|
| State | MachineState transitions |
| Singleton | One machine instance |
| Strategy | Payment (coin vs card) |
| Observer | Low inventory alerts |
| Factory | Create drink brewers |

---

# Interview Questions

## Two users press buttons simultaneously?

`synchronized` methods or lock — one transaction at a time for hardware.

## Partial payment timeout?

Scheduled task: if COLLECTING_PAYMENT idle > 60s → refund and reset to IDLE.

## How to add new drink?

Register in `prices` and `recipes` maps; no code change if using config file.

## Card payment failure?

Rollback to COLLECTING_PAYMENT, allow retry or cancel with refund.

---

# Quick Revision

```text
States: IDLE → SELECTED → PAYMENT → DISPENSING
Inventory.canMake + consume
synchronized for single-user hardware
Recipe maps ingredients per drink
Observer for low stock
```

---

*End of Day 33 Coffee Vending Machine LLD*
