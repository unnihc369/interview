# Day 26 - LLD: Car Rental System

---

# 1. Requirements

## Functional

- Browse vehicles by location, type, availability
- Reserve vehicle for date range
- Pick up and return vehicle
- Calculate rental price (daily + extras)
- Cancel reservation

## Non-Functional

- No double booking of same vehicle
- Support multiple rental locations
- Bill generation on return

---

# 2. Core Entities

```text
Location 1 ---- * Vehicle
Vehicle 1 ---- * Reservation
User 1 ---- * Reservation
Reservation 1 ---- 0..1 Invoice
```

```java
enum VehicleType { ECONOMY, SUV, LUXURY, TRUCK }
enum VehicleStatus { AVAILABLE, RESERVED, RENTED, MAINTENANCE }
enum ReservationStatus { PENDING, CONFIRMED, ACTIVE, COMPLETED, CANCELLED }

class Location {
    private Long id;
    private String city;
    private String address;
}

class Vehicle {
    private Long id;
    private String licensePlate;
    private VehicleType type;
    private VehicleStatus status;
    private Long locationId;
    private BigDecimal dailyRate;
}

class User {
    private Long id;
    private String name;
    private String licenseNumber;
    private String email;
}

class Reservation {
    private Long id;
    private Long userId;
    private Long vehicleId;
    private Long pickupLocationId;
    private Long returnLocationId;
    private LocalDateTime pickupTime;
    private LocalDateTime returnTime;
    private ReservationStatus status;
    private BigDecimal estimatedAmount;
}

class Invoice {
    private Long reservationId;
    private BigDecimal baseCharge;
    private BigDecimal lateFee;
    private BigDecimal total;
}
```

---

# 3. Search & Availability

```java
interface VehicleSearchService {
    List<Vehicle> search(SearchCriteria criteria);
}

class SearchCriteria {
    private Long locationId;
    private VehicleType type;
    private LocalDateTime pickup;
    private LocalDateTime dropoff;
}
```

Availability: vehicle has no overlapping **CONFIRMED/ACTIVE** reservation:

```java
boolean isAvailable(Long vehicleId, LocalDateTime start, LocalDateTime end) {
    return reservationRepository.findByVehicleId(vehicleId).stream()
        .filter(r -> r.getStatus() == CONFIRMED || r.getStatus() == ACTIVE)
        .noneMatch(r -> timesOverlap(start, end, r.getPickupTime(), r.getReturnTime()));
}
```

---

# 4. Pricing Strategy

```java
interface PricingStrategy {
    BigDecimal calculate(Vehicle vehicle, LocalDateTime pickup, LocalDateTime returnTime);
}

class DailyRatePricing implements PricingStrategy {
    public BigDecimal calculate(Vehicle v, LocalDateTime in, LocalDateTime out) {
        long days = ChronoUnit.DAYS.between(in.toLocalDate(), out.toLocalDate());
        days = Math.max(days, 1);
        return v.getDailyRate().multiply(BigDecimal.valueOf(days));
    }
}

class SeasonalSurgePricing implements PricingStrategy {
    // wrap base strategy, multiply during peak season
}
```

Factory selects strategy based on dates and vehicle type.

---

# 5. State Pattern — Reservation Lifecycle

```java
interface ReservationState {
    void confirm(Reservation r);
    void pickup(Reservation r);
    void complete(Reservation r);
    void cancel(Reservation r);
}

class PendingState implements ReservationState {
    public void confirm(Reservation r) { r.setStatus(CONFIRMED); }
    public void cancel(Reservation r) { r.setStatus(CANCELLED); }
    // pickup/complete throw IllegalStateException
}
```

Or simpler enum transitions with validation in service.

---

# 6. Service Layer

```java
interface ReservationService {
    Reservation createReservation(ReservationRequest req);
    void pickupVehicle(Long reservationId);
    Invoice returnVehicle(Long reservationId, ReturnDetails details);
    void cancel(Long reservationId);
}

interface VehicleService {
    List<Vehicle> getAvailable(SearchCriteria criteria);
    void updateStatus(Long vehicleId, VehicleStatus status);
}
```

---

# 7. Pickup & Return Flow

### Pickup

```text
Validate reservation CONFIRMED
Validate user license
Vehicle status → RENTED
Reservation status → ACTIVE
Record odometer / fuel level
```

### Return

```text
Validate reservation ACTIVE
Compute actual days + late fee + damage charges
Vehicle status → AVAILABLE (or MAINTENANCE)
Reservation status → COMPLETED
Generate Invoice
```

---

# 8. Concurrency — Double Booking Prevention

Same as hotel booking:

```java
@Transactional
public Reservation createReservation(ReservationRequest req) {
    lockService.lock("vehicle:" + req.getVehicleId());
    try {
        if (!availabilityService.isAvailable(...)) throw new VehicleNotAvailableException();
        Reservation r = buildReservation(req);
        r.setStatus(CONFIRMED);
        vehicleService.updateStatus(req.getVehicleId(), VehicleStatus.RESERVED);
        return reservationRepository.save(r);
    } finally {
        lockService.unlock(...);
    }
}
```

---

# 9. Class Diagram

```text
User 1 -------- * Reservation
Reservation * --- 1 Vehicle
Vehicle * ------- 1 Location
ReservationService --> AvailabilityService
ReservationService --> PricingStrategy
ReservationService --> VehicleService
ReturnService --> InvoiceService
```

---

# 10. Sequence — Create Reservation

```text
Client -> RentalController: POST /reservations
RentalController -> ReservationService: create(req)
ReservationService -> LockService: acquire(vehicleId)
ReservationService -> AvailabilityService: isAvailable()
ReservationService -> PricingStrategy: calculate()
ReservationService -> ReservationRepository: save(CONFIRMED)
ReservationService -> VehicleService: updateStatus(RESERVED)
ReservationService --> RentalController: ReservationResponse
RentalController --> Client: 201 Created
```

---

# 11. API Design

```text
GET  /api/v1/vehicles?locationId=&type=&pickup=&return=
POST /api/v1/reservations
POST /api/v1/reservations/{id}/pickup
POST /api/v1/reservations/{id}/return
DELETE /api/v1/reservations/{id}
GET  /api/v1/reservations/{id}/invoice
```

---

# 12. Edge Cases

| Case | Handling |
|------|----------|
| One-way rental (different return location) | Transfer fee in pricing |
| Late return | Per-hour/day late fee |
| Vehicle breakdown | Swap vehicle, extend reservation |
| Cancel before pickup | Refund policy by cancellation window |
| Maintenance block | Exclude vehicle from search |

---

# 13. Extension — Fleet Management

- **Factory** for pricing by vehicle type
- **Observer** for reservation confirmation SMS
- **Strategy** for insurance add-ons

---

# Common Interview Questions

## Q1. Car rental vs hotel booking?

Similar availability model; rental adds vehicle state (odometer, fuel), pickup/return locations, one-way fees.

## Q2. Search by type vs specific vehicle?

Search returns available vehicles of type; assign specific vehicle on confirm.

## Q3. How to handle different drop-off location?

Track pickup and return location IDs; apply relocation fee in pricing.

## Q4. Scale search across cities?

Index `(location_id, type, status)`; cache popular location searches.

---

# One-Line Revision

```text
Car Rental = vehicle availability + reservation state machine + pricing strategy + pickup/return billing.
```

---

*End of Day 26 LLD*
