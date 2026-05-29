# Day 12 — Mini-Project: BookStore REST API with JPA

**Topics:** Full layered example · Entity · Repository · Service · Controller · Exception handling · Interview Q&A

---

# 1. Project Structure

```text
bookstore/
├── src/main/java/com/example/bookstore/
│   ├── BookstoreApplication.java
│   ├── entity/Book.java
│   ├── repository/BookRepository.java
│   ├── dto/BookRequest.java, BookResponse.java
│   ├── service/BookService.java
│   ├── controller/BookController.java
│   └── exception/GlobalExceptionHandler.java
└── src/main/resources/application.properties
```

---

# 2. Entity

```java
@Entity
@Table(name = "books")
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String author;

    @Column(nullable = false)
    private BigDecimal price;

    private boolean available = true;

    // constructors, getters, setters
}
```

---

# 3. Repository

```java
public interface BookRepository extends JpaRepository<Book, Long> {

    List<Book> findByAuthorContainingIgnoreCase(String author);

    Page<Book> findByAvailableTrue(Pageable pageable);

    @Query("SELECT b FROM Book b WHERE b.price BETWEEN :min AND :max")
    List<Book> findByPriceRange(@Param("min") BigDecimal min, @Param("max") BigDecimal max);
}
```

---

# 4. DTOs

```java
public record BookRequest(
    @NotBlank String title,
    @NotBlank String author,
    @NotNull @Positive BigDecimal price
) {}

public record BookResponse(Long id, String title, String author, BigDecimal price, boolean available) {
    public static BookResponse from(Book b) {
        return new BookResponse(b.getId(), b.getTitle(), b.getAuthor(), b.getPrice(), b.isAvailable());
    }
}
```

---

# 5. Service Layer

```java
@Service
@Transactional(readOnly = true)
public class BookService {

    private final BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public Page<BookResponse> listAvailable(Pageable pageable) {
        return bookRepository.findByAvailableTrue(pageable)
            .map(BookResponse::from);
    }

    public BookResponse getById(Long id) {
        return bookRepository.findById(id)
            .map(BookResponse::from)
            .orElseThrow(() -> new ResourceNotFoundException("Book not found: " + id));
    }

    @Transactional
    public BookResponse create(BookRequest req) {
        Book book = new Book();
        book.setTitle(req.title());
        book.setAuthor(req.author());
        book.setPrice(req.price());
        return BookResponse.from(bookRepository.save(book));
    }

    @Transactional
    public BookResponse update(Long id, BookRequest req) {
        Book book = bookRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Book not found: " + id));
        book.setTitle(req.title());
        book.setAuthor(req.author());
        book.setPrice(req.price());
        return BookResponse.from(book);
    }

    @Transactional
    public void delete(Long id) {
        if (!bookRepository.existsById(id)) {
            throw new ResourceNotFoundException("Book not found: " + id);
        }
        bookRepository.deleteById(id);
    }

    @Transactional
    public BookResponse purchase(Long id) {
        Book book = bookRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Book not found: " + id));
        if (!book.isAvailable()) {
            throw new BusinessException("Book already sold");
        }
        book.setAvailable(false);
        return BookResponse.from(book);
    }
}
```

---

# 6. REST Controller

```java
@RestController
@RequestMapping("/api/v1/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @GetMapping
    public Page<BookResponse> list(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return bookService.listAvailable(PageRequest.of(page, size, Sort.by("title")));
    }

    @GetMapping("/{id}")
    public BookResponse get(@PathVariable Long id) {
        return bookService.getById(id);
    }

    @PostMapping
    public ResponseEntity<BookResponse> create(@Valid @RequestBody BookRequest req) {
        BookResponse created = bookService.create(req);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public BookResponse update(@PathVariable Long id, @Valid @RequestBody BookRequest req) {
        return bookService.update(id, req);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        bookService.delete(id);
        return ResponseEntity.noContent().build();
    }

    @PostMapping("/{id}/purchase")
    public BookResponse purchase(@PathVariable Long id) {
        return bookService.purchase(id);
    }
}
```

---

# 7. Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Map<String, String>> notFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(Map.of("error", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> validation(MethodArgumentNotValidException ex) {
        String msg = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest().body(Map.of("error", msg));
    }
}
```

---

# 8. application.properties

```properties
spring.application.name=bookstore
spring.datasource.url=jdbc:mysql://localhost:3306/bookstore
spring.datasource.username=root
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
server.port=8080
```

---

# 9. API Test Flow (curl)

```bash
# Create
curl -X POST http://localhost:8080/api/v1/books \
  -H "Content-Type: application/json" \
  -d '{"title":"Clean Code","author":"Robert Martin","price":29.99}'

# List
curl "http://localhost:8080/api/v1/books?page=0&size=10"

# Purchase
curl -X POST http://localhost:8080/api/v1/books/1/purchase
```

---

# 10. Layer Responsibilities (Interview)

| Layer | Responsibility |
|-------|----------------|
| Controller | HTTP, validation, status codes |
| Service | Business rules, transactions |
| Repository | Data access |
| Entity | DB mapping |
| DTO | API contract |

---

# Common Interview Questions

## Q1. Why records for DTOs?

Immutable, concise, good for read-only API payloads (Java 16+).

---

## Q2. Where to put @Transactional?

Service layer — spans multiple repository calls.

---

## Q3. Why not return Entity from controller?

Coupling, lazy-load serialization issues, exposes schema.

---

## Q4. How to add security next?

Spring Security + JWT; protect POST/DELETE; public GET.

---

## Q5. How to test?

`@WebMvcTest` for controller; `@DataJpaTest` for repository; `@SpringBootTest` integration.

---

# One-Line Revision

```text
BookStore = Entity + JpaRepository + @Transactional Service + REST Controller + DTOs + ExceptionHandler.
```

---

*End of Day 12 Spring Boot*
