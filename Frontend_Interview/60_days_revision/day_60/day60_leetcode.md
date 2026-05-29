# Day 60 — LeetCode (Final Mock Contest)

**Format:** Timed mock — 3 problems · **60 min total** · 2 Medium + 1 Hard  
**No peeking at solutions until attempt complete**

| # | Problem | Difficulty | Target time |
|---|---------|------------|-------------|
| 300 | [Longest Increasing Subsequence](#mock-1--longest-increasing-subsequence-300) | Medium | 20 min |
| 72 | [Edit Distance](#mock-2--edit-distance-72) | Medium | 20 min |
| 416 | [Partition Equal Subset Sum](#mock-3--partition-equal-subset-sum-416) | Medium | 20 min |

---

## Table of Contents

1. [Mock Rules & Scoring](#1-mock-rules--scoring)
2. [Mock 1 — LIS (300)](#mock-1--longest-increasing-subsequence-300)
3. [Mock 2 — Edit Distance (72)](#mock-2--edit-distance-72)
4. [Mock 3 — Partition Equal Subset Sum (416)](#mock-3--partition-equal-subset-sum-416)
5. [Post-Mock Review Checklist](#5-post-mock-review-checklist)
6. [Solutions (Reveal After Attempt)](#6-solutions-reveal-after-attempt)
7. [60-Day Final Revision Checklist](#7-60-day-final-revision-checklist)
8. [Pattern Cheat Sheet — All DP](#8-pattern-cheat-sheet--all-dp)

---

## 1. Mock Rules & Scoring

### Before you start

1. Timer: **60 minutes** total.
2. State DP definition before coding each problem.
3. Run mental tests on edge cases.

### Scoring rubric

| Criteria | Points (×3 problems) |
|----------|----------------------|
| Correct pattern identified | 3 |
| Working code | 3 |
| Complexity stated | 2 |
| Edge cases handled | 2 |

| Total / 30 | Grade |
|------------|-------|
| 25–30 | DP ready for interviews |
| 18–24 | Review Days 50–54 |
| < 18 | Re-read Week 8 concepts |

---

## Mock 1 — Longest Increasing Subsequence (300)

### Problem (short)

Return length of longest strictly increasing subsequence.

```
Input: nums = [10,9,2,5,3,7,101,18]
Output: 4  ([2,3,7,101] or [2,5,7,101])
```

### Hints (if stuck 5+ min)

<details>
<summary>Hint 1</summary>
O(n²): dp[i] = max LIS ending at i.
</details>

<details>
<summary>Hint 2</summary>
O(n log n): patience sorting with tails array + binary search.
</details>

### Your workspace

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
function lengthOfLIS(nums) {
  // YOUR CODE
}
```

### Tests

```js
console.assert(lengthOfLIS([10,9,2,5,3,7,101,18]) === 4);
console.assert(lengthOfLIS([0,1,0,3,2,3]) === 4);
console.assert(lengthOfLIS([7,7,7,7,7,7,7]) === 1);
console.assert(lengthOfLIS([]) === 0);
```

---

## Mock 2 — Edit Distance (72)

### Problem (short)

Minimum operations (insert, delete, replace) to convert word1 → word2.

```
Input: word1 = "horse", word2 = "ros"
Output: 3
```

### Hints

<details>
<summary>Hint 1</summary>
dp[i][j] = min ops for word1[0..i-1] → word2[0..j-1].
</details>

<details>
<summary>Hint 2</summary>
If chars match: dp[i][j] = dp[i-1][j-1]. Else 1 + min(3 neighbors).
</details>

### Your workspace

```js
/**
 * @param {string} word1
 * @param {string} word2
 * @return {number}
 */
function minDistance(word1, word2) {
  // YOUR CODE
}
```

### Tests

```js
console.assert(minDistance("horse", "ros") === 3);
console.assert(minDistance("intention", "execution") === 5);
console.assert(minDistance("", "abc") === 3);
console.assert(minDistance("abc", "abc") === 0);
```

---

## Mock 3 — Partition Equal Subset Sum (416)

### Problem (short)

Can nums be partitioned into two subsets with equal sum?

```
Input: nums = [1,5,11,5]
Output: true  ([1,5,5] and [11])
```

### Hints

<details>
<summary>Hint 1</summary>
If total odd → false. Target = total/2.
</details>

<details>
<summary>Hint 2</summary>
0/1 knapsack: dp[t] = can we make sum t?
</details>

### Your workspace

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
function canPartition(nums) {
  // YOUR CODE
}
```

### Tests

```js
console.assert(canPartition([1,5,11,5]) === true);
console.assert(canPartition([1,2,3,5]) === false);
console.assert(canPartition([2,2,3,5]) === false);
console.assert(canPartition([1,1]) === true);
```

---

## 5. Post-Mock Review Checklist

After each problem, answer:

- [ ] What is the DP state in one sentence?
- [ ] What are base cases?
- [ ] Time and space complexity?
- [ ] Can space be optimized?
- [ ] What breaks if input is empty?

---

## 6. Solutions (Reveal After Attempt)

### 300 — LIS O(n log n)

```js
function lengthOfLIS(nums) {
  if (!nums.length) return 0;
  const tails = [];
  for (const x of nums) {
    let lo = 0, hi = tails.length;
    while (lo < hi) {
      const mid = (lo + hi) >> 1;
      if (tails[mid] < x) lo = mid + 1;
      else hi = mid;
    }
    tails[lo] = x;
  }
  return tails.length;
}
```

### 72 — Edit Distance

```js
function minDistance(word1, word2) {
  const m = word1.length, n = word2.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 0; i <= m; i++) dp[i][0] = i;
  for (let j = 0; j <= n; j++) dp[0][j] = j;
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) dp[i][j] = dp[i - 1][j - 1];
      else dp[i][j] = 1 + Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]);
    }
  }
  return dp[m][n];
}
```

### 416 — Partition Equal Subset Sum

```js
function canPartition(nums) {
  const sum = nums.reduce((a, b) => a + b, 0);
  if (sum % 2) return false;
  const target = sum / 2;
  const dp = Array(target + 1).fill(false);
  dp[0] = true;
  for (const n of nums) {
    for (let t = target; t >= n; t--) {
      dp[t] = dp[t] || dp[t - n];
    }
  }
  return dp[target];
}
```

---

## 7. 60-Day Final Revision Checklist

### Week 8 DP (must know cold)

- [ ] Memo vs tabulation tradeoffs
- [ ] 0/1 vs unbounded knapsack loop direction
- [ ] LCS / edit distance 2D fill
- [ ] LIS tails + binary search
- [ ] Tree DP post-order return tuple
- [ ] Stock state machine (4 vars for k=2)
- [ ] Interval DP: burst last balloon

### Weeks 1–7 highlights

- [ ] Stack: valid parentheses
- [ ] Sliding window: longest substring k distinct
- [ ] Two pointers: 3Sum
- [ ] Heap: kth largest, merge k lists
- [ ] Tree: BFS/DFS, LCA
- [ ] BST: validate, insert
- [ ] Trie: autocomplete prefix

### Machine coding

- [ ] Controlled input + debounced search
- [ ] List virtualization concept
- [ ] RN: FlatList, SafeAreaView, keyboard avoiding

### Interview day

- [ ] 8 hours sleep
- [ ] Quiet space, whiteboard or shared editor tested
- [ ] Think aloud — silence is the enemy
- [ ] Ask clarifying questions first

---

## 8. Pattern Cheat Sheet — All DP

```
1D:     dp[i] from dp[i-1], dp[i-2]
Grid:   dp[i][j] from top + left
Knapsack 0/1:   for w high→low
Knapsack unbounded: for a low→high
LCS:    match → diag+1, else max(top,left)
Edit:   match → diag, else 1+min(3)
LIS:    tails + bisect
Tree:   post-order return [pick, skip]
Bitmask: dp[mask][node]
Stock:  buy/sell rolling states
Interval: for len, for i, for k split
```

---

**Congratulations on completing 60 days.**  
**Prev/Next:** [Concepts](day60_concepts.md) · [Machine Coding](day60_machine_coding.md)
