# Day 12 — Tic-Tac-Toe LLD

**Topics:** Board · Player · Game flow · Win detection · Class diagram · Sequence flow · Interview points

---

# 1. Requirements

- 3×3 board (extendable to N×N).
- Two players X and O alternate turns.
- First to get 3 in row/column/diagonal wins.
- Draw if board full with no winner.
- Support new game, invalid move handling.

---

# 2. Class Diagram (Text)

```text
+----------------+
|     Game       |
+----------------+
| - board        |
| - playerX      |
| - playerO      |
| - current      |
| - status       |
+----------------+
| + play(row,col)|
| + getStatus()  |
| + reset()      |
+----------------+
        | uses
        v
+----------------+
|     Board      |
+----------------+
| - cells[][]    |
+----------------+
| + place()      |
| + isFull()     |
| + print()      |
+----------------+

+----------------+
|    Player      |
+----------------+
| - name         |
| - symbol       |
+----------------+

+----------------+<<enum>>
|  GameStatus    |
+----------------+
| IN_PROGRESS    |
| X_WON          |
| O_WON          |
| DRAW           |
+----------------+
```

---

# 3. Core Implementation

```java
enum Symbol { X, O, EMPTY }
enum GameStatus { IN_PROGRESS, X_WON, O_WON, DRAW }

class Board {
    private final Symbol[][] cells = new Symbol[3][3];

    Board() {
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                cells[i][j] = Symbol.EMPTY;
    }

    boolean place(int row, int col, Symbol symbol) {
        if (row < 0 || row > 2 || col < 0 || col > 2) return false;
        if (cells[row][col] != Symbol.EMPTY) return false;
        cells[row][col] = symbol;
        return true;
    }

    boolean isFull() {
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                if (cells[i][j] == Symbol.EMPTY) return false;
        return true;
    }

    Symbol[][] getCells() { return cells; }
}

class Player {
    private final String name;
    private final Symbol symbol;
    Player(String name, Symbol symbol) {
        this.name = name;
        this.symbol = symbol;
    }
    Symbol getSymbol() { return symbol; }
}

class Game {
    private final Board board = new Board();
    private final Player playerX = new Player("Player X", Symbol.X);
    private final Player playerO = new Player("Player O", Symbol.O);
    private Player current = playerX;
    private GameStatus status = GameStatus.IN_PROGRESS;

    public void play(int row, int col) {
        if (status != GameStatus.IN_PROGRESS) {
            throw new IllegalStateException("Game already finished");
        }
        if (!board.place(row, col, current.getSymbol())) {
            throw new IllegalArgumentException("Invalid move");
        }
        status = checkWinner();
        if (status == GameStatus.IN_PROGRESS) {
            current = (current == playerX) ? playerO : playerX;
        }
    }

    private GameStatus checkWinner() {
        Symbol[][] c = board.getCells();
        Symbol s = current.getSymbol();

        for (int i = 0; i < 3; i++) {
            if (c[i][0] == s && c[i][1] == s && c[i][2] == s) return won(s);
            if (c[0][i] == s && c[1][i] == s && c[2][i] == s) return won(s);
        }
        if (c[0][0] == s && c[1][1] == s && c[2][2] == s) return won(s);
        if (c[0][2] == s && c[1][1] == s && c[2][0] == s) return won(s);

        return board.isFull() ? GameStatus.DRAW : GameStatus.IN_PROGRESS;
    }

    private GameStatus won(Symbol s) {
        return s == Symbol.X ? GameStatus.X_WON : GameStatus.O_WON;
    }

    public GameStatus getStatus() { return status; }

    public void reset() {
        // reinitialize board and state
    }
}
```

---

# 4. Sequence Flow — Valid Move

```text
Client -> Game: play(0, 0)
Game -> Board: place(0, 0, X)
Board --> Game: true
Game -> Game: checkWinner() -> IN_PROGRESS
Game -> Game: switch current to O
Game --> Client: ok

Client -> Game: play(1, 1)
Game -> Board: place(1, 1, O)
...
```

---

# 5. Extensions (Interview Discussion)

| Extension | Design change |
|-----------|---------------|
| N×N board | Parameterize size; win condition = N in row |
| Online multiplayer | GameSession + WebSocket; server authoritative |
| AI opponent | Strategy interface `MoveStrategy` |
| Undo move | Command pattern with stack |

---

# 6. Design Patterns

- **Facade:** `Game` coordinates board and players.
- **State:** Could model `GameStatus` as state objects (overkill for 3×3).
- **Observer:** Notify UI on board change.

---

# 7. Interview Points

1. Separate **Board** (data) from **Game** (rules/flow).
2. Validate moves before state change.
3. Win check after each move only — O(1) for 3×3.
4. Thread-safety not needed for single local game; online needs server-side lock per game.
5. API design: `POST /games/{id}/moves {row, col}` with auth.

---

# Quick Revision

```text
Tic-Tac-Toe = Board + 2 Players + turn switch + win/draw detection in Game class.
```

---

*End of Day 12 LLD*
