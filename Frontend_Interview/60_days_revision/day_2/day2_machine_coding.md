# Day 2 — Machine Coding Revision

**React:** Queue visualizer with `useReducer` · add tasks · process with delay  
**React Native:** FlatList with queued animations · incremental load

---

## Table of Contents

### React (Web)

1. [Problem — Queue Visualizer](#1-problem--queue-visualizer)
2. [State Design with useReducer](#2-state-design-with-usereducer)
3. [Reducer Implementation](#3-reducer-implementation)
4. [QueueVisualizer Component — Full Solution](#4-queuevisualizer-component--full-solution)
5. [CSS & UX Details](#5-css--ux-details)
6. [React Interview Points](#6-react-interview-points)

### React Native

7. [Problem — Animated FlatList Queue](#7-problem--animated-flatlist-queue)
8. [Incremental Data Loading](#8-incremental-data-loading)
9. [Queued Animation Pattern](#9-queued-animation-pattern)
10. [Full RN Solution](#10-full-rn-solution)
11. [RN Interview Points](#11-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Queue Visualizer

**Task:** Build a **task queue visualizer** that demonstrates FIFO processing with a configurable delay.

**Requirements (typical interview):**

| Feature | Behavior |
|---------|----------|
| Add task | User enters task name → enqueued at back |
| Process queue | Front task moves to "processing" → completes after delay → removed |
| Pause / Resume | Stop/resume automatic processing |
| Visual states | `pending` (queued), `processing` (active), `done` (history) |
| useReducer | All queue logic in reducer, not scattered `useState` |

**Data model:**

```
queue:     [{ id, name, status: 'pending' }]
processing: { id, name } | null
completed: [{ id, name, finishedAt }]
```

---

## 2. State Design with useReducer

### Why useReducer over useState?

| useState | useReducer |
|----------|------------|
| Simple independent values | Complex state transitions |
| Multiple `setState` calls | Single dispatch with action type |
| Hard to trace updates | Predictable action → state mapping |

### Action types

```ts
type Task = { id: string; name: string };

type State = {
  queue: Task[];
  processing: Task | null;
  completed: (Task & { finishedAt: number })[];
  isPaused: boolean;
};

type Action =
  | { type: "ADD_TASK"; name: string }
  | { type: "START_PROCESSING" }
  | { type: "COMPLETE_TASK" }
  | { type: "TOGGLE_PAUSE" }
  | { type: "CLEAR_COMPLETED" };
```

---

## 3. Reducer Implementation

```tsx
import { useReducer } from "react";

type Task = { id: string; name: string };
type CompletedTask = Task & { finishedAt: number };

type State = {
  queue: Task[];
  processing: Task | null;
  completed: CompletedTask[];
  isPaused: boolean;
};

type Action =
  | { type: "ADD_TASK"; name: string }
  | { type: "START_PROCESSING" }
  | { type: "COMPLETE_TASK" }
  | { type: "TOGGLE_PAUSE" }
  | { type: "CLEAR_COMPLETED" };

const initialState: State = {
  queue: [],
  processing: null,
  completed: [],
  isPaused: false,
};

function generateId() {
  return `${Date.now()}-${Math.random().toString(36).slice(2, 7)}`;
}

function queueReducer(state: State, action: Action): State {
  switch (action.type) {
    case "ADD_TASK": {
      if (!action.name.trim()) return state;
      const task: Task = { id: generateId(), name: action.name.trim() };
      return { ...state, queue: [...state.queue, task] };
    }
    case "START_PROCESSING": {
      if (state.processing || state.queue.length === 0 || state.isPaused) {
        return state;
      }
      const [next, ...rest] = state.queue;
      return { ...state, queue: rest, processing: next };
    }
    case "COMPLETE_TASK": {
      if (!state.processing) return state;
      const finished: CompletedTask = {
        ...state.processing,
        finishedAt: Date.now(),
      };
      return {
        ...state,
        processing: null,
        completed: [finished, ...state.completed],
      };
    }
    case "TOGGLE_PAUSE":
      return { ...state, isPaused: !state.isPaused };
    case "CLEAR_COMPLETED":
      return { ...state, completed: [] };
    default:
      return state;
  }
}
```

---

## 4. QueueVisualizer Component — Full Solution

```tsx
import { useReducer, useEffect, useState, useCallback } from "react";

const PROCESS_DELAY_MS = 1500;

// ... types, initialState, queueReducer from above ...

export default function QueueVisualizer() {
  const [state, dispatch] = useReducer(queueReducer, initialState);
  const [input, setInput] = useState("");

  const addTask = useCallback(() => {
    dispatch({ type: "ADD_TASK", name: input });
    setInput("");
  }, [input]);

  // Auto-start processing when queue has items and nothing is processing
  useEffect(() => {
    if (!state.isPaused && !state.processing && state.queue.length > 0) {
      dispatch({ type: "START_PROCESSING" });
    }
  }, [state.queue.length, state.processing, state.isPaused]);

  // Complete task after delay
  useEffect(() => {
    if (!state.processing) return;
    const timer = setTimeout(() => {
      dispatch({ type: "COMPLETE_TASK" });
    }, PROCESS_DELAY_MS);
    return () => clearTimeout(timer);
  }, [state.processing?.id]);

  return (
    <div className="queue-visualizer">
      <h2>Task Queue Visualizer</h2>

      <div className="controls">
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={(e) => e.key === "Enter" && addTask()}
          placeholder="Enter task name"
        />
        <button onClick={addTask} disabled={!input.trim()}>
          Add to Queue
        </button>
        <button onClick={() => dispatch({ type: "TOGGLE_PAUSE" })}>
          {state.isPaused ? "Resume" : "Pause"}
        </button>
        <button onClick={() => dispatch({ type: "CLEAR_COMPLETED" })}>
          Clear Done
        </button>
      </div>

      <section className="panel processing-panel">
        <h3>Processing</h3>
        {state.processing ? (
          <div className="task-card active">
            <span className="spinner" />
            {state.processing.name}
          </div>
        ) : (
          <p className="empty">Idle</p>
        )}
      </section>

      <section className="panel">
        <h3>Queue ({state.queue.length})</h3>
        <ul className="task-list">
          {state.queue.map((task, i) => (
            <li key={task.id} className="task-card pending">
              <span className="position">{i + 1}</span>
              {task.name}
            </li>
          ))}
          {state.queue.length === 0 && <p className="empty">No pending tasks</p>}
        </ul>
      </section>

      <section className="panel">
        <h3>Completed ({state.completed.length})</h3>
        <ul className="task-list">
          {state.completed.map((task) => (
            <li key={task.id} className="task-card done">
              ✓ {task.name}
            </li>
          ))}
        </ul>
      </section>
    </div>
  );
}
```

---

## 5. CSS & UX Details

```css
.queue-visualizer {
  max-width: 600px;
  margin: 0 auto;
  font-family: system-ui, sans-serif;
}

.controls {
  display: flex;
  gap: 8px;
  margin-bottom: 16px;
}

.controls input {
  flex: 1;
  padding: 8px 12px;
}

.panel {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 12px;
}

.task-card {
  padding: 8px 12px;
  border-radius: 6px;
  margin: 4px 0;
  display: flex;
  align-items: center;
  gap: 8px;
}

.task-card.pending { background: #fff3cd; }
.task-card.active  { background: #cce5ff; }
.task-card.done    { background: #d4edda; }

.spinner {
  width: 14px;
  height: 14px;
  border: 2px solid #007bff;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.position {
  font-weight: bold;
  color: #666;
  min-width: 20px;
}
```

### Edge cases to mention

- Empty input → no-op in reducer
- Pause mid-processing → current task still completes (or freeze — clarify with interviewer)
- Rapid add → queue grows; processing one at a time (FIFO)
- Cleanup `setTimeout` on unmount via effect return

---

## 6. React Interview Points

| Question | Answer |
|----------|--------|
| Why `useReducer` here? | Multiple related state fields; predictable transitions |
| Why two `useEffect`s? | Separation: start processing vs complete after delay |
| Dependency array for processing effect | `[state.processing?.id]` — re-run only when new task starts |
| Alternative to setTimeout? | `async` queue worker, Web Worker, or requestAnimationFrame |
| How to add priority queue? | Sort on ADD or use separate high-priority array |
| Immutable updates? | Spread arrays; never mutate `state.queue.push()` |

### Machine coding checklist

- [ ] useReducer with typed actions
- [ ] Add task to queue
- [ ] FIFO processing with delay
- [ ] Pause/resume
- [ ] Visual distinction: pending / processing / done
- [ ] Cleanup timers on unmount

---

# Part B — React Native

## 7. Problem — Animated FlatList Queue

**Task:** Build a FlatList that loads items incrementally and animates each new item with a **queued animation** (one after another, not all at once).

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Incremental load | Fetch/load 10 items at a time on scroll end |
| Queued animation | New items fade/slide in sequentially |
| FlatList performance | `keyExtractor`, `getItemLayout` if fixed height |
| Pull to refresh | Reset list and animation queue |

---

## 8. Incremental Data Loading

```tsx
const PAGE_SIZE = 10;

function generateItems(start: number, count: number) {
  return Array.from({ length: count }, (_, i) => ({
    id: String(start + i),
    title: `Item ${start + i + 1}`,
    animated: false,
  }));
}

type Item = { id: string; title: string; animated: boolean };
```

### Pagination hook

```tsx
function usePaginatedItems(pageSize: number) {
  const [items, setItems] = useState<Item[]>(() =>
    generateItems(0, pageSize)
  );
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;
    setLoading(true);
    // Simulate API delay
    await new Promise((r) => setTimeout(r, 800));
    setItems((prev) => {
      const nextStart = prev.length;
      if (nextStart >= 50) {
        setHasMore(false);
        return prev;
      }
      return [...prev, ...generateItems(nextStart, pageSize)];
    });
    setLoading(false);
  }, [loading, hasMore, pageSize]);

  const refresh = useCallback(() => {
    setItems(generateItems(0, pageSize));
    setHasMore(true);
  }, [pageSize]);

  return { items, setItems, loading, hasMore, loadMore, refresh };
}
```

---

## 9. Queued Animation Pattern

### Animation queue concept

```
New items added → push IDs to animationQueue
Process queue one at a time:
  1. Animate item[i] (fade in + slide)
  2. On complete → process next in queue
```

```tsx
import { useRef, useCallback } from "react";
import { Animated } from "react-native";

function useAnimationQueue() {
  const queueRef = useRef<string[]>([]);
  const processingRef = useRef(false);
  const animValuesRef = useRef<Map<string, Animated.Value>>(new Map());

  const getAnimValue = (id: string) => {
    if (!animValuesRef.current.has(id)) {
      animValuesRef.current.set(id, new Animated.Value(0));
    }
    return animValuesRef.current.get(id)!;
  };

  const processNext = useCallback(() => {
    if (processingRef.current || queueRef.current.length === 0) return;
    processingRef.current = true;
    const id = queueRef.current.shift()!;
    const anim = getAnimValue(id);

    Animated.timing(anim, {
      toValue: 1,
      duration: 400,
      useNativeDriver: true,
    }).start(() => {
      processingRef.current = false;
      processNext();
    });
  }, []);

  const enqueue = useCallback(
    (ids: string[]) => {
      queueRef.current.push(...ids);
      processNext();
    },
    [processNext]
  );

  const reset = useCallback(() => {
    queueRef.current = [];
    processingRef.current = false;
    animValuesRef.current.clear();
  }, []);

  return { getAnimValue, enqueue, reset };
}
```

---

## 10. Full RN Solution

```tsx
import React, { useEffect, useCallback } from "react";
import {
  View,
  Text,
  FlatList,
  Animated,
  ActivityIndicator,
  RefreshControl,
  StyleSheet,
} from "react-native";

type Item = { id: string; title: string };

function AnimatedListItem({
  item,
  animValue,
}: {
  item: Item;
  animValue: Animated.Value;
}) {
  const translateY = animValue.interpolate({
    inputRange: [0, 1],
    outputRange: [30, 0],
  });
  const opacity = animValue;

  return (
    <Animated.View
      style={[styles.item, { opacity, transform: [{ translateY }] }]}
    >
      <Text style={styles.itemText}>{item.title}</Text>
    </Animated.View>
  );
}

export default function QueuedAnimationList() {
  const { items, setItems, loading, hasMore, loadMore, refresh } =
    usePaginatedItems(PAGE_SIZE);
  const { getAnimValue, enqueue, reset } = useAnimationQueue();
  const prevCountRef = useRef(items.length);

  // Enqueue animation for newly added items
  useEffect(() => {
    if (items.length > prevCountRef.current) {
      const newIds = items
        .slice(prevCountRef.current)
        .map((item) => item.id);
      enqueue(newIds);
    }
    prevCountRef.current = items.length;
  }, [items.length, enqueue]);

  const onRefresh = useCallback(() => {
    reset();
    prevCountRef.current = 0;
    refresh();
  }, [reset, refresh]);

  const renderItem = useCallback(
    ({ item }: { item: Item }) => (
      <AnimatedListItem item={item} animValue={getAnimValue(item.id)} />
    ),
    [getAnimValue]
  );

  return (
    <FlatList
      data={items}
      keyExtractor={(item) => item.id}
      renderItem={renderItem}
      onEndReached={loadMore}
      onEndReachedThreshold={0.3}
      ListFooterComponent={
        loading ? <ActivityIndicator style={{ margin: 16 }} /> : null
      }
      refreshControl={
        <RefreshControl refreshing={false} onRefresh={onRefresh} />
      }
      contentContainerStyle={styles.list}
    />
  );
}

const styles = StyleSheet.create({
  list: { padding: 16 },
  item: {
    padding: 16,
    backgroundColor: "#f5f5f5",
    borderRadius: 8,
    marginBottom: 8,
  },
  itemText: { fontSize: 16 },
});
```

---

## 11. RN Interview Points

| Question | Answer |
|----------|--------|
| Why animation queue? | Avoid janky simultaneous animations; controlled UX |
| `useNativeDriver: true`? | Runs on UI thread; only transform/opacity supported |
| FlatList vs ScrollView | FlatList virtualizes — required for long lists |
| `onEndReached` pitfalls | Can fire multiple times — guard with `loading` flag |
| Reanimated alternative? | `react-native-reanimated` for complex shared transitions |
| Memory with Animated.Value Map | Clear on refresh/unmount |

### RN machine coding checklist

- [ ] FlatList with pagination
- [ ] Loading indicator at footer
- [ ] Queued sequential animations for new items
- [ ] Pull to refresh resets state
- [ ] `keyExtractor` provided
- [ ] Mention `useNativeDriver: true`

---

## Quick Revision — Day 2 Machine Coding

```
React:
  useReducer → ADD_TASK, START_PROCESSING, COMPLETE_TASK
  useEffect #1 → auto-dequeue when idle
  useEffect #2 → setTimeout complete after delay
  FIFO: shift from queue front

React Native:
  FlatList + onEndReached pagination
  Animation queue ref → process one Animated.timing at a time
  getAnimValue Map per item id
  Refresh → reset queue + reload data
```

---

*End of Day 2 machine coding*
