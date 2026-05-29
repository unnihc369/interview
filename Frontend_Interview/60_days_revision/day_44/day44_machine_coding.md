# Day 44 — Machine Coding Revision

**React:** Dictionary prefix search  
**React Native:** Offline dictionary

---

## Table of Contents

### React (Web)

1. [Problem — Live Dictionary Lookup](#1-problem--live-dictionary-lookup)
2. [Prefix Search Trie](#2-prefix-search-trie)
3. [React Dictionary UI](#3-react-dictionary-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Offline Dictionary App](#5-offline-dictionary-app)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Live Dictionary Lookup

**Task:** Type-ahead dictionary. As user types, show **definitions** for all words matching prefix (max 8).

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Prefix match | on word field |
| Show definition | in result list |
| Highlight prefix | bold matched chars |
| Empty state | "No words found" |

---

## 2. Prefix Search Trie

```js
class DictEntry {
  constructor(word, definition) {
    this.word = word;
    this.definition = definition;
  }
}

class DictTrie {
  constructor() {
    this.root = { children: new Map(), entry: null };
  }

  insert(word, definition) {
    let node = this.root;
    for (const ch of word.toLowerCase()) {
      if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), entry: null });
      node = node.children.get(ch);
    }
    node.entry = { word, definition };
  }

  searchPrefix(prefix, limit = 8) {
    const p = prefix.toLowerCase();
    let node = this.root;
    for (const ch of p) {
      if (!node.children.has(ch)) return [];
      node = node.children.get(ch);
    }

    const results = [];
    const dfs = (n, path) => {
      if (results.length >= limit) return;
      if (n.entry) results.push(n.entry);
      for (const [ch, child] of n.children) dfs(child, path + ch);
    };
    dfs(node, p);
    return results;
  }

  lookup(word) {
    let node = this.root;
    for (const ch of word.toLowerCase()) {
      if (!node.children.has(ch)) return null;
      node = node.children.get(ch);
    }
    return node.entry;
  }
}

const DICT = new DictTrie();
[
  ["react", "A JavaScript library for building user interfaces"],
  ["redux", "Predictable state container for JS apps"],
  ["router", "Maps URLs to UI components"],
  ["render", "Process of displaying UI to screen"],
  ["ref", "Reference to DOM or component instance"],
  ["reducer", "Pure function (state, action) => newState"],
].forEach(([w, d]) => DICT.insert(w, d));
```

---

## 3. React Dictionary UI

```tsx
import { useState, useMemo } from "react";

export default function DictionarySearch() {
  const [query, setQuery] = useState("");
  const [selected, setSelected] = useState<{ word: string; definition: string } | null>(null);

  const matches = useMemo(() => {
    if (!query.trim()) return [];
    return DICT.searchPrefix(query, 8);
  }, [query]);

  const exact = useMemo(() => (query ? DICT.lookup(query) : null), [query]);

  return (
    <div style={{ maxWidth: 520, margin: 24, fontFamily: "system-ui" }}>
      <h2>Dictionary Prefix Search</h2>
      <input
        value={query}
        onChange={(e) => { setQuery(e.target.value); setSelected(null); }}
        placeholder="Type 're'…"
        style={{ width: "100%", padding: 12, fontSize: 16, borderRadius: 8, border: "1px solid #ccc" }}
      />

      {exact && (
        <div style={{ marginTop: 16, padding: 16, background: "#f0f7ff", borderRadius: 8 }}>
          <strong>{exact.word}</strong>
          <p style={{ margin: "8px 0 0" }}>{exact.definition}</p>
        </div>
      )}

      {query && !exact && matches.length === 0 && (
        <p style={{ color: "#888", marginTop: 12 }}>No words found for "{query}"</p>
      )}

      {matches.length > 0 && (
        <ul style={{ listStyle: "none", padding: 0, marginTop: 12 }}>
          {matches.map((m) => (
            <li
              key={m.word}
              onClick={() => setSelected(m)}
              style={{
                padding: "10px 14px", borderBottom: "1px solid #eee", cursor: "pointer",
                background: selected?.word === m.word ? "#e8f4fd" : "#fff",
              }}
            >
              <strong>{highlight(m.word, query)}</strong>
              <div style={{ fontSize: 13, color: "#666", marginTop: 4 }}>{m.definition}</div>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

function highlight(word: string, prefix: string) {
  const pl = prefix.toLowerCase();
  if (!word.toLowerCase().startsWith(pl)) return word;
  return (
    <>
      <span style={{ color: "#1976d2" }}>{word.slice(0, pl.length)}</span>
      {word.slice(pl.length)}
    </>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Store definitions in trie? | At terminal node — O(m) lookup |
| Scale to 100k words | Lazy load chunks; trie in Web Worker |
| Fuzzy match? | Trie for prefix + Levenshtein for typos (Day 49) |
| Search API vs local | Local trie for instant; remote for full corpus |

---

# Part B — React Native

## 5. Offline Dictionary App

**Task:** Dictionary works **offline**. Bundle word list; build trie on first launch; persist to AsyncStorage.

```tsx
import { useState, useEffect, useMemo } from "react";
import { View, TextInput, FlatList, Text, StyleSheet, ActivityIndicator } from "react-native";
import AsyncStorage from "@react-native-async-storage/async-storage";

const BUNDLED_WORDS = [
  { word: "hello", def: "Used as a greeting" },
  { word: "help", def: "Make it easier for someone" },
  { word: "hero", def: "Person admired for courage" },
  { word: "hex", def: "Base-16 numeral system" },
  { word: "hydrate", def: "Combine with water" },
];

function buildDictTrie(entries: { word: string; def: string }[]) {
  const trie = new DictTrie();
  entries.forEach(({ word, def }) => trie.insert(word, def));
  return trie;
}

export default function OfflineDictionary() {
  const [trie, setTrie] = useState<DictTrie | null>(null);
  const [query, setQuery] = useState("");
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    (async () => {
      try {
        const raw = await AsyncStorage.getItem("offline-dict-v1");
        if (raw) {
          const entries = JSON.parse(raw);
          setTrie(buildDictTrie(entries));
        } else {
          await AsyncStorage.setItem("offline-dict-v1", JSON.stringify(BUNDLED_WORDS));
          setTrie(buildDictTrie(BUNDLED_WORDS));
        }
      } finally {
        setLoading(false);
      }
    })();
  }, []);

  const results = useMemo(() => {
    if (!trie || !query.trim()) return [];
    return trie.searchPrefix(query, 10);
  }, [trie, query]);

  if (loading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" />
        <Text>Loading offline dictionary…</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.badge}>📴 Offline Ready</Text>
      <TextInput
        value={query}
        onChangeText={setQuery}
        placeholder="Search words…"
        style={styles.input}
        autoCorrect={false}
      />
      <FlatList
        data={results}
        keyExtractor={(item) => item.word}
        keyboardShouldPersistTaps="handled"
        ListEmptyComponent={
          query ? <Text style={styles.empty}>No matches</Text> : null
        }
        renderItem={({ item }) => (
          <View style={styles.card}>
            <Text style={styles.word}>{item.word}</Text>
            <Text style={styles.def}>{item.definition}</Text>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, paddingTop: 56 },
  center: { flex: 1, justifyContent: "center", alignItems: "center" },
  badge: { fontSize: 12, color: "#2e7d32", marginBottom: 8 },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 10, padding: 14, fontSize: 16 },
  card: { padding: 14, borderBottomWidth: StyleSheet.hairlineWidth, borderBottomColor: "#eee" },
  word: { fontSize: 17, fontWeight: "600" },
  def: { fontSize: 14, color: "#555", marginTop: 4 },
  empty: { textAlign: "center", color: "#888", marginTop: 24 },
});
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| AsyncStorage limits | ~6MB iOS — large dicts use SQLite (expo-sqlite) |
| First launch build | Show splash + progress; cache serialized trie |
| OTA updates | Download new word pack; rebuild trie versioned key |
| NetInfo | Show offline badge; disable remote fallback |

---

## Quick Revision — Day 44

```
Web:  Dict trie with definitions + prefix highlight
RN:   Bundle → AsyncStorage → offline prefix search
LC:   648 replaceWords, 677 MapSum, 720 longestWord
```

---

*End of Day 44 machine coding*
