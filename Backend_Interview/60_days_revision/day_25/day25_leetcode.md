# Day 25 - Dynamic Programming: String Matching

---

# 1. Regular Expression Matching

## Problem Statement

Given string `s` and pattern `p`, support `.` (any single char) and `*` (zero or more of preceding element). Return if `s` matches `p`.

---

# Example

```text
s = "aa", p = "a"     → false
s = "aa", p = "a*"    → true
s = "ab", p = ".*"    → true
```

---

# Core Idea — 2D DP

`dp[i][j]` = does `s[0..i-1]` match `p[0..j-1]`?

Transitions:

- If `p[j-1]` is letter or `.`: match if chars equal (or `.`) and `dp[i-1][j-1]`
- If `p[j-1]` is `*`: either skip `x*` (`dp[i][j-2]`) or use one char if match (`dp[i-1][j]`)

---

# Java Solution

```java
class Solution {
    public boolean isMatch(String s, String p) {
        int m = s.length(), n = p.length();
        boolean[][] dp = new boolean[m + 1][n + 1];
        dp[0][0] = true;

        for (int j = 2; j <= n; j += 2) {
            if (p.charAt(j - 1) == '*') dp[0][j] = dp[0][j - 2];
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                char sc = s.charAt(i - 1);
                char pc = p.charAt(j - 1);

                if (pc == '*') {
                    char prev = p.charAt(j - 2);
                    dp[i][j] = dp[i][j - 2]; // zero occurrences
                    if (prev == sc || prev == '.') {
                        dp[i][j] |= dp[i - 1][j];
                    }
                } else if (pc == sc || pc == '.') {
                    dp[i][j] = dp[i - 1][j - 1];
                }
            }
        }
        return dp[m][n];
    }
}
```

---

# Time & Space

```text
Time:  O(m * n)
Space: O(m * n) — can optimize to O(n)
```

---

# 2. Wildcard Matching

## Problem Statement

Given `s` and pattern `p` with `?` (any single char) and `*` (any sequence including empty). Return if match.

---

# Example

```text
s = "aa", p = "a"   → false
s = "aa", p = "*"   → true
s = "cb", p = "?a"  → false
```

---

# Core Idea — DP (Similar but `*` differs)

`dp[i][j]` = match of prefixes.

For `*`:

- Match empty: `dp[i][j-1]`
- Match one+ chars: if `*` used, `dp[i-1][j]`

For `?`: `dp[i-1][j-1]`

---

# Java Solution

```java
class Solution {
    public boolean isMatch(String s, String p) {
        int m = s.length(), n = p.length();
        boolean[][] dp = new boolean[m + 1][n + 1];
        dp[0][0] = true;

        for (int j = 1; j <= n; j++) {
            if (p.charAt(j - 1) == '*') dp[0][j] = dp[0][j - 1];
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                char pc = p.charAt(j - 1);
                if (pc == '*') {
                    dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
                } else if (pc == '?' || pc == s.charAt(i - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                }
            }
        }
        return dp[m][n];
    }
}
```

---

# Regex vs Wildcard (Interview)

| | Regex (LeetCode 10) | Wildcard (LeetCode 44) |
|---|---------------------|------------------------|
| `*` | Zero or more of **preceding** char | Any **sequence** |
| `.` | Any single char | — |
| `?` | — | Any single char |

---

# 3. Edit Distance (Levenshtein)

## Problem Statement

Minimum operations (insert, delete, replace) to convert word `a` to word `b`.

---

# Example

```text
word1 = "horse", word2 = "ros"
Output: 3
(horse → rorse → rose → ros)
```

---

# Core Idea — 2D DP

`dp[i][j]` = min edits for `a[0..i-1]` → `b[0..j-1]`

```text
If a[i-1] == b[j-1]: dp[i][j] = dp[i-1][j-1]
Else: dp[i][j] = 1 + min(insert, delete, replace)
       = 1 + min(dp[i][j-1], dp[i-1][j], dp[i-1][j-1])
```

Base: `dp[i][0]=i`, `dp[0][j]=j`

---

# Java Solution

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 0; i <= m; i++) dp[i][0] = i;
        for (int j = 0; j <= n; j++) dp[0][j] = j;

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = 1 + Math.min(
                        dp[i - 1][j - 1], // replace
                        Math.min(dp[i - 1][j], dp[i][j - 1]) // delete, insert
                    );
                }
            }
        }
        return dp[m][n];
    }
}
```

---

# Space Optimization

Only previous row needed → O(n) space.

---

# Applications (Interview)

- Spell check / fuzzy search
- DNA sequence alignment
- Diff algorithms (git diff conceptually related)

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Regex Matching | 2D DP with `*` rules | O(mn) |
| Wildcard Matching | 2D DP with `*` as sequence | O(mn) |
| Edit Distance | 2D DP min operations | O(mn) |

---

# Common Interview Questions

## Q1. Why handle `a*` base case in regex DP?

Empty string can match patterns like `a*b*c*` — pre-fill row 0 for `x*` pairs.

## Q2. Greedy for regex?

Doesn't work for nested `*` — DP required.

## Q3. Edit distance operations meaning?

Insert = dp[i][j-1], Delete = dp[i-1][j], Replace = dp[i-1][j-1].

## Q4. One-row optimization for edit distance?

Keep `prev[j]` and `curr[j]` rolling arrays.

---

# One-Line Revision

```text
String DP: dp[i][j] on prefixes; regex * = skip or repeat; wildcard * = empty or extend; edit = min of 3 ops.
```

---

*End of Day 25 LeetCode*
