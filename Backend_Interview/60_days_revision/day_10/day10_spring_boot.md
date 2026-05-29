# Day 10 — JPA Query Methods, @Query & Pageable

**Topics:** Derived queries · JPQL · Native SQL · Pagination · Interview Q&A

---

# 1. Derived Query Methods

Spring Data parses method names into queries.

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    List<Product> findByCategory(String category);

    Optional<Product> findBySku(String sku);

    List<Product> findByPriceBetween(double min, double max);

    List<Product> findByNameContainingIgnoreCase(String keyword);

    long countByCategory(String category);

    boolean existsBySku(String sku);

    void deleteByCategory(String category);
}
```

| Keyword | Example |
|---------|---------|
| `And` / `Or` | `findByNameAndCategory` |
| `Between` | `findByPriceBetween` |
| `Like` | `findByNameLike` |
| `OrderBy` | `findByCategoryOrderByPriceDesc` |
| `Top` / `First` | `findTop5ByOrderByPriceDesc` |

---

# 2. @Query (JPQL)

```java
@Query("SELECT p FROM Product p WHERE p.category = :cat AND p.price < :maxPrice")
List<Product> findAffordable(@Param("cat") String cat, @Param("maxPrice") double maxPrice);

@Query("SELECT new com.example.ProductSummary(p.id, p.name, p.price) FROM Product p WHERE p.active = true")
List<ProductSummary> findActiveSummaries();
```

JPQL uses **entity names and fields**, not table columns.

---

# 3. Native Query

```java
@Query(value = "SELECT * FROM products WHERE category = ?1", nativeQuery = true)
List<Product> findByCategoryNative(String category);
```

Use when:

- Complex SQL (window functions, DB-specific features)
- Legacy schema not mapping cleanly to entities

Trade-off: less portable, bypasses entity cache.

---

# 4. Modifying Queries

```java
@Modifying
@Transactional
@Query("UPDATE Product p SET p.price = p.price * :multiplier WHERE p.category = :cat")
int applyDiscount(@Param("cat") String cat, @Param("multiplier") double multiplier);
```

Requires `@Transactional` on service method. Clear persistence context after bulk updates if needed.

---

# 5. Pageable & Sort

```java
Page<Product> findByCategory(String category, Pageable pageable);

List<Product> findByCategory(String category, Sort sort);
```

Controller:

```java
@GetMapping
public Page<Product> list(
        @RequestParam String category,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    return productRepository.findByCategory(
        category,
        PageRequest.of(page, size, Sort.by("price").descending())
    );
}
```

Response includes:

```json
{
  "content": [...],
  "totalElements": 150,
  "totalPages": 8,
  "number": 0,
  "size": 20
}
```

---

# 6. Specifications (Dynamic Filters) — Bonus

```java
public interface ProductRepository extends JpaRepository<Product, Long>, JpaSpecificationExecutor<Product> {}

Specification<Product> spec = (root, query, cb) ->
    cb.and(
        cb.equal(root.get("category"), "BOOKS"),
        cb.lessThan(root.get("price"), 100)
    );

Page<Product> page = productRepository.findAll(spec, pageable);
```

---

# 7. Projection Interfaces

```java
public interface ProductView {
    String getName();
    Double getPrice();
}

List<ProductView> findByCategory(String category);
```

Spring creates proxy — only selected columns fetched.

---

# Common Interview Questions

## Q1. JPQL vs SQL?

JPQL is object-oriented (entities); SQL is table-oriented. JPQL portable across DBs (mostly).

---

## Q2. How does Spring generate implementation?

`SimpleJpaRepository` + `PartTreeJpaQuery` parse method name at startup.

---

## Q3. N+1 with pagination?

`JOIN FETCH` + `Pageable` is tricky — use `@EntityGraph` or separate query for IDs then fetch.

---

## Q4. `Page` vs `Slice`?

`Page` runs count query for total elements. `Slice` only knows if next page exists — cheaper for infinite scroll.

---

## Q5. When native query?

Reporting, complex aggregations, DB-specific optimizations.

---

# One-Line Revision

```text
Derived methods = naming convention queries.
@Query = custom JPQL/SQL; Pageable = paginated sorted results.
```

---

*End of Day 10 Spring Boot*
