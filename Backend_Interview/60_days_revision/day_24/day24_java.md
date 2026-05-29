# Day 24 - Java File I/O & NIO

---

# 1. Classic File I/O (java.io)

---

# Reading a Text File — BufferedReader

```java
try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
}
```

---

# Writing a Text File — BufferedWriter

```java
try (BufferedWriter writer = new BufferedWriter(new FileWriter("out.txt"))) {
    writer.write("Hello");
    writer.newLine();
    writer.write("World");
}
```

---

# Byte Streams vs Character Streams

| Type | Classes | Use |
|------|---------|-----|
| Byte | `InputStream`, `OutputStream`, `FileInputStream` | Binary data (images, PDF) |
| Character | `Reader`, `Writer`, `FileReader` | Text (UTF-16 internally) |

Always wrap with **buffered** streams for performance.

---

# Serialization (java.io)

```java
// Write object
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("user.ser"))) {
    oos.writeObject(user);
}

// Read object
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("user.ser"))) {
    User user = (User) ois.readObject();
}
```

Requires `implements Serializable`. Prefer JSON/Protobuf in modern APIs.

---

# 2. NIO (java.nio) — New I/O

Introduced in Java 1.4; enhanced in Java 7+ with NIO.2.

Key differences from classic I/O:

| Classic I/O | NIO |
|-------------|-----|
| Stream-oriented | Buffer + Channel oriented |
| Blocking | Can be non-blocking (with Selector) |
| `FileReader/Writer` | `Path`, `Files`, `Channels` |

---

# 3. Path & Files (NIO.2 — Java 7+)

```java
import java.nio.file.*;

Path path = Paths.get("data", "config.json");

// Read all lines
List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);

// Write
Files.writeString(path, "content", StandardCharsets.UTF_8,
    StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING);

// Copy / Move
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(oldPath, newPath);

// Walk directory tree
try (Stream<Path> stream = Files.walk(Paths.get("."))) {
    stream.filter(Files::isRegularFile)
          .forEach(System.out::println);
}
```

---

# 4. Buffers & Channels

```java
try (RandomAccessFile file = new RandomAccessFile("data.bin", "rw");
     FileChannel channel = file.getChannel()) {

    ByteBuffer buffer = ByteBuffer.allocate(1024);
    int bytesRead = channel.read(buffer); // read into buffer

    buffer.flip(); // switch to read mode
    while (buffer.hasRemaining()) {
        System.out.print((char) buffer.get());
    }
}
```

```text
Channel → reads/writes to Buffer
Buffer  → flip() before read after write, clear() before reuse
```

---

# 5. Non-Blocking I/O with Selector (Advanced)

Used in high-performance servers (Netty-style):

```text
Selector monitors multiple channels
Single thread handles many connections
Channel registers for OP_READ / OP_WRITE / OP_ACCEPT
```

Spring WebFlux / Netty use NIO under the hood.

---

# 6. try-with-resources

Auto-closes `AutoCloseable` resources:

```java
try (InputStream in = Files.newInputStream(path);
     OutputStream out = Files.newOutputStream(outPath)) {
    in.transferTo(out); // Java 9+
}
```

Prevents resource leaks — always prefer over manual `close()`.

---

# 7. Comparison Table

| Task | Classic I/O | NIO.2 |
|------|-------------|-------|
| Read text file | `BufferedReader` | `Files.readAllLines` |
| Write text | `BufferedWriter` | `Files.writeString` |
| Binary copy | Streams | `Files.copy` / `transferTo` |
| Directory listing | `File.list()` | `Files.walk`, `Files.list` |
| Watch file changes | — | `WatchService` |

---

# 8. Real-World Backend Examples

## Log File Tailing

```java
try (BufferedReader reader = Files.newBufferedReader(logPath)) {
    String line;
    while ((line = reader.readLine()) != null) {
        if (line.contains("ERROR")) alertService.send(line);
    }
}
```

## CSV Import

```java
Files.lines(csvPath)
     .skip(1) // header
     .map(line -> line.split(","))
     .forEach(cols -> userRepository.save(mapRow(cols)));
```

## Config File Loading

```java
Properties props = new Properties();
try (InputStream in = Files.newInputStream(Paths.get("app.properties"))) {
    props.load(in);
}
```

---

# Common Interview Questions

## Q1. Difference between File and Path?

`File` (legacy) vs `Path` (NIO.2) — Path is immutable, better API, integrates with `Files`.

## Q2. Blocking vs non-blocking I/O?

Blocking: thread waits until I/O completes. Non-blocking: thread does other work; NIO Selector notifies when ready.

## Q3. Why BufferedReader?

Reduces system calls by reading chunks into buffer.

## Q4. Character encoding importance?

Always specify `StandardCharsets.UTF_8` — platform default causes bugs in production.

## Q5. Files.readAllBytes vs streaming?

readAllBytes loads entire file in memory — bad for large files; use stream/channel for GB files.

## Q6. How does Spring Boot load application.properties?

`PropertySourceLoader` uses classpath resource loading (NIO/classloader internally).

---

# One-Line Revision

```text
Classic io = streams; NIO.2 = Path/Files/Channels with buffers; always try-with-resources and explicit UTF-8.
```

---

*End of Day 24 Java*
