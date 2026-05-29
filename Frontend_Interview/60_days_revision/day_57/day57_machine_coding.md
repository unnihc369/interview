# Day 57 — Machine Coding

**React:** Stock trading k transactions  
**React Native:** Portfolio graph

---

## Table of Contents

### React (Web)

1. [Problem — Stock Simulator](#1-problem--stock-simulator)
2. [Multi-Transaction DP Engine](#2-multi-transaction-dp-engine)
3. [React Trading UI](#3-react-trading-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Portfolio Graph](#5-portfolio-graph)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Stock Simulator

**Task:** Given price history and max transactions k, show **max profit** and highlight buy/sell days.

---

## 2. Multi-Transaction DP Engine

```js
function maxProfitWithTrades(k, prices) {
  if (k === 0 || !prices.length) return { profit: 0, trades: [] };
  if (k >= prices.length / 2) {
    let profit = 0;
    const trades = [];
    for (let i = 1; i < prices.length; i++) {
      if (prices[i] > prices[i - 1]) {
        profit += prices[i] - prices[i - 1];
        trades.push({ buy: i - 1, sell: i });
      }
    }
    return { profit, trades };
  }

  let buy = Array(k + 1).fill(-Infinity);
  let sell = Array(k + 1).fill(0);
  for (const p of prices) {
    for (let j = k; j >= 1; j--) {
      sell[j] = Math.max(sell[j], buy[j] + p);
      buy[j] = Math.max(buy[j], sell[j - 1] - p);
    }
  }
  return { profit: sell[k], trades: [] };
}
```

---

## 3. React Trading UI

```tsx
import { useState, useMemo } from "react";

const PRICES = [7, 1, 5, 3, 6, 4];

export default function StockSimulator() {
  const [k, setK] = useState(2);
  const { profit, trades } = useMemo(() => maxProfitWithTrades(k, PRICES), [k]);

  return (
    <div style={{ padding: 24 }}>
      <h2>Max profit (k={k}): ${profit}</h2>
      <input type="range" min={1} max={3} value={k} onChange={(e) => setK(+e.target.value)} />
      <div style={{ display: "flex", alignItems: "flex-end", gap: 4, height: 120, marginTop: 16 }}>
        {PRICES.map((p, i) => (
          <div key={i} style={{ width: 32, height: p * 12, background: "#90caf9", position: "relative" }}>
            <span style={{ position: "absolute", bottom: -20, left: 8, fontSize: 10 }}>{i}</span>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 4. React Interview Points

- Rolling DP: explain buy/sell state machine verbally.
- Chart buy/sell markers with green/red overlays.
- k ≥ n/2 reduces to greedy unlimited case.

---

# Part B — React Native

## 5. Portfolio Graph

```tsx
import { View, Text, StyleSheet, Dimensions } from "react-native";
import { useMemo } from "react";

const PRICES = [100, 102, 98, 105, 110, 108, 115];
const W = Dimensions.get("window").width - 32;

function maxProfitOne(prices: number[]) {
  let min = Infinity, max = 0;
  for (const p of prices) {
    min = Math.min(min, p);
    max = Math.max(max, p - min);
  }
  return max;
}

export default function PortfolioGraph() {
  const profit = useMemo(() => maxProfitOne(PRICES), []);
  const maxP = Math.max(...PRICES);
  const minP = Math.min(...PRICES);
  const range = maxP - minP || 1;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Best single trade: +${profit}</Text>
      <View style={styles.chart}>
        {PRICES.map((p, i) => {
          const h = ((p - minP) / range) * 100;
          return (
            <View key={i} style={[styles.bar, { height: `${h}%`, flex: 1 }]} />
          );
        })}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  title: { fontSize: 18, marginBottom: 16 },
  chart: { flexDirection: "row", alignItems: "flex-end", height: 120, gap: 2 },
  bar: { backgroundColor: "#42a5f5", minHeight: 4 },
});
```

---

## 6. RN Interview Points

- Percentage heights for responsive bar chart.
- `Dimensions` for width — listen to orientation changes in production.
- Victory-native or react-native-svg-charts for polished graphs.

---

**Prev/Next:** [Concepts](day57_concepts.md) · [LeetCode](day57_leetcode.md)
