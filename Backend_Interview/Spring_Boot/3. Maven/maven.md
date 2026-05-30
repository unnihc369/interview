# Maven — Spring Boot Build Tool Guide

**Audience:** SDE 1–2 backend interviews · Spring Boot developers  
**Goal:** Understand Maven project layout, lifecycle, dependencies, and how Spring Boot packages executable JARs.

---

## How to Read This Guide

| If you are… | Start here | Time |
|-------------|------------|------|
| **New to Maven** | §1 → §3 → §4 → §5 (basics + lifecycle) | ~45 min |
| **Interview cram** | §5 lifecycle + §7 Spring Boot + §9 Q&A | ~30 min |
| **Deploying to Nexus** | §6 deploy phase + `settings.xml` | ~15 min |

**Mental model:** `pom.xml` declares project + dependencies → Maven lifecycle runs phases → plugins produce JAR/WAR in `target/`.

---

## Table of Contents

1. [What is Maven?](#1-what-is-maven)
2. [Typical Spring Boot Maven Project Structure](#2-typical-spring-boot-maven-project-structure)
3. [pom.xml Essentials](#3-pomxml-essentials)
4. [settings.xml & Repositories](#4-settingsxml--repositories)
5. [Maven Lifecycle](#5-maven-lifecycle)
6. [Deploy Phase Deep Dive](#6-deploy-phase-deep-dive)
7. [Dependencies & Scopes](#7-dependencies--scopes)
8. [Spring Boot Maven Specifics](#8-spring-boot-maven-specifics)
9. [Interview Questions & Answers](#9-interview-questions--answers)

---

## 1. What is Maven?

**Maven** is a **build automation and dependency management** tool for Java projects.

| Without Maven | With Maven |
|---------------|------------|
| Manually download JARs | Dependencies declared in `pom.xml`; Maven fetches them |
| Custom build scripts (Ant) | Standard lifecycle: `compile`, `test`, `package`, `install`, `deploy` |
| Inconsistent project layout | Convention over configuration (`src/main/java`, `src/test/java`) |
| Version conflicts by hand | Transitive dependency resolution + nearest-wins strategy |

**Maven vs Ant:** Ant uses imperative XML tasks (copy, compile, jar) with no built-in dependency management. Maven is **declarative** — you describe *what* the project is; Maven knows *how* to build it.

---

## 2. Typical Spring Boot Maven Project Structure

```text
learningspringboot/
├── .mvn/                          # Maven Wrapper config
│   └── wrapper/
├── mvnw                           # Unix wrapper script
├── mvnw.cmd                       # Windows wrapper script
├── pom.xml                        # Project Object Model (heart of Maven)
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/learningspringboot/
│   │   │       ├── LearningspringbootApplication.java
│   │   │       ├── controller/    # HTTP layer
│   │   │       ├── service/       # Business logic
│   │   │       ├── repository/    # Data access
│   │   │       ├── entity/        # JPA / DB mapping
│   │   │       ├── dto/           # Request/response objects
│   │   │       ├── config/        # @Configuration classes
│   │   │       └── exception/     # Custom exceptions + handlers
│   │   │
│   │   └── resources/
│   │       ├── application.properties   # or application.yml
│   │       ├── static/                  # CSS, JS, images
│   │       └── templates/               # Thymeleaf (if used)
│   │
│   └── test/
│       ├── java/
│       │   └── com/company/learningspringboot/
│       │       └── LearningspringbootApplicationTests.java
│       └── resources/             # Test-specific config
│
└── target/                        # Build output (generated — do not commit)
    ├── classes/
    ├── test-classes/
    └── learningspringboot-0.0.1-SNAPSHOT.jar
```

### Layer Responsibilities

| Layer | Package | Responsibility |
|-------|---------|----------------|
| **Controller** | `controller/` | Handles HTTP requests, validates input, returns responses |
| **Service** | `service/` | Business logic, transactions, orchestration |
| **Repository** | `repository/` | Database access (Spring Data JPA interfaces) |
| **Entity** | `entity/` | Maps to database tables (`@Entity`) |
| **DTO** | `dto/` | Data transfer objects for API request/response (decouple API from DB) |

> **Note:** Layer folders live **inside the base package** (`com.company.learningspringboot`), not directly under `src/main/java`. See [Project File & Folder Structure](../2.%20Create_Project/Project_File_Folder_Structure.md) for a full layered example.

---

## 3. pom.xml Essentials

**POM (Project Object Model)** is the XML file Maven reads to build your project.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Inherit Spring Boot defaults (Java version, dependency versions, plugins) -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.0</version>
        <relativePath/>
    </parent>

    <!-- Project coordinates — unique identity in Maven repos -->
    <groupId>com.company</groupId>           <!-- Organization / company -->
    <artifactId>learningspringboot</artifactId> <!-- Project name -->
    <version>0.0.1-SNAPSHOT</version>          <!-- SNAPSHOT = dev; RELEASE = stable -->
    <packaging>jar</packaging>                 <!-- jar (default) or war -->

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

| Coordinate | Meaning | Example |
|------------|---------|---------|
| **groupId** | Organization or group (like a package prefix) | `com.company` |
| **artifactId** | Unique project/module name | `learningspringboot` |
| **version** | Release version; `-SNAPSHOT` = work in progress | `0.0.1-SNAPSHOT` |

**Super POM:** Every `pom.xml` implicitly inherits from Maven's built-in **Super POM**, which defines default directories (`src/main/java`), default plugin versions, and Central repository URL. Your project's POM + parent POM + Super POM merge into the **effective POM**.

**Parent POM:** A shared `pom.xml` (often in a company or Spring Boot) that child modules inherit from — centralizes plugin versions, dependency management, and build config.

---

## 4. settings.xml & Repositories

### Maven Repository Types

| Type | Location | Purpose |
|------|----------|---------|
| **Local** | `~/.m2/repository` | Cache of downloaded artifacts on your machine |
| **Central** | repo.maven.apache.org | Public Maven Central (default remote) |
| **Remote / Company** | Nexus, Artifactory | Private or mirror repos inside organizations |

### settings.xml

Lives at `~/.m2/settings.xml` (user-level) or `$MAVEN_HOME/conf/settings.xml` (global). Used for:

- Server credentials (for `deploy`)
- Mirror configuration
- Local repository path override
- Active profiles

```xml
<settings>
    <servers>
        <server>
            <id>company-repo</id>
            <username>admin</username>
            <password>${env.REPO_PASSWORD}</password>
        </server>
    </servers>

    <mirrors>
        <mirror>
            <id>company-mirror</id>
            <url>https://nexus.company.com/repository/maven-public/</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
</settings>
```

> The `<server><id>` must **match** the `<repository><id>` in `pom.xml` `distributionManagement`.

---

## 5. Maven Lifecycle

Maven has **three built-in lifecycles**. The most used is the **default lifecycle** (project build):

```text
validate → compile → test → package → verify → install → deploy
```

| Phase | What happens |
|-------|--------------|
| **validate** | Checks POM and project structure are valid |
| **compile** | Compiles `src/main/java` → `target/classes` |
| **test** | Compiles and runs unit tests (`src/test/java`) |
| **package** | Creates JAR/WAR (e.g. `target/app.jar`) |
| **verify** | Runs integration tests and quality checks |
| **install** | Copies artifact to **local** repo (`~/.m2/repository`) |
| **deploy** | Uploads artifact to **remote** repo (Nexus / Artifactory) |

### Common Commands

```bash
mvn clean              # Delete target/ (clean plugin goal)
mvn compile            # Runs validate + compile
mvn test               # Runs up through test
mvn package            # Runs up through package → produces JAR
mvn install            # Runs up through install → local ~/.m2
mvn deploy             # Runs full lifecycle including deploy
mvn clean package -DskipTests   # Build JAR, skip tests
./mvnw spring-boot:run          # Run app via Maven Wrapper (no global Maven install)
```

### package vs install vs deploy

| Command | Output location | Use case |
|---------|-----------------|----------|
| `package` | `target/` only | CI build, run `java -jar` locally |
| `install` | `target/` + local `~/.m2` | Multi-module: other modules on same machine depend on this |
| `deploy` | Remote Nexus/Artifactory | Share artifact with team / production pipeline |

**Why previous phases run automatically:** Maven binds **plugin goals** to lifecycle phases. Running `package` must compile and test first — phases are **sequential and cumulative**.

### Phase vs Goal

| Concept | Definition |
|---------|------------|
| **Phase** | A step in the lifecycle (e.g. `compile`) |
| **Goal** | A specific task a plugin performs (e.g. `compiler:compile`, `surefire:test`) |

A phase may run zero or more goals. Some goals are bound to phases by default; you can also invoke goals directly: `mvn dependency:tree`.

---

## 6. Deploy Phase Deep Dive

**Purpose:** Upload the built artifact (JAR/WAR/POM) to a **remote repository** so other projects or environments can consume it.

**Typical targets:** Nexus, JFrog Artifactory, Maven Central (for open-source releases).

### Requires: distributionManagement in pom.xml

```xml
<distributionManagement>
    <repository>
        <id>company-repo</id>
        <url>https://repo.company.com/releases</url>
    </repository>
    <snapshotRepository>
        <id>company-snapshots</id>
        <url>https://repo.company.com/snapshots</url>
    </snapshotRepository>
</distributionManagement>
```

### Requires: matching credentials in settings.xml

```xml
<servers>
    <server>
        <id>company-repo</id>
        <username>admin</username>
        <password>password</password>
    </server>
    <server>
        <id>company-snapshots</id>
        <username>admin</username>
        <password>password</password>
    </server>
</servers>
```

```bash
mvn clean deploy
```

Flow: `package` creates artifact → `install` puts in local repo → `deploy` plugin uploads to remote URL defined in `distributionManagement`.

---

## 7. Dependencies & Scopes

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

| Scope | Classpath | Typical use |
|-------|-----------|-------------|
| **compile** (default) | compile, test, runtime | Most app dependencies |
| **provided** | compile, test only | Servlet API — container provides at runtime |
| **runtime** | test, runtime only | JDBC driver |
| **test** | test only | JUnit, Mockito |
| **import** | POM only | BOM imports in `dependencyManagement` |

**Transitive dependency:** If A depends on B, and B depends on C, your project gets C automatically without declaring it.

**Dependency conflict:** Two libraries pull different versions of the same artifact (e.g. Guava 28 vs 31).

**Maven resolution (nearest-wins):** Shortest path in the dependency tree wins. You can override with explicit `<dependency>` or `<dependencyManagement>` in parent POM.

```bash
mvn dependency:tree          # Visualize transitive deps
mvn dependency:tree -Dverbose  # Show conflict resolution
```

---

## 8. Spring Boot Maven Specifics

### spring-boot-starter-parent

Parent POM that provides:

- Default Java version and encoding
- **Dependency management** — omit version numbers on Spring starters
- Plugin configuration (compiler, surefire, jar)
- Resource filtering for `application.properties`

### spring-boot-maven-plugin

Repackages the normal JAR into an **executable fat JAR (über-JAR)**:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

| JAR type | Contents | Run |
|----------|----------|-----|
| **Thin JAR** | Your classes only | Needs classpath with all deps |
| **Fat / executable JAR** | Your classes + all dependencies + embedded Tomcat | `java -jar app.jar` |

**How Tomcat gets inside the JAR:** `spring-boot-starter-web` brings embedded Tomcat as a dependency. The repackage goal nests dependency JARs inside `BOOT-INF/lib/` and uses `JarLauncher` as the main entry point — no external Tomcat install needed.

### Maven Wrapper (`.mvn/`, `mvnw`)

Ensures everyone uses the **same Maven version** without installing Maven globally:

```bash
./mvnw clean package
```

### Profiles

Environment-specific config in `pom.xml` or `settings.xml`:

```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>
</profiles>
```

```bash
mvn clean package -Pdev
```

---

## 9. Interview Questions & Answers

### Basic

**1. What is Maven?**  
A build automation and dependency management tool for Java. It uses a declarative `pom.xml`, standard directory layout, and a fixed lifecycle.

**2. Why do we need Maven?**  
Centralized dependency resolution, reproducible builds, convention over configuration, and plugins for compile/test/package/deploy — no manual JAR hunting.

**3. What is pom.xml?**  
Project Object Model — XML describing project coordinates (`groupId`, `artifactId`, `version`), dependencies, plugins, profiles, and build config.

**4. What is settings.xml?**  
User- or machine-level Maven config: credentials, mirrors, local repo path, active profiles. Not committed to the project (usually `~/.m2/settings.xml`).

**5. What is a Maven Repository?**  
Storage for artifacts (JARs). Local (`~/.m2`), Central (public), or remote (Nexus/Artifactory).

**6. What is Super POM?**  
The implicit parent of every POM. Defines defaults: directory structure, Central repo, core plugin versions. Merged into the **effective POM**.

**7. Difference between Maven and Ant?**  
Ant: imperative, task-based, no dependency management. Maven: declarative, lifecycle-driven, built-in dependency resolution and standard layout.

**8. What is groupId?**  
Organization identifier (reverse domain), e.g. `com.company`. Groups related projects.

**9. What is artifactId?**  
Unique project/module name, e.g. `learningspringboot`.

**10. What is version?**  
Release identifier. `-SNAPSHOT` = mutable dev builds; release versions are immutable once deployed.

---

### Lifecycle

**11. Explain Maven Lifecycle.**  
Three lifecycles: **default** (build), **clean** (remove artifacts), **site** (docs). Default: `validate → compile → test → package → verify → install → deploy`.

**12. What happens during compile phase?**  
`compiler:compile` runs — Java sources in `src/main/java` compiled to `target/classes`.

**13. What happens during package phase?**  
JAR/WAR assembled from compiled classes and resources. With Spring Boot plugin, repackage creates executable fat JAR.

**14. What is verify phase?**  
Integration tests and quality checks (e.g. failsafe plugin) run before install/deploy.

**15. Difference between package and install?**  
`package` leaves artifact in `target/`. `install` also copies it to local Maven repo (`~/.m2`) for other local projects.

**16. Difference between install and deploy?**  
`install` → local repo only. `deploy` → remote repo (Nexus/Artifactory) for team/CI consumption.

**17. Why does Maven execute previous phases automatically?**  
Phases are ordered; each builds on the last. Running `package` requires compiled, tested code — Maven runs all prior phases in the lifecycle.

---

### Dependencies

**18. What are dependency scopes?**  
Control classpath visibility: `compile` (default), `provided`, `runtime`, `test`, `import` (BOM).

**19. What is transitive dependency?**  
Indirect dependency — A → B → C means C is on your classpath even if you didn't declare it.

**20. What is dependency conflict?**  
Two deps require different versions of the same library.

**21. How does Maven resolve conflicts?**  
**Nearest definition wins** (shortest path in tree). Override with explicit version or `dependencyManagement`.

**22. What is dependency:tree?**  
Maven plugin goal printing the full dependency hierarchy: `mvn dependency:tree`.

---

### Spring Boot Specific

**23. Why is spring-boot-starter-parent used?**  
Inherits managed dependency versions, Java/compiler settings, and plugin config — less boilerplate in `pom.xml`.

**24. What is spring-boot-maven-plugin?**  
Repackages application into executable fat JAR and supports `spring-boot:run` for local dev.

**25. What is a fat jar?**  
Über-JAR containing application classes + all dependency JARs + embedded server libs in one file.

**26. What is executable jar?**  
JAR with `Main-Class` (Spring Boot uses `JarLauncher`) runnable via `java -jar app.jar`.

**27. How does Spring Boot package Tomcat inside jar?**  
`spring-boot-starter-web` pulls embedded Tomcat; repackage goal puts it in `BOOT-INF/lib/` inside the fat JAR.

---

### Advanced

**28. What is Parent POM?**  
Shared POM that child modules extend via `<parent>` — DRY for versions, plugins, and dependency management.

**29. What are Profiles?**  
Conditional build/config blocks activated by `-P`, OS, JDK, or property — e.g. dev vs prod settings.

**30. What is Maven Wrapper?**  
Scripts (`mvnw`, `mvnw.cmd`) + `.mvn/wrapper` that download a pinned Maven version — consistent builds without global install.

**31. What are Plugins?**  
Extend Maven with goals (compiler, surefire, spring-boot, deploy). Bound to lifecycle phases or invoked directly.

**32. What are Goals?**  
Specific tasks: `compiler:compile`, `surefire:test`, `jar:jar`. Format: `plugin:goal`.

**33. Lifecycle Phase vs Goal?**  
Phase = lifecycle step (abstract). Goal = concrete work by a plugin. Phases bind goals; you can run goals standalone.

**34. How does Maven work internally?**  
Reads POM → merges with parent + Super POM → resolves dependencies from repos → executes lifecycle phases → runs bound plugin goals → writes output to `target/`.

**35. (Bonus) What is dependencyManagement vs dependencies?**  
`dependencies` — adds to classpath. `dependencyManagement` (often in parent/BOM) — pins versions only; child must still declare artifact without version.

---

## One-Page Revision Card

| Topic | Remember |
|-------|----------|
| **Coordinates** | `groupId` + `artifactId` + `version` = unique artifact |
| **Layout** | `src/main/java`, `src/main/resources`, `src/test/java` |
| **Lifecycle** | `compile → test → package → install → deploy` |
| **package** | JAR in `target/` |
| **install** | JAR in `~/.m2/repository` |
| **deploy** | Upload to Nexus/Artifactory |
| **Scopes** | compile, provided, runtime, test |
| **Conflict** | Nearest-wins; use `dependency:tree` |
| **Fat JAR** | `spring-boot-maven-plugin` repackage |
| **Wrapper** | `./mvnw` — no global Maven needed |

---

*End of Maven guide — use with [Project Structure](../2.%20Create_Project/Project_File_Folder_Structure.md) and [Spring Boot Intro](../1.%20Into/Introduction_to_spring_boot.md).*
