# Day 50 — Machine Coding

**React:** Fibonacci visualizer with memo  
**React Native:** Climbing stairs animation

---

## Table of Contents

### React (Web)

1. [Problem — Fibonacci Visualizer](#1-problem--fibonacci-visualizer)
2. [Memoized Fib with Call Tree](#2-memoized-fib-with-call-tree)
3. [React UI — Recursion Tree](#3-react-ui--recursion-tree)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Climbing Stairs Animation](#5-climbing-stairs-animation)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Fibonacci Visualizer

**Task:** Interactive Fibonacci calculator that visualizes **memoization hits vs new computations** on a recursion tree.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Input `n` | Slider 0–20 |
| Compute | Show `F(n)` and call count |
| Tree view | Nodes: computed (blue) vs cache hit (green) |
| Toggle | Compare naive vs memoized call counts |

---

## 2. Memoized Fib with Call Tree

```js
function fibWithTrace(n) {
  const memo = new Map();
  const nodes = []; // { id, parent, label, cached }

  function dp(i, parentId = null) {
    const id = nodes.length;
    nodes.push({ id, parent: parentId, label: `fib(${i})`, cached: false });

    if (i <= 1) {
      nodes[id].result = i;
      return i;
    }
    if (memo.has(i)) {
      nodes[id].cached = true;
      nodes[id].result = memo.get(i);
      return memo.get(i);
    }

    const left = dp(i - 1, id);
    const right = dp(i - 2, id);
    const val = left + right;
    memo.set(i, val);
    nodes[id].result = val;
    return val;
  }

  const result = dp(n);
  return { result, nodes };
}
```

### Naive for comparison

```js
function fibNaiveCount(n, stats = { calls: 0 }) {
  stats.calls++;
  if (n <= 1) return n;
  return fibNaiveCount(n - 1, stats) + fibNaiveCount(n - 2, stats);
}
```

---

## 3. React UI — Recursion Tree

```tsx
import { useState, useMemo } from "react";

type Node = {
  id: number;
  parent: number | null;
  label: string;
  cached: boolean;
  result?: number;
};

function fibWithTrace(n: number): { result: number; nodes: Node[] } {
  const memo = new Map<number, number>();
  const nodes: Node[] = [];

  function dp(i: number, parentId: number | null = null): number {
    const id = nodes.length;
    nodes.push({ id, parent: parentId, label: `fib(${i})`, cached: false });

    if (i <= 1) {
      nodes[id].result = i;
      return i;
    }
    if (memo.has(i)) {
      nodes[id].cached = true;
      nodes[id].result = memo.get(i)!;
      return memo.get(i)!;
    }
    const val = dp(i - 1, id) + dp(i - 2, id);
    memo.set(i, val);
    nodes[id].result = val;
    return val;
  }

  return { result: dp(n), nodes };
}

export default function FibVisualizer() {
  const [n, setN] = useState(8);
  const { result, nodes } = useMemo(() => fibWithTrace(n), [n]);
  const cacheHits = nodes.filter((x) => x.cached).length;

  return (
    <div style={{ fontFamily: "monospace", padding: 24 }}>
      <h2>Fibonacci Memo Visualizer</h2>
      <label>
        n = {n}
        <input
          type="range"
          min={0}
          max={20}
          value={n}
          onChange={(e) => setN(Number(e.target.value))}
        />
      </label>
      <p>
        F({n}) = <strong>{result}</strong> · Cache hits: {cacheHits} / {nodes.length} nodes
      </p>
      <ul style={{ listStyle: "none", padding: 0 }}>
        {nodes.map((node) => (
          <li
            key={node.id}
            style={{
              marginLeft: node.parent != null ? 24 : 0,
              color: node.cached ? "#2e7d32" : "#1565c0",
            }}
          >
            {node.label} → {node.result}
            {node.cached ? " (memo hit)" : ""}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 4. React Interview Points

- Explain **why** memo turns O(2^n) into O(n): each `i` computed once.
- `useMemo` recalculates when `n` changes — separate from algorithmic memo inside `fibWithTrace`.
- For large `n`, cap slider to avoid DOM explosion; mention virtualized tree in production.

---

# Part B — React Native

## 5. Climbing Stairs Animation

**Task:** Animate a character climbing stairs. Show step count and **number of distinct paths** (DP result) as user sets target step.

```tsx
import { useState, useEffect, useRef } from "react";
import { View, Text, Button, Animated, StyleSheet } from "react-native";

function climbStairs(n: number): number {
  const memo: Record<number, number> = {};
  function dp(i: number): number {
    if (i <= 1) return 1;
    if (memo[i] != null) return memo[i];
    memo[i] = dp(i - 1) + dp(i - 2);
    return memo[i];
  }
  return dp(n);
}

export default function ClimbingStairsScreen() {
  const [steps, setSteps] = useState(5);
  const [current, setCurrent] = useState(0);
  const yAnim = useRef(new Animated.Value(0)).current;
  const ways = climbStairs(steps);

  useEffect(() => {
    setCurrent(0);
    yAnim.setValue(0);
  }, [steps]);

  const animateStep = () => {
    if (current >= steps) return;
    const next = Math.min(current + (Math.random() > 0.5 ? 2 : 1), steps);
    Animated.timing(yAnim, {
      toValue: -next * 40,
      duration: 400,
      useNativeDriver: true,
    }).start(() => setCurrent(next));
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Climb {steps} steps</Text>
      <Text>Distinct paths (DP): {ways}</Text>
      <View style={styles.stairwell}>
        {Array.from({ length: steps + 1 }, (_, i) => (
          <View key={i} style={[styles.step, { bottom: i * 40 }]} />
        ))}
        <Animated.View style={[styles.avatar, { transform: [{ translateY: yAnim }] }]} />
      </View>
      <View style={styles.row}>
        <Button title="−" onPress={() => setSteps((s) => Math.max(1, s - 1))} />
        <Button title="Random step" onPress={animateStep} />
        <Button title="+" onPress={() => setSteps((s) => s + 1)} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24 },
  title: { fontSize: 20, fontWeight: "600" },
  stairwell: { height: 300, marginVertical: 24, position: "relative" },
  step: { position: "absolute", left: 0, right: 0, height: 8, backgroundColor: "#ccc" },
  avatar: {
    position: "absolute",
    bottom: 0,
    left: 40,
    width: 32,
    height: 32,
    borderRadius: 16,
    backgroundColor: "#e91e63",
  },
  row: { flexDirection: "row", justifyContent: "space-around" },
});
```

---

## 6. RN Interview Points

- `useNativeDriver: true` for transform animations (GPU).
- DP value (`ways`) is pure computation — keep separate from animation state.
- Reset animation when `steps` changes via `useEffect`.

---

**Prev/Next:** [Concepts](day50_concepts.md) · [LeetCode](day50_leetcode.md)
