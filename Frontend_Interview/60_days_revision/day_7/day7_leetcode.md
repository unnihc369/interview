# Day 7 — LeetCode Mock Contest (Week 1 Assessment)

**Format:** Timed mock · 2 Medium + 1 Hard · 105 minutes total  
**Problems:** [946 Validate Stack Sequences](#1-946-validate-stack-sequences) · [739 Daily Temperatures](#2-739-daily-temperatures) · [224 Basic Calculator](#3-224-basic-calculator)

---

## Table of Contents

1. [Contest Rules & Setup](#1-contest-rules--setup)
2. [946 Validate Stack Sequences](#2-946-validate-stack-sequences)
3. [739 Daily Temperatures](#3-739-daily-temperatures)
4. [224 Basic Calculator](#4-224-basic-calculator)
5. [Scoring Rubric](#5-scoring-rubric)
6. [Post-Contest Review Template](#6-post-contest-review-template)
7. [Pattern Cheat Sheet — Week 1](#7-pattern-cheat-sheet--week-1)

---

## 1. Contest Rules & Setup

### Timing (strict)

| Phase | Duration | Activity |
|-------|----------|----------|
| **Read all 3** | 5 min | Skim problems; pick order |
| **Problem 1 — 946** | 25 min | Medium — warm up |
| **Problem 2 — 739** | 30 min | Medium — core pattern |
| **Problem 3 — 224** | 40 min | Hard — capstone |
| **Buffer** | 5 min | Test edge cases |
| **Total** | **105 min** | No pauses, no hints |

### Environment

```
✓ Timer visible (phone or browser)
✓ Blank editor — no autocomplete from prior solutions
✓ Allow: pen + paper for traces
✗ No looking at day_N_leetcode.md files
✗ No AI assistance
✗ No running untimed "just one more try"
```

### Suggested order

1. **946** — simplest simulation; build confidence
2. **739** — monotonic stack (Week 1 core)
3. **224** — hardest; attempt even if time short

### Before you start

- [ ] Timer set to 105:00
- [ ] Editor open (LeetCode or local)
- [ ] JavaScript or TypeScript chosen — stick to one language
- [ ] Paper ready for stack traces

---

## 2. 946 Validate Stack Sequences

### Problem statement (contest version)

Given two integer arrays `pushed` and `popped` with **distinct** values, return `true` if `popped` could be the result of a stack push/pop sequence using `pushed`, else `false`.

**Examples**

```
Input:  pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
Output: true

Input:  pushed = [1,2,3,4,5], popped = [4,3,5,1,2]
Output: false
```

**Constraints:** `1 <= pushed.length <= 1000`

### Contest hints (only if stuck after 10 min)

<details>
<summary>Hint 1 — Simulation</summary>

Use a stack. Push each element from `pushed`. After each push, try popping while stack top equals next element in `popped`.
</details>

<details>
<summary>Hint 2 — Pointer j</summary>

Maintain index `j` into `popped`. Increment `j` on each successful pop.
</details>

### Your solution space (fill during contest)

```
Approach:
Time:
Space:
Edge cases tested:
```

### Reference solution (check AFTER contest)

```js
function validateStackSequences(pushed, popped) {
  const stack = [];
  let j = 0;
  for (const x of pushed) {
    stack.push(x);
    while (stack.length && stack[stack.length - 1] === popped[j]) {
      stack.pop();
      j++;
    }
  }
  return stack.length === 0;
}
```

```ts
function validateStackSequences(pushed: number[], popped: number[]): boolean {
  const stack: number[] = [];
  let j = 0;
  for (const x of pushed) {
    stack.push(x);
    while (stack.length > 0 && stack[stack.length - 1] === popped[j]) {
      stack.pop();
      j++;
    }
  }
  return stack.length === 0;
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) |

---

## 3. 739 Daily Temperatures

### Problem statement (contest version)

Given array `temperatures` where `temperatures[i]` is daily temperature, return array `answer` where `answer[i]` is the number of days to wait for a **warmer** temperature after day `i`. If none, `answer[i] = 0`.

**Example**

```
Input:  [73,74,75,71,69,72,76,73]
Output: [1,1,4,2,1,1,0,0]
```

**Constraints:** `1 <= temperatures.length <= 10^5`

### Contest hints (only if stuck after 12 min)

<details>
<summary>Hint 1 — Next greater element</summary>

For each day, find next index with higher temperature.
</details>

<details>
<summary>Hint 2 — Monotonic stack</summary>

Stack of **indices** in decreasing temperature order. When current temp > stack top temp, pop and set answer.
</details>

### Your solution space

```
Approach:
Time:
Space:
Edge cases tested:
```

### Reference solution (check AFTER contest)

```js
function dailyTemperatures(temperatures) {
  const n = temperatures.length;
  const answer = new Array(n).fill(0);
  const stack = [];
  for (let i = 0; i < n; i++) {
    while (stack.length && temperatures[i] > temperatures[stack.at(-1)]) {
      const idx = stack.pop();
      answer[idx] = i - idx;
    }
    stack.push(i);
  }
  return answer;
}
```

```ts
function dailyTemperatures(temperatures: number[]): number[] {
  const n = temperatures.length;
  const answer = new Array<number>(n).fill(0);
  const stack: number[] = [];
  for (let i = 0; i < n; i++) {
    while (stack.length > 0 && temperatures[i] > temperatures[stack[stack.length - 1]]) {
      const idx = stack.pop()!;
      answer[idx] = i - idx;
    }
    stack.push(i);
  }
  return answer;
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) |

---

## 4. 224 Basic Calculator

### Problem statement (contest version)

Given string `s` representing a valid expression with integers, `'+'`, `'-'`, `'('`, `')'`, and spaces, implement a basic calculator and return the result.

**Examples**

```
Input:  "1 + 1"           → 2
Input:  " 2-1 + 2 "       → 3
Input:  "(1+(4+5+2)-3)+(6+8)" → 23
```

**Constraints:** `1 <= s.length <= 3 * 10^5`

### Contest hints (only if stuck after 15 min)

<details>
<summary>Hint 1 — Stack on '('</summary>

Push current result and sign before entering parenthesis. Reset result to 0.
</details>

<details>
<summary>Hint 2 — On ')'</summary>

Combine: result = result * sign + popped previous result.
</details>

<details>
<summary>Hint 3 — Track sign and num</summary>

On +/- : apply result += sign * num, reset num, update sign.
</details>

### Your solution space

```
Approach:
Time:
Space:
Edge cases tested:
```

### Reference solution (check AFTER contest)

```js
function calculate(s) {
  const stack = [];
  let result = 0, sign = 1, num = 0;

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

```ts
function calculate(s: string): number {
  const stack: number[] = [];
  let result = 0, sign = 1, num = 0;

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
      stack.push(result, sign);
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

| Time | Space |
|------|-------|
| O(n) | O(n) |

---

## 5. Scoring Rubric

### Per problem

| Score | Criteria |
|-------|----------|
| **3** | Correct solution, optimal complexity, no hints, within time |
| **2** | Correct with hints OR slightly over time OR suboptimal but passes |
| **1** | Partial / wrong on edge cases / significant hints |
| **0** | Could not solve |

### Contest grades

| Total (/9) | Grade | Action |
|------------|-------|--------|
| 8–9 | **A** | Week 1 LeetCode solid — proceed to Week 2 |
| 6–7 | **B** | Re-do missed problems untimed; trace stack by hand |
| 4–5 | **C** | Revisit day 2–5 leetcode files; redo 3 problems/day |
| 0–3 | **D** | Repeat Week 1 LeetCode track before Week 2 |

### Time benchmarks

| Problem | Target | Acceptable | Slow |
|---------|--------|------------|------|
| 946 | ≤ 15 min | ≤ 25 min | > 25 min |
| 739 | ≤ 20 min | ≤ 30 min | > 30 min |
| 224 | ≤ 30 min | ≤ 40 min | > 40 min |

---

## 6. Post-Contest Review Template

Fill immediately after contest (while memory is fresh):

```
Date: ___________
Total time used: ___ / 105 min

946 Validate Stack Sequences
  Solved: Y / N
  Time: ___ min
  Hints used: ___
  Bugs encountered: ___
  Pattern recognized: stack simulation / Y N

739 Daily Temperatures
  Solved: Y / N
  Time: ___ min
  Hints used: ___
  Monotonic stack: confident / shaky / no
  Off-by-one errors: ___

224 Basic Calculator
  Solved: Y / N
  Time: ___ min
  Hints used: ___
  Parenthesis handling: confident / shaky / no

Total score: ___ / 9

Top mistake pattern: _______________
Tomorrow's focus: _______________
```

### Review protocol (30 min after contest)

1. Compare your code to reference — don't just read, **re-type** missed solutions
2. Draw stack state for one failing test case
3. Explain each solution aloud in 60 seconds
4. Add one line to personal "pattern notebook"

---

## 7. Pattern Cheat Sheet — Week 1

| Pattern | Problems | Template |
|---------|----------|----------|
| **Stack simulation** | 946, 735, 150 | Push/pop based on rules |
| **Monotonic stack** | 739, 402, 901, 84 | Pop while condition; store indices |
| **Monotonic deque** | 239 | Front=max; evict stale |
| **Stack + parsing** | 224, 394, 71 | Push state on delimiter |
| **Queue sliding window** | 933 | Dequeue expired entries |
| **Next greater element** | 739, 496 | Decreasing stack of indices |

### If you failed 946

→ Re-read [Day 5 — 946](../day_5/day5_leetcode.md)  
→ Practice: simulate `[1,2,3]` with `popped=[2,3,1]` on paper

### If you failed 739

→ Re-read [Day 2 — 739](../day_2/day2_leetcode.md)  
→ Practice: LC 496 Next Greater Element I (easier variant)

### If you failed 224

→ Re-read [Day 4 — 224](../day_4/day4_leetcode.md)  
→ Practice: LC 227 Basic Calculator II (+, -, *, /) as stretch

---

## Contest Day Checklist

- [ ] 105-minute timer completed
- [ ] All 3 problems attempted
- [ ] Scores recorded in rubric
- [ ] Post-contest review filled
- [ ] Weak patterns identified for Monday revision
- [ ] Reference solutions reviewed **after** attempt only

---

*End of Day 7 mock contest — simulate pressure, learn from gaps*
