# Day 7 — HLD + MySQL

## Topics
- URL Shortener
- Database Normalization & ER Design
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

# 2. Database Normalization (Schema Design)

## Normal Forms

| Form | Rule |
|------|------|
| **1NF** | Atomic values; no repeating groups in one column |
| **2NF** | 1NF + no partial dependency on composite PK |
| **3NF** | 2NF + no transitive dependency (A→B→C) |
| **BCNF** | Every determinant is a candidate key |

**Denormalize when:** read-heavy analytics, cached aggregates — trade write complexity for read speed.

---

## ER → SQL Example (Library)

```sql
CREATE TABLE books (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    isbn VARCHAR(13) UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL
);

CREATE TABLE book_copies (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    book_id BIGINT NOT NULL,
    status ENUM('AVAILABLE','BORROWED') DEFAULT 'AVAILABLE',
    FOREIGN KEY (book_id) REFERENCES books(id) ON DELETE RESTRICT
);

CREATE TABLE loans (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    copy_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    borrowed_at DATETIME NOT NULL,
    FOREIGN KEY (copy_id) REFERENCES book_copies(id),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## Column Type Choices

| Data | Type |
|------|------|
| Money | `DECIMAL(19,4)` — never FLOAT |
| PK | `BIGINT AUTO_INCREMENT` or `CHAR(36)` UUID |
| Status | `ENUM` or lookup table |
| Soft delete | `deleted_at DATETIME NULL` |

---

# 3. MySQL Indexing

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
Normalization → 1NF/2NF/3NF, ER to SQL
Redis → caching
Sharding → split database
B-Tree → indexing structure
EXPLAIN → query analysis
```

---

*End of Day 7 HLD + MySQL*