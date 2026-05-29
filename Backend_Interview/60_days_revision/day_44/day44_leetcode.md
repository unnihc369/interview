# Day 44 — Mock Prep: BFS DP, Interval DP, Dijkstra Grid

**Topics:** Race Car · Strange Printer · Minimum Cost to Make Valid Path in Grid

---

# 1. Race Car

## Problem Statement

Start at position `0`, speed `1`. Each instruction is `A` (accelerate: position += speed, speed *= 2) or `R` (reverse: speed = -1 if positive else 1). Return **minimum** instructions to reach target `target`.

---

# Example

```text
Input: target = 3
Output: 2
Explanation: AA (0→1→3)
```

---

# Brute Force Approach

BFS all (position, speed) states — unbounded position space.

---

# Optimized Approach (BFS on State Space)

## Core Idea

State = `(position, speed)`. BFS from `(0, 1)`.
At each state, two moves: `A` and `R`.
Prune: if `position < 0` or `position > 2 * target`, skip (never optimal to overshoot far beyond target).

---

# Java Solution

```java
class Solution {
    public int racecar(int target) {
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{0, 1, 0}); // pos, speed, steps
        Set<String> visited = new HashSet<>();
        visited.add("0,1");

        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            int pos = curr[0], speed = curr[1], steps = curr[2];

            if (pos == target) return steps;

            // Accelerate
            int newPos = pos + speed;
            int newSpeed = speed * 2;
            String keyA = newPos + "," + newSpeed;
            if (newPos > 0 && newPos < 2 * target && !visited.contains(keyA)) {
                visited.add(keyA);
                queue.offer(new int[]{newPos, newSpeed, steps + 1});
            }

            // Reverse (only if not at start with speed 1 going forward)
            int revSpeed = speed > 0 ? -1 : 1;
            String keyR = pos + "," + revSpeed;
            if (!visited.contains(keyR)) {
                visited.add(keyR);
                queue.offer(new int[]{pos, revSpeed, steps + 1});
            }
        }
        return -1;
    }
}
```

---

# Alternative: DP

`dp[i]` = min instructions to reach exactly `i`.
For each position, simulate sequences of `A` then one `R`.

---

# Time Complexity

```text
O(target × log target) — BFS with bounded state space
```

---

# 2. Strange Printer

## Problem Statement

Strange printer prints sequence of **same character** each turn (can overlap existing). Return minimum turns to print string `s`.

---

# Example

```text
Input: s = "aaabbbb"
Output: 4
Explanation: "aaaa" → "aaabbbb" → "aaabbbb" → "aaabbbb" (or similar 4-turn sequence)
```

---

# Brute Force Approach

Try all partitions — exponential.

---

# Optimized Approach (Interval DP)

## Core Idea

`dp[i][j]` = min turns to print `s[i..j]`.

1. If `s[i] == s[j]`, can extend last print: `dp[i][j] = dp[i][j-1]`
2. Else split at `k`: `dp[i][j] = min(dp[i][k] + dp[k+1][j])`
3. Also: if `s[k] == s[j]`, merge: `dp[i][j] = min(dp[i][j], dp[i][k-1] + dp[k][j-1])`

---

# Java Solution

```java
class Solution {
    public int strangePrinter(String s) {
        int n = s.length();
        int[][] dp = new int[n][n];
        for (int i = 0; i < n; i++) dp[i][i] = 1;

        for (int len = 2; len <= n; len++) {
            for (int i = 0; i + len - 1 < n; i++) {
                int j = i + len - 1;
                dp[i][j] = dp[i][j - 1] + 1; // print s[j] separately
                for (int k = i; k < j; k++) {
                    if (s.charAt(k) == s.charAt(j)) {
                        dp[i][j] = Math.min(dp[i][j],
                            dp[i][k] + (k + 1 <= j - 1 ? dp[k + 1][j - 1] : 0));
                    }
                }
            }
        }
        return dp[0][n - 1];
    }
}
```

---

# Time Complexity

```text
O(n³) time, O(n²) space
```

---

# 3. Minimum Cost to Make at Least One Valid Path in a Grid

## Problem Statement

Grid with arrows (1=right, 2=left, 3=down, 4=up). Cost 0 to follow arrow, cost 1 to change direction. Find min cost from `(0,0)` to `(m-1, n-1)`.

---

# Example

```text
Input: grid = [[1,1,3],[2,3,3],[1,1,4]]
Output: 0  (follow all arrows)
```

---

# Brute Force Approach

Dijkstra without 0-cost optimization — still works but slower.

---

# Optimized Approach (0-1 BFS / Dijkstra)

## Core Idea

Treat as shortest path on grid. Edge weight 0 if move matches arrow, 1 if changed.
Use **deque**: push front for cost 0, push back for cost 1 (0-1 BFS).

Directions: `(0,1), (0,-1), (1,0), (-1,0)` map to 1,2,3,4.

---

# Java Solution

```java
class Solution {
    private static final int[][] DIRS = {{0,1},{0,-1},{1,0},{-1,0}};

    public int minCost(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        int[][] dist = new int[m][n];
        for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);
        dist[0][0] = 0;

        Deque<int[]> dq = new ArrayDeque<>();
        dq.offerFirst(new int[]{0, 0});

        while (!dq.isEmpty()) {
            int[] curr = dq.pollFirst();
            int r = curr[0], c = curr[1];
            if (r == m - 1 && c == n - 1) return dist[r][c];

            for (int d = 0; d < 4; d++) {
                int nr = r + DIRS[d][0], nc = c + DIRS[d][1];
                if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
                int cost = (grid[r][c] == d + 1) ? 0 : 1;
                if (dist[r][c] + cost < dist[nr][nc]) {
                    dist[nr][nc] = dist[r][c] + cost;
                    if (cost == 0) dq.offerFirst(new int[]{nr, nc});
                    else dq.offerLast(new int[]{nr, nc});
                }
            }
        }
        return dist[m - 1][n - 1];
    }
}
```

---

# Time Complexity

```text
O(m × n) — each cell processed once with 0-1 BFS
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Race Car | BFS state (pos, speed) | O(t log t) |
| Strange Printer | Interval DP | O(n³) |
| Min Cost Valid Path | 0-1 BFS / Dijkstra | O(mn) |

---

# Interview Questions

## Why bound position at 2×target in Race Car?

Mathematically, optimal path never needs to overshoot beyond `2*target` before reversing — prunes infinite BFS.

## Strange Printer — why interval DP?

Optimal substructure: min turns for substring depends on smaller substrings inside it.

## 0-1 BFS vs Dijkstra?

When edge weights are only 0 or 1, deque-based 0-1 BFS is O(V+E). Dijkstra with PQ is O(E log V) — both acceptable; 0-1 BFS is cleaner.

---

# One-Line Revision

```text
Race Car = BFS (pos,speed); Strange Printer = interval DP; Valid Path Grid = 0-1 BFS with arrow-matching cost.
```

---

*End of Day 44 LeetCode*
