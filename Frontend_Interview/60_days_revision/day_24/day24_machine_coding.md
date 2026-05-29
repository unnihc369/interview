# Day 24 — Machine Coding Revision

**React:** Median tracker streaming  
**React Native:** Latency monitor

---

## Table of Contents

### React (Web)

1. [Problem — Streaming Median](#1-problem--streaming-median)
2. [Two-Heap Median Finder](#2-two-heap-median-finder)
3. [Live Chart with rAF](#3-live-chart-with-raf)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Latency Monitor](#5-latency-monitor)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Streaming Median

**Task:** Numbers arrive every second (simulated). Display **running median** and sparkline history. Use two-heap algorithm (LC 295).

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Add number | Input or random stream |
| Median | O(log n) insert |
| History | Last 50 medians plotted |
| Render | rAF batch chart updates |

---

## 2. Two-Heap Median Finder

```js
class MedianFinder {
  constructor() {
    this.maxLo = []; // max-heap via negation
    this.minHi = [];
  }

  _pushMax(val) {
    this.maxLo.push(-val);
    this.maxLo.sort((a, b) => a - b);
  }
  _popMax() {
    return -this.maxLo.shift();
  }
  _peekMax() {
    return -this.maxLo[0];
  }

  _pushMin(val) {
    this.minHi.push(val);
    this.minHi.sort((a, b) => a - b);
  }
  _popMin() {
    return this.minHi.shift();
  }

  addNum(num) {
    if (!this.maxLo.length || num <= this._peekMax()) this._pushMax(num);
    else this._pushMin(num);

    if (this.maxLo.length > this.minHi.length + 1) {
      this._pushMin(this._popMax());
    } else if (this.minHi.length > this.maxLo.length) {
      this._pushMax(this._popMin());
    }
  }

  findMedian() {
    if (this.maxLo.length > this.minHi.length) return this._peekMax();
    return (this._peekMax() + this.minHi[0]) / 2;
  }
}
```

---

## 3. Live Chart with rAF

```tsx
import { useEffect, useRef, useState } from "react";

export default function MedianTracker() {
  const finderRef = useRef(new MedianFinder());
  const [median, setMedian] = useState<number | null>(null);
  const [history, setHistory] = useState<number[]>([]);
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const pendingRef = useRef<number[]>([]);
  const rafRef = useRef<number>();

  useEffect(() => {
    const interval = setInterval(() => {
      const n = Math.floor(Math.random() * 100);
      pendingRef.current.push(n);
      if (!rafRef.current) {
        rafRef.current = requestAnimationFrame(flush);
      }
    }, 800);
    return () => clearInterval(interval);
  }, []);

  const flush = () => {
    rafRef.current = undefined;
    const batch = pendingRef.current.splice(0);
    batch.forEach((n) => finderRef.current.addNum(n));
    const m = finderRef.current.findMedian();
    setMedian(m);
    setHistory((h) => [...h.slice(-49), m]);
    drawSparkline();
  };

  const drawSparkline = () => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext("2d")!;
    const { width, height } = canvas;
    ctx.clearRect(0, 0, width, height);
    ctx.strokeStyle = "#1976d2";
    ctx.beginPath();
    history.forEach((v, i, arr) => {
      const x = (i / Math.max(arr.length - 1, 1)) * width;
      const y = height - (v / 100) * height;
      i === 0 ? ctx.moveTo(x, y) : ctx.lineTo(x, y);
    });
    ctx.stroke();
  };

  const addManual = (n: number) => {
    finderRef.current.addNum(n);
    const m = finderRef.current.findMedian();
    setMedian(m);
    setHistory((h) => [...h.slice(-49), m]);
  };

  return (
    <div style={{ padding: 24 }}>
      <h2>Streaming Median</h2>
      <p style={{ fontSize: 32 }}>{median?.toFixed(1) ?? "—"}</p>
      <canvas ref={canvasRef} width={400} height={80} style={{ border: "1px solid #ccc" }} />
      <button onClick={() => addManual(Math.floor(Math.random() * 100))}>Add random</button>
    </div>
  );
}

class MedianFinder {
  maxLo: number[] = [];
  minHi: number[] = [];
  addNum(num: number) {
    if (!this.maxLo.length || num <= -this.maxLo[0]) {
      this.maxLo.push(-num);
      this.maxLo.sort((a, b) => a - b);
    } else {
      this.minHi.push(num);
      this.minHi.sort((a, b) => a - b);
    }
    if (this.maxLo.length > this.minHi.length + 1) {
      const v = -this.maxLo.shift()!;
      this.minHi.push(v);
      this.minHi.sort((a, b) => a - b);
    } else if (this.minHi.length > this.maxLo.length) {
      const v = this.minHi.shift()!;
      this.maxLo.push(-v);
      this.maxLo.sort((a, b) => a - b);
    }
  }
  findMedian() {
    if (this.maxLo.length > this.minHi.length) return -this.maxLo[0];
    return (-this.maxLo[0] + this.minHi[0]) / 2;
  }
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Why two heaps? | O(log n) insert; O(1) median peek |
| rAF batching | Coalesce rapid adds to one paint |
| Canvas vs SVG sparkline | Canvas cheaper for many points |

---

# Part B — React Native

## 5. Latency Monitor

**Task:** Ping API every 2s; show latency line chart updating via rAF-style `requestAnimationFrame` polyfill or Reanimated.

```tsx
import { useEffect, useState } from "react";
import { View, Text, Dimensions } from "react-native";
import Svg, { Polyline } from "react-native-svg";

export default function LatencyMonitor() {
  const [samples, setSamples] = useState<number[]>([]);

  useEffect(() => {
    let mounted = true;
    const poll = async () => {
      const start = Date.now();
      try {
        await fetch("https://httpbin.org/delay/0");
        const ms = Date.now() - start;
        if (mounted) setSamples((s) => [...s.slice(-30), ms]);
      } catch { /* offline */ }
    };
    poll();
    const id = setInterval(poll, 2000);
    return () => { mounted = false; clearInterval(id); };
  }, []);

  const w = Dimensions.get("window").width - 32;
  const h = 100;
  const max = Math.max(...samples, 100);
  const points = samples
    .map((v, i) => `${(i / Math.max(samples.length - 1, 1)) * w},${h - (v / max) * h}`)
    .join(" ");

  return (
    <View style={{ padding: 16 }}>
      <Text style={{ fontSize: 18, fontWeight: "600" }}>API Latency</Text>
      <Text>Latest: {samples[samples.length - 1] ?? "—"} ms</Text>
      <Svg width={w} height={h}>
        {points && <Polyline points={points} fill="none" stroke="#1976d2" strokeWidth={2} />}
      </Svg>
    </View>
  );
}
```

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| rAF in RN? | `requestAnimationFrame` exists on global |
| Victory-native / Skia | Production chart libs |
| Percentile vs median | P95 for SLA dashboards |

---

## Quick Revision — Day 24

```
Web:  two heaps median + rAF batch + canvas sparkline
RN:   interval poll → latency samples → SVG polyline
LC:   295 median; 502 IPO heap; 1167 min cost sticks
```

---

*End of Day 24 machine coding*
