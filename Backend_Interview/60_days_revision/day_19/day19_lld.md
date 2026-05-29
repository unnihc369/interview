# Day 19 — LLD: Chess Game

**Topics:** Requirements · Class Design · Move Validation · Design Patterns

---

# 1. Problem Statement

Design a chess game for two players supporting:

- Standard 8×8 board with all pieces
- Legal move validation
- Turn-based play (White moves first)
- Check and checkmate detection
- Stalemate detection
- Castling, en passant, pawn promotion

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Initialize Board | Standard piece placement |
| Make Move | Move piece if legal |
| Validate Move | Check piece rules + path clear |
| Check Detection | Is king under attack? |
| Checkmate | No legal moves + in check |
| Stalemate | No legal moves + not in check |
| Resign / Draw | End game |

---

# 3. Core Classes

```text
Game
  ├── Board
  ├── Player (White, Black)
  ├── GameStatus (ACTIVE, CHECK, CHECKMATE, STALEMATE, DRAW)
  └── MoveHistory

Board
  ├── Cell[8][8]
  └── placePiece(), getPiece(), isPathClear()

Cell
  ├── row, col
  └── Piece (nullable)

Piece (abstract)
  ├── color, currentCell
  └── canMove(Board, Cell destination)

Concrete Pieces: King, Queen, Rook, Bishop, Knight, Pawn

Move
  ├── sourceCell, destCell
  ├── piece, capturedPiece
  └── isCastling, isEnPassant, promotionPiece
```

---

# 4. Enums

```java
public enum Color {
    WHITE, BLACK;
    public Color opposite() { return this == WHITE ? BLACK : WHITE; }
}

public enum GameStatus {
    ACTIVE, CHECK, CHECKMATE, STALEMATE, DRAW, RESIGNED
}

public enum PieceType {
    KING, QUEEN, ROOK, BISHOP, KNIGHT, PAWN
}
```

---

# 5. Piece Hierarchy

```java
public abstract class Piece {
    protected final Color color;
    protected Cell currentCell;
    protected boolean hasMoved;

    public abstract boolean canMove(Board board, Cell destination);

    protected boolean isSameColor(Piece other) {
        return other != null && other.color == this.color;
    }

    protected boolean isPathClear(Board board, Cell from, Cell to) {
        int dr = Integer.compare(to.getRow(), from.getRow());
        int dc = Integer.compare(to.getCol(), from.getCol());
        int r = from.getRow() + dr, c = from.getCol() + dc;
        while (r != to.getRow() || c != to.getCol()) {
            if (board.getCell(r, c).getPiece() != null) return false;
            r += dr;
            c += dc;
        }
        return true;
    }
}
```

---

# 6. Example — Rook Move Logic

```java
public class Rook extends Piece {
    @Override
    public boolean canMove(Board board, Cell destination) {
        if (isSameColor(destination.getPiece())) return false;

        int rowDiff = Math.abs(destination.getRow() - currentCell.getRow());
        int colDiff = Math.abs(destination.getCol() - currentCell.getCol());

        // Must move in straight line
        if (rowDiff != 0 && colDiff != 0) return false;
        return isPathClear(board, currentCell, destination);
    }
}
```

---

# 7. Game Controller

```java
public class Game {
    private final Board board;
    private Color currentTurn;
    private GameStatus status;
    private final List<Move> moveHistory;

    public Game() {
        board = new Board();
        board.initialize();
        currentTurn = Color.WHITE;
        status = GameStatus.ACTIVE;
        moveHistory = new ArrayList<>();
    }

    public boolean makeMove(Cell source, Cell destination) {
        Piece piece = source.getPiece();
        if (piece == null || piece.getColor() != currentTurn) return false;
        if (!piece.canMove(board, destination)) return false;
        if (wouldLeaveKingInCheck(source, destination)) return false;

        Move move = executeMove(source, destination);
        moveHistory.add(move);

        if (isCheckmate(currentTurn.opposite())) {
            status = GameStatus.CHECKMATE;
        } else if (isStalemate(currentTurn.opposite())) {
            status = GameStatus.STALEMATE;
        } else if (isInCheck(currentTurn.opposite())) {
            status = GameStatus.CHECK;
        } else {
            status = GameStatus.ACTIVE;
        }

        currentTurn = currentTurn.opposite();
        return true;
    }
}
```

---

# 8. Check Detection

```java
public boolean isInCheck(Color color) {
    Cell kingCell = board.findKing(color);
    for (int r = 0; r < 8; r++) {
        for (int c = 0; c < 8; c++) {
            Piece piece = board.getCell(r, c).getPiece();
            if (piece != null && piece.getColor() != color) {
                if (piece.canMove(board, kingCell)) return true;
            }
        }
    }
    return false;
}
```

---

# 9. Checkmate Detection

```java
public boolean isCheckmate(Color color) {
    if (!isInCheck(color)) return false;
    return getAllLegalMoves(color).isEmpty();
}

private List<Move> getAllLegalMoves(Color color) {
    List<Move> legalMoves = new ArrayList<>();
    for (int r = 0; r < 8; r++) {
        for (int c = 0; c < 8; c++) {
            Cell source = board.getCell(r, c);
            Piece piece = source.getPiece();
            if (piece == null || piece.getColor() != color) continue;
            for (int dr = 0; dr < 8; dr++) {
                for (int dc = 0; dc < 8; dc++) {
                    Cell dest = board.getCell(dr, dc);
                    if (piece.canMove(board, dest) &&
                            !wouldLeaveKingInCheck(source, dest)) {
                        legalMoves.add(new Move(source, dest));
                    }
                }
            }
        }
    }
    return legalMoves;
}
```

---

# 10. Special Moves

## Castling

Conditions:
- King and rook haven't moved
- No pieces between them
- King not in check, doesn't pass through check

## En Passant

Pawn captures opponent pawn that just moved two squares.

## Pawn Promotion

When pawn reaches opposite end → promote to Queen (default), Rook, Bishop, or Knight.

---

# 11. Design Patterns Used

| Pattern | Where |
|---------|-------|
| Strategy | Each piece type implements own move rules |
| Command | Move object encapsulates action (enables undo) |
| Observer | Notify UI on board state change |
| Memento | Save/restore board state for undo |
| Singleton | Optional — one active game per session |

---

# 12. Undo with Command Pattern

```java
public class MoveCommand {
    private final Game game;
    private final Move move;
    private final BoardSnapshot snapshotBefore;

    public void execute() {
        game.executeMoveInternal(move);
    }

    public void undo() {
        game.restoreSnapshot(snapshotBefore);
    }
}
```

---

# 13. Board Initialization

```java
public void initialize() {
    // Row 0: White pieces (Rook, Knight, Bishop, Queen, King, Bishop, Knight, Rook)
    setupRow(0, Color.WHITE);
    setupPawnRow(1, Color.WHITE);
    setupPawnRow(6, Color.BLACK);
    setupRow(7, Color.BLACK);
    // Rows 2-5 empty
}
```

---

# Interview Questions

## How to validate move doesn't leave own king in check?

Simulate move on copy of board (or temporarily apply and revert). Check if own king is attacked after move.

---

## How to optimize legal move generation?

Precompute attack maps. Use bitboards for production chess engines. For interview, brute force 64×64 is acceptable with explanation of optimization path.

---

## Why abstract Piece class?

Each piece has different movement rules. Polymorphism via `canMove()` — Open/Closed Principle.

---

## How handle pawn promotion choice?

`makeMove()` accepts optional `promotionPiece` parameter. Default to Queen if not specified.

---

## Multiplayer online chess extension?

Separate `Game` (domain logic) from `GameSession` (WebSocket, player matching). Game state serialized to JSON for network sync.

---

# Quick Revision

```text
Piece hierarchy  → Strategy pattern for move rules
Game controller  → turn management, status
Check/Checkmate  → simulate all opponent moves
wouldLeaveKingInCheck → simulate before commit
Command/Memento → undo support
Board 8×8       → Cell[][] with nullable Piece
```

---

*End of Day 19 Chess Game LLD*
