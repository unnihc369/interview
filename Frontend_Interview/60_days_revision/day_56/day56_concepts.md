# Day 56 — DP Bitmask (TSP)

**Week 8 — Dynamic Programming** · **Topics:** Bitmask DP · State compression · TSP · Held-Karp

---

## Table of Contents

1. [Bitmask State Representation](#1-bitmask-state-representation)
2. [Traveling Salesman Intuition](#2-traveling-salesman-intuition)
3. [Shortest Path Visiting All Nodes (847)](#3-shortest-path-visiting-all-nodes-847)
4. [Held-Karp Template](#4-held-karp-template)
5. [When n ≤ 20](#5-when-n--20)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Bitmask State Representation

Visit set of nodes encoded as integer bitmask:

```js
const visited = 0b1011; // nodes 0,1,3 visited
const visit = (mask, i) => mask | (1 << i);
const isVisited = (mask, i) => (mask >> i) & 1;
const countBits = (mask) => mask.toString(2).split("1").length - 1;
```

`dp[mask][i]` = min cost to reach node `i` having visited exactly the set in `mask`.

---

## 2. Traveling Salesman Intuition

Start at node 0, visit all nodes, return to start (optional).  
State space: O(2^n × n) — feasible for n ≤ 15–20.

```js
function tsp(dist) {
  const n = dist.length;
  const FULL = (1 << n) - 1;
  const dp = Array.from({ length: 1 << n }, () => Array(n).fill(Infinity));
  dp[1][0] = 0; // start at node 0

  for (let mask = 1; mask < (1 << n); mask++) {
    for (let u = 0; u < n; u++) {
      if (!((mask >> u) & 1)) continue;
      for (let v = 0; v < n; v++) {
        if ((mask >> v) & 1) continue;
        const next = mask | (1 << v);
        dp[next][v] = Math.min(dp[next][v], dp[mask][u] + dist[u][v]);
      }
    }
  }
  let ans = Infinity;
  for (let u = 0; u < n; u++) {
    ans = Math.min(ans, dp[FULL][u] + dist[u][0]);
  }
  return ans;
}
```

---

## 3. Shortest Path Visiting All Nodes (847)

Graph may need revisiting nodes. BFS from each node with bitmask state `(mask, node)`.

```js
function shortestPathLength(graph) {
  const n = graph.length;
  const ALL = (1 << n) - 1;
  const q = [];
  const seen = new Set();
  for (let i = 0; i < n; i++) {
    q.push([i, 1 << i, 0]);
    seen.add(`${i},${1 << i}`);
  }
  while (q.length) {
    const [node, mask, dist] = q.shift();
    if (mask === ALL) return dist;
    for (const nei of graph[node]) {
      const nm = mask | (1 << nei);
      const key = `${nei},${nm}`;
      if (!seen.has(key)) {
        seen.add(key);
        q.push([nei, nm, dist + 1]);
      }
    }
  }
  return -1;
}
```

---

## 4. Held-Karp Template

```js
// dp[mask][last] = min cost ending at `last` with visited `mask`
for (let mask = 0; mask < (1 << n); mask++) {
  for (let last = 0; last < n; last++) {
    if (!isVisited(mask, last)) continue;
    for (let next = 0; next < n; next++) {
      if (isVisited(mask, next)) continue;
      relax(dp, mask, last, next);
    }
  }
}
```

---

## 5. When n ≤ 20

| n | 2^n | Feasible? |
|---|-----|-----------|
| 10 | 1024 | Yes |
| 15 | 32768 | Yes |
| 20 | ~1M | Borderline |
| 25 | ~33M | Usually no |

Use bitmask when problem says "visit all" and n is small.

---

## 6. Interview Quick Index

| LC | Approach |
|----|----------|
| 847 | BFS + bitmask |
| 943 | TSP on strings (hard) |
| 1349 | DP mask per row (bitmask seats) |

---

**Next:** [Machine Coding](day56_machine_coding.md) · [LeetCode](day56_leetcode.md)
