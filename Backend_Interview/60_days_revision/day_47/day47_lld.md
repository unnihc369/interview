# Day 47 — LLD: Library Concurrent Reservations (Thread Safety)

**Topics:** Book reservation · Concurrent access · Locking strategies · Interview Q&A

---

# 1. Problem Statement

Design a library management system where:

- Members search and **reserve** books (limited copies)
- Multiple members may try to reserve the **last copy** concurrently
- Support waitlist when all copies reserved
- Cancel reservation / auto-expire holds
- Thread-safe without data corruption

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| search | Find books by title/author/ISBN |
| reserve | Hold a copy for member (24h) |
| cancel | Release reservation |
| borrow | Check out reserved/available copy |
| return | Return copy, notify waitlist |
| waitlist | Queue when no copies available |

---

# 3. Core Entities

```java
public class Book {
    private String isbn;
    private String title;
    private String author;
    private int totalCopies;
    private int availableCopies;
}

public class BookCopy {
    private String copyId;
    private String isbn;
    private CopyStatus status; // AVAILABLE, RESERVED, BORROWED
}

public enum CopyStatus { AVAILABLE, RESERVED, BORROWED }

public class Reservation {
    private String reservationId;
    private String memberId;
    private String copyId;
    private Instant expiresAt;
    private ReservationStatus status;
}

public class Member {
    private String memberId;
    private String name;
    private int maxActiveReservations = 5;
}
```

---

# 4. Concurrency Challenge

```text
Thread A: reserve("ISBN-123") → sees 1 copy available
Thread B: reserve("ISBN-123") → sees 1 copy available
Both succeed → double booking (BUG)
```

**Fix:** atomic check-and-decrement or per-book locking.

---

# 5. Thread-Safe Design — Per-Book Lock

```java
@Service
public class ReservationService {

    private final BookRepository bookRepo;
    private final BookCopyRepository copyRepo;
    private final ReservationRepository reservationRepo;
    private final WaitlistService waitlistService;

    // Fine-grained lock per ISBN
    private final ConcurrentHashMap<String, Object> isbnLocks = new ConcurrentHashMap<>();

    private Object lockFor(String isbn) {
        return isbnLocks.computeIfAbsent(isbn, k -> new Object());
    }

    public ReservationResult reserve(String memberId, String isbn) {
        synchronized (lockFor(isbn)) {
            // Re-check under lock
            validateMemberLimit(memberId);

            Optional<BookCopy> available = copyRepo
                .findFirstByIsbnAndStatus(isbn, CopyStatus.AVAILABLE);

            if (available.isPresent()) {
                BookCopy copy = available.get();
                copy.setStatus(CopyStatus.RESERVED);
                copyRepo.save(copy);

                Reservation res = Reservation.create(memberId, copy.getCopyId(),
                    Instant.now().plus(Duration.ofHours(24)));
                reservationRepo.save(res);
                return ReservationResult.success(res);
            }

            // No copy — add to waitlist
            waitlistService.enqueue(memberId, isbn);
            return ReservationResult.waitlisted(isbn);
        }
    }

    public void cancel(String memberId, String reservationId) {
        Reservation res = reservationRepo.findById(reservationId).orElseThrow();
        if (!res.getMemberId().equals(memberId)) throw new UnauthorizedException();

        synchronized (lockFor(res.getIsbn())) {
            res.setStatus(ReservationStatus.CANCELLED);
            reservationRepo.save(res);

            BookCopy copy = copyRepo.findById(res.getCopyId()).orElseThrow();
            copy.setStatus(CopyStatus.AVAILABLE);
            copyRepo.save(copy);

            waitlistService.notifyNext(res.getIsbn());
        }
    }
}
```

---

# 6. Alternative: Database-Level Concurrency

```java
@Repository
public interface BookCopyRepository extends JpaRepository<BookCopy, String> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT c FROM BookCopy c WHERE c.isbn = :isbn AND c.status = 'AVAILABLE'")
    Optional<BookCopy> findAvailableForUpdate(@Param("isbn") String isbn);
}

@Transactional
public ReservationResult reserve(String memberId, String isbn) {
    BookCopy copy = copyRepo.findAvailableForUpdate(isbn).orElse(null);
    if (copy == null) {
        waitlistService.enqueue(memberId, isbn);
        return ReservationResult.waitlisted(isbn);
    }
    copy.setStatus(CopyStatus.RESERVED);
    return ReservationResult.success(createReservation(memberId, copy));
}
```

**Production preference:** DB pessimistic lock or optimistic `@Version` — survives multi-instance deployment.

---

# 7. Optimistic Locking with @Version

```java
@Entity
public class BookCopy {
    @Id private String copyId;
    @Version private Long version;
    private CopyStatus status;
}

// Concurrent update → OptimisticLockException → retry or fail gracefully
@Retryable(retryFor = OptimisticLockException.class, maxAttempts = 3)
public ReservationResult reserveOptimistic(String memberId, String isbn) { ... }
```

---

# 8. Waitlist Service (Thread-Safe Queue)

```java
@Service
public class WaitlistService {
    // Per-ISBN waitlist queue
    private final ConcurrentHashMap<String, ConcurrentLinkedQueue<String>> waitlists
        = new ConcurrentHashMap<>();

    public void enqueue(String memberId, String isbn) {
        waitlists.computeIfAbsent(isbn, k -> new ConcurrentLinkedQueue<>())
                 .offer(memberId);
    }

    public void notifyNext(String isbn) {
        ConcurrentLinkedQueue<String> queue = waitlists.get(isbn);
        if (queue == null) return;

        String nextMember = queue.poll();
        if (nextMember != null) {
            notificationService.send(nextMember,
                "A copy of " + isbn + " is available. Reserve within 24h.");
        }
    }
}
```

---

# 9. Scheduled Expiration

```java
@Component
public class ReservationExpiryJob {

    @Scheduled(fixedRate = 300_000) // every 5 min
    @Transactional
    public void expireReservations() {
        List<Reservation> expired = reservationRepo
            .findByStatusAndExpiresAtBefore(ACTIVE, Instant.now());

        for (Reservation res : expired) {
            synchronized (lockFor(res.getIsbn())) {
                res.setStatus(EXPIRED);
                BookCopy copy = copyRepo.findById(res.getCopyId()).orElseThrow();
                copy.setStatus(CopyStatus.AVAILABLE);
                copyRepo.save(copy);
                reservationRepo.save(res);
                waitlistService.notifyNext(res.getIsbn());
            }
        }
    }
}
```

---

# 10. Class Diagram

```text
LibraryService
  ├── BookRepository
  ├── BookCopyRepository
  ├── ReservationRepository
  ├── ReservationService (per-ISBN lock)
  └── WaitlistService (ConcurrentLinkedQueue per ISBN)

ReservationService ──uses──▶ NotificationService
BookCopy ──status──▶ AVAILABLE | RESERVED | BORROWED
```

---

# 11. Lock Strategy Comparison

| Strategy | Pros | Cons |
|----------|------|------|
| `synchronized` per ISBN | Simple, in-process | Doesn't work multi-JVM |
| DB pessimistic lock | Multi-instance safe | DB connection held |
| Optimistic @Version | High read throughput | Retries on conflict |
| Distributed lock (Redis) | Multi-instance | Extra infra |

**Interview answer:** Start with DB lock; mention Redis Redisson for scale-out.

---

# Interview Questions

## Q1. Why not one global lock?

Serializes all reservations — throughput bottleneck. Per-ISBN lock allows concurrent reservations on different books.

## Q2. Double booking with JPA default?

Two transactions both read AVAILABLE, both update — lost update. Need `@Lock` or `@Version`.

## Q3. Waitlist fairness?

`ConcurrentLinkedQueue` = FIFO. Priority members? Separate `PriorityBlockingQueue` per ISBN.

## Q4. Idempotent reserve?

Same member double-clicks reserve — check existing active reservation for same ISBN before creating new.

## Q5. How to test concurrency?

`CountDownLatch` + multiple threads + assert only one reservation succeeds for last copy.

```java
@Test
void onlyOneReservationForLastCopy() throws Exception {
    int threads = 10;
    CountDownLatch start = new CountDownLatch(1);
    CountDownLatch done = new CountDownLatch(threads);
    AtomicInteger successes = new AtomicInteger();

    for (int i = 0; i < threads; i++) {
        final String member = "member-" + i;
        Thread.startVirtualThread(() -> {
            try {
                start.await();
                if (reservationService.reserve(member, "ISBN-1").isSuccess())
                    successes.incrementAndGet();
            } finally { done.countDown(); }
        });
    }
    start.countDown();
    done.await();
    assertEquals(1, successes.get());
}
```

---

# One-Line Revision

```text
Library reservations = per-ISBN locking + atomic copy status transition + waitlist queue; production uses DB pessimistic/optimistic lock for multi-instance safety.
```

---

*End of Day 47 LLD*
