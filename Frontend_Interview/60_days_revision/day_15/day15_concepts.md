# Day 15 — Destructuring, Rest & Spread

**Week 3 — Two Pointers** · **Topics:** Destructuring · Rest/Spread · Two-pointer intro

---

## Table of Contents

1. [Destructuring](#1-destructuring)
2. [Rest vs Spread](#2-rest-vs-spread)
3. [Two Pointers — Opposite Ends](#3-two-pointers--opposite-ends)
4. [Interview Quick Index](#4-interview-quick-index)

---

## 1. Destructuring

### Array destructuring

```js
const [first, second, ...rest] = [10, 20, 30, 40];
// first=10, second=20, rest=[30, 40]

const [, , third] = [1, 2, 3]; // skip slots with commas

let [a, b] = [1, 2];
[a, b] = [b, a]; // swap without temp variable
```

### Object destructuring

```js
const user = { id: 1, name: "Ada", role: "admin", city: "BLR" };
const { name, role: jobTitle, ...others } = user;
// jobTitle = "admin", others = { id, city }

function greet({ name = "Guest", age = 0 }) {
  return `${name} is ${age}`;
}
```

### Nested & defaults

```js
const data = { user: { profile: { theme: "dark" } } };
const {
  user: { profile: { theme = "light" } = {} } = {},
} = data;
```

### React / hooks patterns

```js
const [count, setCount] = useState(0);
const { data, isLoading, error } = useQuery(["user", id], fetchUser);

// Props in component
function Card({ title, children, ...domProps }) {
  return <article {...domProps}>{title}{children}</article>;
}
```

### Pitfalls

| Issue | Fix |
|-------|-----|
| Destructure `undefined` | Default param `({ x } = {})` |
| Rename collision | `role: jobTitle` |
| Lose reactivity (Vue) | Destructure refs carefully; in React state is fine |

### TypeScript

```ts
const [head, ...tail]: [number, ...number[]] = [1, 2, 3];

interface User { id: number; name: string; }
function pickName({ name }: User) { return name; }

type WithoutId = Omit<User, "id">;
```

**One-liner:** Destructuring unpacks arrays/objects into variables; use defaults for safe APIs.

---

## 2. Rest vs Spread

| Operator | Where | Meaning |
|----------|-------|---------|
| **Spread** `...` | RHS, literals, function calls | Expand iterable / object |
| **Rest** `...` | LHS (last), function params | Collect remaining |

```js
// Spread — copy, merge, pass array as args
const merged = [...arr1, ...arr2];
const updated = { ...user, city: "Mumbai" };
Math.max(...nums);

// Rest — variadic + object omit
function log(level, ...messages) { }
const { password, ...safeUser } = account;
```

### Shallow copy trap

```js
const nested = { a: { b: 1 } };
const copy = { ...nested };
copy.a.b = 2; // also changes nested.a — deep clone needed for nested data
```

### Immutable updates (React)

```js
setItems((prev) => [
  ...prev.slice(0, index),
  { ...prev[index], done: true },
  ...prev.slice(index + 1),
]);
```

**One-liner:** Spread expands; rest collects — both use `...` but position decides meaning.

---

## 3. Two Pointers — Opposite Ends

**Pattern:** `left = 0`, `right = n - 1`, move inward based on comparison.

| Problem type | Move rule |
|--------------|-----------|
| Pair sum in sorted array | `sum < target` → `left++`; else `right--` |
| Palindrome | mismatch → false; else both move |
| Container water (LC 11) | move shorter height side |

```js
function maxArea(height) {
  let l = 0, r = height.length - 1, best = 0;
  while (l < r) {
    best = Math.max(best, Math.min(height[l], height[r]) * (r - l));
    if (height[l] < height[r]) l++;
    else r--;
  }
  return best;
}
```

**Why move shorter line?** Width shrinks on every step; only a taller line can increase area.

---

## 4. Interview Quick Index

| Question | Section |
|----------|---------|
| Array vs object destructuring | [§1](#1-destructuring) |
| Rest vs spread difference | [§2](#2-rest-vs-spread) |
| Swap two variables | [§1 — array](#1-destructuring) |
| Shallow copy with spread | [§2](#2-rest-vs-spread) |
| Two-pointer when sorted | [§3](#3-two-pointers--opposite-ends) |

---

## Day 15 Cheat Sheet

```
Destructuring → unpack; defaults; rename with :
Rest          → collect (...last in pattern/params)
Spread        → expand (...in array/object/call)
Two pointers  → L/R on sorted or max-area problems
```

---

*End of Day 15 concepts*
