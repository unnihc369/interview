# Day 15 — LLD: Library Management System

**Topics:** Requirements · Class Design · Design Patterns · Interview Flow

---

# 1. Problem Statement

Design a Library Management System supporting:

- Book catalog management
- Member registration
- Book borrowing and return
- Fine calculation for overdue books
- Search books by title/author/ISBN

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Add/Remove Book | Librarian manages catalog |
| Register Member | Users get library card |
| Borrow Book | Member checks out available copy |
| Return Book | Member returns, fine if overdue |
| Search | Find books by title, author, ISBN |
| Reserve | Hold book when all copies borrowed |

---

# 3. Non-Functional Requirements

- Thread-safe concurrent borrow/return
- Extensible fine policy
- Audit trail for transactions

---

# 4. Core Entities

```text
Library
  ├── Catalog (books)
  ├── Members
  └── Transactions

Book
  ├── ISBN
  ├── Title, Author
  └── BookItems (physical copies)

BookItem
  ├── barcode
  ├── status (AVAILABLE, BORROWED, RESERVED, LOST)
  └── rack location

Member
  ├── memberId
  ├── name, contact
  └── borrowedItems[]

Librarian (extends Member or separate role)
```

---

# 5. Class Diagram

```text
Library 1 -------- * Book
Book     1 -------- * BookItem
Member   1 -------- * BookLending
BookItem 1 -------- 0..1 BookLending (active)

class Book {
  - isbn: String
  - title: String
  - author: String
  + isAvailable(): boolean
}

class BookItem {
  - barcode: String
  - status: BookStatus
  + checkout(member: Member): BookLending
  + checkin(): void
}

class Member {
  - memberId: String
  - name: String
  - maxBooks: int
  + borrow(book: Book): BookLending
  + returnBook(lending: BookLending): void
}

class BookLending {
  - lendingId: String
  - issueDate: LocalDate
  - dueDate: LocalDate
  - returnDate: LocalDate
  + calculateFine(): double
}
```

---

# 6. Enums

```java
public enum BookStatus {
    AVAILABLE, BORROWED, RESERVED, LOST, DAMAGED
}

public enum MemberStatus {
    ACTIVE, SUSPENDED, CLOSED
}
```

---

# 7. Key Operations

## Borrow Book Flow

```text
Member requests book
      ↓
Check member status (ACTIVE?)
      ↓
Check borrow limit not exceeded
      ↓
Find available BookItem for Book
      ↓
Create BookLending, set due date
      ↓
Update BookItem status → BORROWED
```

---

## Return Book Flow

```text
Member returns BookItem
      ↓
Find active BookLending
      ↓
Set returnDate = today
      ↓
Calculate fine if overdue
      ↓
Update BookItem status → AVAILABLE
      ↓
Notify reservation queue if any
```

---

# 8. Java Implementation Sketch

```java
public class Library {
    private final Map<String, Book> catalog = new ConcurrentHashMap<>();
    private final Map<String, Member> members = new ConcurrentHashMap<>();
    private final FineStrategy fineStrategy;

    public BookLending borrowBook(String memberId, String isbn) {
        Member member = members.get(memberId);
        if (member == null || member.getStatus() != MemberStatus.ACTIVE) {
            throw new IllegalStateException("Member not eligible");
        }
        if (member.getActiveLendings().size() >= member.getMaxBooks()) {
            throw new IllegalStateException("Borrow limit reached");
        }

        Book book = catalog.get(isbn);
        BookItem item = book.getAvailableItem()
                .orElseThrow(() -> new IllegalStateException("No copies available"));

        BookLending lending = item.checkout(member);
        member.addLending(lending);
        return lending;
    }

    public double returnBook(String lendingId) {
        BookLending lending = findLending(lendingId);
        lending.setReturnDate(LocalDate.now());
        double fine = fineStrategy.calculate(lending);
        lending.getBookItem().checkin();
        return fine;
    }
}
```

---

# 9. Design Patterns Used

| Pattern | Where | Why |
|---------|-------|-----|
| Singleton | Library instance | Single library per system |
| Strategy | FineStrategy | Different fine rules (daily/per-week) |
| Observer | Reservation notification | Notify when book available |
| Factory | BookItem creation | Consistent barcode generation |

---

# 10. Fine Strategy Pattern

```java
public interface FineStrategy {
    double calculate(BookLending lending);
}

public class DailyFineStrategy implements FineStrategy {
    private final double ratePerDay;

    @Override
    public double calculate(BookLending lending) {
        if (lending.getReturnDate().isBefore(lending.getDueDate())) return 0;
        long overdueDays = ChronoUnit.DAYS.between(
                lending.getDueDate(), lending.getReturnDate());
        return overdueDays * ratePerDay;
    }
}
```

---

# 11. Search Implementation

```java
public List<Book> searchByTitle(String title) {
    return catalog.values().stream()
            .filter(b -> b.getTitle().toLowerCase()
                    .contains(title.toLowerCase()))
            .collect(Collectors.toList());
}

public Optional<Book> searchByIsbn(String isbn) {
    return Optional.ofNullable(catalog.get(isbn));
}
```

Use **Catalog index** (HashMap by ISBN) for O(1) lookup. Title search is O(n) — acceptable for interview; mention inverted index for scale.

---

# 12. Concurrency Considerations

- Use `ConcurrentHashMap` for catalog and members
- Synchronize borrow on specific `BookItem` to prevent double checkout
- Optimistic locking with version field for high contention

```java
public synchronized BookLending checkout(Member member) {
    if (status != BookStatus.AVAILABLE) {
        throw new IllegalStateException("Item not available");
    }
    this.status = BookStatus.BORROWED;
    return new BookLending(this, member);
}
```

---

# 13. Sequence Diagram — Borrow Flow

```text
Member → Library: borrowBook(memberId, isbn)
Library → Member: validate status & limit
Library → Book: getAvailableItem()
Book → BookItem: find AVAILABLE copy
BookItem → BookLending: create(dueDate = today + 14 days)
Library → Member: addLending()
Library → Member: return BookLending
```

---

# Interview Questions

## How handle multiple copies of same book?

`Book` is logical entity (ISBN). `BookItem` is physical copy with unique barcode. Borrow operates on `BookItem`, not `Book`.

---

## What if member tries to borrow unavailable book?

Offer reservation. When copy returned, notify first member in reservation queue (FIFO).

---

## How to extend for e-books?

Add `DigitalBook` extending `Book` or separate `DigitalResource` with no `BookItem` — different checkout semantics (no return date, concurrent access limit).

---

## Singleton for Library — thread safe?

Use enum singleton or lazy initialization with double-checked locking. For interview, mention Spring `@Service` scope as modern alternative.

---

# Quick Revision

```text
Book vs BookItem → logical vs physical copy
BookLending → tracks issue/due/return dates
FineStrategy → pluggable overdue calculation
ConcurrentHashMap + synchronized checkout → thread safety
Reservation queue → Observer pattern on return
```

---

*End of Day 15 Library Management LLD*
