# Day 17 — Machine Coding Revision

**React:** Triplet sum game  
**React Native:** Multi-select gesture list

---

## Table of Contents

### React (Web)

1. [Problem — Triplet Sum Game](#1-problem--triplet-sum-game)
2. [Game Logic & Iterator](#2-game-logic--iterator)
3. [React UI Implementation](#3-react-ui-implementation)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Multi-Select Gesture List](#5-multi-select-gesture-list)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Triplet Sum Game

**Task:** Build a mini-game where players pick **three numbers** from a grid; if they sum to **0**, score a point. Show found triplets, prevent reuse of same triplet, timer optional.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Number grid | Click to select up to 3 cells |
| Validation | Sum === 0 → success animation + score++ |
| Dedup | Same sorted triplet cannot score twice |
| Hint | "Show hint" reveals one valid triplet (optional) |
| Reset | New random board |

**Algorithm tie-in:** Precompute valid triplets via sorted array + two pointers (LC 15 pattern).

---

## 2. Game Logic & Iterator

### Find all unique zero-sum triplets

```js
function findZeroTriplets(nums) {
  const sorted = [...nums].sort((a, b) => a - b);
  const res = [];
  const n = sorted.length;

  for (let i = 0; i < n - 2; i++) {
    if (i > 0 && sorted[i] === sorted[i - 1]) continue;
    let l = i + 1, r = n - 1;
    while (l < r) {
      const sum = sorted[i] + sorted[l] + sorted[r];
      if (sum === 0) {
        res.push([sorted[i], sorted[l], sorted[r]]);
        while (l < r && sorted[l] === sorted[l + 1]) l++;
        while (l < r && sorted[r] === sorted[r - 1]) r--;
        l++; r--;
      } else if (sum < 0) l++;
      else r--;
    }
  }
  return res;
}

function tripletKey(t) {
  return t.slice().sort((a, b) => a - b).join(",");
}
```

### Iterable for hints

```js
function* tripletHints(nums) {
  const triplets = findZeroTriplets(nums);
  for (const t of triplets) yield t;
}
```

---

## 3. React UI Implementation

```tsx
import { useMemo, useState } from "react";

const BOARD = [-2, -1, 0, 1, 2, 3, -3, 4];

function tripletKey(a: number, b: number, c: number) {
  return [a, b, c].sort((x, y) => x - y).join(",");
}

export default function TripletSumGame() {
  const [selected, setSelected] = useState<number[]>([]);
  const [score, setScore] = useState(0);
  const [found, setFound] = useState<Set<string>>(new Set());

  const allTriplets = useMemo(() => findZeroTriplets(BOARD), []);

  const toggleCell = (val: number, index: number) => {
    setSelected((prev) => {
      if (prev.includes(index)) return prev.filter((i) => i !== index);
      if (prev.length >= 3) return prev;
      return [...prev, index];
    });
  };

  const checkSelection = () => {
    if (selected.length !== 3) return;
    const values = selected.map((i) => BOARD[i]);
    const sum = values.reduce((a, b) => a + b, 0);
    const key = tripletKey(...values);

    if (sum === 0 && !found.has(key)) {
      setFound((f) => new Set(f).add(key));
      setScore((s) => s + 1);
    }
    setSelected([]);
  };

  const hint = () => {
    const next = allTriplets.find(
      (t) => !found.has(tripletKey(...t))
    );
    if (next) alert(`Hint: try ${next.join(", ")}`);
  };

  return (
    <div style={{ maxWidth: 420, margin: "24px auto" }}>
      <h2>Triplet Sum Game</h2>
      <p>Score: {score} / {allTriplets.length}</p>
      <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 8 }}>
        {BOARD.map((val, i) => (
          <button
            key={i}
            onClick={() => toggleCell(val, i)}
            style={{
              padding: 16,
              background: selected.includes(i) ? "#1976d2" : "#eee",
              color: selected.includes(i) ? "#fff" : "#000",
              border: "none",
              borderRadius: 8,
              cursor: "pointer",
            }}
          >
            {val}
          </button>
        ))}
      </div>
      <div style={{ marginTop: 16, display: "flex", gap: 8 }}>
        <button onClick={checkSelection} disabled={selected.length !== 3}>
          Check
        </button>
        <button onClick={hint}>Hint</button>
      </div>
      <ul>
        {[...found].map((k) => (
          <li key={k}>Found: {k}</li>
        ))}
      </ul>
    </div>
  );
}

function findZeroTriplets(nums: number[]) {
  const sorted = [...nums].sort((a, b) => a - b);
  const res: number[][] = [];
  for (let i = 0; i < sorted.length - 2; i++) {
    if (i > 0 && sorted[i] === sorted[i - 1]) continue;
    let l = i + 1, r = sorted.length - 1;
    while (l < r) {
      const sum = sorted[i] + sorted[l] + sorted[r];
      if (sum === 0) {
        res.push([sorted[i], sorted[l], sorted[r]]);
        while (l < r && sorted[l] === sorted[l + 1]) l++;
        while (l < r && sorted[r] === sorted[r - 1]) r--;
        l++; r--;
      } else if (sum < 0) l++;
      else r--;
    }
  }
  return res;
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Precompute triplets vs check on fly? | Precompute for hints/count; O(n²) once |
| Store selected indices or values? | Indices — duplicate values need disambiguation |
| Set vs array for found | Set for O(1) lookup by string key |

---

# Part B — React Native

## 5. Multi-Select Gesture List

**Task:** List where user **long-press** enters selection mode, then **tap** toggles items; **pan** down with finger selects range (Gmail-style).

**Requirements:**

| Gesture | Action |
|---------|--------|
| Long press | Enter multi-select mode |
| Tap | Toggle single item selection |
| Pan (vertical) | Select all rows under finger path |
| Toolbar | Show count + Delete / Share |

### Using Pressable + selection state

```tsx
import { useState, useCallback } from "react";
import {
  FlatList,
  Pressable,
  Text,
  View,
  StyleSheet,
} from "react-native";

type Item = { id: string; title: string };

const DATA: Item[] = Array.from({ length: 20 }, (_, i) => ({
  id: String(i),
  title: `Message ${i + 1}`,
}));

export default function MultiSelectList() {
  const [selectMode, setSelectMode] = useState(false);
  const [selected, setSelected] = useState<Set<string>>(new Set());

  const toggle = useCallback((id: string) => {
    setSelected((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id);
      else next.add(id);
      return next;
    });
  }, []);

  const onLongPress = (id: string) => {
    setSelectMode(true);
    toggle(id);
  };

  const clearSelection = () => {
    setSelected(new Set());
    setSelectMode(false);
  };

  return (
    <View style={{ flex: 1 }}>
      {selectMode && (
        <View style={styles.toolbar}>
          <Text>{selected.size} selected</Text>
          <Text onPress={clearSelection} style={styles.cancel}>
            Cancel
          </Text>
        </View>
      )}
      <FlatList
        data={DATA}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => {
          const isSelected = selected.has(item.id);
          return (
            <Pressable
              onPress={() => selectMode && toggle(item.id)}
              onLongPress={() => onLongPress(item.id)}
              style={[styles.row, isSelected && styles.rowSelected]}
            >
              <Text>{item.title}</Text>
              {isSelected && <Text>✓</Text>}
            </Pressable>
          );
        }}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  toolbar: {
    flexDirection: "row",
    justifyContent: "space-between",
    padding: 12,
    backgroundColor: "#1976d2",
  },
  cancel: { color: "#fff" },
  row: {
    padding: 16,
    borderBottomWidth: 1,
    borderColor: "#eee",
    flexDirection: "row",
    justifyContent: "space-between",
  },
  rowSelected: { backgroundColor: "#e3f2fd" },
});
```

### Pan range selection (sketch)

Use `react-native-gesture-handler` `Gesture.Pan()` — on `onUpdate`, map `y` to row index via `FlatList` `getItemLayout` and add all indices from `startIndex` to `currentIndex` to `selected` Set.

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Long press delay | `delayLongPress={300}` on Pressable |
| Performance with Set | Immutable new Set on toggle — fine for <500 items |
| Select all | `setSelected(new Set(DATA.map(d => d.id)))` |
| Exit mode on back | `BackHandler` Android listener |

---

## Quick Revision — Day 17

```
Web:  3Sum precompute → grid select → tripletKey dedup → score
RN:   longPress → selectMode → Set<id> toggle → toolbar
Both: iterator/hints; two-pointer triplet generation
```

---

*End of Day 17 machine coding*
