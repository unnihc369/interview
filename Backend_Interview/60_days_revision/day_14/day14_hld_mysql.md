# Day 14 — HLD: Uber/Lyft Backend + MySQL (Joins, Subqueries, CTEs)

**Topics:** Ride-hailing architecture · Core services · Data model · SQL advanced · Interview Q&A

---

# Part A — HLD: Uber / Lyft Backend

---

# 1. Functional Requirements

- Rider requests ride (pickup, drop, vehicle type).
- Driver accepts/rejects; real-time location tracking.
- Fare estimate and final billing.
- Trip lifecycle: requested → matched → ongoing → completed → rated.
- Payments and receipts.
- Notifications (push/SMS).

---

# 2. Non-Functional Requirements

| NFR | Target |
|-----|--------|
| Availability | 99.99% for core trip flow |
| Latency | Match driver < few seconds in city |
| Scalability | Millions concurrent trips globally |
| Consistency | Strong for billing; eventual for location |
| Durability | Trip and payment records never lost |

---

# 3. High-Level Architecture

```text
                    +------------------+
                    |  API Gateway     |
                    +--------+---------+
                             |
        +--------------------+--------------------+
        |                    |                    |
+-------v-------+   +--------v--------+   +-------v-------+
| User Service  |   | Trip Service    |   | Payment Svc   |
+---------------+   +--------+--------+   +---------------+
                             |
              +--------------+--------------+
              |              |              |
      +-------v------+ +-----v-----+ +------v------+
      | Matching Svc | | Location| | Pricing Svc |
      +--------------+ +-----------+ +-------------+
              |
      +-------v-------+
      | Notification  |
      +---------------+

Data:
  MySQL/Postgres — users, trips, payments
  Redis — driver availability, sessions
  Kafka — location stream, trip events
  Cassandra/Dynamo — high-write location history (optional)
```

---

# 4. Core Services

## User Service
- Rider/driver profiles, KYC, ratings.
- Auth via OAuth/JWT.

## Trip Service
- State machine for trip status.
- Source of truth for trip record.

## Matching Service
- Finds nearby available drivers (geospatial index).
- Uses Redis GEO or specialized index (H3, Geohash).

## Location Service
- Ingests GPS updates from driver app (high throughput).
- Publishes to Kafka; consumers update cache and ETA.

## Pricing Service
- Surge, distance, time, base fare.
- Pre-trip estimate vs post-trip reconciliation.

## Payment Service
- Authorize hold at trip start, capture at end.
- Idempotent payment APIs.

---

# 5. Trip State Machine

```text
REQUESTED → MATCHED → DRIVER_ARRIVED → IN_PROGRESS → COMPLETED
                ↓
            CANCELLED (rider/driver/system)
```

Store current state in DB; transitions via Trip Service only (avoid split brain).

---

# 6. Matching Flow (Sequence)

```text
Rider App -> API Gateway: POST /trips (pickup, drop)
API Gateway -> Trip Service: createTrip()
Trip Service -> Pricing Service: estimateFare()
Trip Service -> Matching Service: findDrivers(lat, lng, radius)
Matching Service -> Redis GEO: query nearby drivers
Matching Service -> Trip Service: assign driver (or retry)
Trip Service -> Notification: notify driver
Driver App -> Trip Service: accept()
Trip Service --> Rider App: driver assigned + ETA
```

---

# 7. Data Model (Simplified)

```sql
users (id, name, phone, role)
drivers (id, user_id, vehicle_type, rating, status)
trips (
  id, rider_id, driver_id,
  pickup_lat, pickup_lng, drop_lat, drop_lng,
  status, fare_amount, created_at, completed_at
)
payments (id, trip_id, amount, status, provider_ref)
driver_locations (driver_id, lat, lng, updated_at)  -- or time-series store
```

Indexes:

- `trips(rider_id, created_at)`
- `trips(driver_id, status)`
- `drivers(status)` + geospatial index for location

---

# 8. Scalability & Reliability

| Challenge | Approach |
|-----------|----------|
| Location flood | Kafka + batch writes + regional shards |
| Hot cities | Partition by `city_id` |
| Matching spikes | Queue requests, expand radius, surge pricing |
| Double payment | Idempotency keys, unique constraint on trip payment |
| Split brain trip state | Optimistic locking (`version` column) |

---

# 9. CAP Tradeoffs (Interview)

- **Trip billing:** CP — strong consistency (RDBMS transaction).
- **Driver location on map:** AP — eventual consistency acceptable for few seconds.

---

# Part B — MySQL: Joins, Subqueries, CTEs

---

# 10. Sample Schema (Ride Context)

```sql
CREATE TABLE riders (
  id BIGINT PRIMARY KEY,
  name VARCHAR(100)
);

CREATE TABLE drivers (
  id BIGINT PRIMARY KEY,
  name VARCHAR(100),
  city VARCHAR(50)
);

CREATE TABLE trips (
  id BIGINT PRIMARY KEY,
  rider_id BIGINT,
  driver_id BIGINT,
  city VARCHAR(50),
  fare DECIMAL(10,2),
  status VARCHAR(20),
  completed_at DATETIME
);
```

---

# 11. JOIN Types

## INNER JOIN — only matching rows

```sql
SELECT t.id, r.name AS rider, d.name AS driver, t.fare
FROM trips t
INNER JOIN riders r ON t.rider_id = r.id
INNER JOIN drivers d ON t.driver_id = d.id
WHERE t.status = 'COMPLETED';
```

## LEFT JOIN — all from left + matches from right

```sql
-- Riders who never completed a trip
SELECT r.id, r.name
FROM riders r
LEFT JOIN trips t ON r.id = t.rider_id AND t.status = 'COMPLETED'
WHERE t.id IS NULL;
```

## RIGHT JOIN / FULL OUTER

Rare in MySQL (no native FULL OUTER; emulate with UNION).

---

# Join Interview Tips

| Join | Use when |
|------|----------|
| INNER | Need only matched pairs |
| LEFT | Preserve left table rows |
| Self-join | Same table hierarchy (employee-manager) |

---

# 12. Subqueries

## Subquery in WHERE

```sql
-- Drivers with above-average fare in their city
SELECT d.name, t.fare
FROM trips t
JOIN drivers d ON t.driver_id = d.id
WHERE t.fare > (
  SELECT AVG(t2.fare)
  FROM trips t2
  WHERE t2.city = t.city AND t2.status = 'COMPLETED'
);
```

## Subquery in FROM (derived table)

```sql
SELECT city, avg_fare
FROM (
  SELECT city, AVG(fare) AS avg_fare
  FROM trips
  WHERE status = 'COMPLETED'
  GROUP BY city
) x
WHERE avg_fare > 200;
```

## EXISTS — often faster than IN for large sets

```sql
SELECT d.name
FROM drivers d
WHERE EXISTS (
  SELECT 1 FROM trips t
  WHERE t.driver_id = d.id AND t.status = 'COMPLETED'
);
```

---

# 13. CTE (Common Table Expression)

```sql
WITH completed_trips AS (
  SELECT driver_id, city, fare
  FROM trips
  WHERE status = 'COMPLETED'
),
driver_stats AS (
  SELECT driver_id, city,
         COUNT(*) AS trip_count,
         SUM(fare) AS total_fare,
         AVG(fare) AS avg_fare
  FROM completed_trips
  GROUP BY driver_id, city
)
SELECT d.name, ds.city, ds.trip_count, ds.total_fare
FROM driver_stats ds
JOIN drivers d ON ds.driver_id = d.id
ORDER BY ds.total_fare DESC
LIMIT 10;
```

Benefits:

- Readable multi-step logic
- Can reference CTE multiple times in same query

---

# 14. Recursive CTE (Hierarchy)

```sql
WITH RECURSIVE mgr_chain AS (
  SELECT id, name, manager_id, 1 AS depth
  FROM employees
  WHERE id = 100
  UNION ALL
  SELECT e.id, e.name, e.manager_id, mc.depth + 1
  FROM employees e
  JOIN mgr_chain mc ON e.id = mc.manager_id
)
SELECT * FROM mgr_chain;
```

---

# 15. Window Functions (MySQL 8+)

```sql
SELECT
  driver_id,
  fare,
  AVG(fare) OVER (PARTITION BY driver_id) AS driver_avg,
  RANK() OVER (PARTITION BY city ORDER BY fare DESC) AS fare_rank_in_city
FROM trips
WHERE status = 'COMPLETED';
```

Use for ranking, running totals without collapsing rows.

---

# 15b. GROUP BY & HAVING

```sql
-- Aggregate per group
SELECT city, COUNT(*) AS trip_count, SUM(fare) AS revenue
FROM trips
WHERE status = 'COMPLETED'
GROUP BY city
HAVING revenue > 10000
ORDER BY revenue DESC;
```

| Clause | Purpose |
|--------|---------|
| `WHERE` | Filter **rows** before grouping |
| `GROUP BY` | Collapse rows into groups |
| `HAVING` | Filter **groups** after aggregation |
| `ORDER BY` | Sort final result |

**Rule:** Every non-aggregated column in SELECT must appear in GROUP BY.

```sql
-- WRONG: name not in GROUP BY (ONLY_FULL_GROUP_BY mode)
SELECT city, name, COUNT(*) FROM trips GROUP BY city;

-- RIGHT
SELECT city, COUNT(*) FROM trips GROUP BY city;
```

---

# 16. Practice Queries (Uber Context)

**Top 3 drivers by revenue per city:**

```sql
WITH revenue AS (
  SELECT driver_id, city, SUM(fare) AS total
  FROM trips
  WHERE status = 'COMPLETED'
  GROUP BY driver_id, city
),
ranked AS (
  SELECT *, RANK() OVER (PARTITION BY city ORDER BY total DESC) AS rnk
  FROM revenue
)
SELECT * FROM ranked WHERE rnk <= 3;
```

**Monthly completed trips:**

```sql
SELECT DATE_FORMAT(completed_at, '%Y-%m') AS month, COUNT(*) AS trips
FROM trips
WHERE status = 'COMPLETED'
GROUP BY month
ORDER BY month;
```

---

# Common Interview Questions

## Q1. How does Uber match drivers quickly?

Geospatial index (Redis GEO, Geohash grid), filter available drivers, rank by distance/ETA/rating.

---

## Q2. INNER vs LEFT JOIN?

INNER drops non-matching rows; LEFT keeps all left rows with NULLs for unmatched right.

---

## Q3. Subquery vs JOIN?

Often equivalent; optimizer may rewrite. JOIN usually clearer; EXISTS good for semi-joins.

---

## Q4. When use CTE?

Multi-step readable queries, reuse intermediate result, recursion.

---

## Q5. How to handle surge pricing at scale?

Precompute zones, cache multipliers in Redis, async recalculation from demand/supply ratio.

---

# One-Line Revision

```text
Uber HLD = Trip + Matching + Location + Pricing + Payment, event-driven, geo index.
MySQL = INNER/LEFT joins, subqueries, CTEs, window functions for analytics.
```

---

*End of Day 14 HLD + MySQL*
