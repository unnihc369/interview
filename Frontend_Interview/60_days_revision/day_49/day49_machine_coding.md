# Day 49 — Machine Coding Revision

**React:** Spell checker trie + edit distance  
**React Native:** Voice command prefix matcher

---

## Table of Contents

### React (Web)

1. [Problem — Spell Checker](#1-problem--spell-checker)
2. [Trie + Levenshtein Engine](#2-trie--levenshtein-engine)
3. [React Spell Check UI](#3-react-spell-check-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Voice Command Matcher](#5-voice-command-matcher)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Spell Checker

**Task:** Text input with **red underline** for misspelled words. Click word → see suggestions ranked by edit distance. Dictionary in trie.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Tokenize | split on whitespace/punctuation |
| Misspelling | not in trie exact match |
| Suggestions | edit distance ≤ 2, top 5 |
| Apply fix | click to replace word |

---

## 2. Trie + Levenshtein Engine

```js
class TrieNode {
  constructor() {
    this.children = new Map();
    this.isEnd = false;
  }
}

class SpellTrie {
  constructor() {
    this.root = new TrieNode();
  }

  insert(word) {
    let node = this.root;
    for (const ch of word.toLowerCase()) {
      if (!node.children.has(ch)) node.children.set(ch, new TrieNode());
      node = node.children.get(ch);
    }
    node.isEnd = true;
  }

  has(word) {
    let node = this.root;
    for (const ch of word.toLowerCase()) {
      if (!node.children.has(ch)) return false;
      node = node.children.get(ch);
    }
    return node.isEnd;
  }

  getAllWords() {
    const out = [];
    const dfs = (node, path) => {
      if (node.isEnd) out.push(path);
      for (const [ch, child] of node.children) dfs(child, path + ch);
    };
    dfs(this.root, "");
    return out;
  }
}

function levenshtein(a, b) {
  const m = a.length, n = b.length;
  const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));
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

const DICTIONARY = [
  "react", "javascript", "typescript", "function", "return", "const",
  "component", "interface", "export", "import", "default", "async",
  "await", "promise", "callback", "handler", "render", "effect",
];

const spellTrie = new SpellTrie();
DICTIONARY.forEach((w) => spellTrie.insert(w));
const ALL_WORDS = spellTrie.getAllWords();

function getSuggestions(word, maxDist = 2, limit = 5) {
  const w = word.toLowerCase();
  if (spellTrie.has(w)) return [];
  return ALL_WORDS
    .map((dict) => ({ word: dict, dist: levenshtein(w, dict) }))
    .filter(({ dist }) => dist <= maxDist)
    .sort((a, b) => a.dist - b.dist || a.word.localeCompare(b.word))
    .slice(0, limit)
    .map(({ word }) => word);
}

function tokenize(text) {
  return text.split(/(\s+|[.,!?;])/);
}
```

---

## 3. React Spell Check UI

```tsx
import { useState, useMemo } from "react";

export default function SpellChecker() {
  const [text, setText] = useState("import React from 'reactt'");
  const [activeWord, setActiveWord] = useState<string | null>(null);

  const tokens = useMemo(() => tokenize(text), [text]);

  const misspelled = useMemo(() => {
    const set = new Set<string>();
    tokens.forEach((t) => {
      if (/^[a-zA-Z]+$/.test(t) && !spellTrie.has(t)) set.add(t.toLowerCase());
    });
    return set;
  }, [tokens]);

  const suggestions = useMemo(
    () => (activeWord ? getSuggestions(activeWord) : []),
    [activeWord]
  );

  const applySuggestion = (replacement: string) => {
    if (!activeWord) return;
    const re = new RegExp(`\\b${activeWord}\\b`, "i");
    setText((t) => t.replace(re, replacement));
    setActiveWord(null);
  };

  return (
    <div style={{ maxWidth: 560, margin: 24, fontFamily: "system-ui" }}>
      <h2>Spell Checker (Trie + Edit Distance)</h2>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        rows={4}
        style={{ width: "100%", padding: 12, fontSize: 16, borderRadius: 8, border: "1px solid #ccc" }}
      />
      <p style={{ fontSize: 14, lineHeight: 1.8, marginTop: 12 }}>
        {tokens.map((t, i) => {
          const isWord = /^[a-zA-Z]+$/.test(t);
          const bad = isWord && misspelled.has(t.toLowerCase());
          return isWord ? (
            <span
              key={i}
              onClick={() => bad && setActiveWord(t)}
              style={{
                borderBottom: bad ? "2px wavy #e53935" : "none",
                cursor: bad ? "pointer" : "default",
                marginRight: 2,
              }}
            >
              {t}
            </span>
          ) : (
            <span key={i}>{t}</span>
          );
        })}
      </p>
      {activeWord && (
        <div style={{ padding: 12, background: "#fff3e0", borderRadius: 8 }}>
          <strong>Suggestions for "{activeWord}":</strong>
          <div style={{ display: "flex", gap: 8, marginTop: 8, flexWrap: "wrap" }}>
            {suggestions.length ? suggestions.map((s) => (
              <button key={s} onClick={() => applySuggestion(s)}
                style={{ padding: "4px 12px", borderRadius: 6, border: "1px solid #ccc", cursor: "pointer" }}>
                {s}
              </button>
            )) : <span style={{ color: "#888" }}>No close matches</span>}
          </div>
        </div>
      )}
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Trie vs hash for spell check | Trie narrows candidates by prefix first |
| Edit distance 2 perf | ~5000 words OK brute; BK-tree at scale |
| contentEditable vs textarea | contentEditable for inline underline |
| LanguageTool API | Production uses remote + local trie hybrid |

---

# Part B — React Native

## 5. Voice Command Matcher

**Task:** Simulate voice input transcript matching **command trie**. Show matched command and action.

```tsx
import { useState, useMemo } from "react";
import { View, Text, TextInput, Pressable, StyleSheet } from "react-native";

class CommandTrie {
  root = { children: new Map(), cmd: null as string | null };

  insert(command: string) {
    let node = this.root;
    for (const ch of command) {
      if (!node.children.has(ch)) node.children.set(ch, { children: new Map(), cmd: null });
      node = node.children.get(ch)!;
    }
    node.cmd = command;
  }

  matchPrefix(transcript: string): string | null {
    const t = transcript.toLowerCase().trim();
    let node = this.root;
    let lastMatch: string | null = null;
    for (const ch of t) {
      if (!node.children.has(ch)) break;
      node = node.children.get(ch)!;
      if (node.cmd) lastMatch = node.cmd;
    }
    if (lastMatch && lastMatch.startsWith(t)) return lastMatch;
    return this.findBest(t);
  }

  findBest(transcript: string): string | null {
    const all: string[] = [];
    const dfs = (node: any, path: string) => {
      if (node.cmd) all.push(node.cmd);
      for (const [ch, child] of node.children) dfs(child, path + ch);
    };
    dfs(this.root, "");
    const matches = all.filter((c) => c.startsWith(transcript) || transcript.startsWith(c.split(" ")[0]));
    return matches.sort()[0] ?? null;
  }
}

const cmdTrie = new CommandTrie();
[
  "open settings", "open camera", "play music",
  "pause music", "send message", "set alarm", "navigate home",
].forEach((c) => cmdTrie.insert(c));

const ACTIONS: Record<string, string> = {
  "open settings": "→ SettingsScreen",
  "open camera": "→ CameraScreen",
  "play music": "→ MusicPlayer.play()",
  "pause music": "→ MusicPlayer.pause()",
  "send message": "→ MessageComposer.open()",
  "set alarm": "→ AlarmSet({ minutes: 5 })",
  "navigate home": "→ navigation.navigate('Home')",
};

export default function VoiceCommandMatcher() {
  const [transcript, setTranscript] = useState("open set");
  const match = useMemo(() => cmdTrie.matchPrefix(transcript), [transcript]);
  const action = match ? ACTIONS[match] : null;

  const samples = ["open set", "play mu", "send mes", "navigate ho", "pause"];

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Voice Command Matcher</Text>
      <Text style={styles.sub}>Trie prefix match on transcript</Text>

      <TextInput
        value={transcript}
        onChangeText={setTranscript}
        placeholder="Simulate voice transcript…"
        style={styles.input}
        autoCapitalize="none"
      />

      <View style={[styles.result, match ? styles.ok : styles.miss]}>
        <Text style={styles.resultLabel}>Matched command:</Text>
        <Text style={styles.resultValue}>{match ?? "—"}</Text>
        {action && <Text style={styles.action}>{action}</Text>}
      </View>

      <Text style={styles.samplesLabel}>Try:</Text>
      <View style={styles.chips}>
        {samples.map((s) => (
          <Pressable key={s} style={styles.chip} onPress={() => setTranscript(s)}>
            <Text>{s}</Text>
          </Pressable>
        ))}
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16, paddingTop: 60 },
  title: { fontSize: 20, fontWeight: "700" },
  sub: { color: "#666", marginBottom: 16, marginTop: 4 },
  input: { borderWidth: 1, borderColor: "#ccc", borderRadius: 10, padding: 14, fontSize: 16 },
  result: { marginTop: 16, padding: 16, borderRadius: 10 },
  ok: { backgroundColor: "#e8f5e9" },
  miss: { backgroundColor: "#fafafa" },
  resultLabel: { fontSize: 12, color: "#666" },
  resultValue: { fontSize: 18, fontWeight: "600", marginTop: 4 },
  action: { fontFamily: "monospace", fontSize: 13, marginTop: 8, color: "#1565c0" },
  samplesLabel: { marginTop: 20, fontSize: 13, color: "#888" },
  chips: { flexDirection: "row", flexWrap: "wrap", gap: 8, marginTop: 8 },
  chip: { paddingHorizontal: 12, paddingVertical: 8, backgroundColor: "#e3f2fd", borderRadius: 16 },
});
```

### Real voice integration sketch

```tsx
// import Voice from '@react-native-voice/voice';
// Voice.onSpeechResults = (e) => setTranscript(e.value[0]);
// Voice.start('en-US');
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Fuzzy voice match | Prefix trie + Levenshtein on commands |
| Partial utterance | Match longest command prefix |
| Permissions | iOS NSSpeechRecognitionUsageDescription |
| Offline commands | Local trie — no cloud needed |

---

## Quick Revision — Day 49

```
Web:  Spell trie + Levenshtein suggestions + wavy underline
RN:   Voice transcript → command trie → navigate/action
Mock: 208, 211, 212 timed (45 min)
Week 7 Tries: COMPLETE ✓
```

---

*End of Day 49 machine coding — Week 7 complete*
