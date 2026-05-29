# Day 11 — Adapter Pattern & Proxy Pattern

**Topics:** Structural patterns · Class diagrams · Sequence flows · Interview points

---

# 1. Adapter Pattern

## Intent

Convert interface of a class into another interface clients expect — integrates incompatible APIs.

```text
Client → Target interface ← Adapter → Adaptee (legacy/third-party)
```

---

# Class Diagram (Text)

```text
+-------------+
|   Client    |
+-------------+
      |
      v
+-------------+<<interface>>
| MediaPlayer |
+-------------+
| + play()    |
+-------------+
      △
      |
+-------------+
| MediaAdapter|
+-------------+
| - adaptee   |
+-------------+
      |
      v
+-------------+
| AdvancedPlayer|
+-------------+
| + playVlc() |
| + playMp4() |
+-------------+
```

---

# Java Implementation

```java
interface MediaPlayer {
    void play(String audioType, String fileName);
}

class AdvancedMediaPlayer {
    void playVlc(String file) { System.out.println("VLC: " + file); }
    void playMp4(String file) { System.out.println("MP4: " + file); }
}

class MediaAdapter implements MediaPlayer {
    private final AdvancedMediaPlayer advanced = new AdvancedMediaPlayer();

    public void play(String audioType, String fileName) {
        if ("vlc".equalsIgnoreCase(audioType)) {
            advanced.playVlc(fileName);
        } else if ("mp4".equalsIgnoreCase(audioType)) {
            advanced.playMp4(fileName);
        }
    }
}

class AudioPlayer implements MediaPlayer {
    private final MediaAdapter adapter = new MediaAdapter();

    public void play(String audioType, String fileName) {
        if ("mp3".equalsIgnoreCase(audioType)) {
            System.out.println("MP3: " + fileName);
        } else {
            adapter.play(audioType, fileName);
        }
    }
}
```

---

# Sequence Flow

```text
Client -> AudioPlayer: play("vlc", "movie.vlc")
AudioPlayer -> MediaAdapter: play("vlc", "movie.vlc")
MediaAdapter -> AdvancedMediaPlayer: playVlc("movie.vlc")
AdvancedMediaPlayer --> Client: playing
```

---

# Real-World Uses

- Payment gateway wrapper (Stripe SDK → your `PaymentPort`)
- Legacy XML API → REST JSON client interface
- `InputStreamReader` adapts bytes to chars

---

# 2. Proxy Pattern

## Intent

Provide surrogate/placeholder to control access to real object (lazy load, security, logging, caching).

---

# Class Diagram (Text)

```text
+-------------+<<interface>>
|  Image      |
+-------------+
| + display() |
+-------------+
      △
      |
 +----+----+
 |         |
+------+ +-----------+
|Real  | |ImageProxy |
|Image | +-----------+
+------+ | - real    |
         | + display()|
         +-----------+
```

---

# Java Implementation

```java
interface Image {
    void display();
}

class RealImage implements Image {
    private final String filename;

    RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println("Loading " + filename);
    }

    public void display() {
        System.out.println("Displaying " + filename);
    }
}

class ImageProxy implements Image {
    private final String filename;
    private RealImage real;

    ImageProxy(String filename) {
        this.filename = filename;
    }

    public void display() {
        if (real == null) {
            real = new RealImage(filename); // lazy init
        }
        real.display();
    }
}
```

---

# Sequence Flow — Lazy Load

```text
Client -> ImageProxy: display()
ImageProxy -> RealImage: new RealImage()  // first call only
RealImage -> Disk: loadFromDisk()
ImageProxy -> RealImage: display()
RealImage --> Client: shown
```

---

# Proxy Types (Interview)

| Type | Purpose |
|------|---------|
| Virtual | Lazy loading |
| Protection | Access control |
| Remote | RMI / network stub |
| Caching | Return cached result |

---

# Adapter vs Proxy vs Decorator

| Pattern | Purpose |
|---------|---------|
| Adapter | Interface conversion |
| Proxy | Access control / lazy load |
| Decorator | Add behavior |

---

# Spring Connection

- **Spring AOP** uses dynamic proxy (JDK or CGLIB) for `@Transactional`, `@Cacheable`.
- **RestTemplate/WebClient** adapters for external HTTP APIs.

---

# Interview Points

1. **Adapter:** "Make it fit our interface" — integration problem.
2. **Proxy:** Same interface, control access to real object.
3. **JDK Proxy:** requires interface. **CGLIB:** subclasses concrete classes.
4. Discuss performance: caching proxy reduces DB hits.
5. Don't confuse with Decorator (adds features, same interface, composable stack).

---

# Quick Revision

```text
Adapter = incompatible interface bridge.
Proxy = controlled access / lazy / security / caching surrogate.
```

---

*End of Day 11 LLD*
