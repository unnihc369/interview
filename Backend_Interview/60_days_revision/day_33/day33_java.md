# Day 33 — G1 GC, ZGC, Shenandoah

**Topics:** Low-Latency Collectors · GC Tuning · Interview Comparison

---

# 1. GC Evolution Overview

```text
Serial       → single thread, small apps
Parallel     → throughput (JDK 8 default)
CMS          → low pause, deprecated
G1           → default JDK 9+ (balanced)
ZGC          → ultra-low pause (JDK 15+ prod)
Shenandoah   → low pause (OpenJDK, Red Hat)
```

---

# 2. G1 GC (Garbage First)

**Goal:** Predictable pause times for large heaps (multi-GB).

### Region-Based Heap

```text
Heap divided into equal-sized regions (1-32 MB)
Types: Eden, Survivor, Old, Humongous (objects > 50% region)
```

### Mixed GC

- **Young GC:** stop-the-world, collect Eden regions
- **Concurrent marking:** find mostly garbage Old regions
- **Mixed GC:** reclaim Old regions with most garbage first (Garbage First name)

### Key Flags

```bash
-XX:+UseG1GC                    # default in JDK 9+
-XX:MaxGCPauseMillis=200        # target pause (hint, not guarantee)
-XX:G1HeapRegionSize=16m
-XX:InitiatingHeapOccupancyPercent=45
```

### When to Use

- Heap > 4GB
- Need balance of throughput and pause control
- General-purpose server default

---

# 3. ZGC (Z Garbage Collector)

**Goal:** Sub-millisecond pauses (typically < 1ms), scales to **terabytes**.

### Techniques

- **Colored pointers** — metadata in pointer bits (load barrier)
- **Concurrent** — almost all work outside STW
- **Region-based** like G1

### Key Flags

```bash
-XX:+UseZGC
-Xmx16g
# JDK 17+: generational ZGC
-XX:+ZGenerational
```

### When to Use

- Latency-sensitive (trading, real-time analytics)
- Very large heaps
- Can trade some throughput for pause stability

---

# 4. Shenandoah GC

**Goal:** Low pause independent of heap size (Red Hat / OpenJDK).

### Techniques

- **Brooks forwarding pointers** — concurrent compaction
- Most phases concurrent including compaction

### Key Flags

```bash
-XX:+UseShenandoahGC
-XX:ShenandoahGCHeuristics=adaptive
```

### When to Use

Similar to ZGC — low latency apps. Availability depends on JDK distribution (not in all Oracle builds historically).

---

# 5. Comparison Table

| Collector | Pause Target | Heap Size | Throughput | JDK |
|-----------|--------------|-----------|------------|-----|
| Parallel | Seconds possible | Medium | Highest | 8 default |
| G1 | 10-200ms typical | Large | Good | 9+ default |
| ZGC | <1ms | Very large | Moderate | 15+ |
| Shenandoah | Low ms | Large | Moderate | OpenJDK |

---

# 6. Choosing a Collector

```text
Batch / throughput jobs     → Parallel or G1 with high pause OK
General microservices     → G1 (default)
Latency-critical APIs     → ZGC or Shenandoah
Small heap (< 512MB)      → Serial or G1
```

---

# 7. Monitoring GC

```bash
# GC logs (JDK 9+ unified logging)
-Xlog:gc*:file=gc.log:time,uptime,level

# jstat
jstat -gcutil <pid> 1000

# VisualVM, GCeasy, Datadog JVM metrics
```

**Metrics to watch:** pause time P99, GC frequency, heap after GC, allocation rate.

---

# 8. Tuning Tips (Interview)

```text
1. MaxGCPauseMillis is a goal — monitor actual pauses
2. Avoid premature optimization — measure first
3. Heap too small → frequent GC; too large → long pauses (except ZGC/Shenandoah)
4. Off-heap (DirectByteBuffer) not in heap — still affects system memory
5. Metaspace leaks look like heap issues — check class unloading
```

---

# Interview Questions

## G1 vs CMS?

CMS deprecated. G1 handles compaction better, predictable regions, no fragmentation failure mode of CMS.

## ZGC vs Shenandoah?

Both target low pause. ZGC uses colored pointers; Shenandoah Brooks pointers. Choose based on JDK vendor support and benchmarks on your workload.

## Why not always ZGC?

Throughput may be lower than G1 for some workloads; requires newer JDK; not needed if pauses already acceptable.

## Humongous objects in G1?

Objects > 50% region size go to Humongous regions — can cause fragmentation; tune region size or object design.

## Full GC still possible?

Yes — "stop the world" full GC if concurrent phases fail (Allocation Failure). Investigate heap sizing and leaks.

---

# Quick Revision

```text
G1: regions, MaxGCPauseMillis, JDK 9+ default
ZGC: sub-ms pause, colored pointers, terabyte heaps
Shenandoah: concurrent compact, OpenJDK
Measure → choose collector → tune heap
```

---

*End of Day 33 G1GC ZGC Shenandoah*
