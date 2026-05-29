# Day 17 — LeetCode (Two Pointers)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 611 | [Valid Triangle Number](#1-valid-triangle-number-611) | Medium | Sort + two pointers |
| 881 | [Boats to Save People](#2-boats-to-save-people-881) | Medium | Greedy two pointers |
| 1498 | [Number of Subsequences With Sum >= Target](#3-number-of-subsequences-with-sum--target-1498) | Medium | Sort + two pointers |

---

## Table of Contents

1. [Valid Triangle Number (611)](#1-valid-triangle-number-611)
2. [Boats to Save People (881)](#2-boats-to-save-people-881)
3. [Number of Subsequences With Sum >= Target (1498)](#3-number-of-subsequences-with-sum--target-1498)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Valid Triangle Number (611)

### Problem (short)

Given array `nums`, return count of **triplets** `(i, j, k)` that can form a triangle with positive area.

Triangle inequality: `a + b > c` for sides sorted `a ≤ b ≤ c`.

```
Input: nums = [2,2,3,4]
Output: 3  → (2,3,4), (2,3,4), (2,2,3)
```

### Hint 1 — Fix largest side

Sort ascending. Fix `k` as largest side, use two pointers `i` and `j` on smaller sides.

### Hint 2 — When valid, count all pairs

If `nums[i] + nums[j] > nums[k]`, then **all** pairs between `i` and `j-1` with `j` work → add `j - i` to count, then `j--`.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
function triangleNumber(nums) {
  nums.sort((a, b) => a - b);
  let count = 0;
  const n = nums.length;

  for (let k = n - 1; k >= 2; k--) {
    let i = 0, j = k - 1;
    while (i < j) {
      if (nums[i] + nums[j] > nums[k]) {
        count += j - i;
        j--;
      } else {
        i++;
      }
    }
  }
  return count;
}
```

| Time | Space |
|------|-------|
| O(n²) | O(1) |

---

## 2. Boats to Save People (881)

### Problem (short)

Each boat carries at most **2** people with weight limit `limit`. Minimum boats to carry everyone?

```
Input: people = [1,2], limit = 3
Output: 1
```

### Hint 1 — Sort + greedy

Pair lightest with heaviest when possible.

### Hint 2 — Two pointers

If `people[l] + people[r] <= limit`, both fit → `l++`, `r--`, boats++.  
Else heaviest alone → `r--`, boats++.

### Solution — JavaScript

```js
/**
 * @param {number[]} people
 * @param {number} limit
 * @return {number}
 */
function numRescueBoats(people, limit) {
  people.sort((a, b) => a - b);
  let l = 0, r = people.length - 1, boats = 0;

  while (l <= r) {
    if (people[l] + people[r] <= limit) l++;
    r--;
    boats++;
  }
  return boats;
}
```

| Time | Space |
|------|-------|
| O(n log n) | O(1) |

### Why greedy works

Lightest leftover person is best partner for heaviest — minimizes wasted capacity.

---

## 3. Number of Subsequences With Sum >= Target (1498)

### Problem (short)

Return number of **non-empty subsequences** of `nums` whose sum is **≥ target**. Answer mod `10^9 + 7`.

```
Input: nums = [2,3,3,4,6,7], target = 7
Output: 6
```

### Hint 1 — Sort enables two pointers

Sort `nums`. For each `left`, find smallest `right` such that subarray sum ≥ target — but problem is **subsequence**, not subarray.

### Hint 2 — Sort + two pointers on subsequence pairs

After sort, for fixed `i`, find max `j` where `nums[i] + nums[j] >= target` (i < j). Count `n - j` choices for extending? Classic approach uses two pointers counting pairs then powers of 2.

### Standard approach — sort + two pointers count pairs

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
function numSubseq(nums, target) {
  const MOD = 1e9 + 7;
  nums.sort((a, b) => a - b);
  const n = nums.length;
  const pow2 = new Array(n).fill(1);
  for (let i = 1; i < n; i++) pow2[i] = (pow2[i - 1] * 2) % MOD;

  let ans = 0, l = 0, r = n - 1;

  while (l <= r) {
    if (nums[l] + nums[r] >= target) {
      ans = (ans + pow2[r - l]) % MOD;
      r--;
    } else {
      l++;
    }
  }
  return ans;
}
```

**Intuition:** When `nums[l] + nums[r] >= target`, any subsequence picking `l` and any subset of `{l+1..r-1}` plus `r` works → `2^(r-l)` ways.

| Time | Space |
|------|-------|
| O(n log n) | O(n) |

---

## 4. Pattern Cheat Sheet

| Problem | Pointers | Key move |
|---------|----------|----------|
| **611** | Fix k, i/j inward | `sum > k` → count j-i, j-- |
| **881** | Light/heavy ends | Pair or solo heaviest |
| **1498** | l/r after sort | Pair sum ≥ target → add 2^(r-l) |

### When you see…

| Signal | Think |
|--------|-------|
| Count triplets with inequality | Sort + fix largest + two pointers |
| Pair people / minimize groups | Greedy sort + opposite ends |
| Subsequence sum threshold | Sort + two pointers + powers of 2 |

### Follow-up questions (interview)

| Question | Answer |
|----------|--------|
| 611 without sort? | O(n³) triple loop — sort enables O(n²) |
| 881 why sort ascending? | Greedy pairs lightest with heaviest |
| 1498 why `2^(r-l)`? | Pick l plus any subset of (l+1..r-1) and r |

---

*End of Day 17 LeetCode*
