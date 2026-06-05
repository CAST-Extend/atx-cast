# Template: Language Version Upgrade

Use this template when upgrading language version (e.g., Java 8 → 17, Python 3.8 → 3.12, Node.js 16 → 22, .NET Framework → .NET 8).

## Plan Document Structure

```
TRANSFORMATION GOAL:
[Language being upgraded, from version X to version Y, primary motivation (EOL runtime, security patches, performance, new language features)]

SOURCE APPLICATION:
- Language: [language + current version — from stats/packages/build files]
- Framework: [frameworks in use — these stay but may need compatible versions]
- Size: [LOC from stats, element count from object_profiles]
- Build system: [build tool + config file paths from source_files — e.g., Maven pom.xml, Gradle build.gradle, pyproject.toml]
- Runtime: [current runtime/JDK/interpreter version and distribution]

TARGET STATE:
- Language version: [exact target version — e.g., Java 17, Python 3.12, Node.js 22]
- Runtime: [target runtime — e.g., Amazon Corretto 17, CPython 3.12]
- Framework versions: [any framework version bumps required for language compatibility]
- Build tool version: [if build tool itself needs updating — e.g., Maven 3.9+, Gradle 8+]

DEPRECATED APIs TO REPLACE:
[From quality_insight_violations (cloud-detection-patterns for deprecated versions), advisors rules/violations, and object_details focus="code". List every deprecated/removed API usage with exact location.]

- [Deprecated API/method/class] at [file:line] → Replace with [new API/method/class]

Example entries:
- javax.xml.bind.* at src/main/java/com/app/Util.java:45 → Replace with jakarta.xml.bind.* (add jaxb-api dependency)
- sun.misc.Unsafe at src/main/java/com/app/Memory.java:12 → Replace with VarHandle API
- Thread.stop() at src/main/java/com/app/Worker.java:88 → Replace with interrupt-based cancellation

BUILD/PROJECT FILE CHANGES:
[From source_files — list every build config that needs language version target update.]

- [filepath] — Change [setting name] from [old value] to [new value]

Example entries:
- pom.xml — Change <maven.compiler.source> from 1.8 to 17
- pom.xml — Change <maven.compiler.target> from 1.8 to 17
- .github/workflows/ci.yml — Change java-version from '8' to '17'
- Dockerfile — Change FROM openjdk:8 to FROM amazoncorretto:17

DEPENDENCIES TO UPDATE:
[From packages tool — deps that need version bumps for target language compatibility.]

Upgrade:
- [dep name] [current version] → [target version] — reason: [incompatible with target language / deprecated / CVE]

Remove:
- [dep name] [current version] — reason: [now built into language / no longer needed]

Add:
- [dep name] [version] — reason: [API removed from language, now separate module]

SYNTAX/IDIOM MODERNISATION:
[Language-specific modernisation opportunities. Only include if user requested modernisation beyond bare minimum compatibility. Identified from object_details code patterns.]

- [Old pattern] → [New pattern] — [count] occurrences in [files/directories]

Example entries (Java 8→17):
- Anonymous Runnable → Lambda expressions — 23 occurrences in src/main/java/com/app/services/
- String concatenation in loops → StringBuilder or String.join — 8 occurrences
- Optional.get() without isPresent → Optional.orElseThrow() — 5 occurrences
- try-finally resource management → try-with-resources — 12 occurrences

Example entries (Python 3.8→3.12):
- typing.Dict/List/Tuple → dict/list/tuple built-in generics — throughout codebase
- os.path.join → pathlib.Path — 15 occurrences
- format strings → f-strings — 30 occurrences

QUALITY ISSUES TO FIX:
[Only CVEs and flaws directly related to the language version or deps being upgraded.]

- [CVE-ID] ([severity]): [dep name] [version] — Fixed by upgrading to [version]

ADDITIONAL INSTRUCTIONS:
- Build command to validate: [exact command — e.g., mvn clean verify -DskipTests=false]
- Test command: [exact command — e.g., mvn test, pytest, npm test]
- Minimum test pass rate: [percentage or "all existing tests must pass"]
- Files/directories out of scope: [list any generated code, vendored deps, etc.]
- [Whether to modernise idioms or strictly do minimum-viable upgrade]
- [Whether to update CI/CD pipeline configs]
- [Whether to update container/deployment configs]

PATTERN-SPECIFIC MCP VERIFICATION:
Use CAST Imaging MCP tools throughout the transformation — not just the ones listed below. Any available MCP tool may be called if it provides useful context. These are the key verification checkpoints for this pattern:

- After updating dependency versions: Call `packages` and `package_interactions` to verify the new versions are correct and no code still references old/incompatible APIs
- After replacing deprecated APIs: Call `objects` filtered by old API types to confirm zero remnants of deprecated usage remain
- After modifying build/project files: Call `source_files` to verify all build configs are accounted for (CI/CD, Dockerfiles, etc.)
- After completing all changes: Call `quality_insight_violations` for deprecated-API and language-version patterns to verify violation count matches expectations
- Call `advisors` with language-migration rules to confirm compliance
- Call `transactions_using_object` on any method whose signature changed during modernisation to verify callers were updated
- Call `inter_applications_dependencies` if the application has cross-app consumers — ensure their interfaces weren't broken
CAST IMAGING MCP USAGE DURING EXECUTION:
This plan requires CAST Imaging MCP access during code transformation. Use it for:

1. Before modifying any file:
   - Call `object_details` focus="code" to retrieve current implementation if not already visible
   - Call `object_details` focus="inward" to check all callers before changing a method signature
   - Call `object_details` focus="outward" to understand dependencies before refactoring
   - Call `architectural_graph` or `architectural_graph_focus` to understand which component/layer the object belongs to — prevent accidental cross-layer coupling during refactoring
   - Call `packages` and `package_interactions` before upgrading any dependency to understand what code interacts with it

2. During refactoring:
   - Call `transactions_using_object` on EVERY object being modified (not just when uncertain) to quantify blast radius
   - Call `data_graphs_involving_object` to check data entity interactions affected by the change
   - Call `pathfinder_hierarchy_details` focus="pathfinder" to trace execution paths between source and target objects
   - Call `application_database_explorer` to check table/column details before modifying any data access code

3. After completing a transformation step:
   - Call `object_details` focus="inward" on modified objects to verify no callers were missed
   - Call `transaction_details` focus="type_graph" on affected transactions to confirm flow integrity
   - Call `quality_insight_violations` to cross-reference that the violation location was properly addressed (verify the fix matches the exact file/line reported)

4. When encountering ambiguity:
   - Call `objects` with filters to find related objects not listed in the plan
   - Call `source_file_details` nature="inventory" to see what else is in a file before modifying it
   - Call `object_details` focus="code" on sibling objects to understand surrounding patterns

Application name for all CAST Imaging MCP calls: [APPLICATION_NAME]

```

## Guidance for Populating This Template

### DEPRECATED APIs
- Use `quality_insights` with nature="cloud-detection-patterns" — look for "deprecated language or framework versions" pattern
- Use `advisors` rules — check for language-specific migration rules
- Use `quality_insight_violations` with include_locations=True for each relevant pattern
- Use `object_details` focus="code" to see actual usage patterns

### BUILD/PROJECT FILE CHANGES
- Use `source_files` to find all build configs (pom.xml, build.gradle, package.json, pyproject.toml, *.csproj)
- Check for CI/CD configs (.github/workflows/, Jenkinsfile, .gitlab-ci.yml)
- Check for container configs (Dockerfile, docker-compose.yml)

### DEPENDENCIES
- Use `packages` tool to get full dependency list with versions
- Cross-reference each package's compatibility with target language version
- Check for CVEs that are fixed by the version bump

### SYNTAX MODERNISATION
- Only include if explicitly requested by user
- Use `object_details` focus="code" on objects with high complexity to find modernisation candidates
