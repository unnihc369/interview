# Day 32 - Graph Shortest Path

---

# 1. Shortest Path in Binary Matrix

## Problem Statement

Given `n x n` binary grid, return length of shortest **clear** path from top-left to bottom-right. Path through cells with value `0`; 8-directional movement. Return -1 if no path.

---

# Core Idea

BFS from (0,0) — unweighted shortest path. Mark visited cells.

---

# Java Solution

```java
class Solution {
    private static final int[][] DIRS = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};

    public int shortestPathBinaryMatrix(int[][] grid) {
        int n = grid.length;
        if (grid[0][0] == 1 || grid[n-1][n-1] == 1) return -1;
        if (n == 1) return 1;

        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{0, 0, 1});
        grid[0][0] = 1;

        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            int r = curr[0], c = curr[1], dist = curr[2];
            if (r == n - 1 && c == n - 1) return dist;

            for (int[] d : DIRS) {
                int nr = r + d[0], nc = c + d[1];
                if (nr >= 0 && nc >= 0 && nr < n && nc < n && grid[nr][nc] == 0) {
                    grid[nr][nc] = 1;
                    queue.offer(new int[]{nr, nc, dist + 1});
                }
            }
        }
        return -1;
    }
}
```

---

# JavaScript Solution

```javascript
function shortestPathBinaryMatrix(grid) {
  const n = grid.length;
  if (grid[0][0] === 1 || grid[n-1][n-1] === 1) return -1;
  if (n === 1) return 1;
  const dirs = [[-1,-1],[-1,0],[-1,1],[0,-1],[0,1],[1,-1],[1,0],[1,1]];
  const queue = [[0, 0, 1]];
  grid[0][0] = 1;

  while (queue.length) {
    const [r, c, dist] = queue.shift();
    if (r === n - 1 && c === n - 1) return dist;
    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nc >= 0 && nr < n && nc < n && grid[nr][nc] === 0) {
        grid[nr][nc] = 1;
        queue.push([nr, nc, dist + 1]);
      }
    }
  }
  return -1;
}
```

---

# Time Complexity

```text
O(n^2)
```

---

# 2. Network Delay Time

## Problem Statement

`n` nodes, `times[i] = (ui, vi, wi)` directed weighted edges. Signal sent from node `k`. Return time for all nodes to receive signal, or -1 if impossible.

---

# Core Idea

Dijkstra's algorithm from source `k`. Answer = max distance to all reachable nodes.

---

# Java Solution

```java
class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        Map<Integer, List<int[]>> graph = new HashMap<>();
        for (int[] t : times) {
            graph.computeIfAbsent(t[0], x -> new ArrayList<>())
                 .add(new int[]{t[1], t[2]});
        }

        int[] dist = new int[n + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[k] = 0;

        PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
        pq.offer(new int[]{k, 0});

        while (!pq.isEmpty()) {
            int[] curr = pq.poll();
            int node = curr[0], d = curr[1];
            if (d > dist[node]) continue;

            for (int[] edge : graph.getOrDefault(node, List.of())) {
                int next = edge[0], w = edge[1];
                if (dist[node] + w < dist[next]) {
                    dist[next] = dist[node] + w;
                    pq.offer(new int[]{next, dist[next]});
                }
            }
        }

        int max = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == Integer.MAX_VALUE) return -1;
            max = Math.max(max, dist[i]);
        }
        return max;
    }
}
```

---

# JavaScript Solution

```javascript
function networkDelayTime(times, n, k) {
  const graph = {};
  for (const [u, v, w] of times) {
    if (!graph[u]) graph[u] = [];
    graph[u].push([v, w]);
  }

  const dist = Array(n + 1).fill(Infinity);
  dist[k] = 0;
  const pq = [[0, k]]; // [distance, node]

  while (pq.length) {
    pq.sort((a, b) => a[0] - b[0]);
    const [d, node] = pq.shift();
    if (d > dist[node]) continue;
    for (const [next, w] of graph[node] || []) {
      if (dist[node] + w < dist[next]) {
        dist[next] = dist[node] + w;
        pq.push([dist[next], next]);
      }
    }
  }

  let max = 0;
  for (let i = 1; i <= n; i++) {
    if (dist[i] === Infinity) return -1;
    max = Math.max(max, dist[i]);
  }
  return max;
}
```

---

# Time Complexity

```text
O(E log V) with binary heap
```

---

# 3. Cheapest Flights Within K Stops

## Problem Statement

`n` cities, flights `flights[i] = [from, to, price]`, find cheapest price from `src` to `dst` with at most `k` stops.

---

# Core Idea

**Bellman-Ford style:** relax all edges for `k+1` iterations (at most k+1 edges = k stops).

Or BFS with (node, cost, stops) — prune when stops > k.

---

# Java Solution (Bellman-Ford)

```java
class Solution {
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[src] = 0;

        for (int i = 0; i <= k; i++) {
            int[] temp = dist.clone();
            for (int[] f : flights) {
                int u = f[0], v = f[1], w = f[2];
                if (dist[u] != Integer.MAX_VALUE && dist[u] + w < temp[v]) {
                    temp[v] = dist[u] + w;
                }
            }
            dist = temp;
        }
        return dist[dst] == Integer.MAX_VALUE ? -1 : dist[dst];
    }
}
```

---

# JavaScript Solution

```javascript
function findCheapestPrice(n, flights, src, dst, k) {
  let dist = Array(n).fill(Infinity);
  dist[src] = 0;

  for (let i = 0; i <= k; i++) {
    const temp = [...dist];
    for (const [u, v, w] of flights) {
      if (dist[u] !== Infinity && dist[u] + w < temp[v]) {
        temp[v] = dist[u] + w;
      }
    }
    dist = temp;
  }
  return dist[dst] === Infinity ? -1 : dist[dst];
}
```

---

# Time Complexity

```text
O(k * E)
```

---

# Pattern Quick Summary

| Problem | Algorithm | Time |
|---------|-----------|------|
| Shortest Path Binary Matrix | BFS (unweighted) | O(n²) |
| Network Delay Time | Dijkstra | O(E log V) |
| Cheapest Flights K Stops | Bellman-Ford (k+1 relax) | O(k·E) |

---

# Interview Questions

## Binary Matrix — why BFS not DFS?

Unweighted graph → BFS finds shortest path first time reaching target.

## Network Delay — Bellman-Ford vs Dijkstra?

No negative edges here → Dijkstra preferred. Bellman-Ford O(VE) works for negative edges with cycle check.

## Cheapest Flights — why clone dist each iteration?

Prevents using more than k stops in same round (path with k+2 edges in one iteration).

## Dijkstra with negative edges?

Fails — use Bellman-Ford or SPFA.

---

*End of Day 32 LeetCode*
