# Day 53 — Machine Coding

**React:** Diff viewer text comparison  
**React Native:** Merge conflict resolver

---

## Table of Contents

### React (Web)

1. [Problem — Diff Viewer](#1-problem--diff-viewer)
2. [LCS-Based Diff Algorithm](#2-lcs-based-diff-algorithm)
3. [React Diff UI](#3-react-diff-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Merge Conflict Resolver](#5-merge-conflict-resolver)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Diff Viewer

**Task:** Side-by-side diff of two strings. Highlight **common subsequence** (unchanged), deletions (red), insertions (green).

---

## 2. LCS-Based Diff Algorithm

```js
function buildLcsTable(a, b) {
  const m = a.length, n = b.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      dp[i][j] = a[i - 1] === b[j - 1]
        ? dp[i - 1][j - 1] + 1
        : Math.max(dp[i - 1][j], dp[i][j - 1]);
    }
  }
  return dp;
}

function diffLines(oldText, newText) {
  const a = oldText.split("\n");
  const b = newText.split("\n");
  const dp = buildLcsTable(a, b);
  const result = [];
  let i = a.length, j = b.length;
  while (i > 0 || j > 0) {
    if (i > 0 && j > 0 && a[i - 1] === b[j - 1]) {
      result.unshift({ type: "same", text: a[i - 1] });
      i--; j--;
    } else if (j > 0 && (i === 0 || dp[i][j - 1] >= dp[i - 1][j])) {
      result.unshift({ type: "add", text: b[j - 1] });
      j--;
    } else {
      result.unshift({ type: "del", text: a[i - 1] });
      i--;
    }
  }
  return result;
}
```

---

## 3. React Diff UI

```tsx
import { useMemo, useState } from "react";

export default function DiffViewer() {
  const [oldText, setOldText] = useState("hello\nworld");
  const [newText, setNewText] = useState("hello\nthere\nworld");
  const diff = useMemo(() => diffLines(oldText, newText), [oldText, newText]);

  return (
    <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 16, padding: 24 }}>
      <textarea value={oldText} onChange={(e) => setOldText(e.target.value)} rows={8} />
      <textarea value={newText} onChange={(e) => setNewText(e.target.value)} rows={8} />
      <div style={{ gridColumn: "1 / -1", fontFamily: "monospace" }}>
        {diff.map((row, idx) => (
          <div
            key={idx}
            style={{
              background:
                row.type === "same" ? "#f5f5f5" : row.type === "add" ? "#e8f5e9" : "#ffebee",
            }}
          >
            {row.type === "add" ? "+ " : row.type === "del" ? "- " : "  "}
            {row.text}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## 4. React Interview Points

- LCS backtrack produces minimal edit script for line-level diff.
- Character-level diff: apply same DP on chars (Myers diff is production alternative).
- Debounce textarea updates for large files.

---

# Part B — React Native

## 5. Merge Conflict Resolver

```tsx
import { View, Text, Pressable, ScrollView, StyleSheet } from "react-native";
import { useState } from "react";

type ConflictBlock = { base: string; ours: string; theirs: string };

function resolveConflict(block: ConflictBlock, choice: "ours" | "theirs" | "merge"): string {
  if (choice === "ours") return block.ours;
  if (choice === "theirs") return block.theirs;
  // merge: LCS union heuristic — keep common lines
  const a = block.ours.split("\n");
  const b = block.theirs.split("\n");
  const dp = buildLcsTable(a, b);
  const merged: string[] = [];
  let i = a.length, j = b.length;
  while (i > 0 && j > 0) {
    if (a[i - 1] === b[j - 1]) {
      merged.unshift(a[i - 1]);
      i--; j--;
    } else if (dp[i - 1][j] >= dp[i][j - 1]) { i--; }
    else { j--; }
  }
  return merged.join("\n");
}

export default function MergeConflictScreen() {
  const block: ConflictBlock = {
    base: "const x = 1;",
    ours: "const x = 1;\nconst y = 2;",
    theirs: "const x = 1;\nconst z = 3;",
  };
  const [resolved, setResolved] = useState("");

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Merge conflict</Text>
      <Text style={styles.label}>Ours</Text>
      <Text style={styles.code}>{block.ours}</Text>
      <Text style={styles.label}>Theirs</Text>
      <Text style={styles.code}>{block.theirs}</Text>
      <View style={styles.row}>
        {(["ours", "theirs", "merge"] as const).map((c) => (
          <Pressable key={c} style={styles.btn} onPress={() => setResolved(resolveConflict(block, c))}>
            <Text>{c}</Text>
          </Pressable>
        ))}
      </View>
      {resolved ? <Text style={styles.resolved}>{resolved}</Text> : null}
    </ScrollView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  title: { fontSize: 20, fontWeight: "600" },
  label: { marginTop: 12, fontWeight: "500" },
  code: { fontFamily: "monospace", backgroundColor: "#f0f0f0", padding: 8 },
  row: { flexDirection: "row", gap: 8, marginVertical: 16 },
  btn: { padding: 12, backgroundColor: "#ddd", borderRadius: 8 },
  resolved: { fontFamily: "monospace", color: "#2e7d32" },
});
```

---

## 6. RN Interview Points

- Three-way merge: base/ours/theirs — LCS helps auto-merge non-conflicting lines.
- `Pressable` for touch targets ≥ 44pt.
- Persist resolution to AsyncStorage in real app.

---

**Prev/Next:** [Concepts](day53_concepts.md) · [LeetCode](day53_leetcode.md)
