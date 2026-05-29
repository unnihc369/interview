# Day 8 — Graphs & BFS/DFS

---

# 1. Number of Islands

## Problem Statement

Given a 2D grid of `'1'` (land) and `'0'` (water), count the number of islands. An island is surrounded by water and formed by connecting adjacent lands horizontally or vertically.

---

# Example

```text
Input:
[
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]

Output: 3
```

---

# Approach Hints

- Each unvisited `'1'` starts a new island — run DFS/BFS to mark all connected land.
- Mutate grid to `'0'` (visited) or use `boolean[][] visited`.
- Alternative: Union-Find for dynamic grids (overkill here).

---

# Core Idea (Walkthrough)

1. Loop every cell `(i, j)`.
2. If `grid[i][j] == '1'`, increment count and flood-fill (DFS/BFS) all 4 neighbors.
3. Return total count.

---

# Java Solution

```java
class Solution {
    public int numIslands(char[][] grid) {
        int rows = grid.length, cols = grid[0].length;
        int count = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    count++;
                    dfs(grid, r, c);
                }
            }
        }
        return count;
    }

    private void dfs(char[][] grid, int r, int c) {
        int rows = grid.length, cols = grid[0].length;
        if (r < 0 || c < 0 || r >= rows || c >= cols || grid[r][c] != '1') return;

        grid[r][c] = '0';
        dfs(grid, r + 1, c);
        dfs(grid, r - 1, c);
        dfs(grid, r, c + 1);
        dfs(grid, r, c - 1);
    }
}
```

---

# JavaScript Solution

```javascript
function numIslands(grid) {
  const rows = grid.length;
  const cols = grid[0].length;
  let count = 0;

  function dfs(r, c) {
    if (r < 0 || c < 0 || r >= rows || c >= cols || grid[r][c] !== "1") return;
    grid[r][c] = "0";
    dfs(r + 1, c);
    dfs(r - 1, c);
    dfs(r, c + 1);
    dfs(r, c - 1);
  }

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === "1") {
        count++;
        dfs(r, c);
      }
    }
  }
  return count;
}
```

---

# Time & Space Complexity

```text
Time:  O(m * n)
Space: O(m * n) worst-case recursion stack
```

---

# 2. Rotting Oranges

## Problem Statement

Grid: `0` empty, `1` fresh orange, `2` rotten orange. Each minute, rotten oranges rot adjacent fresh oranges (4-directional). Return minimum minutes until no fresh orange remains, or `-1` if impossible.

---

# Example

```text
Input: [[2,1,1],[1,1,0],[0,1,1]]
Output: 4
```

---

# Approach Hints

- Multi-source BFS: enqueue all rotten oranges at minute `0`.
- Track fresh count; when BFS ends, if fresh > 0 return `-1`.
- Level-order BFS gives minutes (layers).

---

# Core Idea (Walkthrough)

1. Count fresh oranges; push all rotten to queue with time `0`.
2. BFS: for each rotten, rot neighbors and decrement fresh.
3. Track max time; if fresh == 0 return max time else `-1`.

---

# Java Solution

```java
class Solution {
    public int orangesRotting(int[][] grid) {
        int rows = grid.length, cols = grid[0].length;
        Queue<int[]> q = new LinkedList<>();
        int fresh = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 1) fresh++;
                else if (grid[r][c] == 2) q.offer(new int[]{r, c, 0});
            }
        }

        int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
        int minutes = 0;

        while (!q.isEmpty()) {
            int[] cur = q.poll();
            int r = cur[0], c = cur[1], t = cur[2];
            minutes = Math.max(minutes, t);

            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nc >= 0 && nr < rows && nc < cols && grid[nr][nc] == 1) {
                    grid[nr][nc] = 2;
                    fresh--;
                    q.offer(new int[]{nr, nc, t + 1});
                }
            }
        }

        return fresh == 0 ? minutes : -1;
    }
}
```

---

# JavaScript Solution

```javascript
function orangesRotting(grid) {
  const rows = grid.length;
  const cols = grid[0].length;
  const q = [];
  let fresh = 0;

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === 1) fresh++;
      else if (grid[r][c] === 2) q.push([r, c, 0]);
    }
  }

  const dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];
  let minutes = 0;

  while (q.length) {
    const [r, c, t] = q.shift();
    minutes = Math.max(minutes, t);

    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nc >= 0 && nr < rows && nc < cols && grid[nr][nc] === 1) {
        grid[nr][nc] = 2;
        fresh--;
        q.push([nr, nc, t + 1]);
      }
    }
  }

  return fresh === 0 ? minutes : -1;
}
```

---

# Time & Space Complexity

```text
Time:  O(m * n)
Space: O(m * n) queue
```

---

# 3. Course Schedule

## Problem Statement

There are `numCourses` labeled `0` to `numCourses-1` and prerequisites `prerequisites[i] = [a, b]` meaning you must take `b` before `a`. Return `true` if you can finish all courses (no cycle in dependency graph).

---

# Example

```text
Input: numCourses = 2, prerequisites = [[1,0]]
Output: true

Input: numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: false
```

---

# Approach Hints

- Model as directed graph: edge `b -> a` (b before a).
- Cycle detection: DFS with 3 states (unvisited, visiting, visited) OR Kahn's topological sort (BFS on indegree).
- If topological order size == numCourses, no cycle.

---

# Core Idea (Walkthrough) — Kahn's Algorithm

1. Build adjacency list and indegree array.
2. Enqueue all nodes with indegree 0.
3. While queue not empty: pop, add to order, reduce indegree of neighbors; enqueue if indegree becomes 0.
4. Return `order.size() == numCourses`.

---

# Java Solution

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> adj = new ArrayList<>();
        int[] indegree = new int[numCourses];

        for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
        for (int[] p : prerequisites) {
            adj.get(p[1]).add(p[0]);
            indegree[p[0]]++;
        }

        Queue<Integer> q = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) q.offer(i);
        }

        int taken = 0;
        while (!q.isEmpty()) {
            int course = q.poll();
            taken++;
            for (int next : adj.get(course)) {
                if (--indegree[next] == 0) q.offer(next);
            }
        }

        return taken == numCourses;
    }
}
```

---

# JavaScript Solution

```javascript
function canFinish(numCourses, prerequisites) {
  const adj = Array.from({ length: numCourses }, () => []);
  const indegree = Array(numCourses).fill(0);

  for (const [a, b] of prerequisites) {
    adj[b].push(a);
    indegree[a]++;
  }

  const q = [];
  for (let i = 0; i < numCourses; i++) {
    if (indegree[i] === 0) q.push(i);
  }

  let taken = 0;
  while (q.length) {
    const course = q.shift();
    taken++;
    for (const next of adj[course]) {
      if (--indegree[next] === 0) q.push(next);
    }
  }

  return taken === numCourses;
}
```

---

# Time & Space Complexity

```text
Time:  O(V + E)
Space: O(V + E)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Number of Islands | DFS/BFS flood fill | O(m×n) |
| Rotting Oranges | Multi-source BFS | O(m×n) |
| Course Schedule | Topological sort / cycle detection | O(V+E) |

---

*End of Day 8 LeetCode*
