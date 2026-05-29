# Day 9 — JPA Relationships (OneToMany, ManyToOne, ManyToMany)

**Topics:** Cardinality · Owning side · Cascade · Fetch type · Interview Q&A

---

# 1. Relationship Overview

| Annotation | Meaning |
|------------|---------|
| `@OneToOne` | One A ↔ one B |
| `@OneToMany` | One A ↔ many B |
| `@ManyToOne` | Many A ↔ one B |
| `@ManyToMany` | Many A ↔ many B |

---

# 2. ManyToOne / OneToMany (Bidirectional)

**Example:** Many `Order` items belong to one `Customer`.

```java
@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();

    public void addOrder(Order order) {
        orders.add(order);
        order.setCustomer(this);
    }
}

@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private double amount;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;
}
```

---

# Owning Side (Interview Critical)

- **`@ManyToOne`** side owns foreign key (`customer_id` in `orders` table).
- **`@OneToMany(mappedBy = "...")`** is inverse side — no extra FK column.

Always update **both sides** in bidirectional helpers to keep object graph consistent.

---

# 3. ManyToMany

**Example:** Students and Courses.

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}
```

JPA creates join table `student_course` with composite key columns.

---

# 4. Cascade Types

```java
cascade = CascadeType.ALL
// PERSIST, MERGE, REMOVE, REFRESH, DETACH
```

| Cascade | Effect |
|---------|--------|
| PERSIST | Save child with parent |
| REMOVE | Delete children when parent deleted |
| MERGE | Merge children |
| ALL | All of above |

`orphanRemoval = true` — delete child removed from collection.

---

# 5. FetchType: LAZY vs EAGER

```java
@ManyToOne(fetch = FetchType.LAZY)   // default for ManyToOne
@OneToMany(fetch = FetchType.LAZY)   // default for OneToMany
```

| LAZY | EAGER |
|------|-------|
| Load on access | Load with parent |
| Better default | Risk of N+1 or huge joins |

**N+1 problem:**

```java
List<Customer> customers = customerRepo.findAll();
for (Customer c : customers) {
    c.getOrders().size(); // triggers query per customer
}
```

**Fix:**

```java
@Query("SELECT c FROM Customer c JOIN FETCH c.orders")
List<Customer> findAllWithOrders();
```

---

# 5b. @EntityGraph (Alternative N+1 Fix)

```java
@EntityGraph(attributePaths = {"orders", "orders.items"})
@Query("SELECT c FROM Customer c WHERE c.id = :id")
Optional<Customer> findWithOrders(@Param("id") Long id);

// Or on repository method:
@EntityGraph(attributePaths = "orders")
List<Customer> findByActiveTrue();
```

| JOIN FETCH | @EntityGraph |
|------------|--------------|
| In JPQL query | Declarative on method |
| Hard with Pageable | Works better with pagination |
| Explicit | Reusable attribute paths |

---

# 5c. Optimistic Locking (@Version)

```java
@Entity
public class Product {
    @Id
    private Long id;

    @Version
    private Long version; // auto-incremented on each update

    private int stock;
}
```

```java
// Concurrent updates:
// Thread A and B read stock=10
// A saves → version 1→2, stock=9
// B saves → OptimisticLockException (stale version)
```

**Use when:** low contention, read-heavy — avoids DB row locks.

---

# 5d. Flyway / Liquibase (Schema Migrations)

**Never use `ddl-auto=update` in production.**

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```sql
-- db/migration/V1__create_users.sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

```properties
spring.jpa.hibernate.ddl-auto=validate
spring.flyway.enabled=true
```

| Tool | Style |
|------|-------|
| Flyway | Versioned SQL files (V1__, V2__) |
| Liquibase | XML/YAML/SQL changelogs |

---

# 5e. DTO Mapping (MapStruct)

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserResponse toResponse(User entity);
    User toEntity(UserRequest dto);
}
```

**Why not expose Entity in REST?** Lazy loading errors, schema leakage, tight coupling.

---

# 6. Repository with Relationships

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerId(Long customerId);
}
```

---

# 7. DTO Projection (Best Practice)

Avoid returning entities with lazy collections in REST.

```java
public record OrderResponse(Long id, double amount, String customerName) {}
```

Map in service layer to prevent serialization errors and leaking schema.

---

# Common Interview Questions

## Q1. Difference between `mappedBy` and `@JoinColumn`?

`mappedBy` — inverse side, no FK.  
`@JoinColumn` — owning side, FK column created.

---

## Q2. unidirectional vs bidirectional?

Unidirectional: only one side navigates.  
Bidirectional: both sides reference each other (need sync helpers).

---

## Q3. Why LAZY default?

Performance — don't load entire object graph unless needed.

---

## Q4. `@Transactional` required for LAZY?

Yes. Accessing lazy collection outside transaction → `LazyInitializationException`.

---

## Q5. ManyToMany performance issue?

Large join tables; consider link entity with extra fields (enrollment date, role).

```java
@Entity
public class Enrollment {
    @ManyToOne Student student;
    @ManyToOne Course course;
    private LocalDate enrolledOn;
}
```

---

## Q6. @EntityGraph vs JOIN FETCH?

Both solve N+1. JOIN FETCH in JPQL; @EntityGraph declarative on repository — better with Pageable.

---

## Q7. When use @Version?

Optimistic locking for concurrent updates — throws OptimisticLockException on stale version.

---

## Q8. Flyway vs ddl-auto=update?

Production: **validate** + Flyway migrations. ddl-auto=update is dev-only — no version control, risky in prod.

---

# One-Line Revision

```text
ManyToOne owns FK; OneToMany uses mappedBy.
N+1 fix: JOIN FETCH or @EntityGraph.
@Version = optimistic locking.
Flyway = versioned schema migrations.
DTO + MapStruct = never expose entities in REST.
```

---

*End of Day 9 Spring Boot*
