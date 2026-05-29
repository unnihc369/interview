# Day 27 - LLD: Movie Ticket Booking System

---

# 1. Requirements

## Functional

- Browse movies, theaters, and show timings
- View seat map with availability
- Book seats for a show
- Cancel booking (before show start)
- Payment integration

## Non-Functional

- **No double booking** of same seat (critical)
- Handle concurrent users booking same show
- Low latency seat map load

---

# 2. Core Entities

```text
City 1 ---- * Theater
Theater 1 ---- * Screen
Screen 1 ---- * Show
Movie 1 ---- * Show
Show 1 ---- * Seat (layout)
Booking * ---- 1 User
Booking * ---- * Seat (via BookingSeat)
```

```java
enum SeatType { REGULAR, PREMIUM, VIP }
enum SeatStatus { AVAILABLE, LOCKED, BOOKED }
enum BookingStatus { PENDING, CONFIRMED, CANCELLED }
enum PaymentStatus { PENDING, SUCCESS, FAILED }

class Movie {
    private Long id;
    private String title;
    private int durationMins;
    private String language;
}

class Theater {
    private Long id;
    private String name;
    private Long cityId;
}

class Screen {
    private Long id;
    private Long theaterId;
    private String name;
    private int totalRows;
    private int seatsPerRow;
}

class Show {
    private Long id;
    private Long movieId;
    private Long screenId;
    private LocalDateTime startTime;
    private LocalDateTime endTime;
    private BigDecimal basePrice;
}

class Seat {
    private Long id;
    private Long screenId;
    private int row;
    private int col;
    private SeatType type;
}

class ShowSeat {
    private Long showId;
    private Long seatId;
    private SeatStatus status;
    private Long lockedByUserId;   // optional
    private LocalDateTime lockExpiry;
}

class Booking {
    private Long id;
    private Long userId;
    private Long showId;
    private BookingStatus status;
    private BigDecimal totalAmount;
    private List<Long> seatIds;
}

class User {
    private Long id;
    private String name;
    private String email;
}
```

---

# 3. Seat Locking Strategy (Critical)

## Problem

Two users select same seat at same time.

## Solution — Temporary Lock + Confirm

```text
1. User selects seats → LOCK for 5-10 minutes
2. User pays → CONFIRM (status BOOKED)
3. Lock expires without payment → release seats
```

```java
@Transactional
public LockResponse lockSeats(LockRequest req) {
    for (Long seatId : req.getSeatIds()) {
        ShowSeat ss = showSeatRepo.findByShowAndSeat(req.getShowId(), seatId)
            .orElseThrow();
        if (ss.getStatus() == BOOKED) {
            throw new SeatNotAvailableException(seatId);
        }
        if (ss.getStatus() == LOCKED && !ss.getLockedByUserId().equals(req.getUserId())
                && ss.getLockExpiry().isAfter(LocalDateTime.now())) {
            throw new SeatNotAvailableException(seatId);
        }
    }
    // acquire distributed lock per show
    lockService.lock("show:" + req.getShowId());
    try {
        // re-validate and lock
        for (Long seatId : req.getSeatIds()) {
            showSeatRepo.lockSeat(req.getShowId(), seatId, req.getUserId(),
                LocalDateTime.now().plusMinutes(10));
        }
    } finally {
        lockService.unlock("show:" + req.getShowId());
    }
    return new LockResponse(req.getSeatIds(), lockExpiry);
}
```

---

# 4. Pricing

```java
BigDecimal calculatePrice(Show show, List<Seat> seats) {
    return seats.stream()
        .map(seat -> show.getBasePrice().multiply(seat.getType().getMultiplier()))
        .reduce(BigDecimal.ZERO, BigDecimal::add);
}

// REGULAR=1.0, PREMIUM=1.5, VIP=2.0 multipliers
```

---

# 5. Service Layer

```java
interface ShowService {
    List<Show> getShows(Long movieId, Long cityId, LocalDate date);
    SeatMapResponse getSeatMap(Long showId);
}

interface BookingService {
    LockResponse lockSeats(LockRequest req);
    Booking confirmBooking(ConfirmRequest req);
    void cancelBooking(Long bookingId);
}

interface PaymentService {
    PaymentResult charge(Booking booking);
}
```

---

# 6. Confirm Booking Flow

```java
@Transactional
public Booking confirmBooking(ConfirmRequest req) {
    validateLocks(req.getUserId(), req.getShowId(), req.getSeatIds());
    Booking booking = createPendingBooking(req);

    PaymentResult payment = paymentService.charge(booking);
    if (!payment.isSuccess()) {
        releaseLocks(req);
        throw new PaymentFailedException();
    }

    markSeatsBooked(req.getShowId(), req.getSeatIds());
    booking.setStatus(CONFIRMED);
    return bookingRepository.save(booking);
}
```

---

# 7. Class Diagram

```text
Movie 1 ---- * Show
Theater 1 ---- * Screen
Screen 1 ---- * Show
Screen 1 ---- * Seat
Show + Seat → ShowSeat (availability state)

User 1 ---- * Booking
BookingService --> ShowSeatRepository
BookingService --> PaymentService
BookingService --> LockService
ShowService --> ShowRepository
```

---

# 8. Sequence Diagram — Book Seats

```text
Client -> BookingController: POST /shows/{id}/lock
BookingController -> BookingService: lockSeats(req)
BookingService -> LockService: acquire(showId)
BookingService -> ShowSeatRepo: validate + lock seats
BookingService --> Client: lockToken + expiry

Client -> BookingController: POST /bookings/confirm
BookingController -> BookingService: confirm(req)
BookingService -> PaymentService: charge()
PaymentService --> BookingService: SUCCESS
BookingService -> ShowSeatRepo: mark BOOKED
BookingService -> BookingRepo: save CONFIRMED
BookingController --> Client: 201 + ticket
```

---

# 9. Seat Map API Response

```json
{
  "showId": 101,
  "seats": [
    {"seatId": 1, "row": "A", "col": 1, "type": "REGULAR", "status": "AVAILABLE"},
    {"seatId": 2, "row": "A", "col": 2, "type": "REGULAR", "status": "LOCKED"},
    {"seatId": 3, "row": "A", "col": 3, "type": "PREMIUM", "status": "BOOKED"}
  ]
}
```

Cache seat map with short TTL; invalidate on lock/book.

---

# 10. API Design

```text
GET  /api/v1/movies?cityId=
GET  /api/v1/shows?movieId=&cityId=&date=
GET  /api/v1/shows/{showId}/seats
POST /api/v1/shows/{showId}/lock
POST /api/v1/bookings/confirm
DELETE /api/v1/bookings/{id}
GET  /api/v1/bookings/{id}
```

---

# 11. Concurrency Options (Interview Deep Dive)

| Approach | Pros | Cons |
|----------|------|------|
| DB row lock (`SELECT FOR UPDATE`) | Strong consistency | Contention on hot shows |
| Redis distributed lock | Fast, scalable | Extra infra |
| Optimistic locking (version) | Simple | Retries on conflict |
| Seat lock table with expiry | Clear model | Needs cleanup job |

**BookMyShow-style:** lock seats in Redis with TTL + async confirm to DB.

---

# 12. Scheduled Job — Release Expired Locks

```java
@Scheduled(fixedRate = 60000)
public void releaseExpiredLocks() {
    showSeatRepo.releaseLocksWhere(expiry < now());
}
```

---

# 13. Edge Cases

| Case | Handling |
|------|----------|
| Payment timeout | Release lock after TTL |
| Partial seat failure | All-or-nothing lock (transaction) |
| Cancel booking | Refund policy; mark seats AVAILABLE if before cutoff |
| Walk-in booking | Same flow without pre-lock |
| Group booking (adjacent seats) | Optional constraint in search |

---

# 14. Patterns Used

| Pattern | Where |
|---------|-------|
| State | SeatStatus, BookingStatus |
| Strategy | Pricing by seat type |
| Factory | Payment gateway selection |
| Observer | Booking confirmation email/SMS |
| Singleton | Lock service (careful with DI) |

---

# 15. Comparison with Similar LLD

| System | Key Difference |
|--------|----------------|
| Hotel Booking | Room = single unit; date range overlap |
| Movie Booking | Seat granularity; show-time specific; short lock window |
| Flight Booking | Seat + fare class + segment; complex pricing |

---

# Common Interview Questions

## Q1. How to prevent double booking?

Pessimistic/distributed lock + seat status state machine + transactional all-or-nothing lock.

## Q2. Why temporary lock before payment?

Hold seats during checkout; release if user abandons or payment fails.

## Q3. How to scale seat map for blockbuster?

Cache seat map in Redis; shard by showId; WebSocket for live updates (optional).

## Q4. DB schema for seats?

`show_seat(show_id, seat_id, status, locked_by, lock_expiry)` with unique `(show_id, seat_id)`.

## Q5. How is this different from Ticketmaster HLD?

LLD focuses on classes and booking flow; HLD adds CDN, queue for flash sales, multi-region.

---

# One-Line Revision

```text
Movie Booking = show seat map + timed lock + payment confirm + BOOKED state; concurrency is the core challenge.
```

---

*End of Day 27 LLD*
