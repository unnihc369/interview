# Day 47 — LeetCode (Suffix & Duplicate)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 745 | [Prefix and Suffix Search](#1-prefix-and-suffix-search-745) | Hard | Padded index map |
| 1062 | [Longest Repeating Substring](#2-longest-repeating-substring-1062) | Medium | Binary search + hash |
| 1044 | [Longest Duplicate Substring](#3-longest-duplicate-substring-1044) | Hard | Binary search + rolling hash |

---

## Table of Contents

1. [Prefix and Suffix Search (745)](#1-prefix-and-suffix-search-745)
2. [Longest Repeating Substring (1062)](#2-longest-repeating-substring-1062)
3. [Longest Duplicate Substring (1044)](#3-longest-duplicate-substring-1044)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Prefix and Suffix Search (745)

### Problem (short)

`WordFilter(words)` + `f(prefix, suffix)` → index of word with given prefix AND suffix (largest index on tie).

```
WordFilter(["apple"])
f("a", "e") → 0
```

### Hint 1 — Brute index map

For each word, enumerate all prefix/suffix pairs → key `pre#suf`.

### Hint 2 — Trie alternative

Insert `{suffix}#{word}` with weight index for combined search.

### Solution — JavaScript (Map)

```js
class WordFilter {
  constructor(words) {
    this.index = new Map();
    for (let i = 0; i < words.length; i++) {
      const w = words[i];
      for (let pre = 0; pre <= w.length; pre++) {
        for (let suf = 0; suf <= w.length; suf++) {
          const key = w.slice(0, pre) + "#" + w.slice(w.length - suf);
          this.index.set(key, i);
        }
      }
    }
  }

  f(prefix, suffix) {
    const key = prefix + "#" + suffix;
    return this.index.has(key) ? this.index.get(key) : -1;
  }
}
```

| Constructor | f() | Space |
|-------------|-----|-------|
| O(n × L²) | O(1) | O(n × L²) |

---

## 2. Longest Repeating Substring (1062)

### Problem (short)

Length of longest substring appearing at least twice (overlaps allowed).

```
"s = "abbaba" → 2 ("ab" or "ba")
```

### Hint 1 — Binary search length

If length L duplicate exists, then L-1 too. Monotonic.

### Hint 2 — Rolling hash check

O(n) per check → O(n log n) total.

### Solution — JavaScript

```js
/**
 * @param {string} s
 * @return {number}
 */
function longestRepeatingSubstring(s) {
  const n = s.length;
  let lo = 0, hi = n - 1, ans = 0;

  while (lo <= hi) {
    const mid = (lo + hi) >> 1;
    if (hasDup(s, mid)) { ans = mid; lo = mid + 1; }
    else hi = mid - 1;
  }
  return ans;
}

function hasDup(s, len) {
  if (len === 0) return true;
  const set = new Set();
  for (let i = 0; i <= s.length - len; i++) {
    const sub = s.slice(i, i + len);
    if (set.has(sub)) return true;
    set.add(sub);
  }
  return false;
}
```

| Time | Space |
|------|-------|
| O(n² log n) naive / O(n log n) hash | O(n) |

---

## 3. Longest Duplicate Substring (1044)

### Problem (short)

Return **any** longest duplicated substring (appears ≥2 times).

```
s = "banana" → "ana"
```

### Hint 1 — Same binary search as 1062

Track start index when duplicate found.

### Hint 2 — Rolling hash with verification

Hash collision → verify substring.

### Solution — JavaScript

```js
/**
 * @param {string} s
 * @return {string}
 */
function longestDupSubstring(s) {
  const n = s.length;
  const nums = s.split("").map((c) => c.charCodeAt(0) - 97);
  let lo = 1, hi = n - 1;
  let start = 0;

  while (lo <= hi) {
    const mid = (lo + hi) >> 1;
    const idx = search(nums, mid);
    if (idx !== -1) { start = idx; lo = mid + 1; }
    else hi = mid - 1;
  }
  return lo > 1 ? s.slice(start, start + lo - 1) : "";
}

function search(nums, len) {
  const mod = 2n ** 61n - 1n;
  const base = 26n;
  let h = 0n, pow = 1n;
  for (let i = 0; i < len; i++) {
    h = (h * base + BigInt(nums[i])) % mod;
    pow = (pow * base) % mod;
  }
  const seen = new Map();
  seen.set(h.toString(), 0);
  for (let i = len; i < nums.length; i++) {
    h = (h - BigInt(nums[i - len]) * pow % mod + mod) % mod;
    h = (h * base + BigInt(nums[i])) % mod;
    const key = h.toString();
    if (seen.has(key)) {
      const prev = seen.get(key);
      if (nums.slice(prev, prev + len).every((v, j) => v === nums[i - len + 1 + j])) return i - len + 1;
    }
    seen.set(key, i - len + 1);
  }
  return -1;
}
```

| Time | Space |
|------|-------|
| O(n log n) | O(n) |

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | One-liner |
|---------|---------|-----------|
| **745** | prefix#suffix map | Enumerate all splits per word |
| **1062** | Binary search + dup check | Longest repeat length |
| **1044** | Binary search + rolling hash | Return duplicate string |

### Follow-ups

| Question | Answer |
|----------|--------|
| 745 optimize space? | Trie of suffix#prefix combined |
| 1062 suffix tree? | O(n) build — mention as follow-up |
| 1044 mod choice? | Large prime / 2^61-1 reduce collisions |

### Test cases

```js
const wf = new WordFilter(["apple"]);
console.assert(wf.f("a", "e") === 0);
console.assert(wf.f("b", "") === -1);

console.assert(longestRepeatingSubstring("abbaba") === 2);
console.assert(longestDupSubstring("banana") === "ana");
```

### Complexity summary

| Problem | Time | Space |
|---------|------|-------|
| 745 | O(1) query | O(n×L²) |
| 1062 | O(n log n) hash | O(n) |
| 1044 | O(n log n) | O(n) |

---

*End of Day 47 LeetCode*
