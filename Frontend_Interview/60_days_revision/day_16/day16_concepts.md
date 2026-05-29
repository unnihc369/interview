# Day 16 — Array.prototype.every & some

**Week 3 — Two Pointers** · **Topics:** `every` · `some` · Validation patterns · Short-circuit logic

---

## Table of Contents

1. [every — All Must Pass](#1-every--all-must-pass)
2. [some — At Least One Passes](#2-some--at-least-one-passes)
3. [every vs some vs filter vs find](#3-every-vs-some-vs-filter-vs-find)
4. [Form Validation & Auth Guards](#4-form-validation--auth-guards)
5. [Two-Pointer Tie-In](#5-two-pointer-tie-in)
6. [TypeScript & React Patterns](#6-typescript--react-patterns)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. every — All Must Pass

### Definition

`Array.prototype.every(callback)` returns **`true`** if **every** element satisfies the predicate; **`false`** on the **first** failure.

```js
[2, 4, 6].every((n) => n % 2 === 0); // true
[2, 3, 6].every((n) => n % 2 === 0); // false — stops at index 1
```

### Short-circuit behavior

Like `&&` over elements: stops as soon as callback returns falsy.

```js
let calls = 0;
[1, 2, 3, 4].every((n) => {
  calls++;
  return n < 3;
});
// calls === 3 (evaluated 1, 2, 3; 3 fails)
```

### Empty array edge case

```js
[].every(() => false); // true — vacuous truth (no counterexample)
```

**Interview line:** Empty `every` is always `true`; empty `some` is always `false`.

### Signature & thisArg

```js
const scores = { min: 40 };
[50, 60, 70].every(function (s) {
  return s >= this.min;
}, scores);
```

### Real-world uses

| Use | Pattern |
|-----|---------|
| Form validity | `fields.every(f => f.error === null)` |
| Permissions | `requiredPerms.every(p => user.has(p))` |
| All items selected | `items.every(i => i.checked)` |
| Password rules | `[minLen, hasDigit, hasUpper].every(rule => rule(pw))` |

```js
function isValidPassword(pw) {
  const rules = [
    (s) => s.length >= 8,
    (s) => /[A-Z]/.test(s),
    (s) => /[0-9]/.test(s),
  ];
  return rules.every((rule) => rule(pw));
}
```

---

## 2. some — At Least One Passes

### Definition

`Array.prototype.some(callback)` returns **`true`** if **at least one** element satisfies the predicate; **`false`** if none do.

```js
[1, 3, 5].some((n) => n % 2 === 0); // false
[1, 2, 5].some((n) => n % 2 === 0); // true — stops at index 1
```

### Short-circuit

Stops on first `true` — like `||` over elements.

```js
let calls = 0;
[1, 2, 3].some((n) => {
  calls++;
  return n > 1;
});
// calls === 2
```

### Empty array

```js
[].some(() => true); // false
```

### Real-world uses

| Use | Pattern |
|-----|---------|
| Any error in form | `fields.some(f => f.error)` |
| Has admin role | `roles.some(r => r === 'admin')` |
| Overlap detection | `arrA.some(x => setB.has(x))` |
| Unsaved changes | `tabs.some(t => t.dirty)` |

```js
function hasConflict(slots) {
  return slots.some((slot, i) =>
    slots.some((other, j) => i !== j && overlaps(slot, other))
  );
}
```

---

## 3. every vs some vs filter vs find

| Method | Returns | Stops early? | When |
|--------|---------|--------------|------|
| `every` | boolean | Yes (on false) | All pass? |
| `some` | boolean | Yes (on true) | Any pass? |
| `filter` | new array | No | Collect matches |
| `find` | element / undefined | Yes | First match value |
| `findIndex` | index / -1 | Yes | First match index |
| `includes` | boolean | Yes | Primitive equality |

### De Morgan's laws (interview favorite)

```js
// NOT every === some NOT
!arr.every(pred)  === arr.some(x => !pred(x))
!arr.some(pred)   === arr.every(x => !pred(x))
```

Example: "not all adults" ↔ "some are minors"

```js
const users = [{ age: 20 }, { age: 15 }];
users.every(u => u.age >= 18);           // false
users.some(u => u.age < 18);             // true — equivalent negation
```

### Performance note

Prefer `some`/`every` over `filter().length` when you only need a boolean:

```js
// Bad — walks entire array
nums.filter(n => n > 10).length > 0;

// Good
nums.some(n => n > 10);
```

---

## 4. Form Validation & Auth Guards

### Multi-field form (React)

```tsx
type Field = { name: string; value: string; error: string | null };

function validateField(name: string, value: string): string | null {
  if (!value.trim()) return `${name} is required`;
  return null;
}

function useFormValidation(fields: Field[]) {
  const isValid = fields.every((f) => f.error === null);
  const hasErrors = fields.some((f) => f.error !== null);
  return { isValid, hasErrors };
}
```

### Route guard pattern

```js
const ROUTE_GUARDS = [
  (user) => user.isLoggedIn,
  (user) => user.emailVerified,
  (user) => !user.isBanned,
];

function canAccess(user) {
  return ROUTE_GUARDS.every((guard) => guard(user));
}
```

### Feature flags

```js
const FEATURES = { darkMode: true, betaCharts: false };

function allEnabled(keys) {
  return keys.every((k) => FEATURES[k]);
}

function anyEnabled(keys) {
  return keys.some((k) => FEATURES[k]);
}
```

---

## 5. Two-Pointer Tie-In

`every`/`some` answer **existential / universal** questions over arrays — same family as two-pointer **search** problems.

| Question type | Array method | Two-pointer analog |
|---------------|--------------|-------------------|
| Does pair sum to target exist? | `some` with nested loop | Opposite ends O(n) if sorted |
| All pairs non-decreasing? | `every` on adjacent | Single pass |
| Any duplicate? | `some` with Set | Floyd / hash |

```js
// Sorted two-sum existence — O(n)
function hasPairSum(nums, target) {
  let l = 0, r = nums.length - 1;
  while (l < r) {
    const sum = nums[l] + nums[r];
    if (sum === target) return true;
    if (sum < target) l++;
    else r--;
  }
  return false;
}

// Universal: all positive?
nums.every(n => n > 0);
```

---

## 6. TypeScript & React Patterns

### Type predicates with every

```ts
function isString(x: unknown): x is string {
  return typeof x === "string";
}

const mixed: unknown[] = ["a", "b"];
mixed.every(isString); // narrows if true in conditional
```

### Readonly checks

```ts
function allKeysPresent<T extends object>(
  obj: T,
  keys: (keyof T)[]
): boolean {
  return keys.every((k) => k in obj && obj[k] != null);
}
```

### Checkbox "select all"

```tsx
const allSelected = items.every((i) => i.selected);
const someSelected = items.some((i) => i.selected);

<input
  type="checkbox"
  checked={allSelected}
  ref={(el) => {
    if (el) el.indeterminate = someSelected && !allSelected;
  }}
  onChange={toggleAll}
/>
```

---

## 7. Interview Quick Index

| Question | Section |
|----------|---------|
| `every` on empty array? | [§1](#1-every--all-must-pass) — returns `true` |
| `some` vs `includes`? | [§3](#3-every-vs-some-vs-filter-vs-find) — callback vs `===` |
| Short-circuit? | [§1](#1-every--all-must-pass), [§2](#2-some--at-least-one-passes) |
| De Morgan with every/some | [§3](#3-every-vs-some-vs-filter-vs-find) |
| Form "all valid" | [§4](#4-form-validation--auth-guards) |
| Boolean vs filter().length | [§3](#3-every-vs-some-vs-filter-vs-find) |

---

## Day 16 Cheat Sheet

```
every(fn)  → true if ALL pass; empty → true; stops on first false
some(fn)   → true if ANY pass;  empty → false; stops on first true
!every(p)  ≡ some(x => !p(x))
!some(p)   ≡ every(x => !p(x))
Use some/every for booleans; filter when you need the list
```

---

*End of Day 16 concepts*
