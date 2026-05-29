# Day 59 — Machine Coding

**React:** Autocorrect cost display  
**React Native:** Typo suggester

---

## Table of Contents

### React (Web)

1. [Problem — Autocorrect Panel](#1-problem--autocorrect-panel)
2. [Edit Distance Suggester](#2-edit-distance-suggester)
3. [React Autocorrect UI](#3-react-autocorrect-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Typo Suggester](#5-typo-suggester)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Autocorrect Panel

**Task:** As user types, show top-3 dictionary suggestions with **edit distance cost** displayed.

---

## 2. Edit Distance Suggester

```js
function minDistance(a, b) {
  const m = a.length, n = b.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 0; i <= m; i++) dp[i][0] = i;
  for (let j = 0; j <= n; j++) dp[0][j] = j;
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      dp[i][j] = a[i - 1] === b[j - 1]
        ? dp[i - 1][j - 1]
        : 1 + Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]);
    }
  }
  return dp[m][n];
}

function suggest(typed, dict, maxCost = 2, limit = 3) {
  return dict
    .map((w) => ({ word: w, cost: minDistance(typed.toLowerCase(), w) }))
    .filter((x) => x.cost <= maxCost)
    .sort((a, b) => a.cost - b.cost || a.word.localeCompare(b.word))
    .slice(0, limit);
}
```

---

## 3. React Autocorrect UI

```tsx
import { useState, useMemo } from "react";

const DICT = ["hello", "help", "world", "word", "hold", "shell"];

export default function AutocorrectPanel() {
  const [text, setText] = useState("helo");
  const suggestions = useMemo(() => suggest(text, DICT), [text]);

  return (
    <div style={{ padding: 24, maxWidth: 400 }}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        style={{ width: "100%", padding: 12, fontSize: 16 }}
      />
      {suggestions.length > 0 && (
        <ul style={{ listStyle: "none", padding: 0, marginTop: 12 }}>
          {suggestions.map(({ word, cost }) => (
            <li
              key={word}
              onClick={() => setText(word)}
              style={{ padding: 8, cursor: "pointer", borderBottom: "1px solid #eee" }}
            >
              {word} <span style={{ color: "#888" }}>(cost {cost})</span>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## 4. React Interview Points

- Debounce suggestions for performance (`useDeferredValue`).
- Trie + BK-tree for large dictionaries in production.
- Show cost breakdown: insert/delete/replace (optional advanced UI).

---

# Part B — React Native

## 5. Typo Suggester

```tsx
import { View, Text, TextInput, Pressable, FlatList, StyleSheet } from "react-native";
import { useState, useMemo } from "react";

const WORDS = ["react", "native", "typescript", "javascript", "component"];

export default function TypoSuggester() {
  const [query, setQuery] = useState("reacr");
  const items = useMemo(() => suggest(query, WORDS), [query]);

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        value={query}
        onChangeText={setQuery}
        autoCorrect={false}
        placeholder="Type a word..."
      />
      <FlatList
        data={items}
        keyExtractor={(item) => item.word}
        renderItem={({ item }) => (
          <Pressable style={styles.row} onPress={() => setQuery(item.word)}>
            <Text style={styles.word}>{item.word}</Text>
            <Text style={styles.cost}>Δ{item.cost}</Text>
          </Pressable>
        )}
        ListEmptyComponent={<Text style={styles.empty}>No close matches</Text>}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 8, padding: 12, fontSize: 16 },
  row: { flexDirection: "row", justifyContent: "space-between", paddingVertical: 12, borderBottomWidth: 1, borderColor: "#eee" },
  word: { fontSize: 16 },
  cost: { color: "#888" },
  empty: { marginTop: 16, color: "#999" },
});
```

---

## 6. RN Interview Points

- Disable `autoCorrect` when showing custom suggestions.
- `FlatList` for scrollable suggestions.
- Haptic on suggestion tap (`Haptics.impactAsync`).

---

**Prev/Next:** [Concepts](day59_concepts.md) · [LeetCode](day59_leetcode.md)
