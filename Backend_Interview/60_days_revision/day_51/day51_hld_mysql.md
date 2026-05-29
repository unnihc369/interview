# Day 51 — HLD Redo: URL Shortener + Uber + MySQL EXPLAIN

**Week 8 · Tuesday · ~3 hours**  
**Reference:** `day_7/day7_hld_mysql.md`, `day_14/day14_hld_mysql.md`

---

# Part 1 — URL Shortener (45 min mock)

---

## 1. Functional Requirements

- Shorten long URL → unique short code  
- Redirect short URL → original (HTTP 302/301)  
- Optional: custom alias, expiration, analytics (click count)  
- Optional: authenticated users manage links  

---

## 2. Non-Functional Requirements

| NFR | Target |
|-----|--------|
| Read latency | < 50ms p99 redirect |
| Write latency | < 200ms create |
| Availability | 99.99% reads (redirect is critical path) |
| Scale | 100M URLs, 10K redirects/sec |
| Uniqueness | No collision on short codes |

---

## 3. Capacity Estimation (Back-of-envelope)

```text
100M URLs × 500 bytes ≈ 50 GB metadata (fits sharded SQL)
10K RPS read >> 100 RPS write  → optimize for read
Redirect: cache hit 90%+ → Redis fronting DB
```

---

## 4. High-Level Architecture

```text
Client → CDN (static) → LB → API Servers
                              ↓
                    ┌─────────┴─────────┐
                    ↓                   ↓
              Write API            Redirect API
              (create)             (read-heavy)
                    ↓                   ↓
                 MySQL              Redis cache
              (source of truth)    (hot URLs)
                    ↓
              ID generator (Snowflake / DB auto-inc per shard)
```

---

## 5. Shortening Strategies

| Approach | Pros | Cons |
|----------|------|------|
| Hash (MD5 truncate) | Fast | Collisions → retry |
| Base62(auto-inc ID) | No collision | Needs distributed ID |
| Pre-generated keys | Fast write | Key pool management |

**Recommended:** `unique_id` from DB/Snowflake → Base62 encode → 7 chars ≈ 62^7 URLs.

```java
String encode(long id) {
    char[] chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789".toCharArray();
    StringBuilder sb = new StringBuilder();
    while (id > 0) {
        sb.append(chars[(int)(id % 62)]);
        id /= 62;
    }
    return sb.reverse().toString();
}
```

---

## 6. Redirect Flow

```text
GET /abc123
  → API / redirect service
  → Redis GET url:abc123
  → miss → MySQL SELECT long_url WHERE short_code='abc123'
  → populate cache (TTL 24h)
  → HTTP 302 Location: long_url
  → async: increment click counter (Kafka → analytics DB)
```

**301 vs 302:** 301 permanent (SEO to shortener); 302 temporary (track every click).

---

## 7. Database Schema

```sql
CREATE TABLE urls (
    id            BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_code    VARCHAR(10) NOT NULL UNIQUE,
    long_url      VARCHAR(2048) NOT NULL,
    user_id       BIGINT,
    created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at    TIMESTAMP NULL,
    click_count   BIGINT DEFAULT 0,
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id)
);
```

---

## 8. Sharding

- Shard by `short_code` hash or by `id` range  
- Redirect always has `short_code` → single shard lookup  
- Cross-shard only for user dashboard listing  

---

## 9. URL Shortener — Interview Checklist

| # | Topic | Covered? |
|---|-------|----------|
| 1 | Read-heavy → cache | |
| 2 | Base62 / ID generation | |
| 3 | Collision handling | |
| 4 | Rate limiting (abuse) | |
| 5 | Custom alias uniqueness | |
| 6 | Expired URL 404 | |
| 7 | Analytics async path | |

---

# Part 2 — Uber / Ride-Hailing HLD (45 min mock)

---

## 1. Functional Requirements

- Request ride (pickup, drop, vehicle type)  
- Match driver, track trip, ETA  
- Fare estimate + final payment  
- Ratings, trip history  
- Cancel ride (rider/driver)  

---

## 2. Non-Functional Requirements

- Match driver in **seconds** in dense cities  
- **Strong consistency** for billing; **eventual** for live location  
- Global scale, 99.99% availability for trip flow  

---

## 3. Architecture (Redo from memory)

```text
Apps → API Gateway → Services:
  User | Trip | Matching | Location | Pricing | Payment | Notification

Data:
  MySQL/Postgres — trips, users, payments
  Redis — driver online, GEO index, sessions
  Kafka — location stream, trip events
  Object store — receipts (optional)
```

---

## 4. Trip State Machine

```text
REQUESTED → MATCHED → DRIVER_ARRIVED → IN_PROGRESS → COMPLETED
     ↓
 CANCELLED (any stage with rules)
```

Single writer: **Trip Service** owns transitions.

---

## 5. Matching Flow

```text
1. Trip Service creates trip (REQUESTED)
2. Pricing Service → fare estimate
3. Matching Service queries Redis GEO (lat, lng, radius 5km)
4. Filter: online, correct vehicle type, not on trip
5. Rank by ETA, rating, acceptance rate
6. Push to top N drivers; first accept wins
7. Trip → MATCHED; notify rider
```

**Geospatial:** Redis `GEOADD`, `GEORADIUS` or H3/Geohash cells.

---

## 6. Location Pipeline

```text
Driver app (GPS every 3-5s)
  → Location Ingest API
  → Kafka topic driver-locations
  → Consumers: update Redis position, compute ETA, rider map WS
```

High write throughput → don't sync every GPS to MySQL.

---

## 7. Payment Flow

```text
Trip start → authorize hold (Stripe)
Trip end → Pricing final fare → capture payment
Idempotent payment_id per trip_id
```

---

## 8. Failure Modes

| Failure | Mitigation |
|---------|------------|
| No drivers | Expand radius, surge pricing, retry |
| Driver cancels | Re-match from queue |
| Split brain trip state | Optimistic locking / version column |
| Location lag | Show last known + stale indicator |

---

## 9. Uber HLD — Interview Checklist

| # | Topic | Covered? |
|---|-------|----------|
| 1 | Trip state machine | |
| 2 | Matching + GEO | |
| 3 | Location streaming | |
| 4 | Pricing / surge | |
| 5 | Payment authorize/capture | |
| 6 | Notification service | |
| 7 | Strong vs eventual consistency | |

---

# Part 3 — MySQL EXPLAIN Optimization Exercises

---

## Exercise 0 — Setup (run locally or mentally)

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    dept_id INT NOT NULL,
    name VARCHAR(100),
    salary INT,
    hired_at DATE,
    INDEX idx_dept (dept_id),
    INDEX idx_dept_salary (dept_id, salary)
);

CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    region VARCHAR(50),
    INDEX idx_region (region)
);
```

---

## Exercise 1 — Full table scan vs index

```sql
EXPLAIN SELECT * FROM employees WHERE dept_id = 5;
```

**Expected:** `type=ref`, `key=idx_dept`, `rows` << table size.

```sql
EXPLAIN SELECT * FROM employees WHERE salary > 100000;
```

**Expected:** Likely `ALL` scan (no index on salary alone) unless `idx_dept_salary` used with dept filter.

**Task:** Add `INDEX idx_salary (salary)` and re-run EXPLAIN. Compare `rows` and `Extra`.

---

## Exercise 2 — Composite index left prefix

```sql
EXPLAIN SELECT * FROM employees WHERE dept_id = 5 AND salary > 80000;
```

**Expected:** Uses `idx_dept_salary` — both conditions benefit.

```sql
EXPLAIN SELECT * FROM employees WHERE salary > 80000;
```

**Expected:** Cannot use `idx_dept_salary` efficiently (violates left-prefix rule).

**Interview line:** “Composite index (dept_id, salary) supports `dept_id` alone and `(dept_id, salary)`, not `salary` alone.”

---

## Exercise 3 — Covering index

```sql
EXPLAIN SELECT dept_id, salary FROM employees WHERE dept_id = 5;
```

Look for `Extra: Using index` — index-only scan, no table lookup.

---

## Exercise 4 — JOIN order

```sql
EXPLAIN
SELECT e.name, d.name
FROM employees e
JOIN departments d ON e.dept_id = d.id
WHERE d.region = 'APAC';
```

**Analyze:**

- Which table is driven first?  
- `d.region` filtered via `idx_region` → small row set → nested loop into `e`  

**Optimization:** Ensure `employees.dept_id` indexed (already `idx_dept`).

---

## Exercise 5 — Filesort and temporary

```sql
EXPLAIN
SELECT * FROM employees
WHERE dept_id = 5
ORDER BY salary DESC
LIMIT 10;
```

With `idx_dept_salary`, may avoid filesort. Without composite, watch `Extra: Using filesort`.

---

## Exercise 6 — Subquery vs JOIN

```sql
-- A
EXPLAIN SELECT * FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE region = 'APAC');

-- B
EXPLAIN SELECT e.* FROM employees e
JOIN departments d ON e.dept_id = d.id
WHERE d.region = 'APAC';
```

Modern MySQL often optimizes similarly; compare `type`, `rows`, `filtered`.

---

## Exercise 7 — UPDATE / DELETE pitfalls

```sql
EXPLAIN UPDATE employees SET salary = salary * 1.1 WHERE dept_id = 5;
```

Index on `dept_id` helps find rows; still row-level locks.

**Question:** Why is `UPDATE` on non-indexed column slow at scale?  
→ Full scan + lock every row touched.

---

## EXPLAIN Column Cheat Sheet

| Column | Meaning |
|--------|---------|
| `type` | `system` < `const` < `eq_ref` < `ref` < `range` < `index` < `ALL` (worst) |
| `key` | Index actually used |
| `rows` | Estimated rows examined |
| `Extra` | `Using index`, `Using filesort`, `Using temporary`, `Using where` |

---

## Practice Answers (Self-check)

1. **Avoid SELECT *** in production APIs — more I/O, prevents covering index.  
2. **Normalize vs denormalize** — URL shortener clicks might be separate `clicks` table for write scaling.  
3. **Read replica** for analytics queries; master for writes.  
4. **Slow query log** + `EXPLAIN ANALYZE` (MySQL 8.0.18+) for actual timings.

---

# Part 4 — Combined Timed Drill (90 min)

| Block | Time |
|-------|------|
| URL Shortener whiteboard | 40 min |
| Uber HLD whiteboard | 40 min |
| 3 EXPLAIN exercises on paper | 10 min |

---

*End of Day 51 — HLD + MySQL*
