# Day 21 — Machine Coding Revision

**React:** Reorder playlist (two pointers)  
**React Native:** Pinch zoom image

---

## Table of Contents

### React (Web)

1. [Problem — Playlist Reorder](#1-problem--playlist-reorder)
2. [Two-Pointer Swap Logic](#2-two-pointer-swap-logic)
3. [React UI](#3-react-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Pinch Zoom Image](#5-pinch-zoom-image)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Playlist Reorder

**Task:** Reorder tracks using **Move Up / Move Down** buttons (two-pointer swaps) or drag handles. Show current index highlight.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Move up | Swap `i` with `i-1` if `i > 0` |
| Move down | Swap `i` with `i+1` if `i < n-1` |
| Selection | Click row to select index |
| Keyboard | Arrow up/down when focused |

---

## 2. Two-Pointer Swap Logic

```js
function swapAdjacent(list, index, direction) {
  const j = direction === "up" ? index - 1 : index + 1;
  if (j < 0 || j >= list.length) return list;
  const next = [...list];
  [next[index], next[j]] = [next[j], next[index]];
  return next;
}
```

### Reorder range (two-pointer shift)

Move item from `from` to `to` by shifting elements between pointers — O(n) single pass.

---

## 3. React UI

```tsx
import { useState } from "react";

type Track = { id: string; title: string };

export default function PlaylistReorder() {
  const [tracks, setTracks] = useState<Track[]>([
    { id: "1", title: "Song A" },
    { id: "2", title: "Song B" },
    { id: "3", title: "Song C" },
  ]);
  const [selected, setSelected] = useState(0);

  const move = (dir: "up" | "down") => {
    setTracks((prev) => swapAdjacent(prev, selected, dir));
    setSelected((i) => (dir === "up" ? Math.max(0, i - 1) : Math.min(tracks.length - 1, i + 1)));
  };

  return (
    <div style={{ maxWidth: 400, margin: 24 }}>
      <h2>Playlist</h2>
      {tracks.map((t, i) => (
        <div
          key={t.id}
          onClick={() => setSelected(i)}
          style={{
            padding: 12,
            marginBottom: 4,
            background: i === selected ? "#e3f2fd" : "#f5f5f5",
            cursor: "pointer",
          }}
        >
          {i + 1}. {t.title}
        </div>
      ))}
      <div style={{ marginTop: 12, display: "flex", gap: 8 }}>
        <button onClick={() => move("up")} disabled={selected === 0}>↑ Up</button>
        <button onClick={() => move("down")} disabled={selected === tracks.length - 1}>↓ Down</button>
      </div>
    </div>
  );
}

function swapAdjacent<T>(list: T[], index: number, direction: "up" | "down"): T[] {
  const j = direction === "up" ? index - 1 : index + 1;
  if (j < 0 || j >= list.length) return list;
  const next = [...list];
  [next[index], next[j]] = [next[j], next[index]];
  return next;
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Swap vs splice reorder | Adjacent = swap; arbitrary = splice |
| selected index after move | Follow item: decrement/increment index |
| a11y | `role="listbox"` + arrow keys |

---

# Part B — React Native

## 5. Pinch Zoom Image

**Task:** Image viewer with **pinch-to-zoom** and **pan** when zoomed. Double-tap reset.

```tsx
import { Image, StyleSheet } from "react-native";
import { Gesture, GestureDetector } from "react-native-gesture-handler";
import Animated, {
  useAnimatedStyle,
  useSharedValue,
  withTiming,
} from "react-native-reanimated";

export default function PinchZoomImage({ uri }: { uri: string }) {
  const scale = useSharedValue(1);
  const savedScale = useSharedValue(1);
  const translateX = useSharedValue(0);
  const translateY = useSharedValue(0);

  const pinch = Gesture.Pinch()
    .onUpdate((e) => {
      scale.value = savedScale.value * e.scale;
    })
    .onEnd(() => {
      if (scale.value < 1) {
        scale.value = withTiming(1);
        savedScale.value = 1;
        translateX.value = withTiming(0);
        translateY.value = withTiming(0);
      } else savedScale.value = scale.value;
    });

  const pan = Gesture.Pan()
    .onUpdate((e) => {
      if (scale.value > 1) {
        translateX.value = e.translationX;
        translateY.value = e.translationY;
      }
    });

  const composed = Gesture.Simultaneous(pinch, pan);

  const style = useAnimatedStyle(() => ({
    transform: [
      { translateX: translateX.value },
      { translateY: translateY.value },
      { scale: scale.value },
    ],
  }));

  return (
    <GestureDetector gesture={composed}>
      <Animated.View style={styles.container}>
        <Animated.Image source={{ uri }} style={[styles.image, style]} />
      </Animated.View>
    </GestureDetector>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, overflow: "hidden" },
  image: { width: "100%", height: "100%", resizeMode: "contain" },
});
```

### Two-pointer tie-in

Pinch uses **two touch points** — distance between fingers drives scale (opposite ends of gesture vector).

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Reanimated vs Animated | Shared values on UI thread — smoother pinch |
| Simultaneous gestures | Pinch + pan together |
| Bounds clamp | Limit pan so image doesn't leave viewport |

---

## Quick Revision — Day 21

```
Web:  selected index + swapAdjacent up/down → immutable reorder
RN:   Pinch + Pan gestures → shared scale/translate
Both: two-pointer mental model; timed LC 11/15/42
```

---

*End of Day 21 machine coding*
