# Day 56 — Machine Coding

**React:** Route planner small cities  
**React Native:** TSP map preview

---

## Table of Contents

### React (Web)

1. [Problem — City Route Planner](#1-problem--city-route-planner)
2. [Bitmask TSP Solver](#2-bitmask-tsp-solver)
3. [React Route UI](#3-react-route-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [TSP Map Preview](#5-tsp-map-preview)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — City Route Planner

**Task:** 5–8 cities with distance matrix. Find shortest tour visiting all cities (TSP). Visualize route on simple SVG map.

---

## 2. Bitmask TSP Solver

```js
function tsp(dist) {
  const n = dist.length;
  const FULL = (1 << n) - 1;
  const dp = Array.from({ length: 1 << n }, () => Array(n).fill(Infinity));
  const parent = Array.from({ length: 1 << n }, () => Array(n).fill(-1));
  dp[1][0] = 0;

  for (let mask = 1; mask <= FULL; mask++) {
    for (let u = 0; u < n; u++) {
      if (!(mask & (1 << u)) || dp[mask][u] === Infinity) continue;
      for (let v = 0; v < n; v++) {
        if (mask & (1 << v)) continue;
        const nm = mask | (1 << v);
        const cost = dp[mask][u] + dist[u][v];
        if (cost < dp[nm][v]) {
          dp[nm][v] = cost;
          parent[nm][v] = u;
        }
      }
    }
  }

  let best = Infinity, end = -1;
  for (let u = 0; u < n; u++) {
    const c = dp[FULL][u] + dist[u][0];
    if (c < best) { best = c; end = u; }
  }
  return { cost: best, path: reconstruct(parent, FULL, end) };
}

function reconstruct(parent, mask, node) {
  const path = [node];
  while (parent[mask][node] !== -1) {
    const prev = parent[mask][node];
    mask ^= 1 << node;
    node = prev;
    path.push(node);
  }
  return path.reverse();
}
```

---

## 3. React Route UI

```tsx
import { useMemo } from "react";

const CITIES = ["A", "B", "C", "D"];
const DIST = [
  [0, 10, 15, 20],
  [10, 0, 35, 25],
  [15, 35, 0, 30],
  [20, 25, 30, 0],
];

export default function RoutePlanner() {
  const { cost, path } = useMemo(() => tsp(DIST), []);

  return (
    <div style={{ padding: 24 }}>
      <h2>Shortest tour: {cost}</h2>
      <p>Route: {path.map((i) => CITIES[i]).join(" → ")} → {CITIES[path[0]]}</p>
      <svg width={300} height={200}>
        {path.map((cityIdx, i) => {
          const next = path[(i + 1) % path.length];
          const x1 = 50 + cityIdx * 60, y1 = 100;
          const x2 = 50 + next * 60, y2 = 100;
          return <line key={i} x1={x1} y1={y1} x2={x2} y2={y2} stroke="#1565c0" strokeWidth={2} />;
        })}
        {CITIES.map((c, i) => (
          <g key={c}>
            <circle cx={50 + i * 60} cy={100} r={16} fill="#90caf9" />
            <text x={50 + i * 60} y={105} textAnchor="middle" fontSize={12}>{c}</text>
          </g>
        ))}
      </svg>
    </div>
  );
}
```

---

## 4. React Interview Points

- Cap cities at ~12 for client-side TSP.
- `useMemo` for expensive bitmask DP.
- For larger n, use heuristic (nearest neighbor) with disclaimer.

---

# Part B — React Native

## 5. TSP Map Preview

```tsx
import { View, Text, StyleSheet } from "react-native";
import { useMemo } from "react";

const POINTS = [
  { id: 0, label: "Home", x: 20, y: 80 },
  { id: 1, label: "Shop", x: 120, y: 40 },
  { id: 2, label: "Gym", x: 200, y: 100 },
];

function buildDist(points) {
  const n = points.length;
  const d = Array.from({ length: n }, () => Array(n).fill(0));
  for (let i = 0; i < n; i++)
    for (let j = 0; j < n; j++) {
      const dx = points[i].x - points[j].x;
      const dy = points[i].y - points[j].y;
      d[i][j] = Math.round(Math.hypot(dx, dy));
    }
  return d;
}

export default function TspMapPreview() {
  const dist = useMemo(() => buildDist(POINTS), []);
  const { cost, path } = useMemo(() => tsp(dist), [dist]);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Tour distance: {cost}px</Text>
      <View style={styles.map}>
        {POINTS.map((p) => (
          <View key={p.id} style={[styles.pin, { left: p.x, top: p.y }]}>
            <Text style={styles.pinText}>{p.label}</Text>
          </View>
        ))}
      </View>
      <Text>{path.map((i) => POINTS[i].label).join(" → ")}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  title: { fontSize: 18, marginBottom: 12 },
  map: { height: 200, backgroundColor: "#e3f2fd", marginBottom: 12, position: "relative" },
  pin: { position: "absolute", backgroundColor: "#1565c0", padding: 6, borderRadius: 4 },
  pinText: { color: "#fff", fontSize: 10 },
});
```

---

## 6. RN Interview Points

- Absolute positioning for simple map pins — production uses MapView.
- Precompute TSP when points change only.
- Show loading indicator for n > 10.

---

**Prev/Next:** [Concepts](day56_concepts.md) · [LeetCode](day56_leetcode.md)
