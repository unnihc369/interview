# Day 34 — LLD: Snake and Ladder Game

**Topics:** Board Design · Dice · Player Turn · Snake/Ladder Logic · Interview Flow

---

# 1. Problem Statement

Design a Snake and Ladder board game for `N` players:

- Board has 100 cells (1 to 100)
- Snakes: head → tail (move down)
- Ladders: foot → top (move up)
- Players roll dice (1-6), move forward
- Exact roll needed to win (if at 96, need 4 to reach 100)
- First player to reach cell 100 wins

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Add Players | 2-4 players with names |
| Roll Dice | Random 1-6 |
| Move Player | Apply dice + snakes/ladders |
| Win Detection | Reach exactly 100 |
| Display Board | Optional UI/print state |
| Configurable | Board size, snake/ladder positions |

---

# 3. Non-Functional Requirements

- Fair dice (uniform random)
- Single active player at a time
- Extensible board sizes (not only 100)

---

# 4. Core Entities

```text
Game
  ├── Board
  ├── List<Player>
  ├── currentPlayerIndex
  └── GameStatus (IN_PROGRESS, WON)

Board
  ├── size (default 100)
  ├── Map<Integer, Integer> snakes   // head → tail
  └── Map<Integer, Integer> ladders  // foot → top

Player
  ├── id, name
  └── position (0 = off-board, start at 0 or 1)

Dice
  └── roll() → 1..6

SnakeAndLadderService
  └── orchestrates game flow
```

---

# 5. Board Setup

```java
public class Board {
    private final int size;
    private final Map<Integer, Integer> snakes = new HashMap<>();
    private final Map<Integer, Integer> ladders = new HashMap<>();

    public Board(int size) {
        this.size = size;
        initDefaultSnakesAndLadders();
    }

    private void initDefaultSnakesAndLadders() {
        snakes.put(16, 6);
        snakes.put(47, 26);
        snakes.put(49, 11);
        snakes.put(56, 53);
        snakes.put(62, 19);
        snakes.put(64, 60);
        snakes.put(87, 24);
        snakes.put(93, 73);
        snakes.put(95, 75);
        snakes.put(98, 78);

        ladders.put(1, 38);
        ladders.put(4, 14);
        ladders.put(9, 31);
        ladders.put(21, 42);
        ladders.put(28, 84);
        ladders.put(36, 44);
        ladders.put(51, 67);
        ladders.put(71, 91);
        ladders.put(80, 100);
    }

    public int resolvePosition(int position) {
        if (snakes.containsKey(position)) return snakes.get(position);
        if (ladders.containsKey(position)) return ladders.get(position);
        return position;
    }

    public int getSize() { return size; }
}
```

---

# 6. Player & Dice

```java
public class Player {
    private final String id;
    private final String name;
    private int position;

    public Player(String id, String name) {
        this.id = id;
        this.name = name;
        this.position = 0;
    }

    public void setPosition(int position) { this.position = position; }
    public int getPosition() { return position; }
    public String getName() { return name; }
}

public class Dice {
    private final Random random = new Random();

    public int roll() {
        return random.nextInt(6) + 1;
    }
}
```

---

# 7. Game Logic

```java
public class SnakeAndLadderGame {
    private final Board board;
    private final List<Player> players;
    private final Dice dice;
    private int currentPlayerIndex;
    private GameStatus status;
    private Player winner;

    public SnakeAndLadderGame(Board board, List<Player> players, Dice dice) {
        if (players.size() < 2 || players.size() > 4) {
            throw new IllegalArgumentException("2-4 players required");
        }
        this.board = board;
        this.players = new ArrayList<>(players);
        this.dice = dice;
        this.currentPlayerIndex = 0;
        this.status = GameStatus.IN_PROGRESS;
    }

    public MoveResult playTurn() {
        if (status == GameStatus.WON) {
            throw new IllegalStateException("Game already over");
        }

        Player current = players.get(currentPlayerIndex);
        int roll = dice.roll();
        int newPos = calculateNewPosition(current.getPosition(), roll);

        if (newPos == -1) {
            return new MoveResult(current, roll, current.getPosition(),
                "Need exact roll to win");
        }

        int finalPos = board.resolvePosition(newPos);
        current.setPosition(finalPos);

        if (finalPos == board.getSize()) {
            status = GameStatus.WON;
            winner = current;
            return new MoveResult(current, roll, finalPos, "Wins!");
        }

        nextPlayer();
        return new MoveResult(current, roll, finalPos, "OK");
    }

    private int calculateNewPosition(int current, int roll) {
        int next = current + roll;
        if (next > board.getSize()) return -1; // overshoot — stay
        return next;
    }

    private void nextPlayer() {
        currentPlayerIndex = (currentPlayerIndex + 1) % players.size();
    }

    public Player getWinner() { return winner; }
    public Player getCurrentPlayer() { return players.get(currentPlayerIndex); }
}
```

---

# 8. MoveResult DTO

```java
public record MoveResult(
    Player player,
    int diceValue,
    int newPosition,
    String message
) {}
```

---

# 9. Game Controller (API Layer)

```java
@RestController
@RequestMapping("/games/snake-ladder")
public class GameController {
    private final Map<String, SnakeAndLadderGame> games = new ConcurrentHashMap<>();

    @PostMapping
    public String createGame(@RequestBody CreateGameRequest req) {
        String gameId = UUID.randomUUID().toString();
        List<Player> players = req.getPlayerNames().stream()
            .map(name -> new Player(UUID.randomUUID().toString(), name))
            .toList();
        games.put(gameId, new SnakeAndLadderGame(new Board(100), players, new Dice()));
        return gameId;
    }

    @PostMapping("/{gameId}/roll")
    public MoveResult roll(@PathVariable String gameId) {
        return games.get(gameId).playTurn();
    }
}
```

---

# 10. Class Diagram

```text
SnakeAndLadderGame
  ├── Board (snakes, ladders)
  ├── List<Player>
  ├── Dice
  └── GameStatus

Board ──uses──► Map<head/foot, tail/top>
Player ──has──► position
Dice ──► roll()
```

---

# 11. Design Patterns

| Pattern | Use |
|---------|-----|
| Facade | `SnakeAndLadderGame` hides board + turn logic |
| Strategy | Different dice (biased, double dice) |
| Factory | `BoardFactory` for custom boards |
| Observer | UI updates on move (optional) |
| Singleton | Not needed — multiple game instances |

---

# 12. Extensions (Interview)

| Extension | Design |
|-----------|--------|
| Double dice | Sum two rolls; double snake risk |
| Power-ups | Skip snake once — `PlayerPowerUp` decorator |
| 2-player vs AI | `Player` interface, `AIPlayer` strategy |
| Undo move | Command pattern stack |
| Bigger board | `Board(200)` with scaled snakes |

---

# 13. Sample Game Flow

```text
Players: Alice, Bob (positions 0)
Alice rolls 4 → pos 4 → ladder to 14
Bob rolls 6 → pos 6
Alice rolls 5 → pos 19
Bob rolls 3 → pos 9 → ladder to 31
...
Alice rolls 4 from 96 → 100 → WINS
```

---

# Interview Questions

## Why position 0 vs 1 start?

Design choice. Starting at 0 means first roll lands on dice value. Document clearly in interview.

## Snake and ladder on same cell?

Invalid config — validate on board init. Snake takes precedence if both exist (define rule).

## Thread safety for online multiplayer?

`playTurn()` synchronized per `gameId`. WebSocket pushes state to clients.

## How to test without randomness?

Inject `Dice` interface with `FixedDice(returns 4)` for deterministic tests.

## Overshoot rule variants?

Some rules: stay put if overshoot. Others: bounce back `(100 - (next - 100))`. Clarify with interviewer.

---

# Quick Revision

```text
Board: snakes (down), ladders (up), resolvePosition()
Exact roll to win from 96+
Turn rotation: currentPlayerIndex %
2-4 players, Dice roll 1-6
GameStatus WON when position == board.size
Inject Dice for testing
```

---

*End of Day 34 Snake and Ladder Game LLD*
