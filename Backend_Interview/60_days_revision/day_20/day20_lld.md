# Day 20 — LLD: Splitwise

**Topics:** Expense Sharing · Balance Simplification · Class Design · Interview Flow

---

# 1. Problem Statement

Design Splitwise — an expense sharing application where:

- Users create groups and add expenses
- Expenses split equally, by exact amounts, or by percentage
- System tracks who owes whom
- Simplify debts to minimize transactions
- Settle up between users

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Create User | Register with name, email |
| Create Group | Add members to a group |
| Add Expense | Record who paid and how to split |
| Split Types | Equal, Exact, Percentage |
| View Balances | See net owed/owing per user |
| Simplify Debts | Minimize number of settlements |
| Settle Up | Mark debt as paid |

---

# 3. Non-Functional Requirements

- Support thousands of users per group
- Accurate balance calculations (use decimal/BigDecimal)
- Audit trail for all expenses

---

# 4. Core Entities

```text
User
  ├── userId, name, email
  └── balanceSheet (Map<User, Double>)

Group
  ├── groupId, name
  ├── members: List<User>
  └── expenses: List<Expense>

Expense
  ├── expenseId, description, amount
  ├── paidBy: User
  ├── splitType: EQUAL | EXACT | PERCENTAGE
  └── splits: List<Split>

Split
  ├── user: User
  └── amountOwed: BigDecimal

BalanceSheet
  └── Map<User, BigDecimal> (positive = owed to you, negative = you owe)
```

---

# 5. Class Diagram

```text
User 1 -------- * GroupMember
Group 1 -------- * GroupMember
Group 1 -------- * Expense
Expense 1 ------ * Split
User  1 -------- 1 BalanceSheet

class Expense {
  - id: String
  - amount: BigDecimal
  - paidBy: User
  - splitType: SplitType
  + validate(): boolean
}

class SplitStrategy {
  + split(amount, users): List<Split>
}

class EqualSplitStrategy implements SplitStrategy
class ExactSplitStrategy implements SplitStrategy
class PercentageSplitStrategy implements SplitStrategy
```

---

# 6. Split Strategy Pattern

```java
public interface SplitStrategy {
    List<Split> split(BigDecimal totalAmount, List<SplitRequest> requests);
}

public class EqualSplitStrategy implements SplitStrategy {
    @Override
    public List<Split> split(BigDecimal total, List<SplitRequest> requests) {
        int n = requests.size();
        BigDecimal share = total.divide(BigDecimal.valueOf(n), 2, RoundingMode.HALF_UP);
        return requests.stream()
                .map(r -> new Split(r.getUser(), share))
                .collect(Collectors.toList());
    }
}

public class ExactSplitStrategy implements SplitStrategy {
    @Override
    public List<Split> split(BigDecimal total, List<SplitRequest> requests) {
        BigDecimal sum = requests.stream()
                .map(SplitRequest::getAmount)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
        if (sum.compareTo(total) != 0) {
            throw new IllegalArgumentException("Split amounts must equal total");
        }
        return requests.stream()
                .map(r -> new Split(r.getUser(), r.getAmount()))
                .collect(Collectors.toList());
    }
}
```

---

# 7. Adding an Expense

```java
public class Group {
    private final String groupId;
    private final List<User> members;
    private final List<Expense> expenses;

    public Expense addExpense(String description, BigDecimal amount,
                              User paidBy, SplitStrategy strategy,
                              List<SplitRequest> splitRequests) {
        validateMembers(splitRequests);
        List<Split> splits = strategy.split(amount, splitRequests);

        Expense expense = new Expense(description, amount, paidBy, splits);
        expenses.add(expense);
        updateBalances(expense);
        return expense;
    }

    private void updateBalances(Expense expense) {
        User payer = expense.getPaidBy();
        for (Split split : expense.getSplits()) {
            User debtor = split.getUser();
            if (debtor.equals(payer)) continue;

            // debtor owes payer
            payer.getBalanceSheet().addCredit(debtor, split.getAmountOwed());
            debtor.getBalanceSheet().addDebit(payer, split.getAmountOwed());
        }
    }
}
```

---

# 8. Balance Sheet

```java
public class BalanceSheet {
    // positive = other user owes you, negative = you owe other user
    private final Map<User, BigDecimal> balances = new HashMap<>();

    public void addCredit(User fromUser, BigDecimal amount) {
        balances.merge(fromUser, amount, BigDecimal::add);
    }

    public void addDebit(User toUser, BigDecimal amount) {
        balances.merge(toUser, amount.negate(), BigDecimal::add);
    }

    public BigDecimal getBalanceWith(User other) {
        return balances.getOrDefault(other, BigDecimal.ZERO);
    }
}
```

---

# 9. Debt Simplification Algorithm

Goal: Minimize number of transactions to settle all debts.

## Approach — Greedy with Net Balance

1. Compute net balance for each user in group
2. Separate into creditors (positive) and debtors (negative)
3. Match largest creditor with largest debtor
4. Create settlement transaction, adjust balances
5. Repeat until all zero

```java
public List<Settlement> simplifyDebts(Map<User, BigDecimal> netBalances) {
    PriorityQueue<UserBalance> creditors = new PriorityQueue<>(
            (a, b) -> b.amount.compareTo(a.amount));
    PriorityQueue<UserBalance> debtors = new PriorityQueue<>(
            (a, b) -> a.amount.compareTo(b.amount));

    netBalances.forEach((user, amount) -> {
        if (amount.compareTo(BigDecimal.ZERO) > 0) {
            creditors.offer(new UserBalance(user, amount));
        } else if (amount.compareTo(BigDecimal.ZERO) < 0) {
            debtors.offer(new UserBalance(user, amount.abs()));
        }
    });

    List<Settlement> settlements = new ArrayList<>();
    while (!creditors.isEmpty() && !debtors.isEmpty()) {
        UserBalance creditor = creditors.poll();
        UserBalance debtor = debtors.poll();

        BigDecimal settleAmount = creditor.amount.min(debtor.amount);
        settlements.add(new Settlement(debtor.user, creditor.user, settleAmount));

        BigDecimal remainingCredit = creditor.amount.subtract(settleAmount);
        BigDecimal remainingDebt = debtor.amount.subtract(settleAmount);

        if (remainingCredit.compareTo(BigDecimal.ZERO) > 0) {
            creditors.offer(new UserBalance(creditor.user, remainingCredit));
        }
        if (remainingDebt.compareTo(BigDecimal.ZERO) > 0) {
            debtors.offer(new UserBalance(debtor.user, remainingDebt));
        }
    }
    return settlements;
}
```

---

# 10. Example Walkthrough

```text
Expense: $1200 dinner
Paid by: Alice
Split equally: Alice, Bob, Charlie, Diana ($300 each)

Net balances:
  Alice:  +900 (paid 1200, owes 300)
  Bob:    -300
  Charlie:-300
  Diana:  -300

Simplified settlements:
  Bob   → Alice: $300
  Charlie → Alice: $300
  Diana → Alice: $300

(Already minimal — 3 transactions)
```

With multiple expenses, simplification reduces transactions:

```text
Before: A owes B $50, B owes C $50, C owes A $30
After simplify: A owes B $20, C owes B $30
```

---

# 11. Settle Up

```java
public class SettlementService {

    public void settleUp(User from, User to, BigDecimal amount) {
        from.getBalanceSheet().addCredit(to, amount);
        to.getBalanceSheet().addDebit(from, amount);

        Transaction txn = new Transaction(from, to, amount, TransactionType.SETTLEMENT);
        transactionRepository.save(txn);
    }
}
```

---

# 12. Database Schema

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE
);

CREATE TABLE groups (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE group_members (
    group_id BIGINT,
    user_id BIGINT,
    PRIMARY KEY (group_id, user_id)
);

CREATE TABLE expenses (
    id BIGINT PRIMARY KEY,
    group_id BIGINT,
    paid_by BIGINT,
    amount DECIMAL(12,2),
    description VARCHAR(255),
    split_type ENUM('EQUAL','EXACT','PERCENTAGE'),
    created_at TIMESTAMP
);

CREATE TABLE expense_splits (
    expense_id BIGINT,
    user_id BIGINT,
    amount DECIMAL(12,2),
    PRIMARY KEY (expense_id, user_id)
);

CREATE TABLE settlements (
    id BIGINT PRIMARY KEY,
    from_user BIGINT,
    to_user BIGINT,
    amount DECIMAL(12,2),
    settled_at TIMESTAMP
);
```

---

# 13. Sequence Diagram — Add Expense

```text
User → GroupService: addExpense(groupId, amount, paidBy, splits)
GroupService → SplitStrategy: split(amount, requests)
SplitStrategy → GroupService: List<Split>
GroupService → BalanceSheet: updateBalances(expense)
GroupService → ExpenseRepository: save(expense)
GroupService → User: expense created
```

---

# 14. Edge Cases

| Case | Handling |
|------|----------|
| Split amounts don't sum to total | Reject with validation error |
| User not in group | Reject expense |
| Payer not included in split | Allow (payer may not share expense) |
| Rounding in equal split | Assign remainder to payer |
| Negative amounts | Reject |
| Settle more than owed | Reject or cap at owed amount |

---

# Interview Questions

## Why Strategy pattern for splits?

Three split types with different validation and calculation logic. Easy to add new split types (e.g., shares) without modifying Expense class.

---

## How to handle rounding in equal split?

$100 / 3 = $33.33 × 3 = $99.99. Assign extra $0.01 to payer or first participant.

---

## Simplify across groups or within group?

Within group only. Cross-group simplification requires user consent and is complex — mention as future feature.

---

## How to show "you owe" summary?

Aggregate all balance sheets for a user across groups. Sum net negative balances per counterparty.

---

## Concurrency — two expenses added simultaneously?

Use database transaction with row-level lock on balance records. Or optimistic locking with version field on balance sheet.

---

# Quick Revision

```text
SplitStrategy  → Equal, Exact, Percentage splits
BalanceSheet   → net owed between user pairs
Simplify debts → greedy match creditors/debtors
BigDecimal     → never use double for money
Settlement     → record debt payment
Group scope    → expenses and balances per group
```

---

*End of Day 20 Splitwise LLD*
