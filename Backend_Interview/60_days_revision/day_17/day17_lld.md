# Day 17 — LLD: ATM Machine

**Topics:** Requirements · State Pattern · Class Design · Transaction Flow

---

# 1. Problem Statement

Design an ATM system that supports:

- Card insertion and PIN validation
- Balance inquiry
- Cash withdrawal
- Cash deposit
- PIN change
- Transaction history

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Insert Card | Read card, validate with bank |
| Enter PIN | 3 attempts, block card on failure |
| Withdraw | Dispense cash if sufficient balance |
| Deposit | Accept cash, update balance |
| Balance Inquiry | Show current balance |
| Eject Card | Return card, end session |

---

# 3. Actors

```text
Customer  → uses ATM
Bank      → validates card/PIN, manages accounts
ATM       → hardware interface (card reader, cash dispenser, keypad)
```

---

# 4. Core Classes

```text
ATM
  ├── CardReader
  ├── CashDispenser
  ├── CashCollector (deposit)
  ├── Keypad
  ├── Screen
  └── ATMState (State Pattern)

BankService (external)
  ├── validateCard()
  ├── validatePin()
  ├── withdraw()
  ├── deposit()
  └── getBalance()

Card
  ├── cardNumber
  ├── bankId
  └── pin (hashed)

Account
  ├── accountNumber
  ├── balance
  └── transactionHistory
```

---

# 5. State Pattern for ATM

```text
IdleState
    ↓ insert card
HasCardState
    ↓ enter PIN
AuthenticatedState
    ↓ select operation
    ├── WithdrawState
    ├── DepositState
    └── BalanceInquiryState
    ↓ eject card
IdleState
```

---

# State Implementation

```java
interface ATMState {
    void insertCard(ATM atm, Card card);
    void enterPin(ATM atm, String pin);
    void selectOperation(ATM atm, Operation op);
    void ejectCard(ATM atm);
}

class IdleState implements ATMState {
    public void insertCard(ATM atm, Card card) {
        atm.setCurrentCard(card);
        atm.setState(new HasCardState());
        atm.display("Enter PIN");
    }
    public void enterPin(ATM atm, String pin) {
        atm.display("Insert card first");
    }
    // other methods: invalid in idle
}

class HasCardState implements ATMState {
    private int attempts = 0;

    public void enterPin(ATM atm, String pin) {
        if (atm.getBankService().validatePin(atm.getCurrentCard(), pin)) {
            atm.setState(new AuthenticatedState());
            atm.display("Select operation");
        } else {
            attempts++;
            if (attempts >= 3) {
                atm.getBankService().blockCard(atm.getCurrentCard());
                atm.ejectCard();
            } else {
                atm.display("Invalid PIN. Attempts left: " + (3 - attempts));
            }
        }
    }
}
```

---

# 6. Withdrawal Flow

```text
Customer selects Withdraw → enters amount
        ↓
ATM checks: amount <= balance AND amount <= dispenser capacity
        ↓
BankService.withdraw(account, amount)
        ↓
CashDispenser.dispense(amount)
        ↓
Print receipt, log transaction
```

---

# Java Implementation Sketch

```java
public class ATM {
    private ATMState state = new IdleState();
    private Card currentCard;
    private final BankService bankService;
    private final CashDispenser cashDispenser;

    public void withdraw(double amount) {
        if (!(state instanceof AuthenticatedState)) {
            throw new IllegalStateException("Authenticate first");
        }
        Account account = bankService.getAccount(currentCard);
        if (account.getBalance() < amount) {
            display("Insufficient funds");
            return;
        }
        if (!cashDispenser.canDispense(amount)) {
            display("ATM cannot dispense this amount");
            return;
        }
        bankService.withdraw(account, amount);
        cashDispenser.dispense(amount);
        display("Dispensed: $" + amount);
    }
}
```

---

# 7. Cash Dispenser — Greedy Algorithm

Dispense using minimum notes:

```java
public class CashDispenser {
    private Map<Integer, Integer> notes; // denomination → count

    public boolean canDispense(double amount) {
        return computeNotes((int) amount) != null;
    }

    public void dispense(double amount) {
        Map<Integer, Integer> toDispense = computeNotes((int) amount);
        if (toDispense == null) throw new IllegalStateException("Cannot dispense");
        toDispense.forEach((denom, count) ->
                notes.merge(denom, -count, Integer::sum));
    }

    private Map<Integer, Integer> computeNotes(int amount) {
        // Greedy: try largest denominations first
        // Return null if exact change impossible
    }
}
```

---

# 8. Transaction Logging

```java
public class Transaction {
    private String transactionId;
    private TransactionType type;  // WITHDRAW, DEPOSIT, BALANCE_INQUIRY
    private double amount;
    private LocalDateTime timestamp;
    private TransactionStatus status;
}

public enum TransactionType {
    WITHDRAW, DEPOSIT, BALANCE_INQUIRY, PIN_CHANGE
}
```

---

# 9. Singleton ATM

One ATM instance per physical machine:

```java
public enum ATMInstance {
    INSTANCE;
    private final ATM atm = new ATM(/* dependencies */);
    public ATM getAtm() { return atm; }
}
```

---

# 10. Sequence Diagram — Withdraw

```text
Customer → ATM: insertCard(card)
ATM → BankService: validateCard(card)
ATM → Customer: "Enter PIN"
Customer → ATM: enterPin(pin)
ATM → BankService: validatePin(card, pin)
Customer → ATM: withdraw(200)
ATM → BankService: getBalance(account)
ATM → BankService: withdraw(account, 200)
ATM → CashDispenser: dispense(200)
ATM → Customer: receipt + cash
Customer → ATM: ejectCard()
```

---

# 11. Concurrency & Safety

- **Single session:** Only one customer at a time (state machine enforces)
- **Double withdrawal:** Synchronize on account during bank transaction
- **Hardware failure:** Rollback bank debit if dispense fails (two-phase commit pattern)

```java
public void withdrawSafe(Account account, double amount) {
    bankService.debit(account, amount);  // phase 1
    try {
        cashDispenser.dispense(amount);  // phase 2
    } catch (Exception e) {
        bankService.credit(account, amount);  // rollback
        throw e;
    }
}
```

---

# Interview Questions

## Why State pattern for ATM?

ATM behavior changes completely based on current state (idle vs authenticated). State pattern eliminates complex if-else and makes invalid operations impossible per state.

## How handle concurrent ATMs on same account?

Bank service uses optimistic/pessimistic locking on account row. Database transaction with `SELECT FOR UPDATE` or version field.

## What if cash dispense fails after debit?

Implement compensating transaction — credit account back. Log failed dispense for manual reconciliation.

## How to support different card types?

Strategy pattern for card validation (Visa, Mastercard). `CardValidator` interface with implementations.

---

# Quick Revision

```text
State Pattern → Idle → HasCard → Authenticated → Operation
BankService   → external account operations
CashDispenser → greedy note algorithm
3 PIN attempts → block card
Two-phase commit → debit then dispense, rollback on failure
Singleton       → one ATM instance per machine
```

---

*End of Day 17 ATM Machine LLD*
