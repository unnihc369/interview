# Day 46 — LLD: Parking Lot with Electric Charging (Full Code Sketch)

**Topics:** Multi-floor parking · Spot types · EV charging · Payment · Interview flow

---

# 1. Problem Statement

Design a parking lot supporting:

- Multiple floors, multiple spot types (Compact, Large, Handicapped, **Electric**)
- Park/unpark vehicles (Motorcycle, Car, Truck)
- **Electric vehicles** — assign EV spot with charging station
- Track charging session and bill on exit
- Display availability board
- Thread-safe spot allocation

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| enter | Issue ticket, assign spot (or queue if full) |
| exit | Unpark, calculate fee + charging cost |
| availability | Count free spots per type per floor |
| startCharging | Begin EV charging session |
| stopCharging | End session, compute kWh cost |

---

# 3. Core Enums & Entities

```java
public enum VehicleType { MOTORCYCLE, CAR, TRUCK }
public enum SpotType { COMPACT, LARGE, HANDICAPPED, ELECTRIC }
public enum SpotStatus { AVAILABLE, OCCUPIED, OUT_OF_SERVICE }
public enum ChargingStatus { IDLE, CHARGING, COMPLETE }

public abstract class Vehicle {
    protected String licensePlate;
    protected VehicleType type;
    public abstract SpotType preferredSpot();
}

public class Car extends Vehicle {
    public Car(String plate) { this.licensePlate = plate; this.type = VehicleType.CAR; }
    public SpotType preferredSpot() { return SpotType.COMPACT; }
}

public class ElectricCar extends Car {
    private final double batteryCapacityKwh;
    public ElectricCar(String plate, double capacity) {
        super(plate); this.batteryCapacityKwh = capacity;
    }
    @Override public SpotType preferredSpot() { return SpotType.ELECTRIC; }
}
```

---

# 4. Parking Spot & Charging Station

```java
public class ParkingSpot {
    private final String spotId;
    private final int floor;
    private final SpotType type;
    private SpotStatus status = SpotStatus.AVAILABLE;
    private Vehicle parkedVehicle;

    public synchronized boolean park(Vehicle v) {
        if (status != SpotStatus.AVAILABLE) return false;
        if (!canFit(v)) return false;
        this.parkedVehicle = v;
        this.status = SpotStatus.OCCUPIED;
        return true;
    }

    public synchronized Vehicle unpark() {
        Vehicle v = parkedVehicle;
        parkedVehicle = null;
        status = SpotStatus.AVAILABLE;
        return v;
    }

    private boolean canFit(Vehicle v) {
        return switch (v.getType()) {
            case MOTORCYCLE -> true;
            case CAR -> type == SpotType.COMPACT || type == SpotType.LARGE
                     || type == SpotType.ELECTRIC;
            case TRUCK -> type == SpotType.LARGE;
        };
    }
}

public class ChargingStation {
    private final String stationId;
    private final double ratePerKwh;
    private ChargingStatus status = ChargingStatus.IDLE;
    private Instant startTime;
    private double energyDeliveredKwh;

    public synchronized void start() {
        if (status == ChargingStatus.CHARGING) throw new IllegalStateException();
        status = ChargingStatus.CHARGING;
        startTime = Instant.now();
    }

    public synchronized BigDecimal stop() {
        status = ChargingStatus.COMPLETE;
        Duration d = Duration.between(startTime, Instant.now());
        energyDeliveredKwh = d.toMinutes() * 0.5; // simplified: 0.5 kWh/min
        return BigDecimal.valueOf(energyDeliveredKwh * ratePerKwh);
    }
}

public class ElectricSpot extends ParkingSpot {
    private final ChargingStation chargingStation;

    public ElectricSpot(String id, int floor, ChargingStation station) {
        super(id, floor, SpotType.ELECTRIC);
        this.chargingStation = station;
    }
    public ChargingStation getChargingStation() { return chargingStation; }
}
```

---

# 5. Ticket & Pricing

```java
public class ParkingTicket {
    private final String ticketId;
    private final Vehicle vehicle;
    private final ParkingSpot spot;
    private final Instant entryTime;
    private BigDecimal chargingCost = BigDecimal.ZERO;

    public ParkingTicket(String id, Vehicle v, ParkingSpot s) {
        ticketId = id; vehicle = v; spot = s; entryTime = Instant.now();
    }
}

public interface PricingStrategy {
    BigDecimal calculateFee(ParkingTicket ticket);
}

public class HourlyPricingStrategy implements PricingStrategy {
    private final BigDecimal ratePerHour;
    public BigDecimal calculateFee(ParkingTicket t) {
        long hours = Duration.between(t.getEntryTime(), Instant.now()).toHours() + 1;
        return ratePerHour.multiply(BigDecimal.valueOf(hours));
    }
}
```

---

# 6. Parking Lot (Orchestrator)

```java
public class ParkingLot {
    private static ParkingLot instance;
    private final List<ParkingFloor> floors;
    private final Map<String, ParkingTicket> activeTickets = new ConcurrentHashMap<>();
    private final PricingStrategy pricingStrategy;

    private ParkingLot(List<ParkingFloor> floors, PricingStrategy pricing) {
        this.floors = floors;
        this.pricingStrategy = pricing;
    }

    public static synchronized ParkingLot getInstance(List<ParkingFloor> floors,
                                                       PricingStrategy pricing) {
        if (instance == null) instance = new ParkingLot(floors, pricing);
        return instance;
    }

    public Optional<ParkingTicket> park(Vehicle vehicle) {
        for (ParkingFloor floor : floors) {
            Optional<ParkingSpot> spot = floor.findAvailableSpot(vehicle);
            if (spot.isPresent()) {
                ParkingSpot s = spot.get();
                if (s.park(vehicle)) {
                    ParkingTicket ticket = new ParkingTicket(
                        UUID.randomUUID().toString(), vehicle, s);
                    activeTickets.put(ticket.getTicketId(), ticket);

                    if (vehicle instanceof ElectricCar && s instanceof ElectricSpot ev) {
                        ev.getChargingStation().start();
                    }
                    return Optional.of(ticket);
                }
            }
        }
        return Optional.empty(); // lot full — could add wait queue
    }

    public BigDecimal exit(String ticketId) {
        ParkingTicket ticket = activeTickets.remove(ticketId);
        if (ticket == null) throw new IllegalArgumentException("Invalid ticket");

        ParkingSpot spot = ticket.getSpot();
        if (spot instanceof ElectricSpot ev) {
            BigDecimal chargeCost = ev.getChargingStation().stop();
            ticket.setChargingCost(chargeCost);
        }
        spot.unpark();

        return pricingStrategy.calculateFee(ticket).add(ticket.getChargingCost());
    }

    public Map<SpotType, Long> getAvailability() {
        Map<SpotType, Long> avail = new EnumMap<>(SpotType.class);
        for (ParkingFloor f : floors)
            f.addAvailability(avail);
        return avail;
    }
}
```

---

# 7. Parking Floor

```java
public class ParkingFloor {
    private final int floorNumber;
    private final List<ParkingSpot> spots;

    public Optional<ParkingSpot> findAvailableSpot(Vehicle vehicle) {
        SpotType preferred = vehicle.preferredSpot();
        // Prefer exact match, then fallback
        return spots.stream()
            .filter(s -> s.getStatus() == SpotStatus.AVAILABLE)
            .filter(s -> s.getType() == preferred || s.canFit(vehicle))
            .findFirst();
    }

    public void addAvailability(Map<SpotType, Long> map) {
        spots.stream()
            .filter(s -> s.getStatus() == SpotStatus.AVAILABLE)
            .forEach(s -> map.merge(s.getType(), 1L, Long::sum));
    }
}
```

---

# 8. Display Board (Observer Pattern)

```java
public interface AvailabilityObserver {
    void onAvailabilityChanged(Map<SpotType, Long> availability);
}

public class DisplayBoard implements AvailabilityObserver {
    public void onAvailabilityChanged(Map<SpotType, Long> avail) {
        System.out.println("=== AVAILABILITY ===");
        avail.forEach((type, count) ->
            System.out.println(type + ": " + count));
    }
}
```

---

# 9. Class Diagram

```text
ParkingLot (Singleton)
  ├── floors: List<ParkingFloor>
  ├── activeTickets: Map
  └── pricingStrategy: PricingStrategy

ParkingFloor
  └── spots: List<ParkingSpot>

ParkingSpot <|-- ElectricSpot
  └── chargingStation: ChargingStation

Vehicle <|-- Car <|-- ElectricCar
Vehicle <|-- Motorcycle
Vehicle <|-- Truck

PricingStrategy <|.. HourlyPricingStrategy
```

---

# 10. Usage Example

```java
public class Main {
    public static void main(String[] args) {
        ChargingStation cs1 = new ChargingStation("CS-1", 0.25);
        List<ParkingSpot> floor1Spots = List.of(
            new ParkingSpot("C1", 1, SpotType.COMPACT),
            new ElectricSpot("E1", 1, cs1),
            new ParkingSpot("L1", 1, SpotType.LARGE)
        );
        ParkingFloor floor1 = new ParkingFloor(1, floor1Spots);
        ParkingLot lot = ParkingLot.getInstance(
            List.of(floor1), new HourlyPricingStrategy(BigDecimal.valueOf(5)));

        ElectricCar ev = new ElectricCar("EV-123", 75.0);
        ParkingTicket ticket = lot.park(ev).orElseThrow();
        System.out.println("Parked. Ticket: " + ticket.getTicketId());

        BigDecimal total = lot.exit(ticket.getTicketId());
        System.out.println("Total due: $" + total);
    }
}
```

---

# Interview Questions

## Q1. Singleton for ParkingLot?

One physical lot — classic Singleton use case. Mention testability concern → inject factory in production code.

## Q2. EV spot for non-electric car?

Policy decision: allow (waste) or restrict (preferred). State your assumption upfront.

## Q3. Thread safety?

`ConcurrentHashMap` for tickets; `synchronized` on spot park/unpark; or `ReentrantLock` per floor.

## Q4. Strategy pattern where?

Pricing (hourly, flat, weekend), spot assignment (nearest exit, fill compact first).

## Q5. Extend for reservations?

Add `ReservationService` with time-window holds before `park()`.

---

# One-Line Revision

```text
Parking Lot LLD = Singleton lot + floors/spots + vehicle-spot matching + ElectricSpot/ChargingStation + ticket/pricing on exit — mention thread safety and Strategy for pricing.
```

---

*End of Day 46 LLD*
