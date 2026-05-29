# Day 27 — Machine Coding Revision

**React:** Schedule posts heap timestamp  
**React Native:** Offline priority sync

---

## Table of Contents

### React (Web)

1. [Problem — Scheduled Posts Queue](#1-problem--scheduled-posts-queue)
2. [Min-Heap by publishAt](#2-min-heap-by-publishat)
3. [Scheduler UI](#3-scheduler-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Offline Priority Sync](#5-offline-priority-sync)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Scheduled Posts Queue

**Task:** Compose social posts with **schedule time**. Min-heap orders by `publishAt`. Timer publishes when `now >= publishAt`.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Compose | Text + datetime-local |
| Queue | Sorted by soonest first |
| Auto publish | setInterval checks heap peek |
| Published log | History list |

---

## 2. Min-Heap by publishAt

```js
class PostMinHeap {
  constructor() {
    this.data = [];
  }
  push(post) {
    this.data.push(post);
    this.data.sort((a, b) => a.publishAt - b.publishAt);
  }
  peek() { return this.data[0]; }
  pop() { return this.data.shift(); }
  toArray() { return [...this.data].sort((a, b) => a.publishAt - b.publishAt); }
}
```

---

## 3. Scheduler UI

```tsx
import { useEffect, useRef, useState } from "react";

type Post = { id: string; content: string; publishAt: number };

export default function SchedulePostsQueue() {
  const heapRef = useRef<PostMinHeap>(new PostMinHeap());
  const [queue, setQueue] = useState<Post[]>([]);
  const [published, setPublished] = useState<Post[]>([]);
  const [content, setContent] = useState("");
  const [when, setWhen] = useState("");

  const refresh = () => setQueue(heapRef.current.toArray());

  const schedule = () => {
    if (!content || !when) return;
    const post: Post = {
      id: crypto.randomUUID(),
      content,
      publishAt: new Date(when).getTime(),
    };
    heapRef.current.push(post);
    setContent("");
    refresh();
  };

  useEffect(() => {
    const id = setInterval(() => {
      const now = Date.now();
      while (heapRef.current.peek() && heapRef.current.peek()!.publishAt <= now) {
        const p = heapRef.current.pop();
        if (p) setPublished((pub) => [p, ...pub]);
      }
      refresh();
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return (
    <div style={{ padding: 24, maxWidth: 520 }}>
      <h2>Scheduled Posts</h2>
      <textarea value={content} onChange={(e) => setContent(e.target.value)} rows={3} style={{ width: "100%" }} />
      <input type="datetime-local" value={when} onChange={(e) => setWhen(e.target.value)} />
      <button onClick={schedule}>Schedule</button>
      <h3>Queue ({queue.length})</h3>
      <ul>
        {queue.map((p) => (
          <li key={p.id}>{new Date(p.publishAt).toLocaleString()} — {p.content}</li>
        ))}
      </ul>
      <h3>Published</h3>
      <ul>
        {published.map((p) => (
          <li key={p.id}>{p.content}</li>
        ))}
      </ul>
    </div>
  );
}

class PostMinHeap {
  data: Post[] = [];
  push(post: Post) {
    this.data.push(post);
    this.data.sort((a, b) => a.publishAt - b.publishAt);
  }
  peek() { return this.data[0]; }
  pop() { return this.data.shift(); }
  toArray() { return [...this.data].sort((a, b) => a.publishAt - b.publishAt); }
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| setInterval vs rAF | 1s poll fine for schedule |
| Missed posts (tab sleep) | On focus, flush all due |
| Timezone | Store UTC ms; display local |

---

# Part B — React Native

## 5. Offline Priority Sync

**Task:** Queue offline actions with priority. On reconnect, sync **highest priority first** (lower number = higher).

```tsx
import { useEffect, useState } from "react";
import NetInfo from "@react-native-community/netinfo";
import { View, Text, FlatList, Pressable, StyleSheet } from "react-native";

type Action = { id: string; type: string; priority: number; payload: string };

export default function OfflinePrioritySync() {
  const [queue, setQueue] = useState<Action[]>([]);
  const [online, setOnline] = useState(true);
  const [log, setLog] = useState<string[]>([]);

  useEffect(() => {
    const unsub = NetInfo.addEventListener((s) => setOnline(!!s.isConnected));
    return () => unsub();
  }, []);

  useEffect(() => {
    if (!online || queue.length === 0) return;
    const sorted = [...queue].sort((a, b) => a.priority - b.priority);
    const next = sorted[0];
    setLog((l) => [`Synced P${next.priority}: ${next.type}`, ...l]);
    setQueue((q) => q.filter((a) => a.id !== next.id));
  }, [online, queue.length]);

  const enqueue = (type: string, priority: number) => {
    setQueue((q) =>
      [...q, { id: String(Date.now()), type, priority, payload: "{}" }].sort(
        (a, b) => a.priority - b.priority
      )
    );
  };

  return (
    <View style={styles.container}>
      <Text>Network: {online ? "Online" : "Offline"}</Text>
      <Pressable style={styles.btn} onPress={() => enqueue("delete", 1)}>
        <Text>Enqueue critical (P1)</Text>
      </Pressable>
      <Pressable style={styles.btn} onPress={() => enqueue("like", 3)}>
        <Text>Enqueue normal (P3)</Text>
      </Pressable>
      <Text style={{ marginTop: 12 }}>Queue ({queue.length})</Text>
      <FlatList
        data={queue}
        keyExtractor={(a) => a.id}
        renderItem={({ item }) => (
          <Text>P{item.priority} — {item.type}</Text>
        )}
      />
      <Text style={{ marginTop: 12 }}>Sync log</Text>
      {log.slice(0, 5).map((l, i) => (
        <Text key={i} style={{ fontSize: 12 }}>{l}</Text>
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  btn: { padding: 12, backgroundColor: "#eee", marginTop: 8, borderRadius: 8 },
});
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| NetInfo | `@react-native-community/netinfo` |
| Persist queue | AsyncStorage / SQLite |
| Conflict after sync | Server wins or CRDT |

---

## Quick Revision — Day 27

```
Web:  min-heap publishAt → interval flush due posts
RN:   offline priority queue → sync on reconnect
LC:   1046 last stone; 1054 barcodes; 1353 max events
```

---

*End of Day 27 machine coding*
