# Day 22 — Machine Coding Revision

**React:** Task scheduler priority queue  
**React Native:** Audio priority queue

---

## Table of Contents

### React (Web)

1. [Problem — Task Scheduler](#1-problem--task-scheduler)
2. [Min-Heap Priority Queue](#2-min-heap-priority-queue)
3. [React UI](#3-react-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Audio Priority Queue](#5-audio-priority-queue)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Task Scheduler

**Task:** Todo list where tasks have **priority** (1 = highest). Always process/display highest priority first. Add task, complete top task, show queue.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Add | Title + priority number |
| Queue order | Lower number = higher priority (min-heap) |
| Complete | Remove highest-priority task |
| Peek | Show next task prominently |

---

## 2. Min-Heap Priority Queue

```js
class MinHeap {
  constructor(compare = (a, b) => a - b) {
    this.data = [];
    this.compare = compare;
  }
  push(v) { this.data.push(v); this._up(this.data.length - 1); }
  pop() {
    if (!this.data.length) return undefined;
    const t = this.data[0];
    const last = this.data.pop();
    if (this.data.length) { this.data[0] = last; this._down(0); }
    return t;
  }
  peek() { return this.data[0]; }
  _up(i) { /* sift up */ }
  _down(i) { /* sift down */ }
}

// Task compare by priority then createdAt
const taskHeap = new MinHeap((a, b) =>
  a.priority !== b.priority ? a.priority - b.priority : a.createdAt - b.createdAt
);
```

---

## 3. React UI

```tsx
import { useState, useMemo } from "react";

type Task = { id: string; title: string; priority: number; createdAt: number };

function useTaskHeap() {
  const [tasks, setTasks] = useState<Task[]>([]);

  const sorted = useMemo(() => {
    return [...tasks].sort((a, b) =>
      a.priority !== b.priority ? a.priority - b.priority : a.createdAt - b.createdAt
    );
  }, [tasks]);

  const add = (title: string, priority: number) => {
    setTasks((t) => [
      ...t,
      { id: crypto.randomUUID(), title, priority, createdAt: Date.now() },
    ]);
  };

  const completeTop = () => setTasks((t) => sorted.slice(1));

  return { queue: sorted, next: sorted[0], add, completeTop };
}

export default function TaskScheduler() {
  const { queue, next, add, completeTop } = useTaskHeap();
  const [title, setTitle] = useState("");
  const [priority, setPriority] = useState(2);

  return (
    <div style={{ maxWidth: 420, margin: 24 }}>
      <h2>Priority Task Queue</h2>
      {next && (
        <div style={{ padding: 16, background: "#e3f2fd", marginBottom: 16 }}>
          <strong>Next:</strong> {next.title} (P{next.priority})
          <button onClick={completeTop} style={{ marginLeft: 8 }}>Complete</button>
        </div>
      )}
      <input value={title} onChange={(e) => setTitle(e.target.value)} placeholder="Task" />
      <input type="number" value={priority} onChange={(e) => setPriority(+e.target.value)} min={1} max={5} />
      <button onClick={() => { add(title, priority); setTitle(""); }}>Add</button>
      <ul>
        {queue.map((t) => (
          <li key={t.id}>P{t.priority} — {t.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

Interview note: React state uses sorted array for simplicity; real PQ mutates heap in O(log n).

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Heap vs sorted array insert | Heap O(log n); array insert+sort O(n log n) |
| Stable priority | Tie-break with `createdAt` or sequence id |
| Immutable complete | Filter out id or slice sorted copy |

---

# Part B — React Native

## 5. Audio Priority Queue

**Task:** Sound effect queue — **urgent** sounds (notification) play before **ambient** sounds. Min-heap by priority; `expo-av` playback.

```tsx
import { useRef, useState } from "react";
import { Button, Text, View } from "react-native";
import { Audio } from "expo-av";

type SoundJob = { id: string; uri: string; priority: number };

class SoundPriorityQueue {
  private heap: SoundJob[] = [];
  compare(a: SoundJob, b: SoundJob) { return a.priority - b.priority; }
  push(job: SoundJob) { /* heap push */ this.heap.push(job); this._up(this.heap.length - 1); }
  pop(): SoundJob | undefined { /* heap pop */ return undefined; }
  peek() { return this.heap[0]; }
  _up(i: number) { /* ... */ }
  _down(i: number) { /* ... */ }
}

export default function AudioPriorityPlayer() {
  const queueRef = useRef(new SoundPriorityQueue());
  const [playing, setPlaying] = useState<string | null>(null);

  const enqueue = (uri: string, priority: number) => {
    queueRef.current.push({ id: String(Date.now()), uri, priority });
    playNext();
  };

  const playNext = async () => {
    const job = queueRef.current.peek();
    if (!job || playing) return;
    setPlaying(job.id);
    const { sound } = await Audio.Sound.createAsync({ uri: job.uri });
    await sound.playAsync();
    sound.setOnPlaybackStatusUpdate((s) => {
      if (s.isLoaded && s.didJustFinish) {
        queueRef.current.pop();
        setPlaying(null);
        playNext();
      }
    });
  };

  return (
    <View style={{ padding: 24 }}>
      <Text>Audio Priority Queue</Text>
      <Button title="Urgent (P1)" onPress={() => enqueue("urgent.mp3", 1)} />
      <Button title="Normal (P3)" onPress={() => enqueue("click.mp3", 3)} />
      {playing && <Text>Playing: {playing}</Text>}
    </View>
  );
}
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Single Audio instance? | One player; queue rest in heap |
| Interrupt current? | Higher priority (lower number) stops current |
| Background audio | iOS audio session category |

---

## Quick Revision — Day 22

```
Web:  tasks sorted by priority → complete top → min-heap mental model
RN:   SoundPriorityQueue → playNext chain on finish
LC:   215 min-heap K; 347 freq heap; 23 merge K
```

---

*End of Day 22 machine coding*
