# Day 48 — LLD Mock: Chess (Full Code — Check & Checkmate)

**Topics:** Complete move flow · Check detection · Checkmate · Stalemate · Mock interview simulation

---

# Mock Interview Format (45 min)

```text
0–5 min   Clarify scope (2-player, standard rules, no AI)
5–10 min  Class diagram + enums
10–25 min Core: Board, Piece, makeMove, isCheck
25–40 min Checkmate + stalemate detection
40–45 min Edge cases: castling blocked by check, pin
```

---

# 1. Core Enums

```java
public enum Color { WHITE, BLACK;
    public Color opposite() { return this == WHITE ? BLACK : WHITE; }
}

public enum GameStatus {
    ACTIVE, CHECK, CHECKMATE, STALEMATE, DRAW
}
```

---

# 2. Cell & Board

```java
public class Cell {
    private final int row, col;
    private Piece piece;

    public Cell(int row, int col) { this.row = row; this.col = col; }

    public boolean isEmpty() { return piece == null; }
    public Piece getPiece() { return piece; }
    public void setPiece(Piece p) { this.piece = p; }
    public int getRow() { return row; }
    public int getCol() { return col; }
}

public class Board {
    private final Cell[][] cells = new Cell[8][8];

    public Board() {
        for (int r = 0; r < 8; r++)
            for (int c = 0; c < 8; c++)
                cells[r][c] = new Cell(r, c);
        initialize();
    }

    public Cell getCell(int row, int col) {
        if (row < 0 || row > 7 || col < 0 || col > 7) return null;
        return cells[row][col];
    }

    public void initialize() {
        // Pawns
        for (int c = 0; c < 8; c++) {
            setPiece(1, c, new Pawn(Color.BLACK));
            setPiece(6, c, new Pawn(Color.WHITE));
        }
        // Rooks
        setPiece(0, 0, new Rook(Color.BLACK));
        setPiece(0, 7, new Rook(Color.BLACK));
        setPiece(7, 0, new Rook(Color.WHITE));
        setPiece(7, 7, new Rook(Color.WHITE));
        // Knights
        setPiece(0, 1, new Knight(Color.BLACK));
        setPiece(0, 6, new Knight(Color.BLACK));
        setPiece(7, 1, new Knight(Color.WHITE));
        setPiece(7, 6, new Knight(Color.WHITE));
        // Bishops
        setPiece(0, 2, new Bishop(Color.BLACK));
        setPiece(0, 5, new Bishop(Color.BLACK));
        setPiece(7, 2, new Bishop(Color.WHITE));
        setPiece(7, 5, new Bishop(Color.WHITE));
        // Queens & Kings
        setPiece(0, 3, new Queen(Color.BLACK));
        setPiece(0, 4, new King(Color.BLACK));
        setPiece(7, 3, new Queen(Color.WHITE));
        setPiece(7, 4, new King(Color.WHITE));
    }

    private void setPiece(int row, int col, Piece piece) {
        cells[row][col].setPiece(piece);
        piece.setCurrentCell(cells[row][col]);
    }

    public Cell findKing(Color color) {
        for (int r = 0; r < 8; r++)
            for (int c = 0; c < 8; c++) {
                Piece p = cells[r][c].getPiece();
                if (p instanceof King && p.getColor() == color)
                    return cells[r][c];
            }
        return null;
    }

    public void movePiece(Cell from, Cell to) {
        to.setPiece(from.getPiece());
        from.setPiece(null);
        to.getPiece().setCurrentCell(to);
        to.getPiece().setHasMoved(true);
    }

    public void undoMove(Cell from, Cell to, Piece captured) {
        from.setPiece(to.getPiece());
        to.setPiece(captured);
        from.getPiece().setCurrentCell(from);
    }
}
```

---

# 3. Piece Hierarchy

```java
public abstract class Piece {
    protected final Color color;
    protected Cell currentCell;
    protected boolean hasMoved;

    public Piece(Color color) { this.color = color; }

    public abstract boolean canMove(Board board, Cell destination);

    protected boolean isSameColor(Piece other) {
        return other != null && other.color == this.color;
    }

    protected boolean isPathClear(Board board, Cell from, Cell to) {
        int dr = Integer.compare(to.getRow(), from.getRow());
        int dc = Integer.compare(to.getCol(), from.getCol());
        int r = from.getRow() + dr, c = from.getCol() + dc;
        while (r != to.getRow() || c != to.getCol()) {
            if (!board.getCell(r, c).isEmpty()) return false;
            r += dr; c += dc;
        }
        return true;
    }

    public Color getColor() { return color; }
    public Cell getCurrentCell() { return currentCell; }
    public void setCurrentCell(Cell cell) { this.currentCell = cell; }
    public boolean hasMoved() { return hasMoved; }
    public void setHasMoved(boolean moved) { this.hasMoved = moved; }
}
```

---

# 4. Key Pieces

```java
public class King extends Piece {
    public King(Color color) { super(color); }

    @Override
    public boolean canMove(Board board, Cell dest) {
        if (isSameColor(dest.getPiece())) return false;
        int dr = Math.abs(dest.getRow() - currentCell.getRow());
        int dc = Math.abs(dest.getCol() - currentCell.getCol());
        return dr <= 1 && dc <= 1;
    }
}

public class Queen extends Piece {
    public Queen(Color color) { super(color); }

    @Override
    public boolean canMove(Board board, Cell dest) {
        if (isSameColor(dest.getPiece())) return false;
        int dr = Math.abs(dest.getRow() - currentCell.getRow());
        int dc = Math.abs(dest.getCol() - currentCell.getCol());
        boolean straight = dr == 0 || dc == 0;
        boolean diagonal = dr == dc;
        if (!straight && !diagonal) return false;
        return isPathClear(board, currentCell, dest);
    }
}

public class Rook extends Piece {
    public Rook(Color color) { super(color); }

    @Override
    public boolean canMove(Board board, Cell dest) {
        if (isSameColor(dest.getPiece())) return false;
        int dr = Math.abs(dest.getRow() - currentCell.getRow());
        int dc = Math.abs(dest.getCol() - currentCell.getCol());
        if (dr != 0 && dc != 0) return false;
        return isPathClear(board, currentCell, dest);
    }
}

public class Bishop extends Piece {
    public Bishop(Color color) { super(color); }

    @Override
    public boolean canMove(Board board, Cell dest) {
        if (isSameColor(dest.getPiece())) return false;
        int dr = Math.abs(dest.getRow() - currentCell.getRow());
        int dc = Math.abs(dest.getCol() - currentCell.getCol());
        if (dr != dc) return false;
        return isPathClear(board, currentCell, dest);
    }
}

public class Knight extends Piece {
    public Knight(Color color) { super(color); }

    @Override
    public boolean canMove(Board board, Cell dest) {
        if (isSameColor(dest.getPiece())) return false;
        int dr = Math.abs(dest.getRow() - currentCell.getRow());
        int dc = Math.abs(dest.getCol() - currentCell.getCol());
        return (dr == 2 && dc == 1) || (dr == 1 && dc == 2);
    }
}

public class Pawn extends Piece {
    public Pawn(Color color) { super(color); }

    @Override
    public boolean canMove(Board board, Cell dest) {
        if (isSameColor(dest.getPiece())) return false;
        int direction = (color == Color.WHITE) ? -1 : 1;
        int startRow = (color == Color.WHITE) ? 6 : 1;
        int dr = dest.getRow() - currentCell.getRow();
        int dc = Math.abs(dest.getCol() - currentCell.getCol());

        // Forward one
        if (dc == 0 && dr == direction && dest.isEmpty()) return true;
        // Forward two from start
        if (dc == 0 && !hasMoved && currentCell.getRow() == startRow
            && dr == 2 * direction
            && dest.isEmpty()
            && board.getCell(currentCell.getRow() + direction, currentCell.getCol()).isEmpty())
            return true;
        // Capture diagonal
        if (dc == 1 && dr == direction && !dest.isEmpty()) return true;
        return false;
    }
}
```

---

# 5. Game — Move Validation + Check/Checkmate

```java
public class Game {
    private final Board board;
    private Color currentTurn;
    private GameStatus status;

    public Game() {
        board = new Board();
        currentTurn = Color.WHITE;
        status = GameStatus.ACTIVE;
    }

    public boolean makeMove(Cell from, Cell to) {
        if (status == GameStatus.CHECKMATE || status == GameStatus.STALEMATE)
            return false;

        Piece piece = from.getPiece();
        if (piece == null || piece.getColor() != currentTurn) return false;
        if (!piece.canMove(board, to)) return false;

        // Cannot move into check (or leave king in check)
        if (!isLegalMove(from, to, currentTurn)) return false;

        Piece captured = to.getPiece();
        board.movePiece(from, to);

        updateGameStatus(currentTurn.opposite());
        currentTurn = currentTurn.opposite();
        return true;
    }

    /** Simulate move — if own king in check after move, illegal */
    private boolean isLegalMove(Cell from, Cell to, Color color) {
        Piece captured = to.getPiece();
        board.movePiece(from, to);
        boolean inCheck = isKingInCheck(color);
        board.undoMove(from, to, captured);
        return !inCheck;
    }

    /** Is the king of `color` under attack? */
    public boolean isKingInCheck(Color color) {
        Cell kingCell = board.findKing(color);
        if (kingCell == null) return false;
        Color opponent = color.opposite();

        for (int r = 0; r < 8; r++) {
            for (int c = 0; c < 8; c++) {
                Cell cell = board.getCell(r, c);
                Piece p = cell.getPiece();
                if (p != null && p.getColor() == opponent) {
                    if (p.canMove(board, kingCell)) return true;
                }
            }
        }
        return false;
    }

    /** Does `color` have ANY legal move? */
    public boolean hasAnyLegalMove(Color color) {
        for (int r = 0; r < 8; r++) {
            for (int c = 0; c < 8; c++) {
                Cell from = board.getCell(r, c);
                Piece p = from.getPiece();
                if (p == null || p.getColor() != color) continue;

                for (int dr = 0; dr < 8; dr++) {
                    for (int dc = 0; dc < 8; dc++) {
                        Cell to = board.getCell(dr, dc);
                        if (p.canMove(board, to) && isLegalMove(from, to, color))
                            return true;
                    }
                }
            }
        }
        return false;
    }

    private void updateGameStatus(Color colorToMove) {
        boolean inCheck = isKingInCheck(colorToMove);
        boolean hasMove = hasAnyLegalMove(colorToMove);

        if (inCheck && !hasMove) status = GameStatus.CHECKMATE;
        else if (!inCheck && !hasMove) status = GameStatus.STALEMATE;
        else if (inCheck) status = GameStatus.CHECK;
        else status = GameStatus.ACTIVE;
    }

    public GameStatus getStatus() { return status; }
    public Color getCurrentTurn() { return currentTurn; }
}
```

---

# 6. Check / Checkmate Logic Explained

```text
CHECK:
  After opponent's move, is my king attacked by any enemy piece?

ILLEGAL MOVE:
  Simulate move → if own king in check afterward → reject

CHECKMATE:
  In check AND no legal move exists (hasAnyLegalMove returns false)

STALEMATE:
  NOT in check AND no legal move exists
```

---

# 7. Demo — Fool's Mate (Fastest Checkmate)

```java
public class ChessDemo {
    public static void main(String[] args) {
        Game game = new Game();
        Board board = /* access or expose board */;

        // White: f2-f3, Black: e7-e5, White: g2-g4, Black: d8-h4 (checkmate)
        game.makeMove(board.getCell(6, 5), board.getCell(5, 5)); // f2-f3
        game.makeMove(board.getCell(1, 4), board.getCell(3, 4)); // e7-e5
        game.makeMove(board.getCell(6, 6), board.getCell(4, 6)); // g2-g4
        game.makeMove(board.getCell(0, 3), board.getCell(4, 7)); // Qh4#

        System.out.println("Status: " + game.getStatus()); // CHECKMATE
    }
}
```

---

# 8. Extensions (Mention in Interview)

| Feature | Approach |
|---------|----------|
| Castling | King/Rook unmoved, path clear, not in/through check |
| En passant | Track last double pawn push |
| Pawn promotion | Prompt for piece type on rank 8/1 |
| Draw (50-move) | Counter in Game |
| Move history | List<Move> for undo |

---

# 9. Class Diagram

```text
Game
  ├── board: Board
  ├── currentTurn: Color
  ├── status: GameStatus
  ├── makeMove(from, to)
  ├── isKingInCheck(color)
  ├── hasAnyLegalMove(color)
  └── isLegalMove(from, to, color)

Board
  └── cells[8][8]: Cell

Piece (abstract)
  ├── King, Queen, Rook, Bishop, Knight, Pawn
  └── canMove(board, dest)
```

---

# Interview Questions

## Q1. How do you detect check?

Find king cell → iterate all opponent pieces → if any `canMove` to king cell → check.

## Q2. Why simulate move for legality?

Moving may expose own king (pin discovered check). Must verify king safe **after** move.

## Q3. Checkmate vs stalemate?

Checkmate: in check + no legal moves. Stalemate: NOT in check + no legal moves.

## Q4. Performance of hasAnyLegalMove?

O(pieces × 64 × move simulation) — fine for 8×8. Optimizations: generate moves per piece type only.

## Q5. Single Responsibility in design?

`Piece` knows movement rules. `Board` knows layout. `Game` knows turn, check, game status.

---

# Mock Self-Score Rubric

| Criteria | Points |
|----------|--------|
| Board + all 6 pieces | 3 |
| makeMove with turn validation | 2 |
| isKingInCheck | 3 |
| isLegalMove (can't move into check) | 3 |
| Checkmate + stalemate | 4 |
| Clean OOP / extensibility | 2 |
| **Total** | **17** — aim 14+ |

---

# One-Line Revision

```text
Chess LLD = Piece.canMove + Game.isLegalMove (simulate, king safe) + isKingInCheck + hasAnyLegalMove → CHECK / CHECKMATE / STALEMATE.
```

---

*End of Day 48 Chess Mock LLD*
