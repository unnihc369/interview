# Day 21 — HLD: Amazon Product Search + MySQL ACID & Isolation Levels

**Topics:** Search System Design · Inverted Index · Elasticsearch · ACID · Isolation Levels · Interview Questions

---

# Part 1: Amazon Product Search — HLD

---

# 1. Functional Requirements

- Search products by keyword, category, filters (price, rating, brand)
- Autocomplete / typeahead suggestions
- Sort by relevance, price, rating, popularity
- Faceted search (filter by attributes)
- Handle millions of products, thousands of QPS

---

# 2. Non-Functional Requirements

- Low latency: < 200ms search response
- High availability: 99.99%
- Relevance ranking
- Eventually consistent indexing (new products searchable within minutes)

---

# 3. High-Level Architecture

```text
Client (Web/App)
      ↓
CDN (static assets)
      ↓
Load Balancer
      ↓
API Gateway
      ↓
┌─────────────────────────────────────┐
│  Search Service                     │
│  ├── Query Parser                   │
│  ├── Search Engine (Elasticsearch)  │
│  └── Ranking Service                │
└─────────────────────────────────────┘
      ↓                    ↓
Autocomplete Service   Product Catalog DB (MySQL)
      ↓                    ↓
   Redis Cache         Index Pipeline (Kafka)
                            ↓
                      Elasticsearch Cluster
```

---

# 4. Search Flow

```text
User types "wireless headphones"
        ↓
Autocomplete Service (prefix match from Trie/ES completion)
        ↓
User submits search
        ↓
Query Parser (tokenize, spell-check, synonyms)
        ↓
Elasticsearch (inverted index lookup + filters)
        ↓
Ranking Service (relevance score + business rules)
        ↓
Return paginated results with facets
```

---

# 5. Inverted Index — Core Concept

Maps each word → list of documents containing it.

```text
"wireless" → [doc1, doc5, doc12, doc45]
"headphone" → [doc1, doc3, doc5, doc12]
"bluetooth" → [doc1, doc5, doc20]

Query "wireless headphone":
  Intersect [doc1, doc5, doc12] → ranked results
```

Elasticsearch builds and maintains inverted indexes automatically.

---

# 6. Elasticsearch Index Design

```json
{
  "mappings": {
    "properties": {
      "product_id": { "type": "keyword" },
      "title": { "type": "text", "analyzer": "english" },
      "description": { "type": "text" },
      "category": { "type": "keyword" },
      "brand": { "type": "keyword" },
      "price": { "type": "float" },
      "rating": { "type": "float" },
      "review_count": { "type": "integer" },
      "popularity_score": { "type": "float" },
      "in_stock": { "type": "boolean" }
    }
  }
}
```

---

# 7. Ranking / Relevance

Multi-signal ranking:

```text
Final Score = α * text_relevance (BM25)
            + β * popularity_score
            + γ * rating
            + δ * sales_velocity
            + business_boost (sponsored, in-stock)
```

BM25: Best Match 25 — standard TF-IDF variant used by Elasticsearch.

---

# 8. Autocomplete / Typeahead

```text
User types "wire"
        ↓
Prefix query on completion field
        ↓
Return top 10 suggestions from Trie or ES completion suggester
```

Implementation options:
- **Trie** in memory (Redis) — fast prefix lookup
- **Elasticsearch completion suggester** — integrated, FST-based

---

# 9. Index Pipeline (Data Sync)

Product changes must flow to search index:

```text
Product CRUD (MySQL)
        ↓
CDC / Application Event (Kafka)
        ↓
Index Consumer Service
        ↓
Elasticsearch bulk index/update/delete
```

Use **Change Data Capture (Debezium)** or application-level events.

Latency target: product searchable within 1-5 minutes of creation.

---

# 10. Caching Strategy

| Cache | Data | TTL |
|-------|------|-----|
| Redis | Popular search results | 5-15 min |
| Redis | Autocomplete suggestions | 1 hour |
| CDN | Static product images | Long |
| ES query cache | Frequent filter queries | ES managed |

Cache key: `search:{query_hash}:{filters_hash}:{page}`

---

# 11. Faceted Search

Return aggregations alongside results:

```json
{
  "results": [...],
  "facets": {
    "brands": [{"Sony": 45}, {"Bose": 32}],
    "price_ranges": [{"0-50": 120}, {"50-100": 85}],
    "ratings": [{"4+": 200}, {"3+": 350}]
  }
}
```

Elasticsearch aggregations power facets natively.

---

# 12. Scaling Elasticsearch

```text
Index sharding: products index → 5-10 shards
Replica shards: 1-2 replicas for read scaling
Dedicated master nodes: cluster coordination
Hot-warm architecture: recent products on fast nodes
```

---

# 13. Database Schema (Product Catalog — MySQL)

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(500) NOT NULL,
    description TEXT,
    category_id BIGINT,
    brand_id BIGINT,
    price DECIMAL(10,2),
    rating DECIMAL(3,2),
    review_count INT DEFAULT 0,
    in_stock BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_category (category_id),
    INDEX idx_brand (brand_id),
    INDEX idx_price (price),
    FULLTEXT INDEX ft_title_desc (title, description)
);

CREATE TABLE categories (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    parent_id BIGINT,
    INDEX idx_parent (parent_id)
);
```

MySQL FULLTEXT for simple search; Elasticsearch for production scale.

---

# 14. Bottlenecks & Mitigations

| Bottleneck | Mitigation |
|------------|------------|
| ES cluster overload | Rate limiting, query caching, read replicas |
| Index lag | Parallel index consumers, bulk indexing |
| Hot queries | Redis cache for top 1000 queries |
| Large result sets | Cursor-based pagination, limit max page |
| Spell-check latency | Pre-compute dictionary, async correction |

---

# Part 2: MySQL ACID & Isolation Levels

---

# 15. What is ACID?

| Property | Meaning |
|----------|---------|
| **A**tomicity | All or nothing — transaction fully completes or fully rolls back |
| **C**onsistency | Database moves from one valid state to another (constraints honored) |
| **I**solation | Concurrent transactions don't interfere with each other |
| **D**urability | Committed data survives crashes (written to disk/WAL) |

---

# 16. Atomicity Example

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- If second UPDATE fails → ROLLBACK both
COMMIT;
```

---

# 17. Isolation Levels

SQL standard defines 4 isolation levels:

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|---------------------|--------------|
| READ UNCOMMITTED | Yes | Yes | Yes |
| READ COMMITTED | No | Yes | Yes |
| REPEATABLE READ | No | No | Yes* |
| SERIALIZABLE | No | No | No |

*MySQL InnoDB REPEATABLE READ prevents phantom reads via next-key locking.

---

# 18. Phenomena Explained

## Dirty Read
Transaction A reads uncommitted data from Transaction B. B rolls back → A read invalid data.

## Non-Repeatable Read
Transaction A reads row. Transaction B updates and commits. A reads same row again — different value.

## Phantom Read
Transaction A runs range query. Transaction B inserts matching row and commits. A runs same query — extra rows appear.

---

# 19. MySQL Default Isolation Level

```sql
SELECT @@transaction_isolation;
-- Default: REPEATABLE-READ (InnoDB)
```

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
-- queries run at READ COMMITTED level
COMMIT;
```

---

# 20. Spring @Transactional Isolation

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void transferMoney(Long from, Long to, BigDecimal amount) {
    Account source = accountRepo.findById(from).orElseThrow();
    Account dest = accountRepo.findById(to).orElseThrow();

    source.debit(amount);
    dest.credit(amount);

    accountRepo.save(source);
    accountRepo.save(dest);
}
```

---

# Isolation Level Selection Guide

| Use Case | Recommended Level |
|----------|-------------------|
| Reporting/analytics | READ COMMITTED |
| Financial transactions | REPEATABLE READ or SERIALIZABLE |
| High-concurrency reads | READ COMMITTED |
| Inventory/stock deduction | REPEATABLE READ + SELECT FOR UPDATE |

---

# 21. Locking in MySQL

## Pessimistic Lock

```sql
SELECT * FROM products WHERE id = 1 FOR UPDATE;
-- Locks row until transaction commits
```

## Optimistic Lock

```java
@Entity
public class Product {
    @Version
    private Long version;
}
```

JPA increments version on update. Concurrent update throws `OptimisticLockException`.

---

# 22. Deadlock Example

```text
Txn A: locks row 1, waits for row 2
Txn B: locks row 2, waits for row 1
→ Deadlock → MySQL kills one transaction
```

Prevention:
- Access rows in consistent order
- Keep transactions short
- Use appropriate isolation level

---

# 23. ACID in Product Search Context

| Operation | ACID Concern |
|-----------|-------------|
| Place order | Atomicity — deduct stock + create order together |
| Update inventory | Isolation — prevent overselling (REPEATABLE READ + lock) |
| Index product | Durability — product data persisted before indexing |
| Payment processing | Consistency — balance constraints maintained |

---

# Interview Questions

## MySQL vs Elasticsearch for search?

| MySQL FULLTEXT | Elasticsearch |
|----------------|---------------|
| Simple, no extra infra | Scales to billions of docs |
| Limited relevance ranking | BM25, facets, autocomplete |
| Good for <100K products | Production search at scale |

Use both: MySQL as source of truth, ES as search index.

---

## How does Amazon show results in <200ms?

- Pre-built inverted index in ES
- Redis cache for hot queries
- CDN for images
- Pagination (not load all results)
- Async facet computation

---

## READ COMMITTED vs REPEATABLE READ?

READ COMMITTED: each statement sees latest committed data (Oracle/SQL Server default).

REPEATABLE READ: all statements in transaction see same snapshot (MySQL InnoDB default).

---

## When to use SERIALIZABLE?

When absolute consistency required and concurrency is low. High lock contention — use sparingly. Financial reconciliation, inventory audits.

---

## Eventual consistency in search index?

Product created in MySQL (ACID). Index update via Kafka is async. Brief window where product exists but not searchable. Acceptable for most e-commerce — mention SLA (1-5 min).

---

# Quick Revision

```text
Product Search → ES inverted index + ranking + facets
Index Pipeline → MySQL → Kafka → Elasticsearch
Autocomplete   → Trie or ES completion suggester
ACID           → Atomicity, Consistency, Isolation, Durability
Isolation      → READ UNCOMMITTED < COMMITTED < REPEATABLE READ < SERIALIZABLE
MySQL default  → REPEATABLE READ (InnoDB)
FOR UPDATE     → pessimistic row lock
@Version       → optimistic locking in JPA
```

---

*End of Day 21 HLD + MySQL*
