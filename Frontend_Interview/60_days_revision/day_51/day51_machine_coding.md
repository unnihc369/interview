# Day 51 — Machine Coding

**React:** Grid traveler path highlight  
**React Native:** 2D map shortest path

---

## Table of Contents

### React (Web)

1. [Problem — Grid Path Visualizer](#1-problem--grid-path-visualizer)
2. [DP Path Count + Backtrack](#2-dp-path-count--backtrack)
3. [React Grid UI](#3-react-grid-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [2D Map Shortest Path](#5-2d-map-shortest-path)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Grid Path Visualizer

**Task:** M×N grid. User clicks start/end. Show **unique path count** (DP) and highlight one optimal/min path.

| Feature | Behavior |
|---------|----------|
| Grid size | Configurable rows/cols |
| Obstacles | Toggle cells blocked |
| Highlight | BFS/DP backtrack one path |
| Stats | Path count, min sum (if weighted) |

---

## 2. DP Path Count + Backtrack

```js
function buildPathCountGrid(rows, cols, obstacles) {
  const blocked = (r, c) => obstacles.has(`${r},${c}`);
  const dp = Array.from({ length: rows }, () => Array(cols).fill(0));
  if (blocked(0, 0)) return dp;
  dp[0][0] = 1;
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (blocked(r, c)) { dp[r][c] = 0; continue; }
      if (r > 0) dp[r][c] += dp[r - 1][c];
      if (c > 0) dp[r][c] += dp[r][c - 1];
    }
  }
  return dp;
}

function backtrackPath(rows, cols, obstacles) {
  const dp = buildPathCountGrid(rows, cols, obstacles);
  const path = [];
  let r = rows - 1, c = cols - 1;
  while (r >= 0 && c >= 0) {
    path.push([r, c]);
    if (r === 0 && c === 0) break;
    const fromTop = r > 0 ? dp[r - 1][c] : 0;
    const fromLeft = c > 0 ? dp[r][c - 1] : 0;
    if (fromTop >= fromLeft && r > 0) r--;
    else c--;
  }
  return path.reverse();
}
```

---

## 3. React Grid UI

```tsx
import { useState, useMemo } from "react";

function GridPathVisualizer({ rows = 4, cols = 4 }) {
  const [obstacles, setObstacles] = useState<Set<string>>(new Set());

  const { count, highlight } = useMemo(() => {
    const dp = buildPathCountGrid(rows, cols, obstacles);
    const count = dp[rows - 1]?.[cols - 1] ?? 0;
    const highlight = new Set(backtrackPath(rows, cols, obstacles).map(([r, c]) => `${r},${c}`));
    return { count, highlight };
  }, [rows, cols, obstacles]);

  const toggle = (r: number, c: number) => {
    const k = `${r},${c}`;
    setObstacles((prev) => {
      const next = new Set(prev);
      next.has(k) ? next.delete(k) : next.add(k);
      return next;
    });
  };

  return (
    <div>
      <p>Unique paths: <strong>{count}</strong></p>
      <div style={{ display: "grid", gridTemplateColumns: `repeat(${cols}, 40px)`, gap: 4 }}>
        {Array.from({ length: rows * cols }, (_, idx) => {
          const r = Math.floor(idx / cols), c = idx % cols;
          const key = `${r},${c}`;
          const isPath = highlight.has(key);
          const isObs = obstacles.has(key);
          return (
            <button
              key={key}
              onClick={() => toggle(r, c)}
              style={{
                width: 40, height: 40,
                background: isObs ? "#333" : isPath ? "#90caf9" : "#eee",
              }}
            />
          );
        })}
      </div>
    </div>
  );
}
```

---

## 4. React Interview Points

- Separate **pure DP functions** from UI state.
- `useMemo` recomputes when obstacles change — O(m×n) acceptable for small grids.
- For weighted min path, backtrack prefers smaller predecessor.

---

# Part B — React Native

## 5. 2D Map Shortest Path

```tsx
import { View, Text, Pressable, StyleSheet, ScrollView } from "react-native";
import { useState, useMemo } from "react";

function minPathSumGrid(grid: number[][]) {
  const m = grid.length, n = grid[0].length;
  const dp = grid.map((row) => [...row]);
  for (let i = 1; i < m; i++) dp[i][0] += dp[i - 1][0];
  for (let j = 1; j < n; j++) dp[0][j] += dp[0][j - 1];
  for (let i = 1; i < m; i++)
    for (let j = 1; j < n; j++)
      dp[i][j] += Math.min(dp[i - 1][j], dp[i][j - 1]);
  return dp[m - 1][n - 1];
}

export default function MapShortestPath() {
  const grid = [
    [1, 3, 1],
    [1, 5, 1],
    [4, 2, 1],
  ];
  const minCost = useMemo(() => minPathSumGrid(grid), []);

  return (
    <ScrollView contentContainerStyle={styles.container}>
      <Text style={styles.title}>Min path cost: {minCost}</Text>
      {grid.map((row, r) => (
        <View key={r} style={styles.row}>
          {row.map((cell, c) => (
            <View key={c} style={styles.cell}>
              <Text>{cell}</Text>
            </View>
          ))}
        </View>
      ))}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { padding: 16 },
  title: { fontSize: 18, marginBottom: 12 },
  row: { flexDirection: "row" },
  cell: { width: 48, height: 48, borderWidth: 1, alignItems: "center", justifyContent: "center" },
});
```

---

## 6. RN Interview Points

- Use `ScrollView` for large grids on small screens.
- Pinch/zoom extension: scale cell size with `Animated`.
- Offline: precompute DP in worker for big maps.

---

**Prev/Next:** [Concepts](day51_concepts.md) · [LeetCode](day51_leetcode.md)
