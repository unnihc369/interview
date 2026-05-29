# Day 16 — Machine Coding Revision

**React:** Palindrome checker UI  
**React Native:** Chat typing indicator

---

## Table of Contents

### React (Web)

1. [Problem — Palindrome Checker](#1-problem--palindrome-checker)
2. [Two-Pointer Palindrome Logic](#2-two-pointer-palindrome-logic)
3. [Interactive UI Implementation](#3-interactive-ui-implementation)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Chat Typing Indicator](#5-chat-typing-indicator)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Palindrome Checker

**Task:** Build a live palindrome checker that validates user input as they type, highlights matching character pairs, and shows pass/fail status.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Live check | Re-validate on every keystroke (debounced optional) |
| Normalize | Ignore spaces, punctuation, case |
| Visual feedback | Green border if palindrome; red if not |
| Pair highlight | Show which chars compare (two-pointer indices) |
| Empty state | Neutral message when input is empty |

**Two-pointer tie-in:** Compare chars from both ends moving inward — same pattern as LC 125.

---

## 2. Two-Pointer Palindrome Logic

### Core algorithm

```js
function normalize(s) {
  return s.toLowerCase().replace(/[^a-z0-9]/g, "");
}

function isPalindromeTwoPointer(s) {
  const str = normalize(s);
  let l = 0, r = str.length - 1;
  while (l < r) {
    if (str[l] !== str[r]) return { ok: false, failAt: [l, r] };
    l++;
    r--;
  }
  return { ok: true, failAt: null };
}
```

### With `every` on indices (alternative — less efficient)

```js
function isPalindromeEvery(s) {
  const str = normalize(s);
  const n = str.length;
  return [...Array(Math.floor(n / 2)).keys()].every(
    (i) => str[i] === str[n - 1 - i]
  );
}
```

Interview: prefer explicit two-pointer — O(n) time, O(1) extra space after normalize.

---

## 3. Interactive UI Implementation

```tsx
import { useMemo, useState } from "react";

function normalize(s: string) {
  return s.toLowerCase().replace(/[^a-z0-9]/g, "");
}

function checkPalindrome(raw: string) {
  const str = normalize(raw);
  let l = 0, r = str.length - 1;
  const pairs: [number, number][] = [];

  while (l < r) {
    if (str[l] !== str[r]) {
      return { isPalindrome: false, pairs, mismatch: [l, r] as const };
    }
    pairs.push([l, r]);
    l++;
    r--;
  }
  return { isPalindrome: str.length > 0, pairs, mismatch: null };
}

export default function PalindromeChecker() {
  const [text, setText] = useState("");

  const result = useMemo(() => checkPalindrome(text), [text]);

  const borderColor = !text.trim()
    ? "#ccc"
    : result.isPalindrome
    ? "#2e7d32"
    : "#c62828";

  return (
    <div style={{ maxWidth: 480, margin: "24px auto", fontFamily: "sans-serif" }}>
      <h2>Palindrome Checker</h2>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Type a phrase… e.g. A man a plan a canal Panama"
        rows={4}
        style={{
          width: "100%",
          padding: 12,
          border: `2px solid ${borderColor}`,
          borderRadius: 8,
          fontSize: 16,
        }}
      />
      <p style={{ color: borderColor, fontWeight: 600 }}>
        {!text.trim()
          ? "Enter text to check"
          : result.isPalindrome
          ? "✓ Palindrome"
          : "✗ Not a palindrome"}
      </p>
      {result.mismatch && (
        <p style={{ fontSize: 14, color: "#666" }}>
          Mismatch at normalized indices {result.mismatch[0]} and{" "}
          {result.mismatch[1]}
        </p>
      )}
      <div style={{ marginTop: 12 }}>
        <strong>Normalized:</strong>{" "}
        <code>{normalize(text) || "—"}</code>
      </div>
      {result.pairs.length > 0 && (
        <div style={{ marginTop: 8, fontSize: 14 }}>
          Matched pairs: {result.pairs.length}
        </div>
      )}
    </div>
  );
}
```

### Enhancements (mention in interview)

| Enhancement | How |
|-------------|-----|
| Debounce 150ms | `useDeferredValue` or lodash debounce |
| Character-level highlight | Map raw index → normalized index |
| History | Stack of last 5 checks |
| Accessibility | `aria-live="polite"` on status |

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Why `useMemo` for check? | Avoid re-running O(n) on unrelated parent re-renders |
| Normalize in render or effect? | Pure derive in render/memo — no effect needed |
| `every` vs two-pointer? | Two-pointer finds **first** mismatch; `every` still short-circuits but less clear |
| Controlled vs uncontrolled | Controlled `textarea` for live feedback |

---

# Part B — React Native

## 5. Chat Typing Indicator

**Task:** Show animated "typing…" dots when another user (simulated) is typing in a chat thread.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Typing state | Boolean from props or simulated timer |
| Animation | 3 dots bounce or fade sequentially |
| Auto-hide | Clear after 3s of no activity |
| Accessibility | `accessibilityLabel="User is typing"` |

### Animated typing dots

```tsx
import { useEffect, useRef } from "react";
import { View, Animated, StyleSheet, Text } from "react-native";

const DOTS = 3;

export function TypingIndicator({ visible }: { visible: boolean }) {
  const anims = useRef(
    Array.from({ length: DOTS }, () => new Animated.Value(0))
  ).current;

  useEffect(() => {
    if (!visible) return;

    const loops = anims.map((anim, i) =>
      Animated.loop(
        Animated.sequence([
          Animated.delay(i * 150),
          Animated.timing(anim, {
            toValue: 1,
            duration: 300,
            useNativeDriver: true,
          }),
          Animated.timing(anim, {
            toValue: 0,
            duration: 300,
            useNativeDriver: true,
          }),
        ])
      )
    );

    loops.forEach((l) => l.start());
    return () => loops.forEach((l) => l.stop());
  }, [visible, anims]);

  if (!visible) return null;

  return (
    <View style={styles.row} accessibilityLabel="User is typing">
      <Text style={styles.label}>typing</Text>
      {anims.map((anim, i) => (
        <Animated.View
          key={i}
          style={[
            styles.dot,
            {
              opacity: anim.interpolate({
                inputRange: [0, 1],
                outputRange: [0.3, 1],
              }),
              transform: [
                {
                  translateY: anim.interpolate({
                    inputRange: [0, 1],
                    outputRange: [0, -4],
                  }),
                },
              ],
            },
          ]}
        />
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  row: {
    flexDirection: "row",
    alignItems: "center",
    padding: 8,
    marginLeft: 12,
  },
  label: { color: "#888", marginRight: 4, fontSize: 12 },
  dot: {
    width: 6,
    height: 6,
    borderRadius: 3,
    backgroundColor: "#888",
    marginHorizontal: 2,
  },
});
```

### Simulated typing in parent

```tsx
import { useState, useEffect } from "react";

function ChatScreen() {
  const [peerTyping, setPeerTyping] = useState(false);

  useEffect(() => {
    const interval = setInterval(() => {
      setPeerTyping(true);
      setTimeout(() => setPeerTyping(false), 2500);
    }, 8000);
    return () => clearInterval(interval);
  }, []);

  return (
    <View style={{ flex: 1 }}>
      {/* messages */}
      <TypingIndicator visible={peerTyping} />
    </View>
  );
}
```

### WebSocket integration sketch

```js
socket.on("typing", ({ userId, isTyping }) => {
  setTypingUsers((prev) =>
    isTyping ? [...new Set([...prev, userId])] : prev.filter((id) => id !== userId)
  );
});
// Show indicator when typingUsers.some(id => id !== me)
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| `useNativeDriver: true`? | Only for transform/opacity — yes for dots |
| Reanimated vs Animated? | Reanimated 2+ runs on UI thread — smoother lists |
| Debounce typing events? | Emit at most every 300ms; stop after 2s idle |
| FlatList footer | Render `TypingIndicator` in `ListFooterComponent` |

---

## Quick Revision — Day 16

```
Web:  normalize → two pointers l/r → live status + mismatch index
RN:   Animated.loop + stagger delay → typing dots; WebSocket debounce
Both: short-circuit validation; palindrome ≡ two-pointer equality
```

---

*End of Day 16 machine coding*
