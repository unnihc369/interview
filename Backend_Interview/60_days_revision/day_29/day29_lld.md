# Day 29 — LLD: Ride Sharing (Uber-like)

**Topics:** Matching · Trip Lifecycle · Pricing · Class Design · Interview Flow

---

# 1. Problem Statement

Design a ride-sharing platform where:

- Riders request rides (pickup, dropoff)
- Drivers go online and accept nearby requests
- System matches rider to driver
- Trip tracked from request → completion → payment
- Fare calculated by distance/time/surge

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Register Rider/Driver | Profile, vehicle info |
| Go Online/Offline | Driver availability |
| Request Ride | Rider specifies locations |
| Match Driver | Nearest available driver |
| Accept/Reject | Driver responds to request |
| Track Trip | En route, started, completed |
| Fare & Payment | Estimate, final charge, pay |
| Cancel | Rider or driver with policy |
| Rate | Post-trip rating |

---

# 3. Non-Functional Requirements

- Low latency matching (< few seconds)
- Geo-spatial queries for nearby drivers
- Handle concurrent requests for same driver
- Idempotent trip state transitions

---

# 4. Core Entities

```text
User (abstract)
  ├── Rider
  └── Driver (vehicle, rating, isOnline, currentLocation)

Location
  ├── latitude, longitude
  └── address (optional)

Ride
  ├── rideId, rider, driver (nullable until matched)
  ├── pickup, dropoff
  ├── status: REQUESTED | MATCHED | ACCEPTED | IN_PROGRESS | COMPLETED | CANCELLED
  ├── fare, distance, duration
  └── requestedAt, completedAt

Vehicle
  ├── type (SEDAN, SUV), plateNumber, capacity

Payment
  ├── rideId, amount, method, status
```

---

# 5. Class Diagram (Key Relationships)

```text
RideService
  ├── DriverMatchingService
  ├── FareCalculator
  ├── PaymentService
  └── NotificationService

DriverMatchingService
  └── GeoIndex (QuadTree / Redis GEO)

FareCalculator
  └── SurgePricingStrategy

interface PricingStrategy
  └── StandardPricingStrategy, SurgePricingStrategy
```

---

# 6. Ride State Machine

```text
REQUESTED ──match──► MATCHED ──accept──► ACCEPTED ──start──► IN_PROGRESS
    │                    │                    │
    └────cancel──────────┴────cancel───────────┴──► CANCELLED
                                                      │
IN_PROGRESS ──complete──► COMPLETED ──pay──► (Payment SUCCESS)
```

Invalid transitions rejected (e.g., cannot complete from REQUESTED).

---

# 7. Driver Matching (Simplified)

```java
public class DriverMatchingService {
    private final GeoIndex geoIndex;

    public Optional<Driver> findNearestDriver(Location pickup, double radiusKm) {
        List<Driver> nearby = geoIndex.findWithin(pickup, radiusKm)
            .stream()
            .filter(Driver::isOnline)
            .filter(d -> d.getCurrentRide() == null)
            .sorted(Comparator.comparingDouble(d -> distance(d.getLocation(), pickup)))
            .toList();

        return nearby.isEmpty() ? Optional.empty() : Optional.of(nearby.get(0));
    }
}
```

**Interview extension:** expand radius if no driver; queue request; priority for premium riders.

---

# 8. Fare Calculation

```java
public class FareCalculator {
    private static final double BASE_FARE = 50.0;
    private static final double PER_KM = 12.0;
    private static final double PER_MIN = 2.0;

    public BigDecimal estimate(Location pickup, Location dropoff, double surgeMultiplier) {
        double km = GeoUtils.distanceKm(pickup, dropoff);
        double mins = GeoUtils.estimateMinutes(km);
        double fare = BASE_FARE + km * PER_KM + mins * PER_MIN;
        return BigDecimal.valueOf(fare * surgeMultiplier).setScale(2, RoundingMode.HALF_UP);
    }
}
```

---

# 9. Concurrency — Accept Race

Two drivers cannot accept same ride:

```java
public synchronized boolean acceptRide(String rideId, String driverId) {
    Ride ride = rideRepository.findById(rideId);
    if (ride.getStatus() != RideStatus.MATCHED) return false;
    if (!ride.getDriver().getId().equals(driverId)) return false;
    ride.setStatus(RideStatus.ACCEPTED);
    rideRepository.save(ride);
    return true;
}
```

Production: optimistic locking (`@Version`) or distributed lock (Redis).

---

# 10. Key APIs

```text
POST   /riders/{id}/rides          → request ride
GET    /rides/{id}                 → status + driver location
POST   /rides/{id}/accept          → driver accepts
POST   /rides/{id}/start           → trip started
POST   /rides/{id}/complete        → trip ended, fare computed
POST   /rides/{id}/cancel          → cancel with reason
POST   /drivers/{id}/online        → go online
POST   /drivers/{id}/location      → update GPS
```

---

# 11. Design Patterns Used

| Pattern | Use |
|---------|-----|
| State | Ride status transitions |
| Strategy | Pricing (standard vs surge) |
| Observer | Notify rider on driver assigned |
| Factory | Payment gateway selection |
| Repository | Persistence abstraction |

---

# Interview Questions

## How to find nearest drivers at scale?

Redis GEO (`GEORADIUS`), PostGIS, or in-memory geo index per city shard. Partition by city/region.

## Surge pricing design?

Demand/supply ratio per geo-cell. `SurgePricingStrategy` reads multiplier from cache updated by analytics job.

## Split payment between platform and driver?

Separate `Payment` entity with `platformFee`, `driverPayout`. Settlement batch job.

## Rider sees driver location in real time?

WebSocket/SSE; driver app pushes location every N seconds to `LocationService` → subscribers on ride channel.

---

# Quick Revision

```text
Ride states: REQUESTED → MATCHED → ACCEPTED → IN_PROGRESS → COMPLETED
GeoIndex for driver matching
FareCalculator + SurgePricingStrategy
Accept race → lock or optimistic versioning
Feign/notification for async driver ping
```

---

*End of Day 29 Ride Sharing LLD*
