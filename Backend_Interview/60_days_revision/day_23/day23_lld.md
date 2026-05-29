# Day 23 - LLD: Hotel Booking System

---

# 1. Requirements

## Functional

- Search hotels by city, date range, guests
- View room types and availability
- Book room (payment optional in basic version)
- Cancel booking
- Admin: add hotel, rooms, pricing

## Non-Functional

- Handle concurrent bookings (no double booking)
- Support peak pricing / seasonal rates

---

# 2. Core Entities

```text
Hotel 1 ---- * Room
Room 1 ---- * RoomType (or Room has type)
Booking * ---- 1 User
Booking * ---- 1 Room
Payment 0..1 ---- 1 Booking
```

---

# 3. Enums & Key Classes

```java
enum RoomType { SINGLE, DOUBLE, SUITE }
enum BookingStatus { PENDING, CONFIRMED, CANCELLED }
enum PaymentStatus { PENDING, SUCCESS, FAILED }

class Hotel {
    private Long id;
    private String name;
    private String city;
    private List<Room> rooms;
}

class Room {
    private Long id;
    private Long hotelId;
    private RoomType type;
    private int capacity;
    private BigDecimal basePrice;
}

class Booking {
    private Long id;
    private Long userId;
    private Long roomId;
    private LocalDate checkIn;
    private LocalDate checkOut;
    private BookingStatus status;
    private BigDecimal totalAmount;
}

class User {
    private Long id;
    private String name;
    private String email;
}
```

---

# 4. Service Layer

```java
interface HotelSearchService {
    List<Hotel> search(String city, LocalDate in, LocalDate out, int guests);
}

interface BookingService {
    Booking createBooking(BookingRequest req);
    void cancelBooking(Long bookingId);
}

interface AvailabilityService {
    boolean isAvailable(Long roomId, LocalDate in, LocalDate out);
    List<Room> getAvailableRooms(Long hotelId, LocalDate in, LocalDate out);
}
```

---

# 5. Availability Logic

Room is unavailable if any **confirmed** booking overlaps:

```text
Overlap if: NOT (newCheckOut <= existingCheckIn OR newCheckIn >= existingCheckOut)
```

```java
public boolean isAvailable(Long roomId, LocalDate in, LocalDate out) {
    return bookingRepository
        .findByRoomIdAndStatus(roomId, BookingStatus.CONFIRMED)
        .stream()
        .noneMatch(b -> datesOverlap(in, out, b.getCheckIn(), b.getCheckOut()));
}
```

---

# 6. Concurrency — Prevent Double Booking

## Problem

Two users book same room/dates simultaneously.

## Solutions

| Approach | Detail |
|----------|--------|
| DB unique constraint | `(room_id, check_in, check_out)` hard if ranges vary |
| Pessimistic lock | `SELECT FOR UPDATE` on room row |
| Optimistic lock | Version field on booking slot |
| Distributed lock | Redis lock per `roomId+dateRange` |

### Recommended for Interview

```java
@Transactional
public Booking createBooking(BookingRequest req) {
    lockService.lock("room:" + req.getRoomId() + ":" + req.getCheckIn());
    try {
        if (!availabilityService.isAvailable(...)) {
            throw new RoomNotAvailableException();
        }
        Booking booking = buildBooking(req);
        booking.setStatus(BookingStatus.CONFIRMED);
        return bookingRepository.save(booking);
    } finally {
        lockService.unlock(...);
    }
}
```

---

# 7. Pricing Strategy (Pattern)

```java
interface PricingStrategy {
    BigDecimal calculate(Room room, LocalDate in, LocalDate out);
}

class BasePricingStrategy implements PricingStrategy { ... }
class WeekendSurgePricing implements PricingStrategy { ... }
class FestivalPricing implements PricingStrategy { ... }
```

Factory selects strategy based on dates.

---

# 8. Class Diagram (Text)

```text
User 1 -------- * Booking
Booking * ------- 1 Room
Room * ---------- 1 Hotel

BookingService --> AvailabilityService
BookingService --> PaymentService
BookingService --> PricingStrategy
HotelSearchService --> HotelRepository
```

---

# 9. Sequence Diagram — Book Room

```text
Client -> BookingController: POST /bookings
BookingController -> BookingService: createBooking(req)
BookingService -> LockService: acquire(roomId, dates)
BookingService -> AvailabilityService: isAvailable()
AvailabilityService -> BookingRepository: findOverlapping()
BookingService -> PricingStrategy: calculate()
BookingService -> PaymentService: charge()
PaymentService --> BookingService: SUCCESS
BookingService -> BookingRepository: save()
BookingService -> LockService: release()
BookingService --> BookingController: BookingResponse
BookingController --> Client: 201 Created
```

---

# 10. API Design

```text
GET  /api/v1/hotels?city=Mumbai&checkIn=2026-06-01&checkOut=2026-06-03
GET  /api/v1/hotels/{id}/rooms?checkIn=&checkOut=
POST /api/v1/bookings
DELETE /api/v1/bookings/{id}
```

---

# 11. Edge Cases

| Case | Handling |
|------|----------|
| Check-out ≤ check-in | Validation error 400 |
| Cancel after check-in | Policy-based fee |
| Payment fails | Booking stays PENDING or auto-cancel |
| Partial date overlap | Treat as unavailable |
| Waitlist | Optional queue per room type |

---

# 12. Extension Topics (If Asked)

- **Inventory by room type** instead of individual room (allocate any free room of type)
- **Observer** for booking confirmation email
- **State pattern** for booking lifecycle
- **Rate limiting** on search API

---

# Common Interview Questions

## Q1. How to avoid double booking?

Transactional check + pessimistic/distributed lock + unique overlap validation.

## Q2. Hotel vs Room availability?

Search filters hotels with at least one available room matching guest count and dates.

## Q3. How to scale search?

Cache popular city searches in Redis; index DB on `(city, room_type)`.

## Q4. OTA vs direct booking?

OTA adds aggregator layer; inventory sync is hardest part (eventual consistency).

---

# One-Line Revision

```text
Hotel Booking = availability overlap check + lock + pricing strategy + booking state machine.
```

---

*End of Day 23 LLD*
