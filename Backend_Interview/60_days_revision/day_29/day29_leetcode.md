# Day 29 - Backtracking: Grid & Combinatorial Generation

---

# 1. Word Search

## Problem Statement

Given an `m x n` board of characters and a string `word`, return `true` if `word` exists in the grid. Words are formed by sequentially adjacent cells (horizontal or vertical). Same cell cannot be used twice in one word.

---

# Example

```text
Input:
board = [["A","B","C","E"],
         ["S","F","C","S"],
         ["A","D","E","E"]]
word = "ABCCED"

Output: true
```

---

# Core Idea

DFS backtracking from each cell matching first character:

1. Mark cell visited
2. Explore 4 directions for next character
3. Unmark on backtrack (restore state)

---

# Java Solution

```java
class Solution {
    private int rows, cols;
    private static final int[][] DIRS = {{0,1},{0,-1},{1,0},{-1,0}};

    public boolean exist(char[][] board, String word) {
        rows = board.length;
        cols = board[0].length;
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (dfs(board, word, r, c, 0)) return true;
            }
        }
        return false;
    }

    private boolean dfs(char[][] board, String word, int r, int c, int idx) {
        if (idx == word.length()) return true;
        if (r < 0 || c < 0 || r >= rows || c >= cols) return false;
        if (board[r][c] != word.charAt(idx)) return false;

        char temp = board[r][c];
        board[r][c] = '#'; // mark visited

        for (int[] d : DIRS) {
            if (dfs(board, word, r + d[0], c + d[1], idx + 1)) {
                board[r][c] = temp;
                return true;
            }
        }
        board[r][c] = temp; // backtrack
        return false;
    }
}
```

---

# JavaScript Solution

```javascript
function exist(board, word) {
  const rows = board.length, cols = board[0].length;
  const dirs = [[0,1],[0,-1],[1,0],[-1,0]];

  function dfs(r, c, idx) {
    if (idx === word.length) return true;
    if (r < 0 || c < 0 || r >= rows || c >= cols) return false;
    if (board[r][c] !== word[idx]) return false;

    const temp = board[r][c];
    board[r][c] = '#';

    for (const [dr, dc] of dirs) {
      if (dfs(r + dr, c + dc, idx + 1)) {
        board[r][c] = temp;
        return true;
      }
    }
    board[r][c] = temp;
    return false;
  }

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (dfs(r, c, 0)) return true;
    }
  }
  return false;
}
```

---

# Time Complexity

```text
O(m * n * 4^L) where L = word length
Space: O(L) recursion stack
```

---

# 2. Generate Parentheses

## Problem Statement

Given `n` pairs of parentheses, generate all combinations of well-formed parentheses.

---

# Example

```text
Input: n = 3
Output: ["((()))","(()())","(())()","()(())","()()()"]
```

---

# Core Idea

Build string incrementally:

- Add `(` if `open < n`
- Add `)` if `close < open` (ensures validity)

---

# Java Solution

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        backtrack(result, new StringBuilder(), 0, 0, n);
        return result;
    }

    private void backtrack(List<String> result, StringBuilder sb,
                           int open, int close, int n) {
        if (sb.length() == 2 * n) {
            result.add(sb.toString());
            return;
        }
        if (open < n) {
            sb.append('(');
            backtrack(result, sb, open + 1, close, n);
            sb.deleteCharAt(sb.length() - 1);
        }
        if (close < open) {
            sb.append(')');
            backtrack(result, sb, open, close + 1, n);
            sb.deleteCharAt(sb.length() - 1);
        }
    }
}
```

---

# JavaScript Solution

```javascript
function generateParenthesis(n) {
  const result = [];

  function backtrack(sb, open, close) {
    if (sb.length === 2 * n) {
      result.push(sb);
      return;
    }
    if (open < n) backtrack(sb + '(', open + 1, close);
    if (close < open) backtrack(sb + ')', open, close + 1);
  }

  backtrack('', 0, 0);
  return result;
}
```

---

# Time Complexity

```text
O(4^n / sqrt(n)) — Catalan number of valid strings
Space: O(n) recursion depth
```

---

# 3. Letter Combinations of a Phone Number

## Problem Statement

Given a string containing digits `2-9`, return all possible letter combinations that the number could represent (phone keypad mapping).

---

# Example

```text
Input: "23"
Output: ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

---

# Core Idea

Backtracking / DFS: at each digit index, try all mapped letters and recurse.

---

# Java Solution

```java
class Solution {
    private static final String[] MAP = {
        "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
    };

    public List<String> letterCombinations(String digits) {
        List<String> result = new ArrayList<>();
        if (digits.isEmpty()) return result;
        backtrack(digits, 0, new StringBuilder(), result);
        return result;
    }

    private void backtrack(String digits, int idx, StringBuilder sb, List<String> result) {
        if (idx == digits.length()) {
            result.add(sb.toString());
            return;
        }
        String letters = MAP[digits.charAt(idx) - '0'];
        for (char c : letters.toCharArray()) {
            sb.append(c);
            backtrack(digits, idx + 1, sb, result);
            sb.deleteCharAt(sb.length() - 1);
        }
    }
}
```

---

# JavaScript Solution

```javascript
function letterCombinations(digits) {
  if (!digits) return [];
  const map = ['','','abc','def','ghi','jkl','mno','pqrs','tuv','wxyz'];
  const result = [];

  function backtrack(idx, path) {
    if (idx === digits.length) {
      result.push(path);
      return;
    }
    const letters = map[digits[idx]];
    for (const c of letters) {
      backtrack(idx + 1, path + c);
    }
  }

  backtrack(0, '');
  return result;
}
```

---

# Time Complexity

```text
O(4^n * n) — up to 4 letters per digit, n digits
Space: O(n) recursion
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Word Search | Grid DFS + backtrack visited | O(m·n·4^L) |
| Generate Parentheses | Constraint backtracking (open/close) | O(4^n/√n) |
| Letter Combinations | Digit → letter tree DFS | O(4^n·n) |

---

# Interview Questions

## Word Search — can we use a `visited` boolean matrix instead of mutating board?

Yes. Trade-off: extra O(m·n) space vs in-place `#` marker (restores on backtrack).

## Generate Parentheses — relation to Catalan numbers?

Number of valid strings for `n` pairs is the nth Catalan number: C(n) = (2n)! / ((n+1)!·n!).

## Letter Combinations — BFS vs DFS?

Both work. DFS/backtracking is natural for generating all paths; BFS builds level by level.

## When to prune in backtracking?

Prune when partial state cannot lead to valid solution (e.g., `close > open` never happens by construction in parentheses problem).

---

*End of Day 29 LeetCode*
