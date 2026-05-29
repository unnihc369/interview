# Day 39 — LLD: Airline Check-in System

**Topics:** Requirements · Seat Map · Boarding Pass · State Machine · Interview Questions

---

# 1. Problem Statement

Design airline check-in: passengers select seats, print boarding passes, and baggage is tagged before flight departure.

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Search Booking | PNR + last name |
| Select Seat | From seat map; charge for premium |
| Check-in | Confirm; issue boarding pass |
| Baggage Drop | Weight validation, fee if overweight |
| Boarding Pass | QR code, gate, boarding group |

---

# 3. Core Classes

```text
Flight
  ├── id, departure, arrival, aircraft, status

Aircraft
  ├── rows, seatsPerRow, seatConfiguration

Seat
  ├── seatNumber, class (ECONOMY, BUSINESS), status (AVAILABLE, HELD, OCCUPIED)

Passenger
  ├── id, name, bookingRef, seatAssignment, checkInStatus

Booking
  ├── pnr, passengers, flightId

CheckInService
  ├── search(), selectSeat(), completeCheckIn(), dropBaggage()

BoardingPass
  ├── qrCode, gate, seat, boardingTime, group
```

---

# 4. Enums & State

```java
public enum CheckInStatus {
    NOT_STARTED, IN_PROGRESS, COMPLETED, CANCELLED
}

public enum SeatStatus {
    AVAILABLE, HELD, OCCUPIED, BLOCKED
}
```

---

# 5. Seat Selection — Sequence

```text
Passenger -> CheckInController.selectSeat(pnr, seatNo)
          -> CheckInService.selectSeat()
              -> lock seat row (optimistic @Version or Redis lock)
              -> if AVAILABLE → HELD (TTL 10 min)
              -> return seat map update
          -> on completeCheckIn → OCCUPIED, release holds
```

---

# 6. Seat Map Generation

```java
public class SeatMap {
    private final Map<String, Seat> seats;

    public static SeatMap fromAircraft(Aircraft ac) {
        Map<String, Seat> map = new LinkedHashMap<>();
        for (int row = 1; row <= ac.getRows(); row++) {
            for (char col = 'A'; col < 'A' + ac.getSeatsPerRow(); col++) {
                String num = row + String.valueOf(col);
                map.put(num, new Seat(num, ac.classForRow(row), SeatStatus.AVAILABLE));
            }
        }
        return new SeatMap(map);
    }
}
```

---

# 7. Baggage Rules

```java
public BaggageResult dropBaggage(String pnr, double weightKg) {
    double allowance = booking.getFareClass().getBaggageAllowance();
    if (weightKg > allowance) {
        BigDecimal fee = calculateExcessFee(weightKg - allowance);
        paymentService.charge(booking, fee);
    }
    return baggageTagService.issueTag(pnr, weightKg);
}
```

---

# 8. Boarding Pass Generation

```java
public BoardingPass completeCheckIn(String pnr, Long passengerId) {
    Passenger p = booking.getPassenger(passengerId);
    if (p.getSeat() == null) throw new SeatNotSelectedException();
    p.setCheckInStatus(CheckInStatus.COMPLETED);
    return BoardingPass.builder()
        .qrCode(qrService.encode(pnr, p.getId()))
        .gate(flight.getGate())
        .seat(p.getSeat())
        .boardingGroup(computeGroup(p))
        .build();
}
```

---

# 9. Design Patterns

| Pattern | Use |
|---------|-----|
| State | Check-in lifecycle |
| Factory | Boarding pass by airline format |
| Strategy | Fare class baggage rules |
| Observer | Notify gate changes |

---

# 10. Concurrency & Edge Cases

- Two passengers select same seat → one wins, other retries
- Check-in closes N hours before departure
- Exit row eligibility rules
- Connecting flights — multi-leg seat selection

---

# Interview Questions

## Q1. How hold seat temporarily?
Redis key `hold:{flight}:{seat}` TTL 600s; DB status HELD with `heldUntil`.

## Q2. Overbooking?
Airline may sell > capacity; volunteer bump list — mention as business rule extension.

## Q3. Real-time seat map updates?
WebSocket broadcast on seat state change per flight.

## Q4. Integration with DCS?
Departure Control System — check-in is front-end; sync to airline core via message queue.

## Q5. Idempotent check-in?
Client `Idempotency-Key` header; store processed keys 24h.

---

# Quick Revision

```text
Airline Check-in → PNR search, seat lock/hold, complete → boarding pass QR; baggage Strategy by fare class.
```

---

*End of Day 39 LLD*
