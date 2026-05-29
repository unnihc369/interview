# Day 31 - Advanced Backtracking

---

# 1. N-Queens

## Problem Statement

Place `n` queens on an `n×n` chessboard so no two queens attack each other. Return all distinct solutions.

---

# Core Idea

Place queen row by row. For each column, check if safe (no conflict in column, diagonals). Backtrack on failure.

---

# Java Solution

```java
class Solution {
    public List<List<String>> solveNQueens(int n) {
        List<List<String>> result = new ArrayList<>();
        char[][] board = new char[n][n];
        for (char[] row : board) Arrays.fill(row, '.');
        backtrack(board, 0, result);
        return result;
    }

    private void backtrack(char[][] board, int row, List<List<String>> result) {
        if (row == board.length) {
            result.add(construct(board));
            return;
        }
        for (int col = 0; col < board.length; col++) {
            if (!isSafe(board, row, col)) continue;
            board[row][col] = 'Q';
            backtrack(board, row + 1, result);
            board[row][col] = '.';
        }
    }

    private boolean isSafe(char[][] board, int row, int col) {
        for (int i = 0; i < row; i++) {
            if (board[i][col] == 'Q') return false;
            if (col - (row - i) >= 0 && board[i][col - (row - i)] == 'Q') return false;
            if (col + (row - i) < board.length && board[i][col + (row - i)] == 'Q') return false;
        }
        return true;
    }

    private List<String> construct(char[][] board) {
        List<String> res = new ArrayList<>();
        for (char[] row : board) res.add(new String(row));
        return res;
    }
}
```

---

# JavaScript Solution

```javascript
function solveNQueens(n) {
  const result = [];
  const board = Array.from({ length: n }, () => Array(n).fill('.'));

  function isSafe(row, col) {
    for (let i = 0; i < row; i++) {
      if (board[i][col] === 'Q') return false;
      if (col - (row - i) >= 0 && board[i][col - (row - i)] === 'Q') return false;
      if (col + (row - i) < n && board[i][col + (row - i)] === 'Q') return false;
    }
    return true;
  }

  function backtrack(row) {
    if (row === n) {
      result.push(board.map(r => r.join('')));
      return;
    }
    for (let col = 0; col < n; col++) {
      if (!isSafe(row, col)) continue;
      board[row][col] = 'Q';
      backtrack(row + 1);
      board[row][col] = '.';
    }
  }

  backtrack(0);
  return result;
}
```

---

# Time Complexity

```text
O(n!) — try placements with pruning
```

---

# 2. Sudoku Solver

## Problem Statement

Fill empty cells in 9×9 Sudoku so each row, column, and 3×3 box contains digits 1-9 exactly once.

---

# Core Idea

Find empty cell → try digits 1-9 → validate → recurse → backtrack.

Optimize with `boolean[9]` sets for rows, cols, boxes.

---

# Java Solution

```java
class Solution {
    public void solveSudoku(char[][] board) {
        solve(board);
    }

    private boolean solve(char[][] board) {
        for (int r = 0; r < 9; r++) {
            for (int c = 0; c < 9; c++) {
                if (board[r][c] != '.') continue;
                for (char d = '1'; d <= '9'; d++) {
                    if (!isValid(board, r, c, d)) continue;
                    board[r][c] = d;
                    if (solve(board)) return true;
                    board[r][c] = '.';
                }
                return false;
            }
        }
        return true;
    }

    private boolean isValid(char[][] board, int row, int col, char d) {
        for (int i = 0; i < 9; i++) {
            if (board[row][i] == d || board[i][col] == d) return false;
        }
        int br = (row / 3) * 3, bc = (col / 3) * 3;
        for (int r = br; r < br + 3; r++)
            for (int c = bc; c < bc + 3; c++)
                if (board[r][c] == d) return false;
        return true;
    }
}
```

---

# JavaScript Solution

```javascript
function solveSudoku(board) {
  function isValid(row, col, d) {
    for (let i = 0; i < 9; i++) {
      if (board[row][i] === d || board[i][col] === d) return false;
    }
    const br = Math.floor(row / 3) * 3, bc = Math.floor(col / 3) * 3;
    for (let r = br; r < br + 3; r++)
      for (let c = bc; c < bc + 3; c++)
        if (board[r][c] === d) return false;
    return true;
  }

  function solve() {
    for (let r = 0; r < 9; r++) {
      for (let c = 0; c < 9; c++) {
        if (board[r][c] !== '.') continue;
        for (let d = 1; d <= 9; d++) {
          const ch = String(d);
          if (!isValid(r, c, ch)) continue;
          board[r][c] = ch;
          if (solve()) return true;
          board[r][c] = '.';
        }
        return false;
      }
    }
    return true;
  }
  solve();
}
```

---

# 3. Remove Invalid Parentheses

## Problem Statement

Given a string with `'('`, `')'`, and letters, remove the minimum number of invalid parentheses to make it valid. Return all possible results.

---

# Core Idea

1. Count extra `(` and `)` to remove
2. BFS or backtrack: try removing `(` or `)` at each index, skip duplicates
3. Validate before adding to result

---

# Java Solution (BFS — shortest removal)

```java
class Solution {
    public List<String> removeInvalidParentheses(String s) {
        List<String> result = new ArrayList<>();
        Set<String> visited = new HashSet<>();
        Queue<String> queue = new LinkedList<>();
        queue.offer(s);
        visited.add(s);
        boolean found = false;

        while (!queue.isEmpty()) {
            String curr = queue.poll();
            if (isValid(curr)) {
                result.add(curr);
                found = true;
            }
            if (found) continue;

            for (int i = 0; i < curr.length(); i++) {
                if (curr.charAt(i) != '(' && curr.charAt(i) != ')') continue;
                String next = curr.substring(0, i) + curr.substring(i + 1);
                if (!visited.contains(next)) {
                    visited.add(next);
                    queue.offer(next);
                }
            }
        }
        return result;
    }

    private boolean isValid(String s) {
        int count = 0;
        for (char c : s.toCharArray()) {
            if (c == '(') count++;
            else if (c == ')') {
                count--;
                if (count < 0) return false;
            }
        }
        return count == 0;
    }
}
```

---

# JavaScript Solution

```javascript
function removeInvalidParentheses(s) {
  const result = [];
  const visited = new Set([s]);
  const queue = [s];
  let found = false;

  function isValid(str) {
    let count = 0;
    for (const c of str) {
      if (c === '(') count++;
      else if (c === ')') { count--; if (count < 0) return false; }
    }
    return count === 0;
  }

  while (queue.length) {
    const curr = queue.shift();
    if (isValid(curr)) {
      result.push(curr);
      found = true;
    }
    if (found) continue;
    for (let i = 0; i < curr.length; i++) {
      if (curr[i] !== '(' && curr[i] !== ')') continue;
      const next = curr.slice(0, i) + curr.slice(i + 1);
      if (!visited.has(next)) {
        visited.add(next);
        queue.push(next);
      }
    }
  }
  return result;
}
```

---

# Pattern Quick Summary

| Problem | Pattern | Notes |
|---------|---------|-------|
| N-Queens | Row-by-row backtrack + diagonal check | O(n!) |
| Sudoku Solver | Fill empty + constraint check | Bitmask optimization common |
| Remove Invalid Parentheses | BFS min removal or backtrack | Return all min-removal solutions |

---

# Interview Questions

## N-Queens — bit manipulation optimization?

Use three bitmasks for cols and diagonals per row — faster for n=14+.

## Sudoku — why BFS not used?

Single solution needed; DFS backtrack finds one valid complete board.

## Remove Invalid — why BFS?

Guarantees minimum removals first level solutions appear.

## N-Queens count (LeetCode 52)?

Same backtrack, increment counter instead of building board strings.

---

*End of Day 31 LeetCode*
