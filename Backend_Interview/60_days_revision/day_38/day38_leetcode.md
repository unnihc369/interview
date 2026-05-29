# Day 38 — HashMap Design & 2D Fenwick Tree

**Topics:** Encode TinyURL · Insert Delete GetRandom O(1) Duplicates · Range Sum 2D Mutable

---

# 1. Encode and Decode TinyURL

## Problem Statement

Design URL shortener: `encode(longUrl)` → short string; `decode(shortUrl)` → original URL.

---

# Optimized Approach

Bijective map: `longUrl ↔ code` using counter + base62 encoding, or hash with collision handling.

```java
class Codec {
    private final Map<String, String> encodeMap = new HashMap<>();
    private final Map<String, String> decodeMap = new HashMap<>();
    private int counter = 0;
    private static final String BASE = "http://tinyurl.com/";

    public String encode(String longUrl) {
        if (decodeMap.containsKey(longUrl)) return BASE + decodeMap.get(longUrl);
        String code = toBase62(counter++);
        decodeMap.put(longUrl, code);
        encodeMap.put(code, longUrl);
        return BASE + code;
    }

    public String decode(String shortUrl) {
        String code = shortUrl.replace(BASE, "");
        return encodeMap.get(code);
    }

    private String toBase62(int n) {
        char[] chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
        StringBuilder sb = new StringBuilder();
        do {
            sb.append(chars[n % 62]);
            n /= 62;
        } while (n > 0);
        return sb.toString();
    }
}
```

Interview extension: distributed ID (Snowflake), DB persistence, cache.

---

# 2. Insert Delete GetRandom O(1) — Duplicates Allowed

## Problem Statement

Support `insert`, `remove`, `getRandom` — all average O(1); duplicates allowed.

---

# Optimized Approach

- `List<Integer> list` — for O(1) random index
- `Map<Integer, List<Integer>>` — value → list of indices in `list`
- On remove: swap removed with last, update index maps

```java
class RandomizedCollection {
    private final List<int[]> list = new ArrayList<>(); // [val, idxInMap]
    private final Map<Integer, List<Integer>> idxMap = new HashMap<>();
    private final Random rand = new Random();

    public boolean insert(int val) {
        list.add(new int[]{val, idxMap.getOrDefault(val, new ArrayList<>()).size()});
        idxMap.computeIfAbsent(val, k -> new ArrayList<>()).add(list.size() - 1);
        return idxMap.get(val).size() == 1;
    }

    public boolean remove(int val) {
        if (!idxMap.containsKey(val) || idxMap.get(val).isEmpty()) return false;
        List<Integer> indices = idxMap.get(val);
        int removeIdx = indices.remove(indices.size() - 1);
        int[] last = list.get(list.size() - 1);
        list.set(removeIdx, last);
        idxMap.get(last[0]).set(last[1], removeIdx);
        list.remove(list.size() - 1);
        return true;
    }

    public int getRandom() {
        return list.get(rand.nextInt(list.size()))[0];
    }
}
```

---

# 3. Range Sum Query 2D — Mutable

## Problem Statement

Given 2D matrix, `update(row, col, val)` and `sumRegion(row1, col1, row2, col2)` — both efficient.

---

# Brute Force

Recompute sum by nested loops per query.

```text
update O(1), sumRegion O(m*n)
```

---

# Optimized — 2D Fenwick Tree (BIT)

```java
class NumMatrix {
    private final int[][] bit;
    private final int[][] matrix;
    private final int m, n;

    public NumMatrix(int[][] matrix) {
        m = matrix.length;
        n = m == 0 ? 0 : matrix[0].length;
        this.matrix = new int[m][n];
        bit = new int[m + 1][n + 1];
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                update(i, j, matrix[i][j]);
    }

    public void update(int row, int col, int val) {
        int delta = val - matrix[row][col];
        matrix[row][col] = val;
        for (int i = row + 1; i <= m; i += i & -i)
            for (int j = col + 1; j <= n; j += j & -j)
                bit[i][j] += delta;
    }

    public int sumRegion(int r1, int c1, int r2, int c2) {
        return prefix(r2, c2) - prefix(r1 - 1, c2) - prefix(r2, c1 - 1) + prefix(r1 - 1, c1 - 1);
    }

    private int prefix(int row, int col) {
        int sum = 0;
        for (int i = row + 1; i > 0; i -= i & -i)
            for (int j = col + 1; j > 0; j -= j & -j)
                sum += bit[i][j];
        return sum;
    }
}
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| TinyURL | HashMap + counter/base62 | O(1) |
| RandomizedCollection | List + index map swap-remove | O(1) avg |
| 2D Range Sum | 2D Fenwick or 2D prefix diff | O(log²n) update/query |

---

# Interview Questions

## TinyURL — collision handling?
Use counter-based IDs (no collision) or store collision chain if hashing.

## GetRandom duplicates — why list of indices?
One value may appear multiple times; each occurrence needs unique index in backing list.

## 2D BIT vs 2D prefix array?
Prefix array: O(mn) rebuild on update. BIT: O(log m log n) update and query.

---

*End of Day 38 LeetCode*
