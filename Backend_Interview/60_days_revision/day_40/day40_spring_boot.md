# Day 40 — Spring Native & GraalVM

**Topics:** AOT Compilation · Native Image · Trade-offs · Spring Boot 3 Native · Interview Questions

---

# 1. What is GraalVM Native Image?

Compiles Java bytecode **ahead-of-time (AOT)** into standalone native executable.

```text
Traditional JVM:  bytecode → JIT at runtime (warmup, higher memory)
Native Image:    bytecode → native binary at build time (fast start, lower RSS)
```

---

# 2. Why Spring Native?

| Benefit | Detail |
|---------|--------|
| Fast startup | 50–200ms vs seconds on JVM |
| Low memory | Critical for serverless (AWS Lambda, Knative) |
| Smaller footprint | No bundled JVM in container |
| Predictable | No JIT warmup spikes |

---

# 3. Spring Boot 3 Native Support

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.0</version>
</parent>

<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
</plugin>
```

Build:

```bash
mvn -Pnative native:compile
./target/myapp
```

Or Cloud Native Buildpacks:

```bash
mvn spring-boot:build-image -Pnative
```

---

# 4. How AOT Processing Works

At build time Spring analyzes:

- Classpath component scan
- `@Configuration` beans
- Reflection metadata
- Resource files (`application.yml`, templates)

Generates `META-INF/native-image` hints and substitute configurations.

```text
spring-aot-maven-plugin → process-aot → native-image compile
```

---

# 5. Reachability Metadata

Libraries using heavy reflection need hints:

```json
{
  "reflection": [
    {
      "type": "com.example.MyDto",
      "methods": [{"name": "<init>", "parameterTypes": []}]
    }
  ]
}
```

Spring registers many frameworks automatically; custom reflection may need `@RegisterReflectionForBinding`.

---

# 6. Limitations (Interview Critical)

| Limitation | Workaround |
|------------|------------|
| Dynamic class loading | Avoid; use AOT-friendly patterns |
| Reflection at runtime | Build-time registration |
| CGLIB proxies | Prefer JDK interfaces |
| Graal-incompatible libs | Check compatibility matrix |
| Longer build times | CI cache, native build agents |
| No hot reload dev | Use JVM profile for dev |

---

# 7. Profiles Strategy

```text
dev  → JVM (fast iteration, DevTools)
prod → native image (K8s, Lambda)
```

```properties
# application-native.properties
spring.main.lazy-initialization=false
```

---

# 8. Docker with Native

```dockerfile
FROM ghcr.io/graalvm/native-image-community:21 AS build
WORKDIR /app
COPY . .
RUN ./mvnw -Pnative native:compile

FROM debian:bookworm-slim
COPY --from=build /app/target/myapp /app
ENTRYPOINT ["/app"]
```

Image size often **< 100MB** vs 300MB+ JVM images.

---

# 9. When NOT to Use Native

- Long-running monoliths where JIT peak throughput matters
- Heavy dynamic scripting (Groovy rules at runtime)
- Teams without CI capacity for slow native builds
- Apps relying on unsupported agents (some APM tools)

---

# Interview Questions

## Q1. Native vs JVM startup?

Native: no classpath scanning at runtime, no JIT warmup. JVM: optimizes hot paths over minutes.

## Q2. Does Spring Native support all starters?

Most core starters supported; check [Spring Native compatibility](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-with-GraalVM). Some JDBC drivers need extra config.

## Q3. How handle `@Scheduled` and `@Async`?

Supported with AOT; thread pools configured at build time.

## Q4. Observability on native?

Micrometer, OpenTelemetry improving; some Java agents don't support native yet.

## Q5. Serverless fit?

Excellent — Lambda cold start drops dramatically with native Spring Boot.

---

# One-Line Revision

```text
Spring Native + GraalVM = AOT binary, fast cold start, reflection hints; use JVM for dev, native for serverless/K8s scale-to-zero.
```

---

*End of Day 40 Spring Boot*
