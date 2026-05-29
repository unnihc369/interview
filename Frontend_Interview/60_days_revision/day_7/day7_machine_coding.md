# Day 7 — Machine Coding Optimization Guide (Week 1 Assessment)

**Focus:** Performance · UX · Code quality · Interview polish for Week 1 components

---

## Table of Contents

1. [Optimization Framework](#1-optimization-framework)
2. [Day 1 — useStack / Undo-Redo Textarea](#2-day-1--usestack--undo-redo-textarea)
3. [Day 2 — Queue Visualizer (useReducer)](#3-day-2--queue-visualizer-usereducer)
4. [Day 3 — useQueue / Sequential API](#4-day-3--usequeue--sequential-api)
5. [Day 4 — Call Stack Visualizer](#5-day-4--call-stack-visualizer)
6. [Day 5 — Debounced Search](#6-day-5--debounced-search)
7. [Day 6 — Task Manager + Sync Queue](#7-day-6--task-manager--sync-queue)
8. [React Native — Week 1 Components](#8-react-native--week-1-components)
9. [Cross-Cutting React Optimizations](#9-cross-cutting-react-optimizations)
10. [Interview Presentation Guide](#10-interview-presentation-guide)
11. [Self-Assessment Checklist](#11-self-assessment-checklist)

---

## 1. Optimization Framework

Before optimizing, ask (in interview order):

| Priority | Question | Example |
|----------|----------|---------|
| 1 | **Correct?** | Edge cases, cleanup, race conditions |
| 2 | **Readable?** | Clear naming, single responsibility |
| 3 | **Performant?** | Unnecessary re-renders, memory leaks |
| 4 | **Accessible?** | Keyboard, ARIA, focus management |
| 5 | **Polished?** | Loading states, error UX, empty states |

### React optimization toolkit

```tsx
// Prevent unnecessary re-renders
const MemoChild = React.memo(ChildComponent);
const stableCallback = useCallback(() => {}, [deps]);
const stableValue = useMemo(() => compute(), [deps]);

// Avoid inline object/array in JSX props
// BAD:  <List items={items.filter(x => x.active)} />
// GOOD: const activeItems = useMemo(() => items.filter(...), [items]);
```

### When NOT to optimize

- Premature `useMemo` on cheap computations
- `React.memo` on components that always re-render anyway
- Over-abstracting hooks before requirements are clear

---

## 2. Day 1 — useStack / Undo-Redo Textarea

### Component: `useStack` hook + undoable textarea

**File:** [day_1/day1_machine_coding.md](../day_1/day1_machine_coding.md)

### Common issues

| Issue | Fix |
|-------|-----|
| Future not cleared on new edit | `set()` must reset `future: []` |
| Stale closure in undo | Use functional updates or refs |
| Undo on empty past | Guard `canUndo` before calling |
| Memory growth | Cap history length (e.g., max 50 snapshots) |

### Optimization: limit history size

```tsx
const MAX_HISTORY = 50;

case "SET": {
  const newPast = [...state.past, state.present].slice(-MAX_HISTORY);
  return { past: newPast, present: action.tasks, future: [] };
}
```

### Optimization: debounce snapshots for textarea

```tsx
// Don't push every keystroke — debounce 300ms for undo points
const debouncedPush = useDebouncedCallback((value: string) => {
  dispatch({ type: "SET", value });
}, 300);
```

### UX polish

- [ ] Disable undo/redo buttons with `canUndo` / `canRedo`
- [ ] Keyboard shortcuts: Ctrl+Z / Ctrl+Shift+Z
- [ ] aria-label on buttons

### Interview talking points

> "I use past/present/future arrays. On new edit after undo, I clear future — same as VS Code. I debounce history snapshots so undo granularity feels natural, not per-character."

---

## 3. Day 2 — Queue Visualizer (useReducer)

### Component: Task queue with processing delay

**File:** [day_2/day2_machine_coding.md](../day_2/day2_machine_coding.md)

### Common issues

| Issue | Fix |
|-------|-----|
| Timer leak on unmount | `return () => clearTimeout(id)` in effect |
| Double processing | Guard `processing` state in reducer |
| Pause doesn't stop current | Clarify spec: pause queue vs pause mid-task |
| Effect dependency loop | Split auto-start and complete effects |

### Optimization: useRef for interval ID

```tsx
const timerRef = useRef<ReturnType<typeof setTimeout>>();

useEffect(() => {
  if (!state.processing) return;
  timerRef.current = setTimeout(() => dispatch({ type: "COMPLETE_TASK" }), DELAY);
  return () => clearTimeout(timerRef.current);
}, [state.processing?.id]);
```

### UX polish

- [ ] Animate task moving from queue → processing → done
- [ ] Show estimated wait: `queue.length * DELAY`
- [ ] Empty state messages for each panel

### Interview talking points

> "useReducer keeps queue logic in one place — easier to test than scattered useState. Two effects: one starts processing when idle, one completes after timeout with cleanup."

---

## 4. Day 3 — useQueue / Sequential API

### Components: `useQueue<T>` + sequential fetcher

**File:** [day_3/day3_machine_coding.md](../day_3/day3_machine_coding.md)

### Common issues

| Issue | Fix |
|-------|-----|
| Race condition | AbortController or request ID check |
| Stale peek in async | `itemsRef.current` sync ref |
| Queue never drains | `processingRef` guard + recursive processNext |
| Parallel instead of sequential | Single `runningRef` flag |

### Optimization: AbortController

```tsx
const abortRef = useRef<AbortController>();

const fetchSequential = async (url: string) => {
  abortRef.current?.abort();
  abortRef.current = new AbortController();
  return fetch(url, { signal: abortRef.current.signal });
};
```

### Optimization: generic hook reuse

```tsx
function useQueue<T>() { /* generic */ }
function useSequentialRunner<T>(processor: (item: T) => Promise<void>) {
  const queue = useQueue<T>();
  // compose queue + runner
}
```

### Interview talking points

> "Sequential API calls prevent rate limiting and preserve order. I use a ref-based queue so enqueue doesn't trigger re-renders. AbortController cancels stale requests when user changes input."

---

## 5. Day 4 — Call Stack Visualizer

### Component: Debugger simulation with stack frames

**File:** [day_4/day4_machine_coding.md](../day_4/day4_machine_coding.md)

### Common issues

| Issue | Fix |
|-------|-----|
| Stack displayed bottom-up | Reverse for UI (top of stack at top) |
| Interval not cleared | Cleanup in useEffect return |
| ASSIGN doesn't update locals | Immutable update of top frame |
| Run continues after finished | Set `isFinished` and pause |

### Optimization: memoize frame list

```tsx
const visibleFrames = useMemo(
  () => [...state.stack].reverse(),
  [state.stack]
);
```

### UX polish

- [ ] Color-code ENTER (green), EXIT (red), LOG (blue)
- [ ] Step counter progress bar
- [ ] Syntax highlight pseudo-source

### Interview talking points

> "Real debuggers use AST parsing; for interview I use a trace array simulating ENTER/LOG/ASSIGN/EXIT. Stack is LIFO — displayed reversed so top frame appears first."

---

## 6. Day 5 — Debounced Search

### Component: Debounced search with cancellation

**File:** [day_5/day5_machine_coding.md](../day_5/day5_machine_coding.md)

### Common issues

| Issue | Fix |
|-------|-----|
| Search on every keystroke | Debounce 300ms in useEffect |
| Stale results shown | AbortController or ignore outdated request ID |
| No loading indicator | `loading` state during fetch |
| AbortError shown as error | Catch and ignore `AbortError` |

### Optimization checklist

```tsx
useEffect(() => {
  if (!query.trim()) { setResults([]); return; }

  const timer = setTimeout(async () => {
    abortRef.current?.abort();
    abortRef.current = new AbortController();
    // fetch with signal
  }, 300);

  return () => {
    clearTimeout(timer);
    abortRef.current?.abort();
  };
}, [query]);
```

### UX polish

- [ ] Highlight matching text in results
- [ ] Min 2 characters before search
- [ ] Recent searches in localStorage
- [ ] Skeleton loader instead of spinner

### Interview talking points

> "Debounce reduces API calls. AbortController prevents race where slow response overwrites newer results. Cleanup on unmount avoids setState on unmounted component."

---

## 7. Day 6 — Task Manager + Sync Queue

### Component: Undo/redo + background sync

**File:** [day_6/day6_machine_coding.md](../day_6/day6_machine_coding.md)

### Common issues

| Issue | Fix |
|-------|-----|
| Undo doesn't revert sync | Decide: undo local only or also cancel sync op |
| Sync runs offline | Queue in ref; process on `online` event |
| Duplicate sync ops | Deduplicate by task ID + operation type |
| Shallow copy bugs | Deep copy task arrays in history snapshots |

### Optimization: dedupe sync queue

```tsx
const enqueue = (op: SyncOperation) => {
  // Replace pending UPDATE for same task ID
  queueRef.current = queueRef.current.filter(
    (o) => !(o.task.id === op.task.id && o.type === "UPDATE")
  );
  queueRef.current.push(op);
};
```

### Optimization: optimistic UI with rollback

```tsx
const toggleTask = async (id: string) => {
  const prev = tasks;
  updateTasks(optimisticTasks);
  try {
    await api.update(id);
  } catch {
    updateTasks(prev); // rollback on failure
  }
};
```

### Interview talking points

> "Combines Day 1 undo/redo with Day 3 sequential queue. Offline-first: mutations queue in ref, sync when navigator.onLine. Optimistic UI for snappy UX."

---

## 8. React Native — Week 1 Components

### Summary table

| Day | Component | Key optimization |
|-----|-----------|------------------|
| 1 | Stack Navigator | Typed ParamList; native-stack for perf |
| 2 | FlatList + animations | `useNativeDriver: true`; animation queue |
| 3 | Progress bar queue | Single shared Animated.Value for active task |
| 4 | Custom card transitions | `@react-navigation/stack` for interpolator |
| 5 | Throttled infinite scroll | Guard `loading \|\| !hasMore`; throttle 500ms |
| 6 | Deep link queue | `onReady` before processing; sequential nav |

### FlatList performance (all list components)

```tsx
<FlatList
  data={items}
  keyExtractor={(item) => item.id}
  getItemLayout={(_, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
  removeClippedSubviews={true}
  maxToRenderPerBatch={10}
  windowSize={5}
  initialNumToRender={10}
/>
```

### Animation performance

```tsx
// Prefer transform/opacity with native driver
Animated.timing(value, {
  toValue: 1,
  useNativeDriver: true, // transform, opacity only
});

// For width/height → use scaleX or Reanimated layout animations
```

### onEndReached pitfalls

```tsx
// MUST guard — fires multiple times
const loadMore = useCallback(() => {
  if (loading || !hasMore) return;
  // ...
}, [loading, hasMore]);
```

### Deep linking checklist

- [ ] `linking` config on NavigationContainer
- [ ] `getInitialURL` for cold start
- [ ] Queue until `onReady`
- [ ] Test: `npx uri-scheme open myapp://product/42 --ios`

---

## 9. Cross-Cutting React Optimizations

### Custom hooks — rules

| Rule | Why |
|------|-----|
| Return stable callbacks with `useCallback` | Prevent child re-renders |
| Sync refs for async access | Avoid stale closures |
| Cleanup all subscriptions/timers | Prevent memory leaks |
| Co-locate related state | useReducer over many useStates |

### Error boundaries (mention in interview)

```tsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  render() {
    if (this.state.hasError) return <FallbackUI />;
    return this.props.children;
  }
}
```

### Accessibility minimums

```tsx
<button aria-label="Undo last change" disabled={!canUndo}>Undo</button>
<input aria-describedby="search-hint" role="searchbox" />
<div role="status" aria-live="polite">{loading ? "Loading..." : ""}</div>
```

### Testing mention (if interviewer asks)

```tsx
// React Testing Library
render(<DebouncedSearch />);
await userEvent.type(screen.getByRole("searchbox"), "test");
await waitFor(() => expect(screen.getByText(/result/i)).toBeInTheDocument());
```

---

## 10. Interview Presentation Guide

### 45-minute machine coding flow

| Min | Activity |
|-----|----------|
| 0–5 | Clarify requirements; write interface/types |
| 5–25 | Core implementation (happy path) |
| 25–35 | Edge cases + error handling |
| 35–40 | UX polish (loading, disabled states) |
| 40–45 | Explain trade-offs; mention optimizations |

### Phrases that score points

- "I'll use a ref for the async queue to avoid unnecessary re-renders."
- "Cleanup in useEffect return prevents memory leaks."
- "AbortController handles the race condition."
- "I'd add React.memo here if profiling showed unnecessary re-renders."
- "For RN, useNativeDriver only supports transform and opacity."

### Red flags to avoid

- Mutating state directly (`array.push` on state)
- Missing dependency array or wrong deps
- No loading/error/empty states
- Ignoring unmount cleanup
- Over-engineering before basic flow works

---

## 11. Self-Assessment Checklist

Rate each Week 1 component: ✅ built · ⚠️ partial · ❌ not attempted

### React (Web)

| Component | Built | Optimized | Can explain trade-offs |
|-----------|-------|-----------|------------------------|
| useStack / textarea | ⬜ | ⬜ | ⬜ |
| Queue visualizer | ⬜ | ⬜ | ⬜ |
| useQueue / sequential API | ⬜ | ⬜ | ⬜ |
| Call stack visualizer | ⬜ | ⬜ | ⬜ |
| Debounced search | ⬜ | ⬜ | ⬜ |
| Task manager + sync | ⬜ | ⬜ | ⬜ |

### React Native

| Component | Built | Optimized | Can explain trade-offs |
|-----------|-------|-----------|------------------------|
| Stack Navigator | ⬜ | ⬜ | ⬜ |
| Animated FlatList queue | ⬜ | ⬜ | ⬜ |
| Progress bar queue | ⬜ | ⬜ | ⬜ |
| Custom card transitions | ⬜ | ⬜ | ⬜ |
| Throttled infinite scroll | ⬜ | ⬜ | ⬜ |
| Deep link queue | ⬜ | ⬜ | ⬜ |

### Pick ONE to rebuild today (timed 45 min)

```
Component chosen: _______________
Time taken: ___ min
Optimizations applied:
  1. _______________
  2. _______________
  3. _______________
Would pass interview: Y / N
```

---

## Quick Reference — Optimization Patterns

```
Undo/redo     → past/present/future; clear future on edit; cap history
Queue         → useReducer or ref queue; FIFO with guards
Debounce      → useEffect + setTimeout + cleanup
Abort         → AbortController per request; ignore AbortError
Sequential    → ref queue + processNext recursive + runningRef
RN FlatList   → keyExtractor, getItemLayout, onEndReached guard
RN animate    → useNativeDriver for transform/opacity
Deep links    → queue until NavigationContainer onReady
Sync queue    → ref + online event + optimistic UI
```

---

*End of Day 7 optimization guide — polish beats feature creep in interviews*
