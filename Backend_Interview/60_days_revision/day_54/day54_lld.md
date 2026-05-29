# Day 54 — LLD Mock: ATM Machine (1 Hour)

**Week 8 · Friday · Timed 60 minutes**  
**Reference:** `day_17/day17_lld.md` (full content)

---

# Mock Rules

- **0–5 min:** Requirements + clarifications only  
- **5–20 min:** Class diagram + enums + state machine  
- **20–45 min:** Code core flows (PIN, withdraw, eject)  
- **45–55 min:** Extensions + tests discussion  
- **55–60 min:** Self-grade checklist  

No looking at notes until minute 60.

---

# Problem Statement (Read Once)

Design an ATM supporting:

- Card insert, PIN validation (3 attempts)  
- Balance inquiry, withdraw, deposit  
- Cash dispense, card eject  
- Integration with external **BankService**  

---

# Clarifying Questions (Expected)

| Question | Answer |
|----------|--------|
| Single ATM or network? | Single ATM, BankService is external API |
| Concurrent users? | One session at a time |
| Denominations? | Optional: dispense optimal bills |
| Print receipt? | Optional extension |
| Card retained on wrong PIN? | Yes after 3 failures |

---

# Class Diagram (Target)

```text
ATM
  ├── CardReader
  ├── CashDispenser
  ├── CashCollector
  ├── Keypad / Screen (optional interfaces)
  ├── currentCard, currentAccount
  └── ATMState (State pattern)

BankService (interface)
  ├── validateCard(cardNumber)
  ├── validatePin(cardNumber, pin)
  ├── getBalance(accountId)
  ├── withdraw(accountId, amount)
  └── deposit(accountId, amount)

Card, Account, Transaction
Enums: Operation { WITHDRAW, DEPOSIT, BALANCE }
```

---

# State Machine (Must Draw)

```text
IdleState
  insertCard → HasCardState

HasCardState
  enterPin (valid) → AuthenticatedState
  enterPin (invalid x3) → retain card → IdleState
  ejectCard → IdleState

AuthenticatedState
  selectOperation → process → stay Authenticated
  ejectCard → IdleState
```

---

# Core Code — State Interface

```java
interface ATMState {
    void insertCard(ATM atm, Card card);
    void enterPin(ATM atm, String pin);
    void selectOperation(ATM atm, Operation op, int amount);
    void ejectCard(ATM atm);
}
```

---

# AuthenticatedState — Withdraw

```java
class AuthenticatedState implements ATMState {
    @Override
    public void selectOperation(ATM atm, Operation op, int amount) {
        switch (op) {
            case WITHDRAW -> {
                if (amount <= 0) throw new IllegalArgumentException();
                boolean ok = atm.getBankService().withdraw(
                    atm.getCurrentAccountId(), amount);
                if (!ok) {
                    atm.showMessage("Insufficient funds");
                    return;
                }
                atm.getCashDispenser().dispense(amount);
                atm.printReceipt("WITHDRAW", amount);
            }
            case DEPOSIT -> {
                int cash = atm.getCashCollector().collect();
                atm.getBankService().deposit(atm.getCurrentAccountId(), cash);
            }
            case BALANCE -> {
                int bal = atm.getBankService().getBalance(atm.getCurrentAccountId());
                atm.showMessage("Balance: " + bal);
            }
        }
    }
}
```

---

# ATM Orchestrator

```java
class ATM {
    private ATMState state = new IdleState();
    private final BankService bankService;
    private final CashDispenser cashDispenser;
    private Card currentCard;
    private String currentAccountId;
    private int pinAttempts = 0;

    public void insertCard(Card card) { state.insertCard(this, card); }
    public void enterPin(String pin) { state.enterPin(this, pin); }
    public void selectOperation(Operation op, int amount) {
        state.selectOperation(this, op, amount);
    }
    public void ejectCard() { state.ejectCard(this); }

    void changeState(ATMState newState) { this.state = newState; }
    // getters/setters for collaborators
}
```

---

# Sequence — Successful Withdraw ₹2000

```text
Customer → ATM: insertCard
ATM → BankService: validateCard
ATM: state = HasCard
Customer → ATM: enterPin
ATM → BankService: validatePin → OK
ATM: state = Authenticated
Customer → ATM: selectOperation(WITHDRAW, 2000)
ATM → BankService: withdraw(account, 2000) → true
ATM → CashDispenser: dispense(2000)
Customer → ATM: ejectCard
ATM: state = Idle
```

---

# Extensions (If Time Remains)

| Extension | Design |
|-----------|--------|
| Optimal change | DP coin change on dispenser inventory |
| Chain of ATMs | Shared BankService, no change to state machine |
| Observer | Notify bank on each transaction event |
| Thread-safe | `synchronized` on ATM session methods |

---

# Common Mistakes (Avoid)

1. Putting all logic in `ATM` without State pattern  
2. Forgetting to reset `pinAttempts` on new card  
3. Dispensing cash before bank confirms withdraw  
4. No distinction between **ATM hardware** and **BankService**  
5. Missing `ejectCard` from all states  

---

# Scoring Rubric ( / 10)

| Points | Criteria |
|--------|----------|
| 2 | Clear requirements + actors |
| 2 | Class diagram with BankService boundary |
| 2 | State pattern with 3+ states |
| 2 | Withdraw + balance flow correct |
| 1 | PIN retry / card retain |
| 1 | Discuss testing (mock BankService) |

**Pass:** ≥ 7/10

---

# Post-Mock: Compare with day_17

Open `day_17/day17_lld.md` and note gaps. Rewrite state diagram from memory once.

---

*End of Day 54 — LLD Mock*
