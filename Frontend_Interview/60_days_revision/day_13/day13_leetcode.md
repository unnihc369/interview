# Day 13 — LeetCode (Saturday: Sliding Window Revision Guide)

**No new problems** — consolidate Week 2 patterns, templates, and problem map.

---

## Table of Contents

1. [Pattern Decision Tree](#1-pattern-decision-tree)
2. [Templates](#2-templates)
3. [Week 2 Problem Map](#3-week-2-problem-map)
4. [Common Mistakes](#4-common-mistakes)
5. [Timed Drill Plan](#5-timed-drill-plan)
6. [Self-Check Quiz](#6-self-check-quiz)

---

## 1. Pattern Decision Tree

```
Subarray / substring problem?
│
├─ Fixed size k?
│   ├─ Count / max in window → 1456, 1297, 1052
│   ├─ Anagram / permutation → 438, 567
│   └─ Median / complex → 480
│
├─ Variable size?
│   ├─ Longest with constraint → 3, 904, 1004, 487
│   ├─ Shortest with constraint → 209, 76
│   ├─ Count subarrays → 713, 992 (atMost trick)
│   └─ Sum / product threshold → 209, 713
│
└─ Exactly K distinct?
    → atMost(K) - atMost(K-1)  // 992
```

---

## 2. Templates

### Variable window — longest / max length

```js
let left = 0;
let maxLen = 0;
// maintain window state (count, sum, set, etc.)

for (let right = 0; right < n; right++) {
  // add s[right]
  while (invalid) {
    // remove s[left]
    left++;
  }
  maxLen = Math.max(maxLen, right - left + 1);
}
return maxLen;
```

### Variable window — shortest (sum ≥ target)

```js
let left = 0, sum = 0, minLen = Infinity;
for (let right = 0; right < n; right++) {
  sum += nums[right];
  while (sum >= target) {
    minLen = Math.min(minLen, right - left + 1);
    sum -= nums[left++];
  }
}
return minLen === Infinity ? 0 : minLen;
```

### Fixed window — size k

```js
for (let right = 0; right < n; right++) {
  add(s[right]);
  if (right >= k - 1) {
    updateAnswer(left, right);
    remove(s[right - k + 1]);
  }
}
```

### Count subarrays ending at right

```js
while (invalid) shrink left;
count += right - left + 1;
```

### At most K distinct

```js
function atMost(nums, k) {
  const count = new Map();
  let left = 0, total = 0;
  for (let right = 0; right < nums.length; right++) {
    // add nums[right], shrink while size > k
    total += right - left + 1;
  }
  return total;
}
// exactly K: atMost(k) - atMost(k - 1)
```

### Minimum window substring (76)

```js
// expand until have === required
// shrink while valid → update min
// track need map + have count
```

---

## 3. Week 2 Problem Map

| Day | Problems | Pattern |
|-----|----------|---------|
| 8 | 3, 904, 76 | Longest unique; 2 distinct; min window |
| 9 | 438, 567, 480 | Fixed anagram; median |
| 10 | 209, 1004, 992 | Min len sum; k flips; exactly K |
| 11 | 713, 1456, 1590 | Product count; vowels fixed; mod sum |
| 12 | 487, 1297, 1052 | 1 flip; freq substring; grumpy fixed |

### Difficulty ladder (re-solve order)

1. **209** → **3** → **904** → **1004** → **76**
2. **438** → **713** → **992** → **480**

---

## 4. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Shrink before adding right | Add first, then shrink |
| Wrong `have` increment on 76 | Only when `window[c] === need[c]` |
| `count += 1` instead of `count += right - left + 1` | Use window size formula (713, 992) |
| Forgetting `left = max(left, ...)` on 3 | Jump left past duplicate |
| Using `Math.floor` for negative div in JS | Use `Math.trunc` |
| 992 brute force | Use atMost difference |

---

## 5. Timed Drill Plan

| Block | Task | Time |
|-------|------|------|
| 1 | Write 3 templates from memory | 15 min |
| 2 | Re-solve 3, 904, 209 | 45 min |
| 3 | Re-solve 76, 438, 992 | 45 min |
| 4 | One random from map (cold) | 20 min |

**Target:** Medium in 20 min, Hard (76/480/992) in 35 min with working code.

---

## 6. Self-Check Quiz

1. Why does `atMost(K) - atMost(K-1)` give exactly K?
2. When do you use `count += right - left + 1`?
3. Difference between fixed and variable window shrink condition?
4. How is 904 related to 1004?
5. Why is 1297 only checked at `minSize`?

**Answers (skim after attempt):**

1. Exactly K = subarrays with ≤K minus ≤K-1.
2. Counting **number** of valid subarrays (713, 992 atMost).
3. Fixed: every step after k; variable: while invalid.
4. 904 is at most 2 distinct; 1004 is at most k zeros.
5. Longer valid windows contain a minSize substring with same letter count bound.

---

## Cheat Sheet Card

```
LONGEST  → expand right, shrink while bad, max len
SHORTEST → expand until good, shrink while good, min len
FIXED k  → add right, remove right-k+1 when ready
COUNT    → += right - left + 1
EXACT K  → atMost(K) - atMost(K-1)
ANAGRAM  → freq map + matches === need.size
```

---

*End of Day 13 LeetCode revision*
