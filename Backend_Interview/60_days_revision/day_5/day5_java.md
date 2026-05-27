# Day 4 — Java

## Exception Handling in Java

Topics:
- try-catch
- finally
- throw
- throws
- custom exceptions

---

# What is Exception?

Exception is abnormal condition during program execution.

Examples:
- divide by zero
- file not found
- null pointer

---

# Types of Exceptions

| Type | Example |
|---|---|
| Checked | IOException |
| Unchecked | NullPointerException |

---

# try-catch

```java
public class Main {

    public static void main(String[] args) {

        try {

            int result = 10 / 0;

        } catch (ArithmeticException e) {

            System.out.println("Cannot divide by zero");
        }
    }
}
```

---

# finally Block

Always executes.

```java
try {

} catch(Exception e) {

} finally {
    System.out.println("Cleanup");
}
```

---

# throw Keyword

Used to manually throw exception.

```java
throw new RuntimeException("Invalid data");
```

---

# throws Keyword

Declares exception.

```java
public void readFile() throws IOException {

}
```

---

# Custom Exception

## Why?

Business-specific exceptions.

---

## Example

```java
class InvalidAgeException extends Exception {

    public InvalidAgeException(String message) {
        super(message);
    }
}
```

---

## Usage

```java
public class Main {

    public static void validateAge(int age)
            throws InvalidAgeException {

        if (age < 18) {
            throw new InvalidAgeException(
                "Age must be at least 18"
            );
        }
    }
}
```

---

# Exception Hierarchy

```text
Throwable
   ├── Error
   └── Exception
         ├── RuntimeException
         └── Checked Exceptions
```

---

# Interview Questions

## Difference between checked and unchecked exceptions?

Checked:
- checked at compile time

Unchecked:
- occur at runtime

---

## Difference between throw and throws?

| throw | throws |
|---|---|
| used inside method | used in method signature |

---

## Can finally block not execute?
Yes:
- JVM crash
- System.exit()

---

# Quick Revision

```text
try → risky code
catch → handle exception
finally → cleanup
throw → manually throw
throws → declare exception
```

---

*End of Day 4 Java*