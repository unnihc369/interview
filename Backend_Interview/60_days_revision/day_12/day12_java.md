# Day 12 — Optional & java.time API

**Topics:** Null-safe Optional · LocalDate/Time · ZonedDateTime · Duration/Period · Interview Q&A

---

# 1. Optional Class

Introduced in Java 8 to represent optional values and reduce `NullPointerException`.

```java
Optional<String> name = Optional.of("Alice");
Optional<String> empty = Optional.empty();
Optional<String> nullable = Optional.ofNullable(getName()); // may be null inside
```

---

# Creating Optional

| Method | When |
|--------|------|
| `Optional.of(value)` | value must not be null |
| `Optional.ofNullable(value)` | value may be null → empty |
| `Optional.empty()` | explicit empty |

---

# Using Optional (Good Practices)

```java
// BAD — don't use like this
if (optional.isPresent()) {
    System.out.println(optional.get());
}

// GOOD
optional.ifPresent(System.out::println);

String value = optional.orElse("default");
String value2 = optional.orElseGet(() -> computeDefault());
String value3 = optional.orElseThrow(() -> new NotFoundException("missing"));

String mapped = optional.map(String::toUpperCase).orElse("N/A");
```

### flatMap for nested Optional

```java
Optional<User> user = findUserId(id)
    .flatMap(userRepository::findById);
```

---

# Anti-patterns (Interview)

- Don't use `Optional` as method parameter everywhere.
- Don't use `Optional` for fields/serialization by default.
- Don't call `get()` without check.

```java
// BAD
return optional.get();

// GOOD
return optional.orElseThrow();
```

---

# 2. java.time API (JSR-310)

Replaces legacy `Date` / `Calendar` — immutable, thread-safe, clear API.

---

# Core Classes

| Class | Purpose |
|-------|---------|
| `LocalDate` | Date without time (2024-05-29) |
| `LocalTime` | Time without date (14:30:00) |
| `LocalDateTime` | Date + time, no timezone |
| `ZonedDateTime` | Date-time with timezone |
| `Instant` | UTC timestamp (machine epoch) |
| `Duration` | Time-based amount (hours, minutes) |
| `Period` | Date-based amount (days, months) |

---

# Examples

```java
LocalDate today = LocalDate.now();
LocalDate nextWeek = today.plusWeeks(1);
LocalDate parsed = LocalDate.parse("2024-05-29");

LocalTime now = LocalTime.now();
LocalDateTime meeting = LocalDateTime.of(2024, 5, 29, 10, 30);

ZonedDateTime ist = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));
Instant instant = Instant.now();

Duration d = Duration.between(start, end);
Period p = Period.between(birthDate, today);
```

---

# Formatting & Parsing

```java
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm");
String text = meeting.format(fmt);
LocalDateTime parsed = LocalDateTime.parse("29-05-2024 10:30", fmt);

// ISO-8601 built-in
LocalDate.parse("2024-05-29");
```

---

# Converting Legacy Date

```java
Date legacy = new Date();
Instant instant = legacy.toInstant();
LocalDateTime ldt = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
```

---

# 3. Practical Example — Order SLA

```java
public boolean isOverdue(Order order) {
    LocalDateTime deadline = order.getCreatedAt().plusHours(48);
    return LocalDateTime.now().isAfter(deadline);
}

public long hoursUntilExpiry(Instant expiresAt) {
    return Duration.between(Instant.now(), expiresAt).toHours();
}
```

---

# 4. Optional + Repository

```java
public UserResponse getUser(Long id) {
    return userRepository.findById(id)
        .map(UserResponse::from)
        .orElseThrow(() -> new ResourceNotFoundException("User " + id));
}
```

---

# Common Interview Questions

## Q1. Why Optional if we can use null checks?

Expresses intent in API; encourages caller to handle absence; composable map/flatMap.

---

## Q2. LocalDateTime vs ZonedDateTime?

`LocalDateTime` — no timezone (meetings in one zone, logs).  
`ZonedDateTime` — global apps, DST rules, user-local display.

---

## Q3. Instant vs LocalDateTime?

`Instant` — point on UTC timeline (good for DB/storage).  
`LocalDateTime` — human calendar without zone context.

---

## Q4. Duration vs Period?

Duration = hours/minutes/seconds between times.  
Period = years/months/days between dates.

---

## Q5. Is java.time thread-safe?

Yes — all classes immutable.

---

# One-Line Revision

```text
Optional = explicit absence, avoid get() without handling.
java.time = LocalDate/Time/DateTime, ZonedDateTime, Instant, Duration, Period.
```

---

*End of Day 12 Java*
