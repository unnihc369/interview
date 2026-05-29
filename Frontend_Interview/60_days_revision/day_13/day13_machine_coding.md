# Day 13 — Machine Coding Revision (Saturday)

**React:** Real-time price chart with sliding average window  
**React Native:** Reanimated sliding graph

---

## Table of Contents

### React (Web)

1. [Problem — Live Price Chart](#1-problem--live-price-chart)
2. [Sliding Average Hook](#2-sliding-average-hook)
3. [Canvas Chart Component](#3-canvas-chart-component)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Reanimated Sliding Graph](#5-reanimated-sliding-graph)
6. [RN Graph Component](#6-rn-graph-component)
7. [RN Interview Points](#7-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Live Price Chart

Simulate tick stream; display:

- Raw price line (last N points — **sliding window**)
- SMA overlay (simple moving average, window = 20)
- 60fps-friendly canvas render

---

## 2. Sliding Average Hook

```tsx
import { useCallback, useEffect, useRef, useState } from "react";

export function useSlidingWindow(size: number) {
  const [points, setPoints] = useState<number[]>([]);

  const push = useCallback(
    (value: number) => {
      setPoints((prev) => [...prev, value].slice(-size));
    },
    [size]
  );

  return { points, push };
}

export function sma(values: number[], period: number): number[] {
  if (period <= 0) return [];
  const out: number[] = [];
  for (let i = 0; i < values.length; i++) {
    if (i < period - 1) {
      out.push(NaN);
      continue;
    }
    let sum = 0;
    for (let j = i - period + 1; j <= i; j++) sum += values[j];
    out.push(sum / period);
  }
  return out;
}
```

### Simulated tick stream

```tsx
export function usePriceStream(intervalMs = 500) {
  const { points, push } = useSlidingWindow(120);
  const priceRef = useRef(100);

  useEffect(() => {
    const id = setInterval(() => {
      priceRef.current += (Math.random() - 0.48) * 2;
      push(Number(priceRef.current.toFixed(2)));
    }, intervalMs);
    return () => clearInterval(id);
  }, [intervalMs, push]);

  return points;
}
```

---

## 3. Canvas Chart Component

```tsx
import { useEffect, useRef } from "react";
import { sma, usePriceStream } from "./useSlidingWindow";

const W = 600;
const H = 240;
const SMA_PERIOD = 20;

export default function PriceChart() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const prices = usePriceStream(400);
  const averages = sma(prices, SMA_PERIOD);

  useEffect(() => {
    const ctx = canvasRef.current?.getContext("2d");
    if (!ctx || prices.length < 2) return;

    ctx.clearRect(0, 0, W, H);
    const min = Math.min(...prices);
    const max = Math.max(...prices);
    const range = max - min || 1;

    const toY = (v: number) => H - ((v - min) / range) * (H - 20) - 10;
    const toX = (i: number) => (i / (prices.length - 1)) * W;

    // price line
    ctx.strokeStyle = "#4a90d9";
    ctx.lineWidth = 2;
    ctx.beginPath();
    prices.forEach((p, i) => {
      const x = toX(i);
      const y = toY(p);
      i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
    });
    ctx.stroke();

    // SMA line
    ctx.strokeStyle = "#e67e22";
    ctx.beginPath();
    averages.forEach((v, i) => {
      if (Number.isNaN(v)) return;
      const x = toX(i);
      const y = toY(v);
      i === 0 || Number.isNaN(averages[i - 1])
        ? ctx.moveTo(x, y)
        : ctx.lineTo(x, y);
    });
    ctx.stroke();
  }, [prices, averages]);

  return (
    <div style={{ padding: 16 }}>
      <h3>Price · window {prices.length} · SMA({SMA_PERIOD})</h3>
      <canvas ref={canvasRef} width={W} height={H} style={{ border: "1px solid #ddd" }} />
      <p>Latest: {prices[prices.length - 1]?.toFixed(2)}</p>
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Canvas vs SVG | Canvas better for many points / animation |
| Sliding window | `slice(-N)` or ring buffer for O(1) |
| WeakMap for chart | Attach state to canvas ref without leak |

---

# Part B — React Native

## 5. Reanimated Sliding Graph

Use **react-native-reanimated** for smooth path updates on UI thread.

```bash
npx expo install react-native-reanimated react-native-svg
```

---

## 6. RN Graph Component

```tsx
import { useEffect, useState } from "react";
import { View, Text } from "react-native";
import Svg, { Polyline } from "react-native-svg";
import Animated, {
  useAnimatedProps,
  useSharedValue,
} from "react-native-reanimated";

const AnimatedPolyline = Animated.createAnimatedComponent(Polyline);
const W = 320;
const H = 160;
const WINDOW = 60;

function toPoints(values: number[]): string {
  if (values.length < 2) return "";
  const min = Math.min(...values);
  const max = Math.max(...values);
  const range = max - min || 1;
  return values
    .map((v, i) => {
      const x = (i / (values.length - 1)) * W;
      const y = H - ((v - min) / range) * (H - 10) - 5;
      return `${x},${y}`;
    })
    .join(" ");
}

export default function SlidingGraph() {
  const [data, setData] = useState<number[]>([]);
  const points = useSharedValue("");

  useEffect(() => {
    let price = 50;
    const id = setInterval(() => {
      price += (Math.random() - 0.5) * 3;
      setData((prev) => [...prev, price].slice(-WINDOW));
    }, 300);
    return () => clearInterval(id);
  }, []);

  useEffect(() => {
    points.value = toPoints(data);
  }, [data, points]);

  const animatedProps = useAnimatedProps(() => ({
    points: points.value,
  }));

  return (
    <View style={{ padding: 16 }}>
      <Text>Sliding graph ({data.length}/{WINDOW})</Text>
      <Svg width={W} height={H}>
        <AnimatedPolyline
          animatedProps={animatedProps}
          fill="none"
          stroke="#4a90d9"
          strokeWidth={2}
        />
      </Svg>
    </View>
  );
}
```

---

## 7. RN Interview Points

| Question | Answer |
|----------|--------|
| Reanimated vs Animated | Reanimated runs on UI thread |
| react-native-svg | Vector charts; Polyline for line graph |
| Window size | Fixed buffer — shift old points off |

---

## Quick Revision — Day 13

```
Web:  slice(-N) prices + SMA + canvas draw
RN:   Reanimated + SVG Polyline + interval ticks
Weak: chartState WeakMap per canvas element
```

---

*End of Day 13 machine coding*
