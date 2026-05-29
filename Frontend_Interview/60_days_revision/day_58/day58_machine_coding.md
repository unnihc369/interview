# Day 58 — Machine Coding

**React:** Game scoring combos  
**React Native:** Animation timeline optimizer

---

## Table of Contents

### React (Web)

1. [Problem — Combo Scoring Game](#1-problem--combo-scoring-game)
2. [Interval DP for Combos](#2-interval-dp-for-combos)
3. [React Combo UI](#3-react-combo-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Animation Timeline Optimizer](#5-animation-timeline-optimizer)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Combo Scoring Game

**Task:** Array of balloon values. User "pops" in optimal order (interval DP). Show max score and burst sequence animation.

---

## 2. Interval DP for Combos

```js
function maxCoinsWithOrder(nums) {
  const arr = [1, ...nums, 1];
  const n = arr.length;
  const dp = Array.from({ length: n }, () => Array(n).fill(0));
  const choice = Array.from({ length: n }, () => Array(n).fill(-1));

  for (let len = 3; len <= n; len++) {
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      for (let k = i + 1; k < j; k++) {
        const score = dp[i][k] + dp[k][j] + arr[i] * arr[k] * arr[j];
        if (score > dp[i][j]) {
          dp[i][j] = score;
          choice[i][j] = k;
        }
      }
    }
  }
  return { max: dp[0][n - 1], order: getBurstOrder(choice, 0, n - 1, arr) };
}

function getBurstOrder(choice, i, j, arr) {
  if (j - i < 2) return [];
  const k = choice[i][j];
  return [...getBurstOrder(choice, i, k, arr), arr[k], ...getBurstOrder(choice, k, j, arr)];
}
```

---

## 3. React Combo UI

```tsx
import { useState, useMemo } from "react";

export default function ComboScoringGame() {
  const [values, setValues] = useState([3, 1, 5, 8]);
  const result = useMemo(() => maxCoinsWithOrder(values), [values]);
  const [step, setStep] = useState(0);

  return (
    <div style={{ padding: 24 }}>
      <h2>Max combo score: {result.max}</h2>
      <div style={{ display: "flex", gap: 8 }}>
        {values.map((v, i) => (
          <div
            key={i}
            style={{
              width: 48, height: 48, borderRadius: 24,
              background: result.order[step] === v ? "#ff9800" : "#e3f2fd",
              display: "flex", alignItems: "center", justifyContent: "center",
            }}
          >
            {v}
          </div>
        ))}
      </div>
      <button onClick={() => setStep((s) => Math.min(s + 1, result.order.length - 1))}>
        Next burst
      </button>
    </div>
  );
}
```

---

## 4. React Interview Points

- Interval DP fill order: length ascending.
- Animate burst sequence step-by-step.
- Pad array with 1s (boundary balloons).

---

# Part B — React Native

## 5. Animation Timeline Optimizer

**Task:** Keyframes on timeline. Merge overlapping intervals for minimum segments (interval DP analogy).

```tsx
import { View, Text, Pressable, StyleSheet } from "react-native";
import { useMemo, useState } from "react";

type Segment = { start: number; end: number; label: string };

function minScoreTriangulation(values: number[]) {
  const n = values.length;
  const dp = Array.from({ length: n }, () => Array(n).fill(0));
  for (let len = 3; len <= n; len++) {
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      dp[i][j] = Infinity;
      for (let k = i + 1; k < j; k++) {
        dp[i][j] = Math.min(dp[i][j], dp[i][k] + dp[k][j] + values[i] * values[k] * values[j]);
      }
    }
  }
  return dp[0][n - 1];
}

export default function TimelineOptimizer() {
  const weights = [2, 3, 4, 5];
  const cost = useMemo(() => minScoreTriangulation(weights), []);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Timeline merge cost: {cost}</Text>
      <Text style={styles.sub}>Interval DP optimizes keyframe groupings</Text>
      {weights.map((w, i) => (
        <View key={i} style={[styles.segment, { flex: w }]}>
          <Text style={styles.label}>K{i}</Text>
        </View>
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  title: { fontSize: 18, fontWeight: "600" },
  sub: { color: "#666", marginBottom: 16 },
  segment: { height: 40, backgroundColor: "#90caf9", marginVertical: 4, justifyContent: "center", paddingLeft: 8 },
  label: { fontSize: 12 },
});
```

---

## 6. RN Interview Points

- `Animated` stagger uses similar "optimal grouping" thinking.
- Reanimated shared values for 60fps timeline scrubbing.
- Precompute interval DP off UI thread for long timelines.

---

**Prev/Next:** [Concepts](day58_concepts.md) · [LeetCode](day58_leetcode.md)
