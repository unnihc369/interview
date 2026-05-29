# Day 5 — JavaScript & TypeScript Revision

**Topics:** Debouncing & Throttling · Conditional Types · `infer`

---

## Table of Contents

1. [Debouncing — Foundation](#1-debouncing--foundation)
2. [Throttling — Foundation](#2-throttling--foundation)
3. [Debounce vs Throttle — When to Use](#3-debounce-vs-throttle--when-to-use)
4. [Implementations from Scratch](#4-implementations-from-scratch)
5. [TypeScript Conditional Types](#5-typescript-conditional-types)
6. [The `infer` Keyword](#6-the-infer-keyword)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Debouncing — Foundation

### Definition

**Debounce** delays execution until **no new calls** occur for a specified wait period.

> "Wait until user stops typing, then search."

```js
// Without debounce: fires on every keystroke
input.addEventListener("input", (e) => search(e.target.value));

// With debounce: fires once after 300ms pause
input.addEventListener("input", debounce((e) => search(e.target.value), 300));
```

### Use cases

| Use case | Why debounce |
|----------|--------------|
| Search autocomplete | Avoid API call per keystroke |
| Window resize handler | Recalculate layout after resize stops |
| Form validation | Validate after user finishes typing |
| Save draft | Save after editing pause |

### Leading vs trailing edge

```js
// Trailing (default): execute AFTER wait period ends
debounce(fn, 300);

// Leading: execute immediately, then ignore until wait ends
debounce(fn, 300, { leading: true, trailing: false });
```

### Visual timeline

```
Events:  |--x--x--x-------(300ms)----|
Trailing:                    ↑ execute once

Events:  |--x--x--x--x--x--x--x--|
Throttle (200ms): ↑     ↑     ↑     (at most every 200ms)
```

---

## 2. Throttling — Foundation

### Definition

**Throttle** ensures function runs **at most once** per time window, regardless of call frequency.

> "Track scroll position every 100ms, not every pixel."

```js
window.addEventListener("scroll", throttle(() => {
  updateScrollIndicator();
}, 100));
```

### Use cases

| Use case | Why throttle |
|----------|--------------|
| Scroll events | Limit handler frequency |
| Mouse move tracking | Smooth performance |
| Button double-click prevention | Max one click per 500ms |
| Infinite scroll load trigger | Check position periodically |

### Leading vs trailing throttle

```js
// Leading: fire immediately, then ignore for wait period
throttle(fn, 200, { leading: true, trailing: false });

// Trailing: fire at end of window (common for scroll)
throttle(fn, 200, { leading: false, trailing: true });
```

---

## 3. Debounce vs Throttle — When to Use

| Aspect | Debounce | Throttle |
|--------|----------|----------|
| **Trigger** | After activity **stops** | At regular **intervals** during activity |
| **Calls** | One call after burst | Multiple calls, rate-limited |
| **Example** | Search on typing pause | Scroll position update |
| **Analogy** | Elevator waits for everyone | Elevator leaves every N seconds |

### Decision tree

```
Need response DURING continuous event?  → Throttle
Need response AFTER event settles?      → Debounce
Prevent duplicate submit?               → Debounce or throttle (leading)
Real-time drag preview?                 → Throttle or requestAnimationFrame
```

### requestAnimationFrame alternative

For visual updates tied to paint cycle:

```js
let ticking = false;
window.addEventListener("scroll", () => {
  if (!ticking) {
    requestAnimationFrame(() => {
      updateUI();
      ticking = false;
    });
    ticking = true;
  }
});
```

---

## 4. Implementations from Scratch

### Debounce with cancel

```js
function debounce(fn, wait, options = {}) {
  let timeoutId = null;
  const { leading = false, trailing = true } = options;

  function debounced(...args) {
    const invoke = () => {
      timeoutId = null;
      if (trailing) fn.apply(this, args);
    };

    const callNow = leading && !timeoutId;

    clearTimeout(timeoutId);
    timeoutId = setTimeout(invoke, wait);

    if (callNow) fn.apply(this, args);
  }

  debounced.cancel = () => {
    clearTimeout(timeoutId);
    timeoutId = null;
  };

  return debounced;
}
```

### Throttle

```js
function throttle(fn, wait, options = {}) {
  let lastCall = 0;
  let timeoutId = null;
  const { leading = true, trailing = true } = options;

  function throttled(...args) {
    const now = Date.now();
    const remaining = wait - (now - lastCall);

    const invoke = () => {
      lastCall = Date.now();
      fn.apply(this, args);
    };

    if (remaining <= 0) {
      if (timeoutId) {
        clearTimeout(timeoutId);
        timeoutId = null;
      }
      if (leading) invoke();
      else lastCall = now;
    } else if (trailing && !timeoutId) {
      timeoutId = setTimeout(() => {
        invoke();
        timeoutId = null;
      }, remaining);
    }
  }

  throttled.cancel = () => {
    clearTimeout(timeoutId);
    timeoutId = null;
    lastCall = 0;
  };

  return throttled;
}
```

### TypeScript typed versions

```ts
type DebouncedFn<T extends (...args: never[]) => unknown> = ((
  ...args: Parameters<T>
) => void) & { cancel: () => void };

function debounce<T extends (...args: never[]) => unknown>(
  fn: T,
  wait: number
): DebouncedFn<T> {
  let timeoutId: ReturnType<typeof setTimeout> | null = null;

  const debounced = (...args: Parameters<T>) => {
    if (timeoutId) clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), wait);
  };

  debounced.cancel = () => {
    if (timeoutId) clearTimeout(timeoutId);
    timeoutId = null;
  };

  return debounced;
}
```

### React hook pattern

```tsx
function useDebouncedCallback<T extends (...args: never[]) => void>(
  callback: T,
  delay: number
) {
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  return useMemo(() => {
    const fn = (...args: Parameters<T>) => callbackRef.current(...args);
    return debounce(fn, delay);
  }, [delay]);
}
```

---

## 5. TypeScript Conditional Types

### Syntax: `T extends U ? X : Y`

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<"hello">;  // true
type B = IsString<42>;        // false
```

### Distributive conditional types (union)

When `T` is a union, condition distributes over each member:

```ts
type ToArray<T> = T extends unknown ? T[] : never;

type Result = ToArray<string | number>;
// string[] | number[]  (not (string | number)[])
```

### Exclude and Extract (built-in)

```ts
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;

type T1 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T2 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
```

### NonNullable

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
```

### Practical: function return type from API

```ts
type ApiResponse =
  | { success: true; data: User }
  | { success: false; error: string };

type SuccessResponse = Extract<ApiResponse, { success: true }>;
// { success: true; data: User }
```

### Nested conditionals

```ts
type TypeName<T> =
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends Function ? "function" :
  "object";
```

---

## 6. The `infer` Keyword

### Extract type inside conditional

```ts
type ReturnType<T> =
  T extends (...args: never[]) => infer R ? R : never;

type R = ReturnType<() => string>;  // string
type R2 = ReturnType<(x: number) => boolean>;  // boolean
```

### Parameters type

```ts
type Parameters<T> =
  T extends (...args: infer P) => unknown ? P : never;

type P = Parameters<(a: string, b: number) => void>;
// [string, number]
```

### infer in array element type

```ts
type Flatten<T> =
  T extends (infer U)[] ? U : T;

type E = Flatten<string[]>;  // string
type N = Flatten<number>;    // number
```

### infer in Promise unwrap

```ts
type Awaited<T> =
  T extends Promise<infer U> ? U : T;

type Data = Awaited<Promise<User>>;  // User
```

### infer with multiple candidates

```ts
type FirstArg<T> =
  T extends (first: infer F, ...rest: infer _) => unknown ? F : never;
```

### Get property type from array of objects

```ts
type ElementType<T> =
  T extends (infer E)[] ? E : never;

type Item = ElementType<User[]>;  // User
```

### Practical: unwrap React component props

```ts
type ComponentProps<T> =
  T extends React.ComponentType<infer P> ? P : never;
```

### Interview snippets

| Question | Answer |
|----------|--------|
| Debounce vs throttle | After pause vs at most once per interval |
| Leading edge debounce | Execute immediately, suppress trailing |
| Conditional type | `T extends U ? X : Y` — type-level if/else |
| infer purpose | Extract/capture type within conditional branch |
| ReturnType implementation | `infer R` in function return position |
| Distributive conditional | Union T distributes over conditional |

---

## 7. Interview Quick Index

### Debounce & Throttle

| Question | Section |
|----------|---------|
| What is debouncing? | [§1](#1-debouncing--foundation) |
| What is throttling? | [§2](#2-throttling--foundation) |
| Search input — which? | [§3](#3-debounce-vs-throttle--when-to-use) — debounce |
| Scroll handler — which? | [§3](#3-debounce-vs-throttle--when-to-use) — throttle |
| Implement debounce | [§4](#4-implementations-from-scratch) |

### TypeScript

| Question | Section |
|----------|---------|
| Conditional types | [§5](#5-typescript-conditional-types) |
| Exclude / Extract | [§5](#5-typescript-conditional-types) |
| infer keyword | [§6](#6-the-infer-keyword) |
| ReturnType with infer | [§6](#6-the-infer-keyword) |

---

## Day 5 Cheat Sheet (30-second recall)

```
Debounce  → wait for pause; then fire once (search)
Throttle  → fire at most once per interval (scroll)
cancel()  → clear pending timeout on both
Conditional → T extends U ? X : Y
infer     → capture type inside conditional (ReturnType, Parameters)
Leading   → execute at start of window
Trailing  → execute at end of window (debounce default)
Security  → never trust user input in search URL; sanitize display — see security supplement
```

**Related supplements:** [Networking](../supplements/sde1_frontend_networking.md) (AbortController) · [Security](../supplements/sde1_frontend_security.md)

---

*End of Day 5 revision*
