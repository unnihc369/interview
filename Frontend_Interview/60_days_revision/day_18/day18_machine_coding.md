# Day 18 — Machine Coding Revision

**React:** Progress stepper  
**React Native:** Onboarding carousel

---

## Table of Contents

### React (Web)

1. [Problem — Progress Stepper](#1-problem--progress-stepper)
2. [Step State & Generator Flow](#2-step-state--generator-flow)
3. [React Stepper UI](#3-react-stepper-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Onboarding Carousel](#5-onboarding-carousel)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Progress Stepper

**Task:** Multi-step form wizard with horizontal stepper — labels, completed/active/pending states, Next/Back, optional validation gate per step.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Steps | 3–5 labeled steps (Shipping → Payment → Review) |
| Visual | Circles + connector lines; checkmark when done |
| Navigation | Next disabled until step valid; Back preserves data |
| Progress | `currentStep` index 0..n-1 |
| Accessibility | `aria-current="step"` on active |

---

## 2. Step State & Generator Flow

### Generator for step transitions (conceptual)

```js
function* stepFlow(steps) {
  for (const step of steps) {
    const ok = yield step;
    if (!ok) return;
  }
  yield { done: true };
}

const flow = stepFlow(["shipping", "payment", "review"]);
flow.next(); // start shipping
flow.next(true); // validated → payment
```

### Validation per step

```js
const validators = {
  shipping: (data) => data.address?.length > 5,
  payment: (data) => /^\d{16}$/.test(data.card ?? ""),
  review: () => true,
};
```

---

## 3. React Stepper UI

```tsx
import { useState } from "react";

const STEPS = ["Shipping", "Payment", "Review"];

export default function ProgressStepper() {
  const [current, setCurrent] = useState(0);
  const [form, setForm] = useState({ address: "", card: "" });

  const canNext =
    current === 0
      ? form.address.length > 5
      : current === 1
      ? form.card.length === 16
      : true;

  const next = () => canNext && setCurrent((c) => Math.min(c + 1, STEPS.length - 1));
  const back = () => setCurrent((c) => Math.max(c - 1, 0));

  return (
    <div style={{ maxWidth: 560, margin: "24px auto" }}>
      <nav aria-label="Progress">
        <ol style={{ display: "flex", listStyle: "none", padding: 0 }}>
          {STEPS.map((label, i) => (
            <li
              key={label}
              aria-current={i === current ? "step" : undefined}
              style={{ flex: 1, textAlign: "center" }}
            >
              <div
                style={{
                  width: 32,
                  height: 32,
                  borderRadius: "50%",
                  margin: "0 auto",
                  lineHeight: "32px",
                  background:
                    i < current ? "#2e7d32" : i === current ? "#1976d2" : "#ccc",
                  color: "#fff",
                }}
              >
                {i < current ? "✓" : i + 1}
              </div>
              <span style={{ fontSize: 12 }}>{label}</span>
            </li>
          ))}
        </ol>
      </nav>

      <div style={{ margin: "24px 0", minHeight: 120 }}>
        {current === 0 && (
          <input
            placeholder="Address"
            value={form.address}
            onChange={(e) => setForm({ ...form, address: e.target.value })}
          />
        )}
        {current === 1 && (
          <input
            placeholder="Card (16 digits)"
            value={form.card}
            onChange={(e) => setForm({ ...form, card: e.target.value })}
          />
        )}
        {current === 2 && (
          <pre>{JSON.stringify(form, null, 2)}</pre>
        )}
      </div>

      <div style={{ display: "flex", gap: 8 }}>
        <button onClick={back} disabled={current === 0}>
          Back
        </button>
        <button onClick={next} disabled={!canNext || current === STEPS.length - 1}>
          Next
        </button>
      </div>
    </div>
  );
}
```

### Connector lines (CSS)

Use `::after` pseudo on each step except last, or flex border-top between circles.

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Single `current` vs per-step status | Index + compare: `i < current` done, `i === current` active |
| Lift state? | Form object at wizard level; steps are presentational |
| URL sync | `?step=2` for deep link — parse on mount |
| Generator in UI? | Optional for async step machine; index state is simpler |

---

# Part B — React Native

## 5. Onboarding Carousel

**Task:** Swipeable onboarding slides with dots indicator, Skip, Next/Get Started.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Slides | 3–4 full-width screens |
| Swipe | Horizontal FlatList paging |
| Dots | Active dot highlighted |
| Last slide | "Get Started" → navigate home |

```tsx
import { useRef, useState } from "react";
import {
  FlatList,
  View,
  Text,
  Dimensions,
  Pressable,
  StyleSheet,
} from "react-native";

const { width } = Dimensions.get("window");

const SLIDES = [
  { id: "1", title: "Welcome", body: "Track your goals easily." },
  { id: "2", title: "Sync", body: "Works offline and online." },
  { id: "3", title: "Start", body: "You're ready to go!" },
];

export default function OnboardingCarousel({ onDone }: { onDone: () => void }) {
  const [index, setIndex] = useState(0);
  const listRef = useRef<FlatList>(null);

  const onScroll = (e: any) => {
    const i = Math.round(e.nativeEvent.contentOffset.x / width);
    setIndex(i);
  };

  const next = () => {
    if (index === SLIDES.length - 1) onDone();
    else listRef.current?.scrollToIndex({ index: index + 1 });
  };

  return (
    <View style={{ flex: 1 }}>
      <FlatList
        ref={listRef}
        data={SLIDES}
        horizontal
        pagingEnabled
        showsHorizontalScrollIndicator={false}
        onMomentumScrollEnd={onScroll}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={[styles.slide, { width }]}>
            <Text style={styles.title}>{item.title}</Text>
            <Text>{item.body}</Text>
          </View>
        )}
      />
      <View style={styles.dots}>
        {SLIDES.map((_, i) => (
          <View
            key={i}
            style={[styles.dot, i === index && styles.dotActive]}
          />
        ))}
      </View>
      <Pressable onPress={next} style={styles.btn}>
        <Text style={{ color: "#fff" }}>
          {index === SLIDES.length - 1 ? "Get Started" : "Next"}
        </Text>
      </Pressable>
      {index < SLIDES.length - 1 && (
        <Pressable onPress={onDone} style={styles.skip}>
          <Text>Skip</Text>
        </Pressable>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  slide: { flex: 1, justifyContent: "center", alignItems: "center", padding: 24 },
  title: { fontSize: 24, fontWeight: "700", marginBottom: 12 },
  dots: { flexDirection: "row", justifyContent: "center", gap: 8, marginBottom: 16 },
  dot: { width: 8, height: 8, borderRadius: 4, backgroundColor: "#ccc" },
  dotActive: { backgroundColor: "#1976d2", width: 24 },
  btn: {
    backgroundColor: "#1976d2",
    margin: 16,
    padding: 14,
    borderRadius: 8,
    alignItems: "center",
  },
  skip: { alignItems: "center", padding: 8 },
});
```

### Generator for slides config

```js
function* slideConfig() {
  yield { title: "Welcome", image: "welcome.png" };
  yield { title: "Features", image: "features.png" };
  yield { title: "Done", image: "done.png" };
}
// [...slideConfig()] → array for FlatList data
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| FlatList vs ScrollView | FlatList for many slides; 3–4 either works |
| `pagingEnabled` | Snaps to page width — match slide width to screen |
| AsyncStorage skip | Persist `onboardingDone=true` on complete |
| Reanimated carousel | `react-native-reanimated-carousel` for parallax |

---

## Quick Revision — Day 18

```
Web:  currentStep index + per-step validation + aria-current
RN:   horizontal FlatList paging + dots + Get Started
Both: generator yields steps/slides; lazy config loading
```

---

*End of Day 18 machine coding*
