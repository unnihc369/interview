# Day 7 — Sunday Assessment (Week 1 Review)

**Focus:** Weekly JS/TS checklist · Self-assessment · Gap identification

---

## Table of Contents

1. [Week 1 Topic Checklist](#1-week-1-topic-checklist)
2. [JavaScript Self-Assessment](#2-javascript-self-assessment)
3. [TypeScript Self-Assessment](#3-typescript-self-assessment)
4. [Concept Integration Scenarios](#4-concept-integration-scenarios)
5. [Oral Interview Drill (15 min)](#5-oral-interview-drill-15-min)
6. [Gap Analysis Worksheet](#6-gap-analysis-worksheet)
7. [Week 2 Preview](#7-week-2-preview)

---

## 1. Week 1 Topic Checklist

Mark each: ✅ confident · ⚠️ needs review · ❌ re-learn

### JavaScript

| # | Topic | Day | Status |
|---|-------|-----|--------|
| 1 | Closures & lexical scope | 1, 6 | ⬜ |
| 2 | Currying & partial application | 1 | ⬜ |
| 3 | Event loop, call stack, micro/macrotask | 1, 4 | ⬜ |
| 4 | `this`, call/apply/bind | 1 | ⬜ |
| 5 | Prototypes & inheritance | 2 | ⬜ |
| 6 | Class vs Object.create | 2 | ⬜ |
| 7 | Promises & chaining | 3, 6 | ⬜ |
| 8 | async/await | 3 | ⬜ |
| 9 | Implement Promise.all | 6 | ⬜ |
| 10 | Debounce & throttle | 5 | ⬜ |

### TypeScript

| # | Topic | Day | Status |
|---|-------|-----|--------|
| 1 | Basic types, interface vs type | 1 | ⬜ |
| 2 | Union & intersection types | 2 | ⬜ |
| 3 | Type guards (typeof, in, is) | 2 | ⬜ |
| 4 | Generics basics | 3, 6 | ⬜ |
| 5 | Mapped types & keyof | 4 | ⬜ |
| 6 | Conditional types & infer | 5 | ⬜ |
| 7 | Utility types (Partial, Pick, Omit) | 1, 4 | ⬜ |

### Machine Coding (React)

| # | Component | Day | Status |
|---|-----------|-----|--------|
| 1 | useStack undo/redo | 1 | ⬜ |
| 2 | Queue visualizer (useReducer) | 2 | ⬜ |
| 3 | useQueue + sequential API | 3 | ⬜ |
| 4 | Call stack visualizer | 4 | ⬜ |
| 5 | Debounced search + abort | 5 | ⬜ |
| 6 | Task manager + sync queue | 6 | ⬜ |

### Machine Coding (React Native)

| # | Component | Day | Status |
|---|-----------|-----|--------|
| 1 | Stack Navigator + params | 1 | ⬜ |
| 2 | FlatList queued animations | 2 | ⬜ |
| 3 | Animated progress queue | 3 | ⬜ |
| 4 | Custom card transitions | 4 | ⬜ |
| 5 | Throttled infinite scroll | 5 | ⬜ |
| 6 | Deep link queue handler | 6 | ⬜ |

### LeetCode (Stacks & Queues)

| # | Problem | Day | Status |
|---|---------|-----|--------|
| 150 | Evaluate RPN | 1 | ⬜ |
| 394 | Decode String | 1 | ⬜ |
| 32 | Longest Valid Parentheses | 1 | ⬜ |
| 739 | Daily Temperatures | 2 | ⬜ |
| 735 | Asteroid Collision | 2 | ⬜ |
| 239 | Sliding Window Maximum | 2 | ⬜ |
| 402 | Remove K Digits | 3 | ⬜ |
| 933 | Number of Recent Calls | 3 | ⬜ |
| 85 | Maximal Rectangle | 3 | ⬜ |
| 71 | Simplify Path | 4 | ⬜ |
| 456 | 132 Pattern | 4 | ⬜ |
| 224 | Basic Calculator | 4 | ⬜ |
| 901 | Online Stock Span | 5 | ⬜ |
| 946 | Validate Stack Sequences | 5 | ⬜ |
| 407 | Trapping Rain Water II | 5 | ⬜ |
| 84 | Largest Rectangle in Histogram | 6 | ⬜ |
| 316 | Remove Duplicate Letters | 6 | ⬜ |
| 42 | Trapping Rain Water | 6 | ⬜ |

---

## 2. JavaScript Self-Assessment

Answer each in **2 minutes or less** (oral or written). Score: ✅ without hesitation · ⚠️ with hints · ❌ couldn't answer.

### Closures & Scope

1. What is a closure? Give a real-world use case.
2. Why does `var` in a loop with setTimeout print the same value?
3. Implement a function `once(fn)` that runs fn only once.

```js
// Expected:
const init = once(() => console.log("init"));
init(); init(); init(); // "init" once
```

### Event Loop

4. Predict output:

```js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
```

5. Why is `setTimeout(fn, 0)` not immediate?

6. What is the difference between microtask and macrotask?

### Promises & async

7. Implement `Promise.all` from scratch (signature + core logic).

8. When does `Promise.all` reject vs `Promise.allSettled`?

9. Rewrite this with async/await:

```js
fetchUser(id)
  .then((user) => fetchPosts(user.id))
  .then(render)
  .catch(console.error);
```

### Prototypes & OOP

10. Explain prototype chain lookup for `obj.toString()`.

11. What does `Object.create(proto)` do vs `new Constructor()`?

### Debounce & Throttle

12. Search input: debounce or throttle? Why?

13. Scroll position tracker: debounce or throttle? Why?

14. Write a debounce function with `cancel()` method.

---

## 3. TypeScript Self-Assessment

### Types & Guards

1. Difference between `interface` and `type`?

2. When to use `unknown` vs `any`?

3. Write a discriminated union for API response (success | error) and handle exhaustively.

```ts
type ApiResult<T> =
  | { ok: true; data: T }
  | { ok: false; error: string };

function handle<T>(result: ApiResult<T>): T { /* implement */ }
```

### Generics

4. Write a generic `getProperty<T, K extends keyof T>(obj: T, key: K): T[K]`.

5. What does `T extends { length: number }` constrain?

### Advanced

6. Implement `ReturnType<F>` using conditional types and `infer`.

```ts
type MyReturnType<T> = /* your impl */;
type R = MyReturnType<() => string>; // string
```

7. What is `[K in keyof T]: T[K]`?

8. Explain `Extract` and `Exclude` utility types.

---

## 4. Concept Integration Scenarios

Explain approach (no full code required) for each:

### Scenario A — Autocomplete Search

> User types in search box. Show suggestions from API. Handle rapid typing, cancel stale requests, show loading state.

**Topics tested:** debounce, AbortController, Promises, React hooks, TypeScript generics for results

**Checklist for your answer:**
- [ ] Mentioned debounce delay (300ms typical)
- [ ] AbortController or request ID for stale cancellation
- [ ] Loading / error / empty states
- [ ] Cleanup on unmount

### Scenario B — Undo/Redo Editor

> Text editor with undo/redo. Multiple edits. What happens when user undoes then types new text?

**Topics tested:** closures, immutable state, past/present/future stacks

**Checklist:**
- [ ] past + present + future model
- [ ] Clear future on new edit after undo
- [ ] canUndo / canRedo flags

### Scenario C — Offline-First Todo App

> Tasks sync to server when online. Queue mutations while offline.

**Topics tested:** queue, Promises sequential, online/offline events, optimistic UI

**Checklist:**
- [ ] Ref-based sync queue
- [ ] Process on `online` event
- [ ] Optimistic local update first

---

## 5. Oral Interview Drill (15 min)

Set timer. Answer aloud — simulates phone screen.

| Min | Question |
|-----|----------|
| 0–3 | "Explain the JavaScript event loop." |
| 3–6 | "What is a closure? How is it used in React hooks?" |
| 6–9 | "Difference between debounce and throttle with examples." |
| 9–12 | "Explain TypeScript generics. Why not just use any?" |
| 12–15 | "How does prototype inheritance work? Is JS class-based?" |

**Self-score:** ___ / 5 topics at ⚠️ or above

---

## 6. Gap Analysis Worksheet

Fill in after completing checklist and self-assessment:

### Top 3 weak JavaScript topics

| Topic | Revisit | Action |
|-------|---------|--------|
| 1. | Day __ | Re-read section: ______ |
| 2. | Day __ | Re-do problem: ______ |
| 3. | Day __ | Implement from scratch: ______ |

### Top 3 weak TypeScript topics

| Topic | Revisit | Action |
|-------|---------|--------|
| 1. | Day __ | |
| 2. | Day __ | |
| 3. | Day __ | |

### LeetCode gaps (couldn't solve in 30 min)

| Problem | Pattern to review | Retry date |
|---------|-------------------|------------|
| | | |
| | | |
| | | |

### Machine coding gaps

| Component | Blocker | Retry |
|-----------|---------|-------|
| | | |
| | | |

---

## 7. Week 2 Preview

Next week theme: **Sliding Window** (Days 8–14)

| Upcoming JS/TS | Upcoming React/RN |
|----------------|-------------------|
| WeakMap/WeakSet | useMemo/useCallback deep dive |
| Symbol, iterators | Virtual list / windowing |
| Optional chaining | Image lazy loading |
| Template literal types | Gesture handler patterns |

**Prep suggestion:** Ensure Week 1 stack/queue patterns are solid — sliding window builds on similar "window boundary" thinking.

---

## Day 7 Assessment Scorecard

| Section | Score | Target |
|---------|-------|--------|
| JS self-assessment (/14) | __ / 14 | ≥ 11 |
| TS self-assessment (/8) | __ / 8 | ≥ 6 |
| Integration scenarios (/3) | __ / 3 | 3 |
| Oral drill (/5) | __ / 5 | ≥ 4 |
| LeetCode solved this week | __ / 18 | ≥ 12 |
| Machine coding built | __ / 6 | ≥ 4 |

**Overall readiness:**
- **≥ 85%** → Ready for Week 2
- **70–84%** → Spend Monday revisiting ⚠️ items
- **< 70%** → Re-do Week 1 Days 1–5 before continuing

---

*End of Day 7 assessment — honest self-scoring beats false confidence*
