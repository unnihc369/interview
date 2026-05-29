# Day 6 — Machine Coding Saturday Revision

**React:** Mini task manager · undo/redo stack · background sync queue  
**React Native:** Deep link handler with queue

---

## Table of Contents

### React (Web)

1. [Problem — Task Manager with Undo/Redo](#1-problem--task-manager-with-undoredo)
2. [Undo/Redo Stack Architecture](#2-undoredo-stack-architecture)
3. [Background Sync Queue](#3-background-sync-queue)
4. [Full React Solution](#4-full-react-solution)
5. [React Interview Points](#5-react-interview-points)

### React Native

6. [Problem — Deep Link Queue Handler](#6-problem--deep-link-queue-handler)
7. [Linking API Setup](#7-linking-api-setup)
8. [Navigation Queue Pattern](#8-navigation-queue-pattern)
9. [Full RN Solution](#9-full-rn-solution)
10. [RN Interview Points](#10-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Task Manager with Undo/Redo

**Task:** Build a mini task manager combining:
- Add / toggle / delete tasks
- **Undo / redo** for all mutations (stack-based history)
- **Background sync queue** that persists changes to API sequentially

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Add task | Append to list |
| Toggle done | Mark complete/incomplete |
| Delete task | Remove from list |
| Undo / Redo | Revert/restore last N mutations |
| Sync queue | Queue API calls; process when online |
| Optimistic UI | Update UI immediately; sync in background |

---

## 2. Undo/Redo Stack Architecture

```
past:    [state₀, state₁, ...]   ← snapshots before current
present: stateₙ                   ← current tasks
future:  [stateₙ₊₁, ...]          ← redo stack (cleared on new action)
```

```ts
type Task = { id: string; text: string; done: boolean };

type HistoryState = {
  past: Task[][];
  present: Task[];
  future: Task[][];
};

type HistoryAction =
  | { type: "SET"; tasks: Task[] }
  | { type: "UNDO" }
  | { type: "REDO" };

function historyReducer(state: HistoryState, action: HistoryAction): HistoryState {
  switch (action.type) {
    case "SET": {
      return {
        past: [...state.past, state.present],
        present: action.tasks,
        future: [],
      };
    }
    case "UNDO": {
      if (state.past.length === 0) return state;
      const previous = state.past[state.past.length - 1];
      return {
        past: state.past.slice(0, -1),
        present: previous,
        future: [state.present, ...state.future],
      };
    }
    case "REDO": {
      if (state.future.length === 0) return state;
      const next = state.future[0];
      return {
        past: [...state.past, state.present],
        present: next,
        future: state.future.slice(1),
      };
    }
    default:
      return state;
  }
}
```

---

## 3. Background Sync Queue

```tsx
type SyncOperation = {
  id: string;
  type: "CREATE" | "UPDATE" | "DELETE";
  task: Task;
  timestamp: number;
};

function useSyncQueue() {
  const queueRef = useRef<SyncOperation[]>([]);
  const processingRef = useRef(false);
  const [pendingCount, setPendingCount] = useState(0);
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const onOnline = () => setIsOnline(true);
    const onOffline = () => setIsOnline(false);
    window.addEventListener("online", onOnline);
    window.addEventListener("offline", onOffline);
    return () => {
      window.removeEventListener("online", onOnline);
      window.removeEventListener("offline", onOffline);
    };
  }, []);

  const enqueue = useCallback((op: SyncOperation) => {
    queueRef.current.push(op);
    setPendingCount(queueRef.current.length);
    processQueue();
  }, []);

  const processQueue = useCallback(async () => {
    if (processingRef.current || !isOnline) return;
    processingRef.current = true;

    while (queueRef.current.length > 0 && navigator.onLine) {
      const op = queueRef.current[0];
      try {
        await syncToServer(op);
        queueRef.current.shift();
        setPendingCount(queueRef.current.length);
      } catch {
        break; // retry later
      }
    }

    processingRef.current = false;
  }, [isOnline]);

  useEffect(() => {
    if (isOnline) processQueue();
  }, [isOnline, processQueue]);

  return { enqueue, pendingCount, isOnline };
}

async function syncToServer(op: SyncOperation): Promise<void> {
  await new Promise((r) => setTimeout(r, 500)); // simulate API
  console.log("Synced:", op.type, op.task.id);
}
```

---

## 4. Full React Solution

```tsx
import { useReducer, useCallback, useState } from "react";

const initialHistory: HistoryState = {
  past: [],
  present: [],
  future: [],
};

export default function TaskManager() {
  const [history, dispatchHistory] = useReducer(historyReducer, initialHistory);
  const { enqueue, pendingCount, isOnline } = useSyncQueue();
  const [input, setInput] = useState("");

  const tasks = history.present;
  const canUndo = history.past.length > 0;
  const canRedo = history.future.length > 0;

  const updateTasks = useCallback(
    (newTasks: Task[], syncOp?: SyncOperation) => {
      dispatchHistory({ type: "SET", tasks: newTasks });
      if (syncOp) enqueue(syncOp);
    },
    [enqueue]
  );

  const addTask = () => {
    if (!input.trim()) return;
    const task: Task = {
      id: crypto.randomUUID(),
      text: input.trim(),
      done: false,
    };
    updateTasks([...tasks, task], { id: task.id, type: "CREATE", task, timestamp: Date.now() });
    setInput("");
  };

  const toggleTask = (id: string) => {
    const updated = tasks.map((t) =>
      t.id === id ? { ...t, done: !t.done } : t
    );
    const task = updated.find((t) => t.id === id)!;
    updateTasks(updated, { id, type: "UPDATE", task, timestamp: Date.now() });
  };

  const deleteTask = (id: string) => {
    const task = tasks.find((t) => t.id === id)!;
    updateTasks(
      tasks.filter((t) => t.id !== id),
      { id, type: "DELETE", task, timestamp: Date.now() }
    );
  };

  return (
    <div style={{ maxWidth: 480, margin: "0 auto", padding: 24 }}>
      <h2>Task Manager</h2>

      <div style={{ display: "flex", gap: 8, marginBottom: 8 }}>
        <span style={{ color: isOnline ? "green" : "red" }}>
          {isOnline ? "● Online" : "● Offline"}
        </span>
        {pendingCount > 0 && (
          <span style={{ color: "#666" }}>Syncing {pendingCount}...</span>
        )}
      </div>

      <div style={{ display: "flex", gap: 8, marginBottom: 16 }}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={(e) => e.key === "Enter" && addTask()}
          placeholder="New task..."
          style={{ flex: 1, padding: 8 }}
        />
        <button onClick={addTask}>Add</button>
      </div>

      <div style={{ display: "flex", gap: 8, marginBottom: 16 }}>
        <button onClick={() => dispatchHistory({ type: "UNDO" })} disabled={!canUndo}>
          Undo
        </button>
        <button onClick={() => dispatchHistory({ type: "REDO" })} disabled={!canRedo}>
          Redo
        </button>
      </div>

      <ul style={{ listStyle: "none", padding: 0 }}>
        {tasks.map((task) => (
          <li
            key={task.id}
            style={{
              display: "flex",
              alignItems: "center",
              gap: 8,
              padding: 8,
              borderBottom: "1px solid #eee",
            }}
          >
            <input
              type="checkbox"
              checked={task.done}
              onChange={() => toggleTask(task.id)}
            />
            <span style={{ textDecoration: task.done ? "line-through" : "none", flex: 1 }}>
              {task.text}
            </span>
            <button onClick={() => deleteTask(task.id)}>✕</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 5. React Interview Points

| Question | Answer |
|----------|--------|
| Undo/redo data structure | past + present + future stacks |
| Clear future on new action? | Yes — standard editor behavior |
| Optimistic UI | Update local state first; sync async |
| Offline queue | Ref-based queue; process on `online` event |
| Immutable snapshots | Deep copy tasks array on each SET |
| vs Day 1 useStack | Same pattern; here integrated with sync |

### Checklist

- [ ] Add / toggle / delete tasks
- [ ] Undo / redo with disabled states
- [ ] Background sync queue
- [ ] Online/offline indicator
- [ ] Pending sync count

---

# Part B — React Native

## 6. Problem — Deep Link Queue Handler

**Task:** Handle deep links that arrive **before navigation is ready** or in rapid succession — queue and process sequentially.

**Scenario:**

```
App cold start → deep link arrives → navigator not mounted yet
Multiple links in quick succession → process one at a time
```

---

## 7. Linking API Setup

```tsx
// App.tsx
import { Linking } from "react-native";
import { NavigationContainer, NavigationContainerRef } from "@react-navigation/native";

const linking = {
  prefixes: ["myapp://", "https://myapp.com"],
  config: {
    screens: {
      Home: "",
      Product: "product/:id",
      Profile: "user/:userId",
    },
  },
};

export default function App() {
  const navigationRef = useRef<NavigationContainerRef<RootStackParamList>>(null);
  const linkQueue = useDeepLinkQueue(navigationRef);

  return (
    <NavigationContainer
      ref={navigationRef}
      linking={linking}
      onReady={() => linkQueue.processPending()}
    >
      {/* Stack Navigator */}
    </NavigationContainer>
  );
}
```

---

## 8. Navigation Queue Pattern

```tsx
type DeepLink = {
  url: string;
  timestamp: number;
};

function useDeepLinkQueue(
  navigationRef: React.RefObject<NavigationContainerRef<RootStackParamList>>
) {
  const queueRef = useRef<DeepLink[]>([]);
  const processingRef = useRef(false);
  const isReadyRef = useRef(false);

  const parseAndNavigate = useCallback((url: string) => {
    const nav = navigationRef.current;
    if (!nav) return false;

    // Parse myapp://product/42 or https://myapp.com/product/42
    const productMatch = url.match(/product\/(\d+)/);
    if (productMatch) {
      nav.navigate("Product", { id: productMatch[1] });
      return true;
    }

    const userMatch = url.match(/user\/(\w+)/);
    if (userMatch) {
      nav.navigate("Profile", { userId: userMatch[1] });
      return true;
    }

    nav.navigate("Home");
    return true;
  }, [navigationRef]);

  const processNext = useCallback(async () => {
    if (processingRef.current || !isReadyRef.current) return;
    if (queueRef.current.length === 0) return;

    processingRef.current = true;
    const link = queueRef.current.shift()!;

    parseAndNavigate(link.url);

    // Small delay between navigations for animation
    await new Promise((r) => setTimeout(r, 300));
    processingRef.current = false;
    processNext();
  }, [parseAndNavigate]);

  const enqueue = useCallback(
    (url: string) => {
      queueRef.current.push({ url, timestamp: Date.now() });
      processNext();
    },
    [processNext]
  );

  const processPending = useCallback(() => {
    isReadyRef.current = true;
    processNext();
  }, [processNext]);

  useEffect(() => {
    // Handle initial URL (cold start)
    Linking.getInitialURL().then((url) => {
      if (url) enqueue(url);
    });

    // Handle URLs while app is running
    const sub = Linking.addEventListener("url", ({ url }) => {
      enqueue(url);
    });

    return () => sub.remove();
  }, [enqueue]);

  return { enqueue, processPending, queueSize: () => queueRef.current.length };
}
```

---

## 9. Full RN Solution

```tsx
// screens/ProductScreen.tsx
import { RouteProp, useRoute } from "@react-navigation/native";
import { View, Text, StyleSheet } from "react-native";

type RootStackParamList = {
  Home: undefined;
  Product: { id: string };
  Profile: { userId: string };
};

type ProductRoute = RouteProp<RootStackParamList, "Product">;

export default function ProductScreen() {
  const route = useRoute<ProductRoute>();
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Product #{route.params.id}</Text>
      <Text>Opened via deep link</Text>
    </View>
  );
}

// Testing deep links
// iOS: xcrun simctl openurl booted "myapp://product/42"
// Android: adb shell am start -W -a android.intent.action.VIEW -d "myapp://product/42"
```

### Linking config in app.json (Expo)

```json
{
  "expo": {
    "scheme": "myapp",
    "ios": { "associatedDomains": ["applinks:myapp.com"] },
    "android": {
      "intentFilters": [{
        "action": "VIEW",
        "data": [{ "scheme": "myapp" }, { "scheme": "https", "host": "myapp.com" }],
        "category": ["BROWSABLE", "DEFAULT"]
      }]
    }
  }
}
```

---

## 10. RN Interview Points

| Question | Answer |
|----------|--------|
| Why queue deep links? | Navigator may not be ready on cold start |
| `getInitialURL` vs listener | Initial for cold start; listener for warm |
| `onReady` callback | Process queued links after NavigationContainer mounts |
| Universal links vs custom scheme | https (verified) vs myapp:// |
| React Navigation linking | Declarative config maps paths to screens |
| Rapid links | Queue prevents navigation race/conflict |

### Checklist

- [ ] Linking prefixes configured
- [ ] getInitialURL + addEventListener
- [ ] Queue until navigation ready
- [ ] Sequential processing with delay
- [ ] Parse URL and navigate typed screens

---

## Quick Revision — Day 6 Machine Coding

```
React:
  historyReducer → past + present + future
  updateTasks → SET + enqueue sync op
  useSyncQueue → ref queue, process when online
  Optimistic UI + background sync

React Native:
  useDeepLinkQueue → queueRef + isReadyRef
  getInitialURL + addEventListener → enqueue
  onReady → processPending
  parseAndNavigate with regex or linking config
```

---

*End of Day 6 machine coding*
