# Day 7 — HLD + MySQL

## Topics
- URL Shortener
- Base62 Hashing
- Caching
- DB Sharding
- MySQL Indexing

---

# 1. URL Shortener

## Functional Requirements

- Generate short URL
- Redirect short URL
- Analytics

---

# High Level Design

```text
Client
   ↓
Load Balancer
   ↓
Application Servers
   ↓
Cache (Redis)
   ↓
Database
```

---

# URL Shortening Flow

```text
Long URL
   ↓
Hashing
   ↓
Base62 Encoding
   ↓
Short URL
```

---

# Base62 Characters

```text
a-z
A-Z
0-9
```

Total:
```text
62 characters
```

---

# Base62 Example

```text
125 → cb
```

---

# Redirection Flow

```text
short.ly/abc
      ↓
lookup in cache
      ↓
DB lookup if cache miss
      ↓
redirect to original URL
```

---

# Cache

Use:
```text
Redis
```

Why?
- Fast access
- Reduce DB load

---

# Database Schema

```sql
CREATE TABLE url_mapping (
    id BIGINT PRIMARY KEY,
    short_url VARCHAR(20),
    long_url TEXT
);
```

---

# DB Sharding

## Why?

Huge traffic and data.

---

## Sharding Strategy

```text
userId % numberOfShards
```

---

# Bottlenecks

- DB hotspot
- Cache misses
- Redirection latency

---

# Scaling

- CDN
- Distributed cache
- Async analytics

---

# 2. MySQL Indexing

## What is Index?

Data structure improving query speed.

---

# B-Tree Index

MySQL uses:
```text
B-Tree
```

---

# Why B-Tree?

- Sorted data
- Fast search
- Logarithmic lookup

---

# Primary Index

Clustered index.

Data stored physically.

```sql
PRIMARY KEY(id)
```

---

# Secondary Index

Separate structure storing:
- indexed column
- primary key reference

---

# Composite Index

```sql
INDEX(name, age)
```

---

# Leftmost Prefix Rule

Works for:
```sql
(name)
(name, age)
```

Not for:
```sql
(age)
```

---

# EXPLAIN Query

Used to analyze query execution.

```sql
EXPLAIN SELECT * FROM users
WHERE age = 25;
```

---

# Important Columns in EXPLAIN

| Column | Meaning |
|---|---|
| type | access type |
| key | used index |
| rows | estimated rows |

---

# Interview Questions

## Why indexes improve performance?
Avoid full table scan.

---

## Why too many indexes bad?
Slower inserts and updates.

---

## Difference between clustered and non-clustered index?

| Clustered | Non-clustered |
|---|---|
| actual data stored | pointer/reference stored |

---

# Quick Revision

```text
URL Shortener → hashing + cache + DB
Redis → caching
Sharding → split database
B-Tree → indexing structure
EXPLAIN → query analysis
```

---

*End of Day 7 HLD + MySQL*