# Day 26 — Machine Coding Revision

**React:** Top K expensive dashboard  
**React Native:** Notification priority queue

---

## Table of Contents

### React (Web)

1. [Problem — Top K Expensive Dashboard](#1-problem--top-k-expensive-dashboard)
2. [Max-Heap Top K by Price](#2-max-heap-top-k-by-price)
3. [Dashboard UI](#3-dashboard-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Notification Priority Queue](#5-notification-priority-queue)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Top K Expensive Dashboard

**Task:** Product list with prices. Show **top K** most expensive in real-time as prices update (simulate ticker). Use min-heap size K.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Products | id, name, price |
| Top K slider | K = 3..10 |
| Live update | Random price tick every 2s |
| Display | Ranked cards with price bars |

---

## 2. Max-Heap Top K by Price

Keep **min-heap of size K** on price — root is lowest among top K (threshold).

```js
function topKExpensive(products, k) {
  const sorted = [...products].sort((a, b) => b.price - a.price);
  return sorted.slice(0, k);
}

// Heap O(n log k):
function topKHeap(products, k) {
  const heap = [];
  for (const p of products) {
    heap.push(p);
    heap.sort((a, b) => a.price - b.price);
    if (heap.length > k) heap.shift();
  }
  return heap.sort((a, b) => b.price - a.price);
}
```

---

## 3. Dashboard UI

```tsx
import { useEffect, useState, useMemo } from "react";

type Product = { id: string; name: string; price: number };

const INITIAL: Product[] = [
  { id: "1", name: "Laptop", price: 1200 },
  { id: "2", name: "Phone", price: 899 },
  { id: "3", name: "Watch", price: 399 },
  { id: "4", name: "Tablet", price: 649 },
  { id: "5", name: "Headphones", price: 249 },
];

export default function TopKExpensiveDashboard() {
  const [products, setProducts] = useState(INITIAL);
  const [k, setK] = useState(3);

  useEffect(() => {
    const id = setInterval(() => {
      setProducts((prev) =>
        prev.map((p) => ({
          ...p,
          price: Math.max(50, p.price + (Math.random() - 0.5) * 100),
        }))
      );
    }, 2000);
    return () => clearInterval(id);
  }, []);

  const topK = useMemo(
    () => [...products].sort((a, b) => b.price - a.price).slice(0, k),
    [products, k]
  );

  const maxPrice = topK[0]?.price ?? 1;

  return (
    <div style={{ padding: 24, maxWidth: 480 }}>
      <h2>Top {k} Expensive</h2>
      <input type="range" min={2} max={5} value={k} onChange={(e) => setK(+e.target.value)} />
      {topK.map((p, i) => (
        <div key={p.id} style={{ marginBottom: 12 }}>
          <div style={{ display: "flex", justifyContent: "space-between" }}>
            <span>#{i + 1} {p.name}</span>
            <strong>${p.price.toFixed(0)}</strong>
          </div>
          <div style={{ background: "#eee", height: 8, borderRadius: 4 }}>
            <div
              style={{
                width: `${(p.price / maxPrice) * 100}%`,
                background: "#e65100",
                height: "100%",
                borderRadius: 4,
              }}
            />
          </div>
        </div>
      ))}
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Re-sort vs heap each tick | n small → sort OK; n large → heap |
| SharedArrayBuffer | Worker updates prices; main reads for chart |
| Stale closure in interval | Functional setState |

---

# Part B — React Native

## 5. Notification Priority Queue

**Task:** Push notifications queue — **critical** (P1) shown before **normal** (P3). Min-heap by priority; display banner for head.

```tsx
import { useState, useCallback } from "react";
import { View, Text, Pressable, StyleSheet } from "react-native";

type Notif = { id: string; title: string; priority: number };

function pushHeap(heap: Notif[], item: Notif) {
  const next = [...heap, item].sort((a, b) => a.priority - b.priority);
  return next;
}

function popHeap(heap: Notif[]) {
  return heap.slice(1);
}

export default function NotificationPriorityQueue() {
  const [queue, setQueue] = useState<Notif[]>([]);

  const enqueue = (title: string, priority: number) => {
    const n: Notif = { id: String(Date.now()), title, priority };
    setQueue((q) => pushHeap(q, n));
  };

  const dismiss = () => setQueue((q) => popHeap(q));

  const head = queue[0];

  return (
    <View style={styles.container}>
      {head && (
        <View style={[styles.banner, head.priority === 1 && styles.critical]}>
          <Text style={styles.title}>{head.title}</Text>
          <Text style={styles.meta}>Priority {head.priority}</Text>
          <Pressable onPress={dismiss}><Text style={styles.dismiss}>Dismiss</Text></Pressable>
        </View>
      )}
      <Pressable style={styles.btn} onPress={() => enqueue("Sale alert", 3)}>
        <Text>Normal notif</Text>
      </Pressable>
      <Pressable style={styles.btn} onPress={() => enqueue("Security breach", 1)}>
        <Text>Critical notif</Text>
      </Pressable>
      <Text style={{ marginTop: 16 }}>Queue: {queue.length}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  banner: { padding: 16, backgroundColor: "#e3f2fd", borderRadius: 8, marginBottom: 16 },
  critical: { backgroundColor: "#ffcdd2" },
  title: { fontWeight: "700", fontSize: 16 },
  meta: { fontSize: 12, color: "#666" },
  dismiss: { color: "#1976d2", marginTop: 8 },
  btn: { padding: 12, backgroundColor: "#eee", marginBottom: 8, borderRadius: 8 },
});
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| notifee / expo-notifications | OS-level priority channels |
| In-app queue vs OS queue | In-app for stacking when app foreground |
| LC 621 tie-in | Task scheduler cooldown + max heap |

---

## Quick Revision — Day 26

```
Web:  live prices → top K heap/sort → ranked bar chart
RN:   min-heap notification queue → dismiss pop
LC:   621 task scheduler; 767 reorganize; 358 k distance
```

---

*End of Day 26 machine coding*
