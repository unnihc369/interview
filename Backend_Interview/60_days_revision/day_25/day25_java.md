# Day 25 - java.util.regex Pattern & Matcher

---

# 1. Overview

Java regex lives in `java.util.regex`:

| Class | Role |
|-------|------|
| `Pattern` | Compiled regular expression (immutable, thread-safe) |
| `Matcher` | Engine that performs match operations on input |

```text
Pattern.compile(regex) → Pattern
pattern.matcher(input) → Matcher
matcher.find/matches/... → results
```

---

# 2. Basic Matching

```java
import java.util.regex.*;

String text = "Order ID: ORD-12345 confirmed";
Pattern pattern = Pattern.compile("ORD-\\d+");
Matcher matcher = pattern.matcher(text);

if (matcher.find()) {
    System.out.println("Found: " + matcher.group()); // ORD-12345
    System.out.println("Start: " + matcher.start()); // 10
    System.out.println("End: " + matcher.end());     // 19
}
```

---

# 3. find vs matches vs lookingAt

| Method | Matches |
|--------|---------|
| `matches()` | **Entire** input must match pattern |
| `lookingAt()` | From beginning, rest can remain |
| `find()` | Search anywhere in input |

```java
Pattern p = Pattern.compile("\\d+");

p.matcher("abc123").find();      // true (123 found)
p.matcher("abc123").matches();   // false (whole string)
p.matcher("123abc").lookingAt(); // true (starts with digits)
```

---

# 4. Common Regex Metacharacters

| Symbol | Meaning |
|--------|---------|
| `.` | Any char (except newline) |
| `\d` | Digit [0-9] |
| `\D` | Non-digit |
| `\w` | Word char [a-zA-Z0-9_] |
| `\s` | Whitespace |
| `[abc]` | Character class |
| `[^abc]` | Negated class |
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one |
| `{n,m}` | Between n and m |
| `^` | Start of line |
| `$` | End of line |
| `( )` | Capturing group |
| `(?: )` | Non-capturing group |
| `(?<name> )` | Named group (Java 7+) |

---

# 5. Capturing Groups

```java
Pattern p = Pattern.compile("(\\d{4})-(\\d{2})-(\\d{2})");
Matcher m = p.matcher("2026-05-29");

if (m.matches()) {
    System.out.println(m.group(0)); // full: 2026-05-29
    System.out.println(m.group(1)); // 2026
    System.out.println(m.group(2)); // 05
    System.out.println(m.group(3)); // 29
}
```

Named groups:

```java
Pattern p = Pattern.compile("(?<year>\\d{4})-(?<month>\\d{2})-(?<day>\\d{2})");
Matcher m = p.matcher("2026-05-29");
if (m.matches()) {
    System.out.println(m.group("year"));  // 2026
}
```

---

# 6. Replace & Split

```java
String input = "user@email.com, admin@corp.org";

// Replace all emails
String masked = input.replaceAll("[\\w.-]+@[\\w.-]+", "***@***");

// Matcher appendReplacement for complex replace
Pattern p = Pattern.compile("(\\w+)@(\\w+)");
Matcher m = p.matcher(input);
StringBuffer sb = new StringBuffer();
while (m.find()) {
    m.appendReplacement(sb, m.group(1) + "@masked");
}
m.appendTail(sb);

// Split
String[] parts = "a,b,c".split(",");
```

---

# 7. Flags

```java
Pattern p = Pattern.compile("hello", Pattern.CASE_INSENSITIVE);
Pattern p2 = Pattern.compile("^start", Pattern.MULTILINE);
Pattern p3 = Pattern.compile(".*", Pattern.DOTALL); // . matches newline
```

Compile once, reuse — compilation is expensive.

---

# 8. Performance — Pattern.compile vs String.matches

```java
// BAD in loop — recompiles every call
if (email.matches("^[\\w.-]+@[\\w.-]+\\.\\w+$")) { }

// GOOD — compile once
private static final Pattern EMAIL = Pattern.compile("^[\\w.-]+@[\\w.-]+\\.\\w+$");
if (EMAIL.matcher(email).matches()) { }
```

---

# 9. Real-World Backend Examples

## Input Validation

```java
private static final Pattern PHONE = Pattern.compile("^\\+?[1-9]\\d{9,14}$");

public void validatePhone(String phone) {
    if (!PHONE.matcher(phone).matches()) {
        throw new ValidationException("Invalid phone");
    }
}
```

## Log Parsing

```java
Pattern LOG = Pattern.compile(
    "(?<ts>\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}:\\d{2}) (?<level>\\w+) (?<msg>.*)"
);
Matcher m = LOG.matcher(logLine);
if (m.matches()) {
    String level = m.group("level");
}
```

## URL Path Extraction

```java
Pattern API = Pattern.compile("/api/v1/users/(\\d+)/orders/(\\d+)");
```

---

# 10. Regex vs Manual Parsing

| Use Regex | Avoid Regex |
|-----------|-------------|
| Validation (email, PAN, IFSC) | HTML/XML parsing (use parser) |
| Log extraction | Complex nested structures |
| Simple tokenization | Performance-critical hot paths |

---

# Common Interview Questions

## Q1. Pattern vs Matcher thread safety?

Pattern is immutable and thread-safe. Matcher is NOT — create per thread.

## Q2. Greedy vs reluctant quantifiers?

`*` greedy (max match), `*?` reluctant (min match). Important for nested patterns.

## Q3. How to escape regex special chars?

`Pattern.quote("literal.text")` wraps with `\Q...\E`.

## Q4. Difference from LeetCode regex DP?

Java regex engine uses NFA/backtracking — different from simplified `.*` / `a*` interview problem.

## Q5. ReDoS (Regular Expression Denial of Service)?

Catastrophic backtracking on evil input — use possessive quantifiers or limit input length.

---

# One-Line Revision

```text
Compile Pattern once, use Matcher for find/matches/groups; reuse patterns, never recompile in hot loops.
```

---

*End of Day 25 Java*
