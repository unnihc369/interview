# Day 3 — Machine Coding Revision

**React:** `useQueue` hook · sequential API calls  
**React Native:** Animated queue with progress bar

---

## Table of Contents

### React (Web)

1. [Problem — useQueue Hook](#1-problem--usequeue-hook)
2. [useQueue Implementation](#2-usequeue-implementation)
3. [Sequential API Caller](#3-sequential-api-caller)
4. [Full Demo Component](#4-full-demo-component)
5. [React Interview Points](#5-react-interview-points)

### React Native

6. [Problem — Animated Progress Queue](#6-problem--animated-progress-queue)
7. [Queue + Animated Progress Bar](#7-queue--animated-progress-bar)
8. [Full RN Solution](#8-full-rn-solution)
9. [RN Interview Points](#9-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — useQueue Hook

**Task:** Build a generic `useQueue<T>` hook and use it to execute **sequential API calls** (one after another, not parallel).

**Requirements:**

| Feature | Behavior |
|---------|----------|
| `enqueue(item)` | Add to back of queue |
| `dequeue()` | Remove and return front |
| `peek()` | View front without removing |
| `size` | Current queue length |
| `isEmpty` | Boolean helper |
| `clear()` | Empty the queue |
| Sequential runner | Process queue items one at a time via async fn |

---

## 2. useQueue Implementation

```tsx
import { useCallback, useRef, useState } from "react";

type UseQueueReturn<T> = {
  enqueue: (item: T) => void;
  dequeue: () => T | undefined;
  peek: () => T | undefined;
  clear: () => void;
  size: number;
  isEmpty: boolean;
  items: T[];
};

export function useQueue<T>(initial: T[] = []): UseQueueReturn<T> {
  const [items, setItems] = useState<T[]>(initial);
  const itemsRef = useRef(items);
  itemsRef.current = items;

  const enqueue = useCallback((item: T) => {
    setItems((prev) => [...prev, item]);
  }, []);

  const dequeue = useCallback((): T | undefined => {
    let removed: T | undefined;
    setItems((prev) => {
      if (prev.length === 0) return prev;
      removed = prev[0];
      return prev.slice(1);
    });
    return removed;
  }, []);

  const peek = useCallback((): T | undefined => {
    return itemsRef.current[0];
  }, []);

  const clear = useCallback(() => {
    setItems([]);
  }, []);

  return {
    enqueue,
    dequeue,
    peek,
    clear,
    size: items.length,
    isEmpty: items.length === 0,
    items,
  };
}
```

### Why ref + state?

- **State** drives re-renders for UI.
- **Ref** gives synchronous peek inside async callbacks without stale closure.

---

## 3. Sequential API Caller

```tsx
import { useCallback, useRef, useState } from "react";

type ApiJob = {
  id: string;
  url: string;
  label: string;
};

type JobResult = {
  id: string;
  status: "pending" | "running" | "success" | "error";
  data?: unknown;
  error?: string;
};

export function useSequentialFetcher() {
  const [results, setResults] = useState<JobResult[]>([]);
  const [isRunning, setIsRunning] = useState(false);
  const queueRef = useRef<ApiJob[]>([]);
  const runningRef = useRef(false);

  const updateResult = useCallback(
    (id: string, patch: Partial<JobResult>) => {
      setResults((prev) =>
        prev.map((r) => (r.id === id ? { ...r, ...patch } : r))
      );
    },
    []
  );

  const processNext = useCallback(async () => {
    if (runningRef.current) return;
    const job = queueRef.current.shift();
    if (!job) {
      setIsRunning(false);
      return;
    }

    runningRef.current = true;
    setIsRunning(true);
    updateResult(job.id, { status: "running" });

    try {
      const res = await fetch(job.url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      updateResult(job.id, { status: "success", data });
    } catch (err) {
      updateResult(job.id, {
        status: "error",
        error: err instanceof Error ? err.message : "Unknown error",
      });
    } finally {
      runningRef.current = false;
      processNext(); // process next in queue
    }
  }, [updateResult]);

  const addJobs = useCallback(
    (jobs: ApiJob[]) => {
      setResults((prev) => [
        ...prev,
        ...jobs.map((j) => ({
          id: j.id,
          status: "pending" as const,
        })),
      ]);
      queueRef.current.push(...jobs);
      processNext();
    },
    [processNext]
  );

  const cancelPending = useCallback(() => {
    queueRef.current = [];
    setResults((prev) =>
      prev.map((r) =>
        r.status === "pending" ? { ...r, status: "error", error: "Cancelled" } : r
      )
    );
  }, []);

  return { results, isRunning, addJobs, cancelPending };
}
```

### Key design decisions

| Decision | Rationale |
|----------|-----------|
| Ref-based queue | Async processing without re-render on every enqueue |
| Recursive `processNext` | Natural sequential flow after each completion |
| `runningRef` guard | Prevent concurrent processing |
| Status per job | UI can show pending/running/done per row |

---

## 4. Full Demo Component

```tsx
import { useQueue } from "./useQueue";
import { useSequentialFetcher } from "./useSequentialFetcher";

const DEMO_JOBS = [
  { id: "1", url: "https://jsonplaceholder.typicode.com/posts/1", label: "Post 1" },
  { id: "2", url: "https://jsonplaceholder.typicode.com/posts/2", label: "Post 2" },
  { id: "3", url: "https://jsonplaceholder.typicode.com/posts/3", label: "Post 3" },
];

export default function SequentialApiDemo() {
  const { results, isRunning, addJobs, cancelPending } = useSequentialFetcher();
  const localQueue = useQueue<string>();

  const handleRunAll = () => {
    addJobs(DEMO_JOBS);
  };

  return (
    <div style={{ maxWidth: 500, margin: "0 auto", padding: 16 }}>
      <h2>Sequential API Queue</h2>

      <div style={{ display: "flex", gap: 8, marginBottom: 16 }}>
        <button onClick={handleRunAll} disabled={isRunning}>
          Run 3 API Calls (Sequential)
        </button>
        <button onClick={cancelPending}>Cancel Pending</button>
      </div>

      {isRunning && <p>⏳ Processing queue...</p>}

      <ul style={{ listStyle: "none", padding: 0 }}>
        {results.map((r) => (
          <li
            key={r.id}
            style={{
              padding: 12,
              marginBottom: 8,
              borderRadius: 8,
              background:
                r.status === "success"
                  ? "#d4edda"
                  : r.status === "error"
                  ? "#f8d7da"
                  : r.status === "running"
                  ? "#cce5ff"
                  : "#fff3cd",
            }}
          >
            <strong>Job {r.id}</strong> — {r.status}
            {r.error && <span style={{ color: "red" }}> ({r.error})</span>}
          </li>
        ))}
      </ul>

      <hr />

      <h3>Local useQueue Demo</h3>
      <button onClick={() => localQueue.enqueue(`Item ${Date.now()}`)}>
        Enqueue
      </button>
      <button onClick={() => localQueue.dequeue()} disabled={localQueue.isEmpty}>
        Dequeue
      </button>
      <p>Size: {localQueue.size} | Front: {localQueue.peek() ?? "empty"}</p>
      <pre>{JSON.stringify(localQueue.items, null, 2)}</pre>
    </div>
  );
}
```

---

## 5. React Interview Points

| Question | Answer |
|----------|--------|
| Queue vs parallel fetch? | Sequential preserves order, avoids rate limits |
| Why ref for processing queue? | Async loop needs mutable queue without stale state |
| How to add retry? | On error, re-push job to front/back of queueRef |
| How to add concurrency limit? | Process N jobs at once with semaphore pattern |
| useQueue vs useReducer? | useQueue = generic FIFO; reducer = complex transitions |
| AbortController? | Pass signal to fetch; abort pending on cancel |

### Checklist

- [ ] Generic `useQueue<T>` with enqueue/dequeue/peek
- [ ] Sequential async processor
- [ ] Status tracking per job
- [ ] Cancel pending jobs
- [ ] Guard against concurrent runners

---

# Part B — React Native

## 6. Problem — Animated Progress Queue

**Task:** Show a list of tasks processed **one at a time** with an **animated progress bar** for the active task.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Task queue | Add multiple tasks; process sequentially |
| Progress bar | Animated 0→100% for current task |
| Status | pending → running → done |
| Native driver | Use `useNativeDriver: true` where possible |

---

## 7. Queue + Animated Progress Bar

```tsx
import { useRef, useCallback, useState } from "react";
import { Animated, Easing } from "react-native";

type Task = {
  id: string;
  title: string;
  status: "pending" | "running" | "done";
};

export function useTaskQueue() {
  const [tasks, setTasks] = useState<Task[]>([]);
  const queueRef = useRef<string[]>([]);
  const processingRef = useRef(false);
  const progressAnim = useRef(new Animated.Value(0)).current;

  const updateTask = useCallback((id: string, status: Task["status"]) => {
    setTasks((prev) =>
      prev.map((t) => (t.id === id ? { ...t, status } : t))
    );
  }, []);

  const animateProgress = useCallback(
    (duration: number): Promise<void> => {
      progressAnim.setValue(0);
      return new Promise((resolve) => {
        Animated.timing(progressAnim, {
          toValue: 1,
          duration,
          easing: Easing.linear,
          useNativeDriver: false, // width animation needs false
        }).start(() => resolve());
      });
    },
    [progressAnim]
  );

  const processNext = useCallback(async () => {
    if (processingRef.current || queueRef.current.length === 0) return;

    processingRef.current = true;
    const id = queueRef.current.shift()!;
    updateTask(id, "running");

    await animateProgress(2000); // simulate 2s work
    updateTask(id, "done");
    processingRef.current = false;
    processNext();
  }, [animateProgress, updateTask]);

  const addTask = useCallback(
    (title: string) => {
      const id = `${Date.now()}`;
      setTasks((prev) => [...prev, { id, title, status: "pending" }]);
      queueRef.current.push(id);
      processNext();
    },
    [processNext]
  );

  return { tasks, progressAnim, addTask };
}
```

### Progress bar width interpolation

```tsx
// For useNativeDriver: false (layout properties)
const widthInterpolated = progressAnim.interpolate({
  inputRange: [0, 1],
  outputRange: ["0%", "100%"],
});

// Alternative with native driver: use transform scaleX
const scaleX = progressAnim; // 0 to 1
// style={{ transform: [{ scaleX }], transformOrigin: 'left' }}
```

---

## 8. Full RN Solution

```tsx
import React from "react";
import {
  View,
  Text,
  TouchableOpacity,
  Animated,
  FlatList,
  StyleSheet,
} from "react-native";

function ProgressBar({ progress }: { progress: Animated.Value }) {
  const width = progress.interpolate({
    inputRange: [0, 1],
    outputRange: ["0%", "100%"],
  });

  return (
    <View style={styles.progressTrack}>
      <Animated.View style={[styles.progressFill, { width }]} />
    </View>
  );
}

function TaskItem({
  task,
  progressAnim,
}: {
  task: Task;
  progressAnim: Animated.Value;
}) {
  const statusIcon =
    task.status === "done" ? "✓" : task.status === "running" ? "▶" : "○";

  return (
    <View style={styles.taskItem}>
      <Text style={styles.taskTitle}>
        {statusIcon} {task.title}
      </Text>
      {task.status === "running" && <ProgressBar progress={progressAnim} />}
    </View>
  );
}

export default function AnimatedQueueScreen() {
  const { tasks, progressAnim, addTask } = useTaskQueue();

  return (
    <View style={styles.container}>
      <Text style={styles.header}>Task Progress Queue</Text>

      <TouchableOpacity
        style={styles.button}
        onPress={() => addTask(`Download file ${tasks.length + 1}`)}
      >
        <Text style={styles.buttonText}>Add Task</Text>
      </TouchableOpacity>

      <FlatList
        data={tasks}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TaskItem task={item} progressAnim={progressAnim} />
        )}
        contentContainerStyle={styles.list}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, paddingTop: 60, paddingHorizontal: 16 },
  header: { fontSize: 22, fontWeight: "bold", marginBottom: 16 },
  button: {
    backgroundColor: "#007AFF",
    padding: 14,
    borderRadius: 8,
    alignItems: "center",
    marginBottom: 16,
  },
  buttonText: { color: "#fff", fontSize: 16, fontWeight: "600" },
  list: { paddingBottom: 40 },
  taskItem: {
    backgroundColor: "#f8f8f8",
    padding: 14,
    borderRadius: 8,
    marginBottom: 10,
  },
  taskTitle: { fontSize: 16, marginBottom: 8 },
  progressTrack: {
    height: 6,
    backgroundColor: "#e0e0e0",
    borderRadius: 3,
    overflow: "hidden",
  },
  progressFill: {
    height: "100%",
    backgroundColor: "#007AFF",
  },
});
```

---

## 9. RN Interview Points

| Question | Answer |
|----------|--------|
| Why `useNativeDriver: false` for width? | Layout props not supported on native driver |
| Alternative for native driver? | `transform: [{ scaleX }]` with `transformOrigin: 'left'` |
| Single progressAnim shared? | OK — only one task runs at a time |
| Reanimated v2? | `useSharedValue` + `withTiming` for smoother UI thread anim |
| Queue in ref vs state? | Ref for processing order; state for UI list |

### Checklist

- [ ] Sequential task processing
- [ ] Animated progress for active task
- [ ] Visual status indicators
- [ ] Add task button
- [ ] FlatList for task list

---

## Quick Revision — Day 3 Machine Coding

```
React:
  useQueue<T> → enqueue, dequeue, peek, ref for sync access
  useSequentialFetcher → queueRef + processNext recursive
  One fetch at a time; status per job

React Native:
  useTaskQueue → queueRef + progressAnim
  Animated.timing 0→1 for progress bar
  Only running task shows progress
  width interpolation (nativeDriver: false)
```

---

*End of Day 3 machine coding*
