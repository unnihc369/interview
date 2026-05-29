# Day 23 — Machine Coding Revision

**React:** K most frequent words chart  
**React Native:** Emoji heap sort animation

---

## Table of Contents

### React (Web)

1. [Problem — Word Frequency Chart](#1-problem--word-frequency-chart)
2. [Top-K with Heap / Sort](#2-top-k-with-heap--sort)
3. [Bar Chart UI](#3-bar-chart-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Emoji Heap Sort](#5-emoji-heap-sort)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Word Frequency Chart

**Task:** Text area input → live **top K** (default 5) frequent words bar chart. Update debounced 300ms. Tie-break alphabetically (LC 692 style).

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Parse | Split on whitespace/punctuation, lowercase |
| Top K | Configurable K (3–10) |
| Chart | Horizontal bars proportional to count |
| Refresh | Debounced recompute |

---

## 2. Top-K with Heap / Sort

```js
function topKFrequentWords(words, k) {
  const freq = new Map();
  for (const w of words) {
    const clean = w.toLowerCase().replace(/[^a-z]/g, "");
    if (!clean) continue;
    freq.set(clean, (freq.get(clean) ?? 0) + 1);
  }

  return [...freq.entries()]
    .sort((a, b) => {
      if (b[1] !== a[1]) return b[1] - a[1];
      return a[0].localeCompare(b[0]);
    })
    .slice(0, k);
}
```

### Min-heap approach (LC 692)

Keep heap size k comparing by freq then lex order (invert for min-heap logic).

---

## 3. Bar Chart UI

```tsx
import { useState, useEffect, useMemo } from "react";

function useDebounced<T>(value: T, ms: number) {
  const [deb, setDeb] = useState(value);
  useEffect(() => {
    const t = setTimeout(() => setDeb(value), ms);
    return () => clearTimeout(t);
  }, [value, ms]);
  return deb;
}

export default function WordFrequencyChart() {
  const [text, setText] = useState("");
  const [k, setK] = useState(5);
  const debounced = useDebounced(text, 300);

  const top = useMemo(() => {
    const words = debounced.split(/\s+/);
    return topKFrequentWords(words, k);
  }, [debounced, k]);

  const maxCount = top[0]?.[1] ?? 1;

  return (
    <div style={{ maxWidth: 560, margin: 24 }}>
      <h2>Top {k} Words</h2>
      <textarea
        rows={5}
        value={text}
        onChange={(e) => setText(e.target.value)}
        style={{ width: "100%", marginBottom: 12 }}
      />
      <label>
        K:{" "}
        <input type="range" min={3} max={10} value={k} onChange={(e) => setK(+e.target.value)} />
        {k}
      </label>
      <div style={{ marginTop: 16 }}>
        {top.map(([word, count]) => (
          <div key={word} style={{ marginBottom: 8 }}>
            <div style={{ display: "flex", justifyContent: "space-between" }}>
              <span>{word}</span>
              <span>{count}</span>
            </div>
            <div style={{ background: "#eee", height: 8, borderRadius: 4 }}>
              <div
                style={{
                  width: `${(count / maxCount) * 100}%`,
                  background: "#1976d2",
                  height: "100%",
                  borderRadius: 4,
                  transition: "width 0.3s",
                }}
              />
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

function topKFrequentWords(words: string[], k: number) {
  const freq = new Map<string, number>();
  for (const w of words) {
    const clean = w.toLowerCase().replace(/[^a-z]/g, "");
    if (!clean) continue;
    freq.set(clean, (freq.get(clean) ?? 0) + 1);
  }
  return [...freq.entries()]
    .sort((a, b) => (b[1] !== a[1] ? b[1] - a[1] : a[0].localeCompare(b[0])))
    .slice(0, k);
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Debounce hook cleanup | clearTimeout in effect return |
| useMemo deps | debounced text + k |
| Chart library? | CSS bars sufficient; mention Recharts |

---

# Part B — React Native

## 5. Emoji Heap Sort

**Task:** Animate sorting emojis by "priority" using **heap extract** — repeatedly pop min and append to sorted row with `setInterval` steps.

```tsx
import { useState, useEffect, useRef } from "react";
import { View, Text, StyleSheet } from "react-native";

const EMOJIS = ["🔥", "⭐", "💎", "🎯", "🚀", "❤️"];
const PRIORITY = [3, 1, 5, 2, 4, 1]; // lower = higher priority

function buildMinHeap(indices: number[], prio: number[]) {
  // return heap indices sorted by priority — simplified sort for demo
  return [...indices].sort((a, b) => prio[a] - prio[b] || a - b);
}

export default function EmojiHeapSort() {
  const [heap, setHeap] = useState<number[]>(() => buildMinHeap([0,1,2,3,4,5], PRIORITY));
  const [sorted, setSorted] = useState<number[]>([]);
  const [running, setRunning] = useState(false);
  const intervalRef = useRef<ReturnType<typeof setInterval>>();

  useEffect(() => {
    if (!running || heap.length === 0) return;
    intervalRef.current = setInterval(() => {
      setHeap((h) => {
        if (h.length === 0) {
          clearInterval(intervalRef.current);
          setRunning(false);
          return h;
        }
        const [min, ...rest] = h;
        setSorted((s) => [...s, min]);
        return rest;
      });
    }, 600);
    return () => clearInterval(intervalRef.current);
  }, [running, heap.length]);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Heap Extract Sort</Text>
      <Text>Heap: {heap.map((i) => EMOJIS[i]).join(" ")}</Text>
      <Text>Sorted: {sorted.map((i) => EMOJIS[i]).join(" ")}</Text>
      <Text onPress={() => { setHeap(buildMinHeap([0,1,2,3,4,5], PRIORITY)); setSorted([]); setRunning(true); }} style={styles.btn}>
        Start
      </Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 24 },
  title: { fontSize: 18, fontWeight: "700", marginBottom: 16 },
  btn: { marginTop: 16, color: "#1976d2", fontSize: 16 },
});
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| setInterval in RN | Same as web; cleanup on unmount |
| Animation per step | Reanimated Layout for slide |
| Real heap pop | O(log n) sift down each step |

---

## Quick Revision — Day 23

```
Web:  debounce text → Map freq → sort top K → bar chart
RN:   setInterval heap extract → move emoji to sorted row
LC:   692 word freq; 973 closest points; 787 cheapest flights
```

---

*End of Day 23 machine coding*
