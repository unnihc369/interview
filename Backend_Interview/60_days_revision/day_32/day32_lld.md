# Day 32 — LLD: Logging Framework

**Topics:** Logger Hierarchy · Appenders · Formatters · Thread Safety · Interview Design

---

# 1. Problem Statement

Design a logging framework similar to Log4j/SLF4J supporting:

- Log levels: DEBUG, INFO, WARN, ERROR
- Multiple appenders (Console, File, Remote)
- Configurable format patterns
- Logger hierarchy (parent/child inheritance)
- Thread-safe logging

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Log Message | level + message + timestamp |
| Level Filter | Ignore below configured level |
| Appenders | Output to one or more destinations |
| Formatters | Pattern layout (%d %level %msg) |
| Hierarchy | `com.app.service` inherits from `com.app` |
| Configuration | Runtime or file-based |

---

# 3. Core Classes

```text
Logger (interface)
  └── getLogger(String name)
  └── debug/info/warn/error(String msg)

LoggerImpl
  ├── name, level, appenders, parent
  └── log(Level, String msg)

Appender (interface)
  └── append(LogRecord)

ConsoleAppender, FileAppender, AsyncAppender

Formatter (interface)
  └── format(LogRecord)

PatternFormatter

LogRecord
  ├── timestamp, level, loggerName, message, threadName
```

---

# 4. Logger Hierarchy

```java
public class LoggerFactory {
    private static final Map<String, LoggerImpl> loggers = new ConcurrentHashMap<>();
    private static final LoggerImpl root = new LoggerImpl("ROOT", Level.INFO);

    public static Logger getLogger(String name) {
        return loggers.computeIfAbsent(name, n -> {
            LoggerImpl logger = new LoggerImpl(n, null, new ArrayList<>());
            String parentName = parentName(n);
            LoggerImpl parent = parentName == null ? root : (LoggerImpl) getLogger(parentName);
            logger.setParent(parent);
            return logger;
        });
    }

    private static String parentName(String name) {
        int idx = name.lastIndexOf('.');
        return idx < 0 ? null : name.substring(0, idx);
    }
}
```

Effective level: walk up parent chain until non-null level found.

---

# 5. Logging Flow

```java
public class LoggerImpl implements Logger {
    private Level level;
    private final List<Appender> appenders = new CopyOnWriteArrayList<>();
    private LoggerImpl parent;

    public void info(String message) {
        log(Level.INFO, message);
    }

    public void log(Level msgLevel, String message) {
        if (!isEnabled(msgLevel)) return;

        LogRecord record = new LogRecord(
            Instant.now(), msgLevel, name, message, Thread.currentThread().getName());

        for (Appender appender : getAllAppenders()) {
            appender.append(record);
        }
    }

    private boolean isEnabled(Level msgLevel) {
        Level effective = this.level != null ? this.level : resolveParentLevel();
        return msgLevel.ordinal() >= effective.ordinal();
    }

    private List<Appender> getAllAppenders() {
        if (!appenders.isEmpty()) return appenders;
        return parent != null ? parent.getAllAppenders() : List.of();
    }
}
```

---

# 6. Appender & Formatter

```java
public class ConsoleAppender implements Appender {
    private Formatter formatter = new SimpleFormatter();

    public synchronized void append(LogRecord record) {
        System.out.println(formatter.format(record));
    }
}

public class PatternFormatter implements Formatter {
    private final String pattern; // "%d [%level] %logger - %msg"

    public String format(LogRecord r) {
        return pattern
            .replace("%d", r.getTimestamp().toString())
            .replace("%level", r.getLevel().name())
            .replace("%logger", r.getLoggerName())
            .replace("%msg", r.getMessage())
            .replace("%thread", r.getThreadName());
    }
}
```

---

# 7. Async Appender (Decorator)

```java
public class AsyncAppender implements Appender {
    private final Appender delegate;
    private final BlockingQueue<LogRecord> queue = new LinkedBlockingQueue<>(10000);
    private final ExecutorService executor = Executors.newSingleThreadExecutor();

    public AsyncAppender(Appender delegate) {
        this.delegate = delegate;
        executor.submit(this::drain);
    }

    public void append(LogRecord record) {
        queue.offer(record); // non-blocking for caller
    }

    private void drain() {
        while (true) {
            try {
                delegate.append(queue.take());
            } catch (InterruptedException e) { break; }
        }
    }
}
```

---

# 8. Configuration (Simplified)

```java
public class LoggingConfig {
    public static void configure(Properties props) {
        // log4j.rootLevel=INFO
        // log4j.appender.console=ConsoleAppender
        // log4j.logger.com.myapp=DEBUG, console, file
    }
}
```

---

# 9. Design Patterns

| Pattern | Use |
|---------|-----|
| Singleton | LoggerFactory |
| Chain of Responsibility | Parent logger level inheritance |
| Decorator | AsyncAppender wraps FileAppender |
| Strategy | Formatter interchangeable |
| Factory | getLogger(name) |

---

# Interview Questions

## Why CopyOnWriteArrayList for appenders?

Rare writes (config), frequent reads (log). Thread-safe iteration during log.

## FATAL vs ERROR?

Some frameworks add FATAL above ERROR. Ordinal comparison handles filtering.

## SLF4J vs Log4j?

SLF4J is facade API; Log4j/Logback is implementation. Design separates interface (`Logger`) from appenders.

## Rolling file appender?

When file size/date threshold hit, rotate `app.log` → `app.log.1`, open new file.

---

# Quick Revision

```text
Logger hierarchy → effective level + appenders inherit
LogRecord → immutable event
Appender + Formatter
AsyncAppender → queue + background thread
Level ordinal for filter: msgLevel >= configured
```

---

*End of Day 32 Logging Framework LLD*
