# Day 25 — Machine Coding Revision

**React:** Merge K sorted visualizer (Worker)  
**React Native:** Merge image sources

---

## Table of Contents

### React (Web)

1. [Problem — Merge K Sorted Visualizer](#1-problem--merge-k-sorted-visualizer)
2. [Worker-Based Merge](#2-worker-based-merge)
3. [React UI with Progress](#3-react-ui-with-progress)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Merge Image Sources](#5-merge-image-sources)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Merge K Sorted Visualizer

**Task:** User provides K sorted arrays. Merge off main thread via **Web Worker**. Show progress + final merged result without UI freeze.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Input | K text fields (comma-separated sorted nums) |
| Merge | Min-heap in worker |
| Progress | postMessage steps optional |
| UI | Loading spinner; non-blocking |

---

## 2. Worker-Based Merge

```js
// mergeWorker.js
function mergeKSorted(lists) {
  const pointers = lists.map(() => 0);
  const heap = [];

  for (let i = 0; i < lists.length; i++) {
    if (lists[i].length) heap.push({ val: lists[i][0], listIdx: i });
  }
  heap.sort((a, b) => a.val - b.val);

  const result = [];
  while (heap.length) {
    heap.sort((a, b) => a.val - b.val);
    const { val, listIdx } = heap.shift();
    result.push(val);
    pointers[listIdx]++;
    const next = lists[listIdx][pointers[listIdx]];
    if (next !== undefined) heap.push({ val: next, listIdx });
    if (result.length % 10 === 0) {
      self.postMessage({ type: "progress", count: result.length });
    }
  }
  return result;
}

self.onmessage = ({ data: lists }) => {
  self.postMessage({ type: "done", merged: mergeKSorted(lists) });
};
```

---

## 3. React UI with Progress

```tsx
import { useState, useRef, useCallback } from "react";

export default function MergeKVisualizer() {
  const [inputs, setInputs] = useState(["1,4,7", "2,5,8", "3,6,9"]);
  const [merged, setMerged] = useState<number[]>([]);
  const [loading, setLoading] = useState(false);
  const [progress, setProgress] = useState(0);
  const workerRef = useRef<Worker | null>(null);

  const runMerge = useCallback(() => {
    setLoading(true);
    setProgress(0);
    const lists = inputs.map((s) =>
      s.split(",").map((x) => parseInt(x.trim(), 10)).filter((n) => !isNaN(n))
    );

    workerRef.current?.terminate();
    const worker = new Worker(
      new URL("./mergeWorker.js", import.meta.url),
      { type: "module" }
    );
    workerRef.current = worker;

    worker.onmessage = (e) => {
      if (e.data.type === "progress") setProgress(e.data.count);
      if (e.data.type === "done") {
        setMerged(e.data.merged);
        setLoading(false);
        worker.terminate();
      }
    };
    worker.postMessage(lists);
  }, [inputs]);

  return (
    <div style={{ padding: 24 }}>
      <h2>Merge K Sorted (Worker)</h2>
      {inputs.map((val, i) => (
        <input
          key={i}
          value={val}
          onChange={(e) => {
            const next = [...inputs];
            next[i] = e.target.value;
            setInputs(next);
          }}
          style={{ display: "block", width: "100%", marginBottom: 8 }}
        />
      ))}
      <button onClick={runMerge} disabled={loading}>
        {loading ? `Merging… (${progress})` : "Merge"}
      </button>
      <pre style={{ marginTop: 16 }}>{merged.join(", ")}</pre>
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Worker cleanup | terminate on unmount + after done |
| Comlink library | RPC wrapper over postMessage |
| Fallback | Main thread merge with Web Worker feature detect |

---

# Part B — React Native

## 5. Merge Image Sources

**Task:** Combine multiple image URIs into one collage grid (2×2). Use `Image` + `View` layout; heavy compositing via `expo-image-manipulator` or Skia off JS thread.

```tsx
import { View, Image, StyleSheet, Dimensions } from "react-native";

const URIS = [
  "https://picsum.photos/200/200?1",
  "https://picsum.photos/200/200?2",
  "https://picsum.photos/200/200?3",
  "https://picsum.photos/200/200?4",
];

export default function MergeImageGrid() {
  const size = Dimensions.get("window").width / 2 - 24;

  return (
    <View style={styles.grid}>
      {URIS.map((uri, i) => (
        <Image key={i} source={{ uri }} style={{ width: size, height: size, margin: 4 }} />
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  grid: { flexDirection: "row", flexWrap: "wrap", padding: 16, justifyContent: "center" },
});
```

### expo-image-manipulator merge (sketch)

```js
import * as ImageManipulator from "expo-image-manipulator";
// Composite layers — run in background task / InteractionManager.runAfterInteractions
```

RN has no Web Workers in classic sense — use **react-native-multithreading** or native modules.

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| RN parallel JS? | Hermes + JSI workers (experimental) |
| Image merge perf | Native Skia / GPU |
| Prefetch | `Image.prefetch(uri)` before merge |

---

## Quick Revision — Day 25

```
Web:  Worker merge K sorted → progress messages → terminate
RN:   Image grid collage; native for heavy merge
LC:   378 kth matrix; 264 ugly II; 373 k pairs
```

---

*End of Day 25 machine coding*
