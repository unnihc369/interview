# Day 44 — LLD: Cache System (LRU, LFU, TTL)

**Topics:** Eviction policies · Thread safety · Design for scale · Interview Q&A

---

# 1. Problem Statement

Design an in-memory cache supporting:

- `get(key)` / `put(key, value)`
- **Capacity limit** with eviction policy (LRU or LFU)
- **TTL** — entries expire after configurable time
- Thread-safe for concurrent access

---

# 2. Functional Requirements

| Operation | Description |
|-----------|-------------|
| get | Return value or miss; update access metadata |
| put | Insert/update; evict if over capacity |
| delete | Remove key |
| TTL | Auto-expire after N seconds |

---

# 3. Non-Functional Requirements

- O(1) get/put (average)
- Thread-safe reads/writes
- Optional: distributed cache (Redis) for multi-instance — mention in HLD follow-up

---

# 4. LRU — Least Recently Used

## Data Structures

```text
HashMap<K, Node>     → O(1) lookup
Doubly Linked List   → O(1) move to front / evict tail
```

```java
class LRUCache {
    class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0), tail = new Node(0, 0);

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail; tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        moveToHead(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) {
            Node node = map.get(key);
            node.val = value;
            moveToHead(node);
        } else {
            Node node = new Node(key, value);
            map.put(key, node);
            addToHead(node);
            if (map.size() > capacity) {
                Node lru = removeTail();
                map.remove(lru.key);
            }
        }
    }

    private void moveToHead(Node node) {
        removeNode(node);
        addToHead(node);
    }
    // removeNode, addToHead, removeTail helpers...
}
```

---

# Thread-Safe LRU

```java
public class ConcurrentLRUCache<K, V> {
    private final int capacity;
    private final Map<K, V> map = new LinkedHashMap<>(capacity, 0.75f, true) {
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return size() > capacity;
        }
    };

    public synchronized V get(K key) { return map.get(key); }
    public synchronized void put(K key, V val) { map.put(key, val); }
}
```

Production: **Caffeine** or **Guava Cache** — don't reinvent in real apps.

---

# 5. LFU — Least Frequently Used

## Data Structures

```text
HashMap<key, Node>           → key → node with freq
HashMap<freq, DoublyLinkedList> → freq → list of nodes at that freq
int minFreq                  → track minimum for O(1) eviction
```

```java
class LFUCache {
    class Node {
        int key, val, freq;
        Node prev, next;
    }

    private final int capacity;
    private final Map<Integer, Node> keyMap = new HashMap<>();
    private final Map<Integer, DoublyLinkedList> freqMap = new HashMap<>();
    private int minFreq;

    public int get(int key) {
        if (!keyMap.containsKey(key)) return -1;
        Node node = keyMap.get(key);
        increaseFreq(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (capacity == 0) return;
        if (keyMap.containsKey(key)) {
            Node node = keyMap.get(key);
            node.val = value;
            increaseFreq(node);
        } else {
            if (keyMap.size() >= capacity) {
                Node evict = freqMap.get(minFreq).removeLast();
                keyMap.remove(evict.key);
            }
            Node node = new Node(key, value, 1);
            keyMap.put(key, node);
            freqMap.computeIfAbsent(1, k -> new DoublyLinkedList()).addFirst(node);
            minFreq = 1;
        }
    }

    private void increaseFreq(Node node) {
        freqMap.get(node.freq).remove(node);
        if (node.freq == minFreq && freqMap.get(minFreq).isEmpty()) minFreq++;
        node.freq++;
        freqMap.computeIfAbsent(node.freq, k -> new DoublyLinkedList()).addFirst(node);
    }
}
```

---

# LRU vs LFU

| | LRU | LFU |
|---|-----|-----|
| Evicts | Least recently accessed | Least frequently accessed |
| Good for | Temporal locality | Hot key retention |
| Weakness | One-time scan evicts hot data | Old high-freq keys stick |
| Real-world | General purpose | CDN, DB buffer pool |

---

# 6. TTL Support

## Approach A: Lazy expiration (on access)

```java
class CacheEntry<V> {
    V value;
    long expireAt;  // System.currentTimeMillis() + ttlMs
    boolean isExpired() { return System.currentTimeMillis() > expireAt; }
}

public V get(K key) {
    CacheEntry<V> entry = map.get(key);
    if (entry == null || entry.isExpired()) {
        map.remove(key);
        return null;
    }
    return entry.value;
}
```

## Approach B: Active expiration (background thread)

```java
ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();
scheduler.scheduleAtFixedRate(this::evictExpired, 1, 1, TimeUnit.SECONDS);

void evictExpired() {
    long now = System.currentTimeMillis();
    map.entrySet().removeIf(e -> e.getValue().expireAt < now);
}
```

## Approach C: Redis-style — separate expiry index

`TreeMap<expireTime, key>` for efficient range eviction.

---

# 7. Combined Design (Interview Whiteboard)

```text
┌─────────────────────────────────────┐
│           CacheService              │
│  get(key) / put(key,val,ttl)        │
└──────────────┬──────────────────────┘
               │
    ┌──────────┴──────────┐
    │   EvictionPolicy    │  ← Strategy pattern
    │  LRU | LFU          │
    └──────────┬──────────┘
               │
    ┌──────────┴──────────┐
    │  ConcurrentHashMap  │
    │  + metadata (DLL)   │
    └─────────────────────┘
               │
    ┌──────────┴──────────┐
    │  TTL Scheduler      │
    └─────────────────────┘
```

---

# 8. Distributed Cache (HLD Extension)

```text
App Instance 1 ──┐
App Instance 2 ──┼── Redis Cluster (LRU/LFU via maxmemory-policy)
App Instance 3 ──┘

Local L1 (Caffeine, small) → Redis L2 → Database
```

Redis policies: `volatile-lru`, `allkeys-lfu`, etc.

---

# 9. Rate Limiter LLD (Token Bucket)

Standalone design — often asked separately from cache. Full design: [supplements/sde1_lld_missing_designs.md](supplements/sde1_lld_missing_designs.md).

## Token Bucket

- Bucket holds max `capacity` tokens
- Refill at `refillRate` tokens per second
- Each request consumes 1 token; reject if empty

```java
public class TokenBucketRateLimiter {
    private final long capacity;
    private final double refillRatePerSec;
    private double tokens;
    private long lastRefillNanos;

    public synchronized boolean allowRequest() {
        refill();
        if (tokens >= 1) {
            tokens -= 1;
            return true;
        }
        return false;
    }

    private void refill() {
        long now = System.nanoTime();
        double elapsed = (now - lastRefillNanos) / 1_000_000_000.0;
        tokens = Math.min(capacity, tokens + elapsed * refillRatePerSec);
        lastRefillNanos = now;
    }
}
```

| Algorithm | Behavior |
|-----------|----------|
| Token bucket | Allows bursts up to capacity |
| Sliding window | Strict rate over rolling window |
| Fixed window | Simple counter per minute — boundary burst issue |

**Distributed:** Redis `INCR` + TTL or Lua script for atomic check-and-decrement.

---

# Interview Questions

## Q1. Why doubly linked list for LRU?

Singled linked list can't remove arbitrary node in O(1) without predecessor pointer.

## Q2. LFU — why track minFreq?

After eviction at minFreq, need O(1) next victim — incrementing minFreq only when that bucket empties.

## Q3. Cache stampede?

Many threads miss same key → all hit DB. Fix: single-flight lock, early refresh, probabilistic early expiration.

## Q4. LRU with TTL — eviction order?

Evict expired first (lazy or proactive), then LRU among valid entries.

## Q5. @Cacheable in Spring — which policy?

Provider-dependent; Caffeine supports size + TTL; Redis uses server-side eviction.

---

# One-Line Revision

```text
LRU = HashMap + DLL O(1); LFU = HashMap + freq buckets + minFreq; TTL = lazy on get or scheduled sweep; production → Caffeine/Redis.
```

---

*End of Day 44 LLD*
