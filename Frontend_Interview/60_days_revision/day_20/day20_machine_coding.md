# Day 20 — Machine Coding Revision

**React:** Merge sorted arrays visualizer  
**React Native:** Merge databases

---

## Table of Contents

### React (Web)

1. [Problem — Merge Visualizer](#1-problem--merge-visualizer)
2. [Two-Pointer Merge Algorithm](#2-two-pointer-merge-algorithm)
3. [Animated React UI](#3-animated-react-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Merge Databases (Local Sync)](#5-merge-databases-local-sync)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Merge Visualizer

**Task:** Visualize merging two **sorted** arrays step-by-step — highlight active pointers `i`, `j`, and write position `k`. Play / pause / step controls.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Input | Two sorted arrays (editable) |
| Output | Merged result building live |
| Pointers | Highlight `i`, `j` on source arrays |
| Controls | Step, Auto-play 500ms, Reset |

---

## 2. Two-Pointer Merge Algorithm

```js
function* mergeSteps(a, b) {
  const out = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length) {
    yield { a, b, i, j, out: [...out], phase: "compare" };
    if (a[i] <= b[j]) out.push(a[i++]);
    else out.push(b[j++]);
    yield { a, b, i, j, out: [...out], phase: "write" };
  }
  while (i < a.length) {
    out.push(a[i++]);
    yield { a, b, i, j, out: [...out], phase: "drain-a" };
  }
  while (j < b.length) {
    out.push(b[j++]);
    yield { a, b, i, j, out: [...out], phase: "drain-b" };
  }
  yield { a, b, i, j, out, phase: "done" };
}
```

---

## 3. Animated React UI

```tsx
import { useState, useMemo, useEffect } from "react";

export default function MergeVisualizer() {
  const [a] = useState([1, 4, 7, 10]);
  const [b] = useState([2, 3, 8]);
  const steps = useMemo(() => [...mergeSteps(a, b)], [a, b]);
  const [stepIdx, setStepIdx] = useState(0);
  const [playing, setPlaying] = useState(false);

  const frame = steps[stepIdx] ?? steps[steps.length - 1];

  useEffect(() => {
    if (!playing) return;
    const t = setInterval(() => {
      setStepIdx((i) => (i >= steps.length - 1 ? (setPlaying(false), i) : i + 1));
    }, 600);
    return () => clearInterval(t);
  }, [playing, steps.length]);

  return (
    <div style={{ padding: 24, fontFamily: "monospace" }}>
      <h2>Merge Sorted Arrays</h2>
      <div>
        A: {a.map((v, idx) => (
          <span key={idx} style={cellStyle(idx === frame.i)}>{v} </span>
        ))}
      </div>
      <div>
        B: {b.map((v, idx) => (
          <span key={idx} style={cellStyle(idx === frame.j)}>{v} </span>
        ))}
      </div>
      <div style={{ marginTop: 16 }}>
        Out: {frame.out.join(", ")}
      </div>
      <div style={{ marginTop: 16, display: "flex", gap: 8 }}>
        <button onClick={() => setStepIdx((i) => Math.max(0, i - 1))}>Prev</button>
        <button onClick={() => setStepIdx((i) => Math.min(steps.length - 1, i + 1))}>Next</button>
        <button onClick={() => setPlaying(true)}>Play</button>
        <button onClick={() => { setStepIdx(0); setPlaying(false); }}>Reset</button>
      </div>
    </div>
  );
}

function cellStyle(active: boolean) {
  return {
    padding: "4px 8px",
    background: active ? "#ffeb3b" : "#eee",
    marginRight: 4,
    borderRadius: 4,
  };
}

function* mergeSteps(a: number[], b: number[]) {
  const out: number[] = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length) {
    yield { a, b, i, j, out: [...out], phase: "compare" };
    if (a[i] <= b[j]) out.push(a[i++]);
    else out.push(b[j++]);
  }
  while (i < a.length) out.push(a[i++]);
  while (j < b.length) out.push(b[j++]);
  yield { a, b, i, j, out, phase: "done" };
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Generator for steps? | Clean pause/resume; precompute all steps for simple UI |
| Interval cleanup | `clearInterval` in effect return |
| TypedArray output? | `Int32Array` same merge logic |

---

# Part B — React Native

## 5. Merge Databases (Local Sync)

**Task:** Merge two **sorted** local record lists (by `updatedAt`) into one deduplicated list — last-write-wins on same `id`.

```tsx
import { useMemo } from "react";
import { FlatList, Text, View } from "react-native";

type Record = { id: string; title: string; updatedAt: number };

function mergeSortedById(local: Record[], remote: Record[]): Record[] {
  const byId = new Map<string, Record>();
  let i = 0, j = 0;

  while (i < local.length && j < remote.length) {
    const a = local[i], b = remote[j];
    if (a.id === b.id) {
      byId.set(a.id, a.updatedAt >= b.updatedAt ? a : b);
      i++; j++;
    } else if (a.updatedAt <= b.updatedAt) {
      byId.set(a.id, a);
      i++;
    } else {
      byId.set(b.id, b);
      j++;
    }
  }
  while (i < local.length) byId.set(local[i].id, local[i++]);
  while (j < remote.length) byId.set(remote[j].id, remote[j++]);

  return [...byId.values()].sort((x, y) => x.updatedAt - y.updatedAt);
}

export default function MergedDatabaseScreen() {
  const local: Record[] = [
    { id: "1", title: "A local", updatedAt: 100 },
    { id: "2", title: "B local", updatedAt: 200 },
  ];
  const remote: Record[] = [
    { id: "1", title: "A remote", updatedAt: 150 },
    { id: "3", title: "C remote", updatedAt: 250 },
  ];

  const merged = useMemo(() => mergeSortedById(local, remote), []);

  return (
    <FlatList
      data={merged}
      keyExtractor={(r) => r.id}
      renderItem={({ item }) => (
        <View style={{ padding: 12, borderBottomWidth: 1, borderColor: "#eee" }}>
          <Text>{item.title}</Text>
          <Text style={{ fontSize: 12, color: "#888" }}>
            id={item.id} · updated={item.updatedAt}
          </Text>
        </View>
      )}
    />
  );
}
```

### SQLite / WatermelonDB mention

Production apps merge in SQL `UNION` + `GROUP BY id MAX(updatedAt)` or CRDT sync.

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Conflict resolution | LWW (last-write-wins) vs merge function per field |
| Offline queue | Merge on reconnect; show sync badge |
| Sort stability | Tie-break by id after updatedAt |

---

## Quick Revision — Day 20

```
Web:  generator mergeSteps → step/play UI → pointer highlights
RN:   two sorted DBs → Map by id LWW → sorted FlatList
Both: classic merge i/j pointers; LC 88 / 986
```

---

*End of Day 20 machine coding*
