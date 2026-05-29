# Day 4 — LeetCode (Stack Simulation & Parsing)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 71 | [Simplify Path](#1-simplify-path-71) | Medium | Stack |
| 456 | [132 Pattern](#2-132-pattern-456) | Medium | Monotonic Stack |
| 224 | [Basic Calculator](#3-basic-calculator-224) | Hard | Stack + Parsing |

---

## Table of Contents

1. [Simplify Path (71)](#1-simplify-path-71)
2. [132 Pattern (456)](#2-132-pattern-456)
3. [Basic Calculator (224)](#3-basic-calculator-224)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Simplify Path (71)

### Problem (short)

Given Unix-style absolute path, simplify to canonical path.

Rules:
- `.` → ignore (current dir)
- `..` → go up one level (pop stack)
- Multiple slashes → treat as one
- Result starts with `/`

**Example**

```
Input:  "/home//foo/"
Output: "/home/foo"

Input:  "/../"
Output: "/"

Input:  "/home/user/Documents/../Pictures"
Output: "/home/user/Pictures"
```

### Hint 1 — Split and stack

Split by `/`, filter empty strings, use stack for directory names.

### Hint 2 — Rules per token

```
"" or "."  → skip
".."       → pop if stack not empty
else       → push directory name
Return "/" + stack.join("/")
```

### Walkthrough — `"/a/./b/../../c/"`

```
Tokens: ["a", ".", "b", "..", "..", "c"]
"a"  → ["a"]
"."  → skip
"b"  → ["a", "b"]
".." → pop → ["a"]
".." → pop → []
"c"  → ["c"]
Result: "/c"
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### JavaScript

```js
/**
 * @param {string} path
 * @return {string}
 */
function simplifyPath(path) {
  const stack = [];

  for (const part of path.split("/")) {
    if (part === "" || part === ".") continue;
    if (part === "..") {
      stack.pop();
    } else {
      stack.push(part);
    }
  }

  return "/" + stack.join("/");
}
```

### TypeScript

```ts
function simplifyPath(path: string): string {
  const stack: string[] = [];

  for (const part of path.split("/")) {
    if (part === "" || part === ".") continue;
    if (part === "..") {
      stack.pop();
    } else {
      stack.push(part);
    }
  }

  return "/" + stack.join("/");
}
```

### Common mistakes

- Not handling `"/../"` → should be `"/"` not empty string.
- Pushing `".."` onto stack instead of popping.
- Forgetting leading `/` in result.

---

## 2. 132 Pattern (456)

### Problem (short)

Given array `nums`, return `true` if there exists `i < j < k` such that `nums[i] < nums[k] < nums[j]` (132 pattern).

**Example**

```
Input:  nums = [3, 1, 4, 2]
Output: true   (1 < 2 < 4 → indices 1, 3, 2)

Input:  nums = [1, 2, 3, 4]
Output: false  (no decreasing middle)
```

### Hint 1 — Brute O(n³) too slow

Need O(n) approach.

### Hint 2 — Track minimum (1) and candidate for 3

Traverse right to left:
- `third` = largest value that could be the "2" in pattern (middle peak)
- When `nums[i] < third` → found 132 pattern

Use stack to maintain decreasing sequence (candidates for middle `j`).

### Hint 3 — Algorithm (right to left)

```
third = -Infinity
stack = []  // candidates for nums[j] (middle)

for i from n-1 down to 0:
  if nums[i] < third:
    return true   // nums[i] is "1", third is "2" in 132
  while stack not empty AND nums[i] > stack.top:
    third = stack.pop()  // nums[i] could be "3" for earlier j
  push nums[i]
return false
```

### Walkthrough — `[3, 1, 4, 2]`

```
i=3 (2): stack [2]
i=2 (4): 4>2 pop, third=2; stack [4]
i=1 (1): 1<2 (third) → true ✓
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### JavaScript

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
function find132pattern(nums) {
  const stack = [];
  let third = -Infinity;

  for (let i = nums.length - 1; i >= 0; i--) {
    if (nums[i] < third) return true;

    while (stack.length > 0 && nums[i] > stack[stack.length - 1]) {
      third = stack.pop();
    }
    stack.push(nums[i]);
  }
  return false;
}
```

### TypeScript

```ts
function find132pattern(nums: number[]): boolean {
  const stack: number[] = [];
  let third = -Infinity;

  for (let i = nums.length - 1; i >= 0; i--) {
    if (nums[i] < third) return true;

    while (stack.length > 0 && nums[i] > stack[stack.length - 1]) {
      third = stack.pop()!;
    }
    stack.push(nums[i]);
  }
  return false;
}
```

### Common mistakes

- Traversing left-to-right (harder); right-to-left is standard.
- Confusing which variable is "1", "3", "2" in pattern.

---

## 3. Basic Calculator (224)

### Problem (short)

Evaluate string expression with non-negative integers, `+`, `-`, `(`, `)`. No `*` or `/`. Assume valid input.

**Example**

```
Input:  "1 + 1"       → 2
Input:  " 2-1 + 2 "   → 3
Input:  "(1+(4+5+2)-3)+(6+8)" → 23
```

### Hint 1 — Stack for parentheses

When seeing `(`, push current result and sign onto stack.
When seeing `)`, pop and combine.

### Hint 2 — Track sign and current number

```
sign = +1 or -1
result = running total
num = building current number digit by digit
```

On `+` or `-` (or end): apply `result += sign * num`, reset num, update sign.

### Hint 3 — On `(`

```
stack.push(result)
stack.push(sign)
result = 0
sign = 1
```

On `)`:
```
result += sign * num; num = 0
result *= stack.pop()  // sign before paren
result += stack.pop()  // result before paren
```

### Walkthrough — `"1-(2+3)"`

```
Parse: result=0, sign=1
'1': num=1
'-': result=1, sign=-1, num=0
'(': push 1, push -1; result=0, sign=1
'2': num=2
'+': result=2, num=0
'3': num=3
')': result=2+3=5; result=5*(-1)=-5; result=-5+1=-4
Answer: -4? Wait let me recalculate 1-(2+3)=1-5=-4 ✓
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### JavaScript

```js
/**
 * @param {string} s
 * @return {number}
 */
function calculate(s) {
  const stack = [];
  let result = 0;
  let sign = 1;
  let num = 0;

  for (let i = 0; i < s.length; i++) {
    const ch = s[i];

    if (ch >= "0" && ch <= "9") {
      num = num * 10 + (ch.charCodeAt(0) - 48);
    } else if (ch === "+") {
      result += sign * num;
      num = 0;
      sign = 1;
    } else if (ch === "-") {
      result += sign * num;
      num = 0;
      sign = -1;
    } else if (ch === "(") {
      stack.push(result);
      stack.push(sign);
      result = 0;
      sign = 1;
      num = 0;
    } else if (ch === ")") {
      result += sign * num;
      num = 0;
      result *= stack.pop();
      result += stack.pop();
    }
  }

  result += sign * num;
  return result;
}
```

### TypeScript

```ts
function calculate(s: string): number {
  const stack: number[] = [];
  let result = 0;
  let sign = 1;
  let num = 0;

  for (let i = 0; i < s.length; i++) {
    const ch = s[i];

    if (ch >= "0" && ch <= "9") {
      num = num * 10 + (ch.charCodeAt(0) - 48);
    } else if (ch === "+") {
      result += sign * num;
      num = 0;
      sign = 1;
    } else if (ch === "-") {
      result += sign * num;
      num = 0;
      sign = -1;
    } else if (ch === "(") {
      stack.push(result);
      stack.push(sign);
      result = 0;
      sign = 1;
      num = 0;
    } else if (ch === ")") {
      result += sign * num;
      num = 0;
      result *= stack.pop()!;
      result += stack.pop()!;
    }
  }

  result += sign * num;
  return result;
}
```

### Common mistakes

- Forgetting final `result += sign * num` after loop.
- Wrong order of stack pop on `)`.
- Not handling spaces (skip or they're ignored in digit check).

---

## 4. Pattern Cheat Sheet

| Problem | Stack use | Trigger |
|---------|-----------|---------|
| **71 Simplify Path** | Directory names | `..` pop, else push |
| **456 132 Pattern** | Decreasing candidates | Right-to-left; third variable |
| **224 Calculator** | Result + sign | `(` push state, `)` pop combine |

### When you see…

| Signal | Think |
|--------|-------|
| Path canonicalization | Split + stack |
| Pattern i<j<k with ordering | Monotonic stack + auxiliary variable |
| Expression with parentheses | Stack of accumulated results |

### Solve order (revision)

1. **71** — Split `/`, stack push/pop rules.
2. **456** — Reverse loop; `third` tracks "2" in 132.
3. **224** — Stack on `(`; track sign, num, result.

---

*End of Day 4 LeetCode*
