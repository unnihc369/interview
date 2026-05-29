# Day 54 — Machine Coding

**React:** Card game longest increasing  
**React Native:** Leaderboard streak

---

## Table of Contents

### React (Web)

1. [Problem — Card Streak Game](#1-problem--card-streak-game)
2. [LIS Game Logic](#2-lis-game-logic)
3. [React Card UI](#3-react-card-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Leaderboard Streak](#5-leaderboard-streak)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Card Streak Game

**Task:** Player draws cards with values. Build **longest increasing subsequence** from played cards. Show current streak and max possible LIS from hand.

---

## 2. LIS Game Logic

```js
function lisLength(arr) {
  const dp = arr.map(() => 1);
  for (let i = 1; i < arr.length; i++) {
    for (let j = 0; j < i; j++) {
      if (arr[j] < arr[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
    }
  }
  return Math.max(...dp, 0);
}

function canPlay(hand, played, cardIdx) {
  const card = hand[cardIdx];
  if (played.length === 0) return true;
  return card > played[played.length - 1];
}
```

---

## 3. React Card UI

```tsx
import { useState } from "react";

const DECK = [3, 7, 2, 9, 5, 11, 4, 8];

export default function CardStreakGame() {
  const [hand] = useState(DECK);
  const [played, setPlayed] = useState<number[]>([]);
  const [remaining, setRemaining] = useState<number[]>(DECK.map((_, i) => i));

  const play = (idx: number) => {
    const val = hand[idx];
    if (played.length && val <= played[played.length - 1]) return;
    setPlayed((p) => [...p, val]);
    setRemaining((r) => r.filter((i) => i !== idx));
  };

  const maxStreak = lisLength(played);
  const optimal = lisLength(hand);

  return (
    <div style={{ padding: 24 }}>
      <h2>Longest Increasing Streak</h2>
      <p>Current streak length: {maxStreak} · Deck LIS potential: {optimal}</p>
      <div style={{ display: "flex", gap: 8, marginBottom: 16 }}>
        {remaining.map((idx) => (
          <button key={idx} onClick={() => play(idx)} style={{ width: 48, height: 64 }}>
            {hand[idx]}
          </button>
        ))}
      </div>
      <p>Played: {played.join(" → ") || "(none)"}</p>
    </div>
  );
}
```

---

## 4. React Interview Points

- LIS on played sequence validates increasing rule in O(n²) per check — fine for small n.
- Show optimal LIS from full deck as hint/education.
- Animate card flip with CSS transitions.

---

# Part B — React Native

## 5. Leaderboard Streak

```tsx
import { View, Text, FlatList, StyleSheet } from "react-native";
import { useMemo } from "react";

type Player = { id: string; name: string; scores: number[] };

function longestStreak(scores: number[]): number {
  if (!scores.length) return 0;
  const dp = scores.map(() => 1);
  for (let i = 1; i < scores.length; i++) {
    for (let j = 0; j < i; j++) {
      if (scores[j] < scores[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
    }
  }
  return Math.max(...dp);
}

export default function LeaderboardStreak() {
  const players: Player[] = [
    { id: "1", name: "Alice", scores: [10, 20, 15, 25, 30] },
    { id: "2", name: "Bob", scores: [5, 12, 18, 22, 40] },
  ];

  const ranked = useMemo(
    () =>
      [...players]
        .map((p) => ({ ...p, streak: longestStreak(p.scores) }))
        .sort((a, b) => b.streak - a.streak),
    []
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Best Increasing Streak</Text>
      <FlatList
        data={ranked}
        keyExtractor={(p) => p.id}
        renderItem={({ item, index }) => (
          <View style={styles.row}>
            <Text>#{index + 1} {item.name}</Text>
            <Text style={styles.streak}>{item.streak} days</Text>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  title: { fontSize: 20, fontWeight: "600", marginBottom: 12 },
  row: { flexDirection: "row", justifyContent: "space-between", paddingVertical: 12, borderBottomWidth: 1, borderColor: "#eee" },
  streak: { fontWeight: "700", color: "#1565c0" },
});
```

---

## 6. RN Interview Points

- LIS models "strictly improving" streak — clarify vs consecutive-day streak (different DP).
- `useMemo` for ranking when scores update infrequently.
- Pull-to-refresh to reload scores.

---

**Prev/Next:** [Concepts](day54_concepts.md) · [LeetCode](day54_leetcode.md)
