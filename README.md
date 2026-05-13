# gradle9-toolchain-only

> **Reproducer for [TKA-10324](https://mend-io.atlassian.net/browse/TKA-10324)**

## Issue

The Mend GHE integration (SCA Wrapper / scanner) fails to resolve Gradle 9 dependencies when the project uses the **toolchain-only** Java configuration:

```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}
```

...without also setting `sourceCompatibility` / `targetCompatibility`.

The scanner runs `gradle whitesourceDependenciesTask` which triggers `:compileJava`. Since no JDK 17 is pre-installed in the scanner container and no toolchain download repository is configured, the build fails:

```
Cannot find a Java installation on your machine (Linux ... amd64) matching:
  {languageVersion=17, vendor=any vendor, implementation=vendor-specific, nativeImageCapable=false}.
Toolchain download repositories have not been configured.
BUILD FAILED in 8s
```

The result is **0 dependencies detected** even though the project has real dependencies.

## What makes this different from other Gradle 9 projects

| Configuration style | Behaviour |
|---|---|
| `sourceCompatibility = JavaVersion.VERSION_17` | ✅ Works — Gradle does NOT invoke the Java toolchain |
| `java { toolchain { languageVersion = JavaLanguageVersion.of(17) } }` | ❌ Fails — Gradle demands JDK 17 via toolchain resolution |

The `kotlin { jvmToolchain(N) }` shorthand (used in `gradle9-modern`) uses the Kotlin plugin's own toolchain wrapper and behaves differently.

## Project structure

```
├── build.gradle          # Groovy DSL — plain java plugin + toolchain block
├── settings.gradle
├── gradle/wrapper/
│   └── gradle-wrapper.properties  # Gradle 9.0.0
├── src/main/java/com/example/App.java
├── .whitesource
└── autotest_config.json  # Expects 0 deps (negative test)
```

## Environment

- **Gradle**: 9.0.0
- **Java toolchain**: `languageVersion = JavaLanguageVersion.of(17)` (toolchain-only, no `sourceCompatibility`)
- **Build file**: Groovy DSL (`build.gradle`) — matches customer project
- **Mend product**: Mend for GitHub Enterprise (GHE), SCA Wrapper v26.3.1+
- **Customer**: Capital One (Platinum)
