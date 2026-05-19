# Maven — Complete Interview Guide

## Table of Contents

1. [What is Maven?](#1-what-is-maven)
2. [Why Maven Came Into Picture (ANT vs Maven)](#2-why-maven-came-into-picture-ant-vs-maven)
3. [Maven Project Structure](#3-maven-project-structure)
4. [What is pom.xml?](#4-what-is-pomxml)
5. [Important Elements in pom.xml](#5-important-elements-in-pomxml)
6. [Parent POM & Super POM](#6-parent-pom--super-pom)
7. [Maven Repositories](#7-maven-repositories)
8. [Maven Build Lifecycle](#8-maven-build-lifecycle)
9. [Maven Plugins](#9-maven-plugins)
10. [Goals](#10-goals)
11. [Build Section](#11-build-section)
12. [Common Maven Plugins](#12-common-maven-plugins)
13. [Dependency Scopes](#13-dependency-scopes)
14. [Maven Commands](#14-maven-commands)
15. [Clean Lifecycle](#15-clean-lifecycle)
16. [Maven Dependency Tree](#16-maven-dependency-tree)
17. [SNAPSHOT vs Release Version](#17-snapshot-vs-release-version)
18. [Maven vs Gradle](#18-maven-vs-gradle)
19. [Spring Boot Maven Plugin](#19-spring-boot-maven-plugin)
20. [Advanced Topics](#20-advanced-topics)
21. [Final Mental Model](#21-final-mental-model)
22. [Interview Questions Index](#22-interview-questions-index)

---

## 1. What is Maven?

### Definition

**Maven** is a **Project Management + Build Automation Tool** used mainly in Java and Spring Boot projects.

### What Maven Actually Does

| Feature | Purpose |
|---------|---------|
| Build generation | Compile / package code |
| Dependency management | Download libraries automatically |
| Project structure | Standard folder structure |
| Testing | Run unit tests |
| Packaging | Generate JAR / WAR |
| Deployment | Deploy artifacts |
| Documentation | Generate docs |
| Plugin system | Extend functionality |

---

## 2. Why Maven Came Into Picture (ANT vs Maven)

Before Maven, **ANT** was popular.

### Problem in ANT

In ANT you had to specify both **WHAT** to do and **HOW** to do it.

```xml
<javac srcdir="src" destdir="classes"/>
```

You manually specify: compile command, directories, and every build step.

### Maven Solves This

```bash
mvn compile
```

You only tell **WHAT** to do. Maven already knows **HOW** (via plugins and goals).

### ANT vs Maven

| ANT | Maven |
|-----|-------|
| Procedural | Convention based |
| Need to define steps | Standard lifecycle |
| No dependency management | Strong dependency management |
| Flexible but verbose | Easy and structured |
| Need WHAT + HOW | Only WHAT |

---

## 3. Maven Project Structure

Standard Maven structure:

```
project/
 ├── pom.xml
 ├── src/
 │   ├── main/
 │   │   ├── java/
 │   │   └── resources/
 │   └── test/
 │       ├── java/
 │       └── resources/
 └── target/
```

### Important Folders

| Folder | Purpose |
|--------|---------|
| `src/main/java` | Main source code |
| `src/main/resources` | Properties / config files |
| `src/test/java` | Unit tests |
| `target/` | Generated build output |

### What is the `target` folder?

Generated build output after `mvn compile` or `mvn package`:

- `.class` files
- `.jar` / `.war` artifacts
- Temporary build files

---

## 4. What is pom.xml?

### Definition

**POM = Project Object Model** — the heart of Maven.

Maven reads `pom.xml` to understand:

- Dependencies
- Plugins
- Configurations
- Build lifecycle

> **Interview line:** Maven is highly dependent on `pom.xml`.

---

## 5. Important Elements in pom.xml

### `project` — Root element

```xml
<project>
    <!-- all configuration -->
</project>
```

### `groupId`

Usually company / package name.

```xml
<groupId>com.example</groupId>
```

### `artifactId`

Project name.

```xml
<artifactId>user-service</artifactId>
```

### `version`

```xml
<version>1.0.0</version>
```

**Coordinates:** `groupId` + `artifactId` + `version` uniquely identify a project:

```
com.example:user-service:1.0.0
```

### `packaging`

```xml
<packaging>jar</packaging>
```

Possible values: `jar`, `war`, `pom`, etc.

### `dependencies`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### `properties`

Reusable variables for centralized version management.

```xml
<properties>
    <java.version>17</java.version>
</properties>
```

Usage: `${java.version}`

**Why use properties?** Reusability, centralized versions, easier maintenance.

---

## 6. Parent POM & Super POM

### Parent POM

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>
```

Child project **inherits**:

- Dependencies (versions)
- Plugin configurations
- Properties

### Super POM

Every Maven POM internally inherits from the **Super POM** provided by Maven itself.

Contains:

- Default plugin configuration
- Default repositories
- Default build settings

---

## 7. Maven Repositories

Repositories store dependencies (JARs).

### Local Repository

**Path:** `~/.m2/repository`

**Purpose:** Cache downloaded JARs locally.

### Remote Repository

Examples:

- Maven Central
- Company Nexus
- JFrog Artifactory

### Dependency Resolution Flow

```
1. Check local repository (~/.m2/repository)
2. If not found → download from remote repository
3. Cache in local repository
```

---

## 8. Maven Build Lifecycle

> **Most asked interview topic.**

### Core Lifecycle Phases

```
validate → compile → test → package → verify → install → deploy
```

### Important Concept

Lifecycle phases are **sequential**. If you run:

```bash
mvn package
```

Maven automatically runs: `validate` → `compile` → `test` → `package`

### Phase-by-Phase

| Phase | Command | What happens | Output |
|-------|---------|----------------|--------|
| **validate** | `mvn validate` | Checks project structure and POM | — |
| **compile** | `mvn compile` | `.java` → `.class` via compiler plugin | `target/classes` |
| **test** | `mvn test` | Runs unit tests (JUnit via surefire) | Test reports |
| **package** | `mvn package` | Creates JAR / WAR | `target/app.jar` |
| **verify** | `mvn verify` | Integration / quality checks (PMD, Checkstyle) | — |
| **install** | `mvn install` | Copies artifact to local repo | `~/.m2/repository` |
| **deploy** | `mvn deploy` | Uploads to remote repo (CI/CD, Nexus) | Remote repo |

### compile vs package

| compile | package |
|---------|---------|
| Generates `.class` files | Generates JAR / WAR |
| No distributable artifact | Creates deployable artifact |

### package vs install

| package | install |
|---------|---------|
| Creates JAR in `target/` | Stores JAR in local `~/.m2/repository` |

### install vs deploy

| install | deploy |
|---------|--------|
| Local repository | Remote repository (team / CI) |

### What happens during compile?

Maven uses **maven-compiler-plugin**, runs `javac` internally, and generates bytecode.

### Which plugin runs unit tests?

**maven-surefire-plugin**

---

## 9. Maven Plugins

Maven works using **plugins**. Lifecycle phases themselves do not do work — **plugins perform the actual work**.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
</plugin>
```

**Maven Plugin** = reusable component that performs specific build tasks.

---

## 10. Goals

A **Goal** is a specific task performed by a plugin.

```
compiler:compile
   ↑         ↑
 plugin    goal
```

### Lifecycle Phase vs Goal

| Lifecycle Phase | Goal |
|-----------------|------|
| High-level stage | Specific task |

Example: `package` phase binds to `jar:jar` goal.

---

## 11. Build Section

Custom build configurations in `pom.xml`:

```xml
<build>
    <plugins>
        <!-- add and configure plugins -->
    </plugins>
</build>
```

Used to: add plugins, configure plugins, customize lifecycle behavior.

---

## 12. Common Maven Plugins

| Plugin | Purpose |
|--------|---------|
| `maven-compiler-plugin` | Compile Java source |
| `maven-surefire-plugin` | Run unit tests |
| `maven-jar-plugin` | Create JAR |
| `spring-boot-maven-plugin` | Spring Boot fat JAR packaging |
| `maven-checkstyle-plugin` | Code style checks |
| `maven-pmd-plugin` | Static analysis |

---

## 13. Dependency Scopes

| Scope | When available | Included in JAR? | Example |
|-------|----------------|------------------|---------|
| **compile** (default) | Compile + test + runtime | Yes | Spring, Jackson |
| **test** | Test only | No | JUnit, Mockito |
| **provided** | Compile + test; runtime supplied elsewhere | No | `servlet-api` (Tomcat provides) |
| **runtime** | Test + runtime only | Yes | JDBC driver |

### compile vs provided

| compile | provided |
|---------|----------|
| Included in final JAR | Not included in final JAR |
| Needed at runtime | Container provides at runtime |

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```

---

## 14. Maven Commands

| Command | Purpose |
|---------|---------|
| `mvn clean` | Delete `target/` folder |
| `mvn compile` | Compile source code |
| `mvn test` | Run unit tests |
| `mvn package` | Create JAR / WAR |
| `mvn install` | Install to local repository |
| `mvn deploy` | Deploy to remote repository |
| `mvn dependency:tree` | Show dependency tree |

### `mvn clean`

Deletes `target/` — ensures a **fresh build** with no stale artifacts.

### Skip tests

```bash
mvn install -DskipTests
```

### Force update dependencies

```bash
mvn clean install -U
```

---

## 15. Clean Lifecycle

Separate lifecycle from default:

```
pre-clean → clean → post-clean
```

`mvn clean` runs the **clean** phase and removes `target/`.

---

## 16. Maven Dependency Tree

```bash
mvn dependency:tree
```

Used to debug dependency conflicts and visualize transitive dependencies.

### Transitive Dependency

A **dependency of a dependency**.

```
A → B → C
```

If A depends on B, and B depends on C, then A indirectly gets C.

---

## 17. SNAPSHOT vs Release Version

### SNAPSHOT

```xml
<version>1.0-SNAPSHOT</version>
```

- Under active development
- **Mutable** — can be overwritten on each build

### Release

```xml
<version>1.0.0</version>
```

- Stable, immutable release
- Should not change once published

---

## 18. Maven vs Gradle

| Feature | Maven | Gradle |
|---------|-------|--------|
| Config format | XML (`pom.xml`) | Groovy / Kotlin DSL |
| Philosophy | Convention over configuration | Flexible |
| Build speed | Slower | Faster (incremental) |
| Learning curve | Easier | Steeper |
| Customization | Structured | Highly customizable |

---

## 19. Spring Boot Maven Plugin

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

Used for:

- **Executable fat JAR** (all dependencies bundled)
- Embedded Tomcat packaging
- `java -jar app.jar` runnable artifact

### Fat JAR

Spring Boot creates an executable JAR containing your code + all dependencies + embedded server.

---

## 20. Advanced Topics

### Dependency Mediation

When multiple versions of the same library exist:

```
A → B → log4j 1.0
A → C → log4j 2.0
```

Maven chooses the **nearest** dependency in the tree (shortest path wins).

### Excluding a Dependency

```xml
<dependency>
    <groupId>some.group</groupId>
    <artifactId>some-artifact</artifactId>
    <exclusions>
        <exclusion>
            <groupId>unwanted.group</groupId>
            <artifactId>unwanted-artifact</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### Multi-module Projects

Parent POM manages multiple child modules — common in microservices monorepos.

```xml
<modules>
    <module>user-service</module>
    <module>order-service</module>
</modules>
```

### Profiles

Environment-specific configuration:

```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <env>development</env>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <env>production</env>
        </properties>
    </profile>
</profiles>
```

Activate: `mvn package -Pprod`

### Maven Wrapper (`mvnw`)

Ensures the **same Maven version** across all developers and CI — no local Maven install required.

```bash
./mvnw clean package
```

### How Maven Works Internally

```
mvn command
    ↓
Read pom.xml (+ parent + Super POM)
    ↓
Resolve dependencies (local → remote)
    ↓
Execute lifecycle phases
    ↓
Bind and execute plugin goals
    ↓
Generate artifact in target/
```

---

## 21. Final Mental Model

| Concept | Role |
|---------|------|
| `pom.xml` | Configuration |
| Repository | Dependency storage |
| Lifecycle | Build stages |
| Plugin | Actual worker |
| Goal | Specific task |

### One-Line Interview Summary

> Maven is a convention-based project management and build automation tool that manages dependencies, standardizes project structure, and automates the software build lifecycle using plugins and POM configuration.

---

## 22. Interview Questions Index

All questions link to the section where the answer is explained in detail.

### Basic

| # | Question | Answer in section |
|---|----------|-------------------|
| Q1 | [What is Maven?](#1-what-is-maven) | [§1 What is Maven?](#1-what-is-maven) |
| Q2 | [Why is Maven used?](#2-why-maven-came-into-picture-ant-vs-maven) | [§2 ANT vs Maven](#2-why-maven-came-into-picture-ant-vs-maven) |
| Q3 | [What is POM?](#4-what-is-pomxml) | [§4 pom.xml](#4-what-is-pomxml) |
| Q4 | [What is the Maven lifecycle?](#8-maven-build-lifecycle) | [§8 Build Lifecycle](#8-maven-build-lifecycle) |
| Q5 | [Difference between install and deploy?](#8-maven-build-lifecycle) | [§8 — install vs deploy](#8-maven-build-lifecycle) |
| Q6 | [Difference between package and install?](#8-maven-build-lifecycle) | [§8 — package vs install](#8-maven-build-lifecycle) |
| Q7 | [What is the local repository?](#7-maven-repositories) | [§7 Repositories — Local](#7-maven-repositories) |
| Q8 | [What is dependency scope?](#13-dependency-scopes) | [§13 Dependency Scopes](#13-dependency-scopes) |

### Intermediate

| # | Question | Answer in section |
|---|----------|-------------------|
| Q9 | [What is a transitive dependency?](#16-maven-dependency-tree) | [§16 Dependency Tree](#16-maven-dependency-tree) |
| Q10 | [What is a Maven plugin?](#9-maven-plugins) | [§9 Maven Plugins](#9-maven-plugins) |
| Q11 | [What is a goal?](#10-goals) | [§10 Goals](#10-goals) |
| Q12 | [What is a parent POM?](#6-parent-pom--super-pom) | [§6 Parent POM](#6-parent-pom--super-pom) |
| Q13 | [What is Super POM?](#6-parent-pom--super-pom) | [§6 Super POM](#6-parent-pom--super-pom) |
| Q14 | [What is dependency mediation?](#20-advanced-topics) | [§20 — Dependency Mediation](#20-advanced-topics) |
| Q15 | [What is SNAPSHOT?](#17-snapshot-vs-release-version) | [§17 SNAPSHOT vs Release](#17-snapshot-vs-release-version) |
| Q16 | [Difference between ANT and Maven?](#2-why-maven-came-into-picture-ant-vs-maven) | [§2 ANT vs Maven table](#2-why-maven-came-into-picture-ant-vs-maven) |
| Q17 | [What is the target folder?](#3-maven-project-structure) | [§3 — target folder](#3-maven-project-structure) |
| Q18 | [Why use properties in pom.xml?](#5-important-elements-in-pomxml) | [§5 — properties](#5-important-elements-in-pomxml) |
| Q19 | [groupId + artifactId + version?](#5-important-elements-in-pomxml) | [§5 — coordinates](#5-important-elements-in-pomxml) |
| Q20 | [Difference between compile and package?](#8-maven-build-lifecycle) | [§8 — compile vs package](#8-maven-build-lifecycle) |
| Q21 | [What happens during compile phase?](#8-maven-build-lifecycle) | [§8 — compile phase](#8-maven-build-lifecycle) |
| Q22 | [Which plugin runs unit tests?](#8-maven-build-lifecycle) | [§8 — surefire plugin](#8-maven-build-lifecycle) |
| Q23 | [Difference between compile and provided scope?](#13-dependency-scopes) | [§13 — compile vs provided](#13-dependency-scopes) |
| Q24 | [Where does Maven store dependencies?](#7-maven-repositories) | [§7 — ~/.m2/repository](#7-maven-repositories) |

### Advanced

| # | Question | Answer in section |
|---|----------|-------------------|
| Q25 | [How does Maven resolve dependency conflicts?](#20-advanced-topics) | [§20 — Dependency Mediation](#20-advanced-topics) |
| Q26 | [How to skip tests?](#14-maven-commands) | [§14 — `-DskipTests`](#14-maven-commands) |
| Q27 | [How to force update dependencies?](#14-maven-commands) | [§14 — `-U` flag](#14-maven-commands) |
| Q28 | [How does Maven work internally?](#20-advanced-topics) | [§20 — Internal flow](#20-advanced-topics) |
| Q29 | [How to exclude a dependency?](#20-advanced-topics) | [§20 — Exclusions](#20-advanced-topics) |
| Q30 | [What are multi-module projects?](#20-advanced-topics) | [§20 — Multi-module](#20-advanced-topics) |
| Q31 | [What are Maven profiles?](#20-advanced-topics) | [§20 — Profiles](#20-advanced-topics) |
| Q32 | [What is Maven Wrapper (mvnw)?](#20-advanced-topics) | [§20 — Maven Wrapper](#20-advanced-topics) |
| Q33 | [Maven vs Gradle?](#18-maven-vs-gradle) | [§18 Maven vs Gradle](#18-maven-vs-gradle) |
| Q34 | [What is Spring Boot Maven plugin for?](#19-spring-boot-maven-plugin) | [§19 Spring Boot Plugin](#19-spring-boot-maven-plugin) |
| Q35 | [What is a fat JAR?](#19-spring-boot-maven-plugin) | [§19 — Fat JAR](#19-spring-boot-maven-plugin) |
| Q36 | [Lifecycle phase vs goal?](#10-goals) | [§10 Goals](#10-goals) |
| Q37 | [What does `mvn clean` do?](#14-maven-commands) | [§14 — mvn clean](#14-maven-commands) · [§15 Clean Lifecycle](#15-clean-lifecycle) |
| Q38 | [What is `mvn dependency:tree` for?](#16-maven-dependency-tree) | [§16 Dependency Tree](#16-maven-dependency-tree) |

### Quick revision checklist

- [ ] Can explain lifecycle order: validate → … → deploy → [§8](#8-maven-build-lifecycle)
- [ ] Know package vs install vs deploy differences → [§8](#8-maven-build-lifecycle)
- [ ] Know all dependency scopes → [§13](#13-dependency-scopes)
- [ ] Can draw dependency resolution flow → [§7](#7-maven-repositories)
- [ ] Know plugin vs goal vs lifecycle phase → [§8](#8-maven-build-lifecycle), [§9](#9-maven-plugins), [§10](#10-goals)

---

*End of document*
