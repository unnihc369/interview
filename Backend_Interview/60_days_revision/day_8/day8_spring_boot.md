# Day 8 — Spring Data JPA (Entities, CrudRepository, CRUD)

**Topics:** `@Entity` · `@Id` · `CrudRepository` · Service layer CRUD · Interview Q&A

---

# 1. What is Spring Data JPA?

Spring Data JPA simplifies database access by providing repository abstractions on top of JPA/Hibernate.

```text
Controller → Service → Repository → Database
```

You write interfaces; Spring generates implementations at runtime.

---

# 2. Entity Basics

## `@Entity` and `@Table`

```java
@Entity
@Table(name = "books")
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(nullable = false)
    private String author;

    private Double price;

    // constructors, getters, setters
}
```

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Marks JPA entity |
| `@Table` | Maps to table name |
| `@Id` | Primary key |
| `@GeneratedValue` | Auto-increment strategy |
| `@Column` | Column constraints |

---

# 3. CrudRepository

```java
public interface BookRepository extends CrudRepository<Book, Long> {
    // inherits: save, findById, findAll, deleteById, count, existsById
}
```

Enable scanning:

```java
@SpringBootApplication
public class BookstoreApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookstoreApplication.class, args);
    }
}
```

`application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/bookstore
spring.datasource.username=root
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

# 4. Service Layer CRUD

```java
@Service
public class BookService {

    private final BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public Book create(Book book) {
        return bookRepository.save(book);
    }

    public Optional<Book> getById(Long id) {
        return bookRepository.findById(id);
    }

    public Iterable<Book> getAll() {
        return bookRepository.findAll();
    }

    public Book update(Long id, Book updated) {
        Book existing = bookRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Book not found: " + id));
        existing.setTitle(updated.getTitle());
        existing.setAuthor(updated.getAuthor());
        existing.setPrice(updated.getPrice());
        return bookRepository.save(existing);
    }

    public void delete(Long id) {
        if (!bookRepository.existsById(id)) {
            throw new ResourceNotFoundException("Book not found: " + id);
        }
        bookRepository.deleteById(id);
    }
}
```

---

# 5. REST Controller CRUD

```java
@RestController
@RequestMapping("/api/v1/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @PostMapping
    public ResponseEntity<Book> create(@RequestBody Book book) {
        Book saved = bookService.create(book);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Book> get(@PathVariable Long id) {
        return bookService.getById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @GetMapping
    public Iterable<Book> getAll() {
        return bookService.getAll();
    }

    @PutMapping("/{id}")
    public Book update(@PathVariable Long id, @RequestBody Book book) {
        return bookService.update(id, book);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        bookService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

# 6. Repository Hierarchy (Interview)

```text
Repository (marker)
  └── CrudRepository<T, ID>
        └── PagingAndSortingRepository<T, ID>
              └── JpaRepository<T, ID>
```

| Interface | Extra features |
|-----------|----------------|
| CrudRepository | Basic CRUD |
| PagingAndSortingRepository | Pageable, Sort |
| JpaRepository | flush, batch delete, saveAll |

---

# 7. save() Behavior

- If `id` is null → INSERT
- If `id` exists → UPDATE (merge)

```java
Book b = new Book();
b.setTitle("Clean Code");
bookRepository.save(b); // insert

b.setPrice(29.99);
bookRepository.save(b); // update
```

---

# 8. DTO vs Entity (Best Practice)

Expose DTOs from API; keep entities inside service/repository.

```java
public record BookRequest(String title, String author, Double price) {}
public record BookResponse(Long id, String title, String author, Double price) {}
```

---

# Common Interview Questions

## Q1. What is JPA vs Hibernate vs Spring Data JPA?

| JPA | Hibernate | Spring Data JPA |
|-----|-----------|-----------------|
| Specification (API) | Implementation | Abstraction over JPA |
| Interfaces/annotations | ORM engine | Repository magic |

---

## Q2. What does `ddl-auto=update` do?

Hibernate updates schema to match entities (dev only). Production often uses `validate` + Flyway/Liquibase.

---

## Q3. Why interface for repository?

Spring creates proxy at runtime implementing CRUD + query methods.

---

## Q4. Difference between `save()` and `saveAndFlush()`?

`saveAndFlush()` forces SQL flush to DB immediately (JpaRepository).

---

## Q5. N+1 problem preview?

Lazy loading associations can cause many queries. Fix with `JOIN FETCH` or `@EntityGraph` (Day 9).

---

# One-Line Revision

```text
@Entity maps class to table; CrudRepository gives CRUD without boilerplate SQL.
Service owns business logic; Controller exposes REST.
```

---

*End of Day 8 Spring Boot*
