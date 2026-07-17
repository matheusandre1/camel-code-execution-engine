# CLAUDE.md

## Project Overview

Camel Code Execution Engine (CCE) — a microservice that executes Apache Camel YAML routes via gRPC and exposes MCP-compliant code generation tools for AI agents. Part of the [Wanaku](https://github.com/wanaku-ai/wanaku) platform.

## Build

- **Java 21**, single Maven module
- **Dependency**: requires `wanaku-capabilities-java-sdk` installed locally first:
  ```
  cd ../wanaku-capabilities-java-sdk && mvn -DskipTests -B install
  ```
- **Build**: `mvn -B clean package`
- **Test**: `mvn -B install` (includes unit + integration via failsafe)
- **Static analysis**: `mvn -B verify -Pstatic-analysis -DskipTests`
- **Format**: automatic via Spotless (Palantir Java Format) during `compile` phase
- **Do not parallelize** Maven — runs fine single-threaded

## Code Style

- Palantir Java Format (enforced by `spotless-maven-plugin`)
- Import order: `jakarta|javax`, `org.w3c|org.xml`, `java|org|io`, then everything else; wildcards last
- Unused imports removed automatically
- No wildcard imports

## Project Structure

```
src/main/java/ai/wanaku/code/engine/camel/
├── CamelEngineMain.java          # CLI entry point (picocli)
├── WanakuCamelManager.java       # Camel context management
├── codegen/                      # Code generation tools subsystem
│   ├── tools/                    # Individual MCP tool implementations
│   │   ├── SearchServicesTool    # Lists available Kamelets
│   │   ├── ReadKameletTool       # Returns Kamelet YAML definition
│   │   └── GenerateOrchestrationTool  # Returns route templates
│   ├── CodeGenConfig.java
│   ├── CodeGenDiscoveryCallback.java
│   ├── CodeGenResourceLoader.java
│   ├── CodeGenToolRegistrar.java
│   └── CodeGenToolService.java
├── downloader/TarBz2Downloader.java
├── grpc/
│   ├── CodeExecutorService.java       # gRPC service: route execution
│   └── CodeGenToolInvokerService.java # gRPC service: tool invocation
└── util/
    ├── ArchiveExtractor.java
    └── VersionHelper.java
```

## Key Dependencies

- Apache Camel 4.x (core, yaml-dsl, kamelet-main)
- gRPC (server, via wanaku SDK)
- picocli (CLI argument parsing)
- JGit (Git repository cloning)
- Commons Compress (tar.bz2 extraction)
- JUnit Jupiter 6.x (tests)

## CI

- GitHub Actions: multi-platform (ubuntu x86/arm, macOS, Windows experimental)
- PR builds run on all platforms; main builds also push container images to `quay.io/wanaku/camel-code-execution-engine`
- Static analysis (PMD + SpotBugs) runs on PRs as a separate workflow

## Container

- Base image: `ubi9/openjdk-21-runtime`
- Fat JAR: `target/camel-code-execution-engine-app.jar`
- gRPC port: 9190 (default)

## Related Repositories

- `wanaku-ai/wanaku` — MCP Router platform
- `wanaku-ai/wanaku-capabilities-java-sdk` — capabilities SDK (build dependency)
- `wanaku-ai/camel-integration-capability` — Camel integration capability
