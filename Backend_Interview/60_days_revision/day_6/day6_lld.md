# Day 6 — LLD: Parking Lot System Design

## Topics
- Entity Classes
- Parking Spot Assignment
- Billing
- SOLID Principles
- Low Level Design Code

---

# 1. Requirements

Design Parking Lot System supporting:
- Multiple floors
- Different parking spot types
- Entry and Exit
- Ticket generation
- Billing system

---

# 2. Core Entities

```text
ParkingLot
   ↓
ParkingFloor
   ↓
ParkingSpot
   ↓
Vehicle
   ↓
Ticket
   ↓
Payment
```

---

# 3. Vehicle

```java
enum VehicleType {
    BIKE,
    CAR,
    TRUCK
}
```

---

## Vehicle Class

```java
class Vehicle {

    private String number;
    private VehicleType type;

    public Vehicle(String number,
                   VehicleType type) {

        this.number = number;
        this.type = type;
    }

    public VehicleType getType() {
        return type;
    }
}
```

---

# 4. Parking Spot

```java
abstract class ParkingSpot {

    private int id;
    private boolean occupied;

    public ParkingSpot(int id) {
        this.id = id;
    }

    public boolean isOccupied() {
        return occupied;
    }

    public void parkVehicle() {
        occupied = true;
    }

    public void removeVehicle() {
        occupied = false;
    }

    abstract boolean canParkVehicle(
            Vehicle vehicle);
}
```

---

# Car Parking Spot

```java
class CarSpot extends ParkingSpot {

    public CarSpot(int id) {
        super(id);
    }

    @Override
    boolean canParkVehicle(
            Vehicle vehicle) {

        return vehicle.getType()
                == VehicleType.CAR;
    }
}
```

---

# 5. Parking Floor

```java
class ParkingFloor {

    private List<ParkingSpot> spots;

    public ParkingFloor(
            List<ParkingSpot> spots) {

        this.spots = spots;
    }

    public ParkingSpot assignSpot(
            Vehicle vehicle) {

        for (ParkingSpot spot : spots) {

            if (!spot.isOccupied()
                && spot.canParkVehicle(vehicle)) {

                spot.parkVehicle();

                return spot;
            }
        }

        return null;
    }
}
```

---

# 6. Ticket

```java
class Ticket {

    private String ticketId;
    private LocalDateTime entryTime;
    private ParkingSpot parkingSpot;

    public Ticket(String ticketId,
                  ParkingSpot parkingSpot) {

        this.ticketId = ticketId;
        this.parkingSpot = parkingSpot;
        this.entryTime = LocalDateTime.now();
    }

    public LocalDateTime getEntryTime() {
        return entryTime;
    }
}
```

---

# 7. Billing Service

```java
class BillingService {

    public double calculateBill(
            Ticket ticket) {

        long hours = Duration.between(
                ticket.getEntryTime(),
                LocalDateTime.now())
                .toHours();

        return hours * 20;
    }
}
```

---

# 8. Parking Lot

```java
class ParkingLot {

    private List<ParkingFloor> floors;

    public ParkingLot(
            List<ParkingFloor> floors) {

        this.floors = floors;
    }

    public Ticket parkVehicle(
            Vehicle vehicle) {

        for (ParkingFloor floor : floors) {

            ParkingSpot spot =
                    floor.assignSpot(vehicle);

            if (spot != null) {

                return new Ticket(
                        UUID.randomUUID().toString(),
                        spot
                );
            }
        }

        return null;
    }
}
```

---

# 9. SOLID Principles Used

| Principle | Usage |
|---|---|
| SRP | Separate billing and parking |
| OCP | Add new spot types easily |
| LSP | CarSpot replaces ParkingSpot |
| ISP | Focused abstractions |
| DIP | High-level modules depend on interfaces |

---

# 10. Parking Spot Assignment Strategy

Can implement:
- Nearest Spot
- Random Spot
- VIP Spot

Using:
```text
Strategy Design Pattern
```

---

# 11. Interview Questions

## Why abstract ParkingSpot?
To support extensibility.

---

## How to scale parking lot?
- Multiple floors
- Distributed services
- DB persistence

---

## Which design patterns used?
- Strategy
- Factory
- Singleton

---

# Quick Revision

```text
Vehicle → Spot → Ticket → Billing
SOLID → scalable architecture
```

---

*End of Day 6 LLD*