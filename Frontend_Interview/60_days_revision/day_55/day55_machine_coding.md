# Day 55 — Machine Coding

**React:** Org chart max happiness  
**React Native:** Family inheritance

---

## Table of Contents

### React (Web)

1. [Problem — Org Chart Optimizer](#1-problem--org-chart-optimizer)
2. [Tree DP on Org Chart](#2-tree-dp-on-org-chart)
3. [React Org Chart UI](#3-react-org-chart-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Family Inheritance](#5-family-inheritance)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Org Chart Optimizer

**Task:** Each employee has a "happiness score." Select a subset where **no manager and report are both selected**. Maximize total happiness.

*(Same as House Robber III on a tree.)*

---

## 2. Tree DP on Org Chart

```js
type Employee = { id: string; name: string; score: number; reports: Employee[] };

function maxHappiness(root) {
  function dfs(node) {
    if (!node) return { pick: 0, skip: 0 };
    let pick = node.score;
    let skip = 0;
    for (const child of node.reports) {
      const { pick: cp, skip: cs } = dfs(child);
      pick += cs;
      skip += Math.max(cp, cs);
    }
    return { pick, skip };
  }
  const { pick, skip } = dfs(root);
  return Math.max(pick, skip);
}
```

---

## 3. React Org Chart UI

```tsx
import { useMemo } from "react";

const ORG = {
  id: "ceo", name: "CEO", score: 10,
  reports: [
    { id: "eng", name: "Eng Lead", score: 8, reports: [
      { id: "dev1", name: "Dev 1", score: 5, reports: [] },
      { id: "dev2", name: "Dev 2", score: 6, reports: [] },
    ]},
    { id: "sales", name: "Sales Lead", score: 7, reports: [] },
  ],
};

function OrgNode({ node, selected }: { node: typeof ORG; selected: Set<string> }) {
  const isSelected = selected.has(node.id);
  return (
    <div style={{ marginLeft: 24, borderLeft: "2px solid #ccc", paddingLeft: 12 }}>
      <div style={{ background: isSelected ? "#c8e6c9" : "#f5f5f5", padding: 8, display: "inline-block" }}>
        {node.name} (+{node.score})
      </div>
      {node.reports.map((r) => (
        <OrgNode key={r.id} node={r} selected={selected} />
      ))}
    </div>
  );
}

export default function OrgChartOptimizer() {
  const maxScore = useMemo(() => maxHappiness(ORG), []);
  return (
    <div style={{ padding: 24 }}>
      <h2>Max team happiness: {maxScore}</h2>
      <OrgNode node={ORG} selected={new Set(["ceo", "dev1", "sales"])} />
    </div>
  );
}
```

---

## 4. React Interview Points

- Tree DP maps naturally to org charts / component trees.
- Visual selection must respect parent-child constraint.
- Recursive component for tree rendering — key on `id`.

---

# Part B — React Native

## 5. Family Inheritance

```tsx
import { View, Text, StyleSheet } from "react-native";

type Relative = { id: string; name: string; wealth: number; children: Relative[] };

function maxInheritance(node: Relative | null): number {
  if (!node) return 0;
  function dfs(n: Relative): [number, number] {
    if (!n.children.length) return [n.wealth, 0];
    let rob = n.wealth, skip = 0;
    for (const c of n.children) {
      const [cr, cs] = dfs(c);
      rob += cs;
      skip += Math.max(cr, cs);
    }
    return [rob, skip];
  }
  const [r, s] = dfs(node);
  return Math.max(r, s);
}

const FAMILY: Relative = {
  id: "1", name: "Grandparent", wealth: 100,
  children: [
    { id: "2", name: "Parent A", wealth: 50, children: [
      { id: "4", name: "Child", wealth: 20, children: [] },
    ]},
    { id: "3", name: "Parent B", wealth: 40, children: [] },
  ],
};

export default function InheritanceScreen() {
  const total = maxInheritance(FAMILY);
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Optimal inheritance</Text>
      <Text style={styles.amount}>${total}</Text>
      <Text style={styles.note}>Cannot inherit from adjacent generations</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24, justifyContent: "center" },
  title: { fontSize: 22 },
  amount: { fontSize: 36, fontWeight: "700", color: "#2e7d32", marginVertical: 16 },
  note: { color: "#666" },
});
```

---

## 6. RN Interview Points

- Same tree DP as web — platform-agnostic logic.
- Flatten tree for `FlatList` if displaying all members.
- Accessibility: announce optimal total via `accessibilityLiveRegion`.

---

**Prev/Next:** [Concepts](day55_concepts.md) · [LeetCode](day55_leetcode.md)
