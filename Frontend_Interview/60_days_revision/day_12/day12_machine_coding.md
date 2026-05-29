# Day 12 — Machine Coding Revision

**React:** LRU cache visualizer with sliding window of access history  
**React Native:** Image carousel with sliding preload window

---

## Table of Contents

### React (Web)

1. [Problem — LRU Visualizer](#1-problem--lru-visualizer)
2. [LRUCache Class](#2-lrucache-class)
3. [LRUVisualizer Component](#3-lruvisualizer-component)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Image Carousel Preload Window](#5-image-carousel-preload-window)
6. [Carousel Implementation](#6-carousel-implementation)
7. [RN Interview Points](#7-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — LRU Visualizer

Build UI for LRU cache:

| Feature | Behavior |
|---------|----------|
| Capacity | Configurable (e.g. 4) |
| get(key) | Highlight access; move to MRU |
| put(key, val) | Insert/update; evict LRU if full |
| Access window | Show last N operations as sliding log |
| Visual order | MRU left → LRU right |

---

## 2. LRUCache Class

```ts
export class LRUCache<K, V> {
  private capacity: number;
  private map = new Map<K, V>();

  constructor(capacity: number) {
    this.capacity = capacity;
  }

  get(key: K): V | undefined {
    if (!this.map.has(key)) return undefined;
    const val = this.map.get(key)!;
    this.map.delete(key);
    this.map.set(key, val); // refresh MRU
    return val;
  }

  put(key: K, value: V): K | null {
    let evicted: K | null = null;
    if (this.map.has(key)) this.map.delete(key);
    else if (this.map.size >= this.capacity) {
      evicted = this.map.keys().next().value ?? null;
      this.map.delete(evicted!);
    }
    this.map.set(key, value);
    return evicted;
  }

  keysMRUtoLRU(): K[] {
    return [...this.map.keys()].reverse();
  }

  size() {
    return this.map.size;
  }
}
```

---

## 3. LRUVisualizer Component

```tsx
import { useCallback, useMemo, useRef, useState } from "react";
import { LRUCache } from "./LRUCache";

type LogEntry = { op: "get" | "put"; key: string; evicted?: string };

const CAP = 4;
const LOG_WINDOW = 8; // sliding window of last 8 ops

export default function LRUVisualizer() {
  const cacheRef = useRef(new LRUCache<string, number>(CAP));
  const [version, setVersion] = useState(0);
  const [keyInput, setKeyInput] = useState("a");
  const [valInput, setValInput] = useState("1");
  const [log, setLog] = useState<LogEntry[]>([]);

  const order = useMemo(
    () => cacheRef.current.keysMRUtoLRU(),
    [version]
  );

  const pushLog = useCallback((entry: LogEntry) => {
    setLog((prev) => [...prev, entry].slice(-LOG_WINDOW));
  }, []);

  const handleGet = () => {
    const v = cacheRef.current.get(keyInput);
    pushLog({ op: "get", key: keyInput });
    setVersion((n) => n + 1);
    alert(v === undefined ? "miss" : `hit: ${v}`);
  };

  const handlePut = () => {
    const evicted = cacheRef.current.put(keyInput, Number(valInput));
    pushLog({ op: "put", key: keyInput, evicted: evicted ?? undefined });
    setVersion((n) => n + 1);
  };

  return (
    <div style={{ padding: 16, fontFamily: "sans-serif" }}>
      <h3>LRU Cache (cap={CAP})</h3>

      <div style={{ display: "flex", gap: 8, marginBottom: 16 }}>
        <input value={keyInput} onChange={(e) => setKeyInput(e.target.value)} placeholder="key" />
        <input value={valInput} onChange={(e) => setValInput(e.target.value)} placeholder="value" />
        <button onClick={handleGet}>get</button>
        <button onClick={handlePut}>put</button>
      </div>

      <div style={{ display: "flex", gap: 8, marginBottom: 16 }}>
        {order.map((k) => (
          <div
            key={k}
            style={{
              padding: "12px 16px",
              background: "#4a90d9",
              color: "#fff",
              borderRadius: 8,
            }}
          >
            {k}: {cacheRef.current.get(k)}
          </div>
        ))}
        {order.length === 0 && <span style={{ color: "#999" }}>empty</span>}
      </div>
      <p style={{ fontSize: 12, color: "#666" }}>MRU ← left · right → LRU</p>

      <h4>Access log (window {LOG_WINDOW})</h4>
      <ul style={{ fontFamily: "monospace", fontSize: 13 }}>
        {log.map((e, i) => (
          <li key={i}>
            {e.op}({e.key}){e.evicted ? ` → evicted ${e.evicted}` : ""}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| LRU O(1) | Map + doubly linked list (or Map order trick) |
| Why ref for cache? | Mutable structure; avoid re-render on internal ops |
| Sliding log window | `slice(-N)` keeps last N entries |

---

# Part B — React Native

## 5. Image Carousel Preload Window

Preload images in **sliding window**: current index ± `PRELOAD_RADIUS`.

| Index | Load? |
|-------|-------|
| active | Full res |
| active ± 1 | Prefetch |
| else | Placeholder / low-res |

---

## 6. Carousel Implementation

```tsx
import { useCallback, useEffect, useRef, useState } from "react";
import {
  Dimensions,
  FlatList,
  Image,
  NativeScrollEvent,
  NativeSyntheticEvent,
  View,
  ActivityIndicator,
} from "react-native";

const { width: W } = Dimensions.get("window");
const PRELOAD_RADIUS = 2;

const IMAGES = Array.from({ length: 20 }, (_, i) => ({
  id: String(i),
  uri: `https://picsum.photos/seed/${i}/800/600`,
}));

export default function ImageCarousel() {
  const [active, setActive] = useState(0);
  const prefetched = useRef(new Set<string>());

  const shouldLoad = (index: number) =>
    Math.abs(index - active) <= PRELOAD_RADIUS;

  useEffect(() => {
    IMAGES.forEach((img, i) => {
      if (shouldLoad(i) && !prefetched.current.has(img.uri)) {
        Image.prefetch(img.uri).then(() => prefetched.current.add(img.uri));
      }
    });
  }, [active]);

  const onScrollEnd = useCallback(
    (e: NativeSyntheticEvent<NativeScrollEvent>) => {
      setActive(Math.round(e.nativeEvent.contentOffset.x / W));
    },
    []
  );

  return (
    <FlatList
      data={IMAGES}
      horizontal
      pagingEnabled
      showsHorizontalScrollIndicator={false}
      keyExtractor={(item) => item.id}
      onMomentumScrollEnd={onScrollEnd}
      getItemLayout={(_, i) => ({ length: W, offset: W * i, index: i })}
      renderItem={({ item, index }) => (
        <View style={{ width: W, height: 300, backgroundColor: "#111" }}>
          {shouldLoad(index) ? (
            <Image
              source={{ uri: item.uri }}
              style={{ width: W, height: 300 }}
              resizeMode="cover"
            />
          ) : (
            <ActivityIndicator style={{ marginTop: 140 }} color="#fff" />
          )}
        </View>
      )}
    />
  );
}
```

---

## 7. RN Interview Points

| Question | Answer |
|----------|--------|
| `Image.prefetch` | Cache image before display |
| Preload radius | Trade memory vs swipe smoothness |
| FastImage | Third-party — disk + memory cache |

---

## Quick Revision — Day 12

```
LRU:     Map order = MRU at end; evict first key
Log:     slice(-N) sliding window of ops
RN:      prefetch active ± radius on index change
```

---

*End of Day 12 machine coding*
