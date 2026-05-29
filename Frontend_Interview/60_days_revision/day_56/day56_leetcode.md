# Day 56 — LeetCode (Bitmask DP)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 847 | [Shortest Path Visiting All Nodes](#1-shortest-path-visiting-all-nodes-847) | Hard | BFS + bitmask |
| 943 | [Find the Shortest Superstring](#2-find-the-shortest-superstring-943) | Hard | TSP bitmask |
| 1349 | [Maximum Students Taking Exam](#3-maximum-students-taking-exam-1349) | Hard | Row bitmask DP |

---

## Table of Contents

1. [Shortest Path Visiting All Nodes (847)](#1-shortest-path-visiting-all-nodes-847)
2. [Find the Shortest Superstring (943)](#2-find-the-shortest-superstring-943)
3. [Maximum Students Taking Exam (1349)](#3-maximum-students-taking-exam-1349)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Shortest Path Visiting All Nodes (847)

### Solution — Multi-source BFS

```js
function shortestPathLength(graph) {
  const n = graph.length;
  const ALL = (1 << n) - 1;
  const q = [[0, 1, 0]]; // start all nodes
  for (let i = 1; i < n; i++) q.push([i, 1 << i, 0]);
  const seen = new Set(q.map(([n, m]) => `${n},${m}`));

  while (q.length) {
    const [node, mask, d] = q.shift();
    if (mask === ALL) return d;
    for (const nei of graph[node]) {
      const nm = mask | (1 << nei);
      const key = `${nei},${nm}`;
      if (!seen.has(key)) {
        seen.add(key);
        q.push([nei, nm, d + 1]);
      }
    }
  }
  return 0;
}
```

---

## 2. Find the Shortest Superstring (943)

### Hint

Reduce to TSP: cost(i,j) = overlap when concatenating words[i] before words[j].

### Solution sketch

```js
function shortestSuperstring(words) {
  const n = words.length;
  const overlap = Array.from({ length: n }, () => Array(n).fill(0));
  for (let i = 0; i < n; i++)
    for (let j = 0; j < n; j++)
      for (let k = Math.min(words[i].length, words[j].length); k > 0; k--)
        if (words[i].slice(-k) === words[j].slice(0, k)) {
          overlap[i][j] = k;
          break;
        }
  // Held-Karp on overlap → maximize total overlap
  const dp = Array.from({ length: 1 << n }, () => Array(n).fill(-Infinity));
  const parent = Array.from({ length: 1 << n }, () => Array(n).fill(-1));
  for (let i = 0; i < n; i++) { dp[1 << i][i] = 0; }

  for (let mask = 1; mask < (1 << n); mask++) {
    for (let last = 0; last < n; last++) {
      if (!(mask & (1 << last)) || dp[mask][last] < 0) continue;
      for (let next = 0; next < n; next++) {
        if (mask & (1 << next)) continue;
        const nm = mask | (1 << next);
        const val = dp[mask][last] + overlap[last][next];
        if (val > dp[nm][next]) { dp[nm][next] = val; parent[nm][next] = last; }
      }
    }
  }
  // reconstruct superstring from best ending node
  // ...
}
```

---

## 3. Maximum Students Taking Exam (1349)

### Solution sketch — DP over rows with seat bitmask

```js
function maxStudents(seats) {
  const m = seats.length, n = seats[0].length;
  const valid = [];
  for (let mask = 0; mask < (1 << n); mask++) {
    if (mask & (mask << 1)) continue; // no adjacent in row
    let ok = true;
    for (let j = 0; j < n; j++)
      if ((mask >> j) & 1 && seats[0][j] === '#') ok = false;
    if (ok) valid.push(mask);
  }
  // dp[row][mask] = max students with row config `mask`
  // transition: check upper-left/right conflicts between rows
}
```

---

## 4. Pattern Cheat Sheet

| Problem | State |
|---------|-------|
| 847 | (node, visitedMask) |
| 943 | (visitedMask, lastWord) |
| 1349 | (row, seatMask) |

---

**Prev/Next:** [Concepts](day56_concepts.md) · [Machine Coding](day56_machine_coding.md)
