# Day 13 — Elevator System LLD

**Topics:** Direction · Requests · Door · State machine · SCAN algorithm · Interview points

---

# 1. Requirements (Clarify)

- Building has N floors and M elevators.
- Users press **up/down** hall buttons or **floor panel** inside cabin.
- Elevator moves up/down, stops at floors, opens/closes door.
- Handle multiple pending requests efficiently.
- States: IDLE, MOVING_UP, MOVING_DOWN, DOOR_OPEN.
- Optional: weight limit, maintenance mode, peak-hour behavior.

---

# 2. Class Diagram (Text)

```text
+-------------------+
| ElevatorController|
+-------------------+
| - elevators[]     |
| - requestQueue    |
+-------------------+
| + requestElevator()|
| + step()           |
+-------------------+
        | manages
        v
+-------------------+
| Elevator          |
+-------------------+
| - id              |
| - currentFloor    |
| - direction       |
| - state           |
| - upStops (Set)   |
| - downStops (Set) |
| - door            |
+-------------------+
| + addRequest()    |
| + move()          |
| + openDoor()      |
+-------------------+
        | has
        v
+-------------------+
| Door              |
+-------------------+
| - isOpen          |
+-------------------+
| + open()          |
| + close()         |
+-------------------+

+-------------------+
| Request           |
+-------------------+
| - sourceFloor     |
| - destFloor       |
| - direction       |
| - type (HALL/CABIN)|
+-------------------+

+----------------+<<enum>>
| Direction      |
+----------------+
| UP, DOWN, IDLE |
+----------------+

+----------------+<<enum>>
| ElevatorState  |
+----------------+
| IDLE, MOVING,  |
| DOOR_OPEN      |
+----------------+
```

---

# 3. Core Enums & Request

```java
enum Direction { UP, DOWN, IDLE }
enum ElevatorState { IDLE, MOVING, DOOR_OPEN }
enum RequestType { HALL, CABIN }

class Request {
    private final int sourceFloor;
    private final int destinationFloor;
    private final Direction direction; // for hall button
    private final RequestType type;

    // constructor, getters
}
```

---

# 4. Elevator — Stop Sets & Movement

```java
class Elevator {
    private final int id;
    private int currentFloor = 0;
    private Direction direction = Direction.IDLE;
    private ElevatorState state = ElevatorState.IDLE;
    private final TreeSet<Integer> upStops = new TreeSet<>();
    private final TreeSet<Integer> downStops = new TreeSet<>(Collections.reverseOrder());
    private final Door door = new Door();

    public void addRequest(Request req) {
        if (req.getDestinationFloor() > currentFloor) {
            upStops.add(req.getDestinationFloor());
        } else if (req.getDestinationFloor() < currentFloor) {
            downStops.add(req.getDestinationFloor());
        }
        if (req.getSourceFloor() != currentFloor) {
            if (req.getSourceFloor() > currentFloor) upStops.add(req.getSourceFloor());
            else downStops.add(req.getSourceFloor());
        }
        updateDirection();
    }

    private void updateDirection() {
        if (!upStops.isEmpty()) direction = Direction.UP;
        else if (!downStops.isEmpty()) direction = Direction.DOWN;
        else direction = Direction.IDLE;
    }

    public void step() {
        if (state == ElevatorState.DOOR_OPEN) {
            door.close();
            state = ElevatorState.MOVING;
            return;
        }

        if (direction == Direction.IDLE) return;

        if (direction == Direction.UP) {
            if (upStops.isEmpty()) {
                direction = downStops.isEmpty() ? Direction.IDLE : Direction.DOWN;
                return;
            }
            int next = upStops.first();
            moveToward(next);
            if (currentFloor == next) {
                upStops.remove(next);
                openAtFloor();
            }
        } else {
            if (downStops.isEmpty()) {
                direction = upStops.isEmpty() ? Direction.IDLE : Direction.UP;
                return;
            }
            int next = downStops.first();
            moveToward(next);
            if (currentFloor == next) {
                downStops.remove(next);
                openAtFloor();
            }
        }
        updateDirection();
    }

    private void moveToward(int target) {
        if (currentFloor < target) currentFloor++;
        else if (currentFloor > target) currentFloor--;
        state = ElevatorState.MOVING;
    }

    private void openAtFloor() {
        door.open();
        state = ElevatorState.DOOR_OPEN;
    }
}

class Door {
    private boolean open;
    void open() { open = true; }
    void close() { open = false; }
    boolean isOpen() { return open; }
}
```

---

# 5. Elevator Controller — Dispatch

```java
class ElevatorController {
    private final List<Elevator> elevators;

    public void requestElevator(Request req) {
        Elevator best = selectBestElevator(req);
        best.addRequest(req);
    }

    private Elevator selectBestElevator(Request req) {
        // Heuristic: minimize distance + same direction bonus
        return elevators.stream()
            .min(Comparator.comparingInt(e -> e.estimateCost(req)))
            .orElse(elevators.get(0));
    }

    public void runSimulation(int ticks) {
        for (int t = 0; t < ticks; t++) {
            for (Elevator e : elevators) {
                e.step();
            }
        }
    }
}
```

---

# 6. State Machine (Door + Movement)

```text
IDLE ──(request)──> MOVING
MOVING ──(reach stop)──> DOOR_OPEN
DOOR_OPEN ──(close)──> MOVING or IDLE
```

| State | Allowed actions |
|-------|-----------------|
| IDLE | Accept requests, assign direction |
| MOVING | Move one floor per tick/step |
| DOOR_OPEN | Load/unload passengers, then close |

---

# 7. Sequence Flow — Hall Request Up

```text
User -> ElevatorController: requestElevator(floor=3, UP)
ElevatorController -> Elevator: selectBest + addRequest
Elevator -> Elevator: upStops.add(3), direction=UP

loop each tick
  Client -> ElevatorController: runSimulation()
  ElevatorController -> Elevator: step()
  Elevator -> Elevator: move 2→3
  Elevator -> Door: open()
  Elevator --> User: arrived floor 3
end
```

---

# 8. Scheduling Algorithms (Interview)

| Algorithm | Idea |
|-----------|------|
| **FCFS** | First request first — simple, inefficient |
| **SCAN / Elevator** | Continue current direction, then reverse |
| **LOOK** | SCAN but don't go to last floor if no stops |

Production systems use LOOK-like behavior with **upStops** / **downStops** TreeSets.

---

# 9. Edge Cases

1. **Concurrent requests** — synchronize `addRequest` per elevator.
2. **Full capacity** — skip hall pickups when full.
3. **Same floor press** — open door immediately.
4. **Fire emergency** — override to ground floor (priority state).
5. **Multiple elevators** — avoid two elevators racing to same hall call.

---

# 10. Interview Points

1. Clarify single vs multi elevator.
2. Separate **Request**, **Elevator**, **Controller**, **Door**.
3. Use **TreeSet** for ordered stops in each direction.
4. State machine prevents moving while door open.
5. HLD extension: message queue, IoT sensors, central scheduler service.

---

# Quick Revision

```text
Elevator LLD = Request queue + direction + up/down stop sets + door state machine + controller dispatch.
```

---

*End of Day 13 LLD*
