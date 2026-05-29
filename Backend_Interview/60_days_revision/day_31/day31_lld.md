# Day 31 — LLD: Parking Lot (Extended)

**Topics:** Multi-level · Multiple Vehicle Types · Spot Allocation · Payment · Interview Flow

---

# 1. Problem Statement (Extended)

Design a multi-floor parking lot supporting:

- Multiple vehicle types: Motorcycle, Car, Truck
- Different spot sizes: Compact, Large, Handicapped
- Entry/exit with ticket and hourly fee
- Display availability per floor
- Concurrent entry from multiple gates

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Park Vehicle | Assign nearest valid spot |
| Unpark Vehicle | Free spot, calculate fee |
| Availability | Spots free per type/floor |
| Multiple Entrances | Thread-safe assignment |
| Payment | Cash/card at exit |
| Full Lot | Reject when no compatible spot |

---

# 3. Core Entities

```text
Vehicle (abstract)
  ├── Motorcycle, Car, Truck
  └── canFitIn(SpotSize)

ParkingSpot
  ├── spotId, floor, SpotSize, isOccupied
  └── park(Vehicle) / unpark()

ParkingFloor
  ├── floorNumber
  └── List<ParkingSpot>

ParkingLot (Singleton)
  ├── List<ParkingFloor>
  ├── List<EntrancePanel>
  └── List<ExitPanel>

ParkingTicket
  ├── ticketId, vehicle, spot, entryTime
  └── calculateFee(exitTime)

PricingStrategy
  └── hourly rate by vehicle type
```

---

# 4. Spot Allocation Strategy

```java
public class SpotAllocationStrategy {
    public Optional<ParkingSpot> findSpot(ParkingLot lot, Vehicle vehicle) {
        for (ParkingFloor floor : lot.getFloors()) {
            for (ParkingSpot spot : floor.getSpots()) {
                if (!spot.isOccupied() && vehicle.canFitIn(spot.getSize())) {
                    return Optional.of(spot);
                }
            }
        }
        return Optional.empty();
    }
}
```

**Optimized:** index free spots by `SpotSize` in `Map<SpotSize, Queue<ParkingSpot>>`.

---

# 5. Vehicle-Spot Compatibility

```java
public enum SpotSize { COMPACT, LARGE, HANDICAPPED }

public abstract class Vehicle {
    abstract boolean canFitIn(SpotSize size);
}

public class Car extends Vehicle {
    boolean canFitIn(SpotSize size) {
        return size == SpotSize.COMPACT || size == SpotSize.LARGE;
    }
}

public class Truck extends Vehicle {
    boolean canFitIn(SpotSize size) {
        return size == SpotSize.LARGE; // or 2 adjacent spots — extension
    }
}
```

**Truck extension:** allocate 2 consecutive large spots; release both on exit.

---

# 6. Entry Flow

```java
public class ParkingLot {
    private final Map<String, ParkingTicket> activeTickets = new ConcurrentHashMap<>();

    public synchronized Optional<ParkingTicket> park(Vehicle vehicle) {
        Optional<ParkingSpot> spot = allocationStrategy.findSpot(this, vehicle);
        if (spot.isEmpty()) return Optional.empty();

        spot.get().park(vehicle);
        ParkingTicket ticket = new ParkingTicket(UUID.randomUUID().toString(),
            vehicle, spot.get(), Instant.now());
        activeTickets.put(ticket.getTicketId(), ticket);
        return Optional.of(ticket);
    }
}
```

`synchronized` or `ReentrantLock` per floor for interview; production uses DB row lock or Redis.

---

# 7. Exit & Payment

```java
public class ExitPanel {
    public PaymentReceipt processExit(String ticketId, PaymentMethod method) {
        ParkingTicket ticket = lot.getTicket(ticketId);
        Instant exitTime = Instant.now();
        BigDecimal fee = pricingStrategy.calculate(ticket, exitTime);
        Payment payment = paymentService.process(fee, method);
        ticket.getSpot().unpark();
        lot.removeTicket(ticketId);
        return new PaymentReceipt(fee, payment.getTransactionId());
    }
}
```

```java
public class HourlyPricingStrategy implements PricingStrategy {
    public BigDecimal calculate(ParkingTicket ticket, Instant exit) {
        long hours = Duration.between(ticket.getEntryTime(), exit).toHours() + 1;
        double rate = RATES.get(ticket.getVehicle().getType());
        return BigDecimal.valueOf(hours * rate);
    }
}
```

---

# 8. Display Board (Observer)

```java
public class AvailabilityDisplay implements ParkingObserver {
    public void onSpotChanged(ParkingLot lot) {
        Map<SpotSize, Long> free = lot.countFreeSpotsBySize();
        // update LED panel: COMPACT: 12, LARGE: 5
    }
}
```

`ParkingLot` notifies observers on park/unpark.

---

# 9. Class Diagram Summary

```text
ParkingLot 1──* ParkingFloor 1──* ParkingSpot
ParkingLot 1──* ParkingTicket
Vehicle <|-- Car, Motorcycle, Truck
SpotAllocationStrategy → used by ParkingLot
PricingStrategy <|-- HourlyPricingStrategy
PaymentService → process fee
```

---

# 10. Interview Extensions

| Question | Approach |
|----------|----------|
| VIP reserved spots | `SpotType.RESERVED`, check permit |
| Electric charging spots | subtype of spot, extra fee |
| Multi-tenant (mall vs office) | separate `ParkingLot` instances |
| Find vehicle by license plate | index `ticketByPlate` map |

---

# Interview Questions

## Singleton for ParkingLot?

Yes for one physical lot. Multiple lots → factory/registry of `ParkingLot` instances.

## Why strategy for allocation?

Nearest-first, lowest-floor-first, fill compact before large — swap strategies without changing `ParkingLot`.

## Handle lost ticket?

Admin override with license plate lookup + penalty fee.

---

# Quick Revision

```text
Vehicle → canFitIn(SpotSize)
SpotAllocationStrategy + free-spot index
ParkingTicket → entry time → fee on exit
synchronized / lock for concurrent park
Observer for display board
Truck = 2 spots (advanced)
```

---

*End of Day 31 Parking Lot Extended*
