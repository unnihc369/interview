# Day 52 — Machine Coding

**React:** Budget allocation knapsack  
**React Native:** Backpack packing

---

## Table of Contents

### React (Web)

1. [Problem — Budget Allocator](#1-problem--budget-allocator)
2. [0/1 Knapsack Max Value](#2-01-knapsack-max-value)
3. [React Budget UI](#3-react-budget-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Backpack Packing](#5-backpack-packing)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Budget Allocator

**Task:** Given budget and projects (cost, ROI), select projects **at most once each** to maximize ROI without exceeding budget.

| Feature | Behavior |
|---------|----------|
| Projects list | name, cost, value |
| Budget slider | max spend |
| Selection | highlight chosen projects |
| Output | total cost, total value |

---

## 2. 0/1 Knapsack Max Value

```js
function knapsack01(items, capacity) {
  const dp = Array(capacity + 1).fill(0);
  const pick = Array.from({ length: capacity + 1 }, () => []);

  for (const { id, cost, value } of items) {
    for (let w = capacity; w >= cost; w--) {
      const candidate = dp[w - cost] + value;
      if (candidate > dp[w]) {
        dp[w] = candidate;
        pick[w] = [...pick[w - cost], id];
      }
    }
  }
  return { maxValue: dp[capacity], selected: pick[capacity] };
}
```

---

## 3. React Budget UI

```tsx
import { useState, useMemo } from "react";

const PROJECTS = [
  { id: "a", name: "Feature A", cost: 5, value: 12 },
  { id: "b", name: "Feature B", cost: 3, value: 7 },
  { id: "c", name: "Feature C", cost: 4, value: 10 },
  { id: "d", name: "Feature D", cost: 2, value: 4 },
];

export default function BudgetAllocator() {
  const [budget, setBudget] = useState(8);
  const result = useMemo(() => knapsack01(PROJECTS, budget), [budget]);

  return (
    <div style={{ padding: 24 }}>
      <h2>Budget: ${budget}</h2>
      <input type="range" min={0} max={15} value={budget} onChange={(e) => setBudget(+e.target.value)} />
      <p>Max ROI: {result.maxValue}</p>
      <ul>
        {PROJECTS.map((p) => (
          <li key={p.id} style={{ fontWeight: result.selected.includes(p.id) ? "bold" : "normal" }}>
            {p.name} — cost {p.cost}, value {p.value}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 4. React Interview Points

- 0/1 knapsack: iterate capacity **descending** when updating in-place.
- Reconstruction via `pick[]` or backtrack from `dp[i][w]`.
- Greedy fails — must use DP.

---

# Part B — React Native

## 5. Backpack Packing

```tsx
import { View, Text, FlatList, StyleSheet } from "react-native";
import { useMemo, useState } from "react";

type Item = { id: string; name: string; weight: number; value: number };

function packBackpack(items: Item[], maxWeight: number) {
  const dp = Array(maxWeight + 1).fill(0);
  const chosen: string[][] = Array.from({ length: maxWeight + 1 }, () => []);
  for (const item of items) {
    for (let w = maxWeight; w >= item.weight; w--) {
      const v = dp[w - item.weight] + item.value;
      if (v > dp[w]) {
        dp[w] = v;
        chosen[w] = [...chosen[w - item.weight], item.id];
      }
    }
  }
  return { value: dp[maxWeight], ids: chosen[maxWeight] };
}

export default function BackpackScreen() {
  const [capacity] = useState(10);
  const items: Item[] = [
    { id: "1", name: "Tent", weight: 4, value: 8 },
    { id: "2", name: "Stove", weight: 2, value: 5 },
    { id: "3", name: "Rope", weight: 3, value: 6 },
  ];
  const packed = useMemo(() => packBackpack(items, capacity), []);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Capacity: {capacity}kg · Value: {packed.value}</Text>
      <FlatList
        data={items}
        keyExtractor={(i) => i.id}
        renderItem={({ item }) => (
          <Text style={packed.ids.includes(item.id) ? styles.packed : undefined}>
            {item.name} ({item.weight}kg)
          </Text>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  title: { fontSize: 18, marginBottom: 12 },
  packed: { fontWeight: "700", color: "#2e7d32" },
});
```

---

## 6. RN Interview Points

- `FlatList` for scrollable item list.
- Memoize knapsack when items/capacity static.
- Haptic feedback on optimal pack recalc (optional polish).

---

**Prev/Next:** [Concepts](day52_concepts.md) · [LeetCode](day52_leetcode.md)
