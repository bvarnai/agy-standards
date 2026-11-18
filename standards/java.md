# Java engineering standards

These standards apply to JVM-based applications whenever `pom.xml`, `build.gradle`, or `build.gradle.kts` is present in the workspace.

## Architecture & decomposition
1. **Interface-first boundaries & package privacy**:
   - Separate application lifecycle/entry points, DI wiring modules, and domain execution; never mix wiring logic into domain services.
   - Program to narrow, single-purpose interfaces and keep concrete implementations package-private (`Default*` or `*Impl`) unless external modules need direct construction.
   - Favor composition over inheritance; mark value types and implementations behind interfaces `final` (leave classes open when bytecode proxies such as JPA or AOP require subclassing).
2. **Composable workflows & behavioral patterns**:
   - Decompose multi-step pipelines into small, single-responsibility step/command classes over a typed context object rather than monolithic procedural methods.
   - Lock shared orchestration, error mapping, and cleanup in `final` template methods with protected `do*` hooks where a class family shares a fixed lifecycle.
   - Extend behavior via composition and Decorator/Forwarding wrappers; separate formatting/dispatch from data using **sealed interfaces + exhaustive pattern-matching `switch`** for closed variants, or **Strategy/Visitor** for open hierarchies.
3. **Modern Java baseline (21+ LTS) & utility libraries**:
   - Use `record`s for immutable DTOs, value objects, and parameter/context bundles; prefer `List.copyOf` / `Map.copyOf` for defensive copies.
   - Use `java.time` (`Instant`, `Duration`, injected `Clock` for deterministic tests) instead of `Date`/`Calendar`, and `BigDecimal` with an explicit `RoundingMode` (never `double`/`float`) for monetary or exact decimal math.
   - Prefer JDK standard collections first; use **[Google Guava](https://github.com/google/guava)** (`Multimap`, `BiMap`, `Splitter`) or **[Caffeine](https://github.com/ben-manes/caffeine)** (caching) instead of hand-rolling collection, parsing, or cache plumbing.

## Code discipline, null safety & logging
1. **Formatting & complexity guardrails**:
   - Follow the **[Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)** (enforced via Spotless `googleJavaFormat` or Checkstyle `google_checks.xml` when configured), unless the repository already defines a Checkstyle/formatter config.
   - Never use wildcard imports (`*`) or unused imports; add explicit `@Override` and concise Javadoc on public interfaces and contract methods.
   - Keep methods focused (≤ 50 lines target) and parameter lists short (≤ 4–5 parameters; group related inputs into a typed `record` or context object).
2. **Null safety & preconditions**:
   - Use **[JSpecify](https://jspecify.dev/)** (`@NullMarked`, `@Nullable`) or `Optional<T>` for return values; never use `Optional` for fields or parameters, and never return `null` for collections (`List.of()`).
   - Fail fast on public boundaries with explicit preconditions (`Objects.requireNonNull(...)`, Guava `Preconditions`, or Jakarta Bean Validation `@Valid`).
3. **Exceptions, resources & concurrency**:
   - Never swallow exceptions silently or catch raw `Exception` / `Throwable` without contextual wrapping (`throw new DomainException("...", cause)`) or explicit recovery.
   - Always manage `AutoCloseable` resources with `try-with-resources`; use `java.nio.file.Path` / `Files` with explicit `StandardCharsets.UTF_8` instead of `FileWriter` or platform-default encodings.
   - Use Java 21+ virtual threads for I/O-bound concurrency; avoid `synchronized` blocks around blocking I/O (prefer `ReentrantLock` or `ConcurrentHashMap` to prevent carrier-thread pinning on Java 21–23).
4. **Logging**:
   - Use **[SLF4J 2.x](https://www.slf4j.org/) + [Logback](https://logback.qos.ch/)** with parameterized messages (`log.error("Failed for {}", id, ex)`) or the SLF4J 2.x fluent API (`log.atError().setCause(ex).addKeyValue("id", id).log("Failed")`); never use `System.out` or `printStackTrace()`.

## Verification & testing standards
1. **Unit tests**:
   - Use **[JUnit 5](https://junit.org/junit5/) + [Mockito](https://site.mockito.org/) + [AssertJ](https://assertj.github.io/doc/)** (`@Test`, `@ParameterizedTest`, `@Nested`, `@ExtendWith(MockitoExtension.class)`, `@Mock`, `@InjectMocks`, `assertThat(...)`, `assertThatThrownBy(...)`).
2. **Integration & architecture tests**:
   - For persistence or broker integration tests, use [Testcontainers](https://java.testcontainers.org/) against real databases/brokers rather than in-memory substitutes (e.g., avoid H2 for PostgreSQL).
   - Enforce package dependency direction and layering rules with [ArchUnit](https://www.archunit.org/) when configured.
3. **Build & static analysis**:
   - Run `./mvnw test` / `./mvnw verify` (Maven) or `./gradlew check` (Gradle) with zero compiler warnings and zero configured Checkstyle, SpotBugs, PMD, Spotless, or ErrorProne/NullAway violations.
