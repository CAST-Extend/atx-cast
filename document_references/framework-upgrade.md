# Template: Framework Upgrade

Use this template when upgrading to a newer version of the SAME framework (e.g., Spring Boot 2 → 3, Angular 14 → 18, Django 3 → 5, .NET Framework 4.8 → .NET 8, React 16 → 18, Rails 5 → 7).

This is NOT for switching frameworks (use framework-migration.md) or upgrading only the language (use language-upgrade.md).

## Plan Document Structure

```
TRANSFORMATION GOAL:
[Framework being upgraded, from version X to version Y, primary motivation (EOL version, CVEs in current version, new features needed, dependency compatibility)]

SOURCE APPLICATION:
- Language: [from stats]
- Framework: [exact framework name + current version — from packages]
- Size: [LOC from stats, element counts from object_profiles — list framework-specific object types and counts]
- Build system: [build tool + config file path from source_files]
- Framework config files: [list all framework configuration files from source_files — e.g., application.yml, angular.json, settings.py]
- Plugins/Extensions: [framework plugins or modules in use — from packages and architectural_graph]

TARGET STATE:
- Framework: [exact framework name + target version]
- Language: [if the framework upgrade requires a language version bump — e.g., Spring Boot 3 requires Java 17+]
- Build tool: [if build tool version must change — e.g., Angular 18 requires Node 18+]
- Breaking changes scope: [summary of major breaking changes between versions from framework's migration guide]

BREAKING CHANGES TO ADDRESS:
[From quality_insight_violations (deprecated patterns), advisors rules, packages tool, and object_details focus="code". Organized by breaking change category.]

Namespace/Package Renames:
- [Old namespace] → [New namespace]: [count] occurrences in [files]
  Example: javax.* → jakarta.* (Spring Boot 3), @angular/http → @angular/common/http

Removed/Replaced APIs:
- [Removed API/class/method] at [file:line] → Replace with [new API]
  Reason: [why it was removed in target version]

Configuration Changes:
- [Old config property/format] → [New config property/format] in [config file path]

Behavioral Changes:
- [What changed in behavior] — affects [file:line or component]
  Fix: [what code needs to change to preserve behavior]

Dependency Compatibility:
- [Dep name] [current version] — incompatible with [target framework version]
  Upgrade to: [compatible version]
  Breaking changes in dep upgrade: [if any — list what changes in the dep's API]

FRAMEWORK CONFIGURATION FILE CHANGES:
[List every framework config file that needs modification. From source_files.]

- [filepath] — Changes:
  - [specific property/section]: [old value/format] → [new value/format]
  - [specific property/section]: [old value/format] → [new value/format]

Example entries:
- pom.xml — Changes:
  - spring-boot-starter-parent version: 2.7.x → 3.3.x
  - java.version property: 11 → 17
  - Remove: springfox-swagger dependencies (replaced by springdoc)
- application.yml — Changes:
  - spring.redis.* → spring.data.redis.* (property prefix changed)
  - server.servlet.context-path (verify still valid)
- angular.json — Changes:
  - build architect target options for new compiler

DEPRECATED PATTERNS TO MODERNISE:
[Framework patterns that still work but are deprecated in target version. From quality_insight_violations and object_details focus="code".]

- [Deprecated pattern] → [Modern replacement]: [count] occurrences
  Locations: [file:line, file:line, ...]
  Priority: [Must fix (removed in next version) / Should fix (deprecated warning) / Optional (still works)]

Example entries (Spring Boot 2→3):
- WebSecurityConfigurerAdapter → SecurityFilterChain @Bean: 3 occurrences
  Locations: SecurityConfig.java:15, AdminSecurityConfig.java:22, ApiSecurityConfig.java:10
  Priority: Must fix (class removed in Spring Security 6)
- @RequestMapping for all HTTP methods → specific @GetMapping/@PostMapping: 45 occurrences
  Priority: Optional (still works but deprecated style)

Example entries (Angular 14→18):
- Module-based architecture → Standalone components: 30 components
  Priority: Should fix (modules still work but standalone is new default)
- RxJS pipe operators import from 'rxjs/operators' → import from 'rxjs': 25 occurrences
  Priority: Must fix (old import paths removed)

FILES IN SCOPE:
[Every file that needs modification. From source_files, packages, quality_insight_violations.]

- [filepath] — [what changes: namespace rename / API replacement / config update / dep version bump]

QUALITY ISSUES FIXED BY UPGRADE:
[CVEs and flaws that are resolved by moving to the target framework version.]

- [CVE-ID] ([severity]): [description] — Fixed by upgrading [framework] to [version]
- [Structural flaw]: [description] — Fixed by using [new framework feature]

REMAINING QUALITY ISSUES:
[CVEs and flaws that are NOT fixed by the framework upgrade alone and need additional attention.]

- [CVE-ID] ([severity]): [dep name] — Still affected, needs separate upgrade to [version]

DEPENDENCIES TO UPDATE:
[From packages tool — all deps that need version changes for framework compatibility.]

Upgrade (required for compatibility):
- [groupId:artifactId] [current] → [target] — reason: [incompatible with target framework / transitive conflict]

Upgrade (recommended):
- [groupId:artifactId] [current] → [target] — reason: [deprecated version / CVE fix / better integration]

Remove:
- [groupId:artifactId] [current] — reason: [now built into framework / replaced by framework module / incompatible]

Add:
- [groupId:artifactId] [version] — reason: [replaces removed module / new framework requirement]

Replace:
- [old dep] → [new dep] [version] — reason: [framework changed preferred library — e.g., springfox → springdoc]

MIGRATION STEPS (ORDERED):
[Framework upgrades often have a specific order. Document it.]

1. [First step — e.g., "Update language version to minimum required"]
2. [Second step — e.g., "Update framework version in build file"]
3. [Third step — e.g., "Fix compilation errors from namespace renames"]
4. [Fourth step — e.g., "Replace removed APIs"]
5. [Fifth step — e.g., "Update configuration files"]
6. [Sixth step — e.g., "Update dependent libraries"]
7. [Seventh step — e.g., "Fix deprecated patterns"]
8. [Eighth step — e.g., "Run tests and fix failures"]

ADDITIONAL INSTRUCTIONS:
- Build command: [command — run after each major step to catch issues early]
- Test command: [command]
- Framework migration guide reference: [URL to official migration guide if available]
- Intermediate version stops: [if jumping multiple major versions, list intermediate stops — e.g., "Spring Boot 2.7 → 3.0 → 3.3, not directly 2.3 → 3.3"]
- Scope exclusions: [files/directories not to touch]
- [Whether to address deprecated patterns or only breaking changes]
- [Whether to update test dependencies/patterns to match new framework conventions]

PATTERN-SPECIFIC MCP VERIFICATION:
Use CAST Imaging MCP tools throughout the transformation — not just the ones listed below. Any available MCP tool may be called if it provides useful context. These are the key verification checkpoints for this pattern:

- After namespace/package renames: Call `objects` filtered by old namespace types to confirm zero remnants (e.g., no javax.* objects remaining after jakarta migration)
- After replacing removed APIs: Call `object_details` focus="code" on converted objects to verify new patterns are correct
- After updating dependencies: Call `packages` and `package_interactions` to verify all ecosystem deps are at compatible versions and no code references old package APIs
- After configuration changes: Call `source_files` to verify all framework config files were found and updated
- After completing all steps: Call `quality_insight_violations` for deprecated-framework patterns to verify violation count is zero
- Call `advisors` with framework-specific upgrade rules to confirm all rules satisfied
- Call `transactions` to verify all API/UI endpoints still exist and function (no endpoints lost during migration)
- Call `architectural_graph` at component level to verify architecture integrity wasn't degraded
- Call `data_graphs_involving_object` on any modified data access objects to ensure data flows are intact
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

### BREAKING CHANGES
- Use `quality_insights` nature="cloud-detection-patterns" — "deprecated language or framework versions" pattern
- Use `advisors` rules — check for framework-specific upgrade rules
- Use `quality_insight_violations` with include_locations=True for deprecated pattern violations
- Use `packages` to identify current framework version and all related ecosystem packages
- Use `object_details` focus="code" on objects using deprecated APIs to understand current usage

### CONFIGURATION FILES
- Use `source_files` to find all framework config files
- Use `source_file_details` nature="inventory" to see what objects are defined/configured in each file

### DEPRECATED PATTERNS
- Use `object_profiles` to understand the distribution of framework-specific object types
- Use `objects` filtered by framework-specific types to get full inventory
- Use `object_details` focus="code" on samples to see current pattern usage

### DEPENDENCY COMPATIBILITY
- Use `packages` to get complete dependency list with versions
- Cross-reference each package against target framework's compatibility matrix
- Check SaferClosestVersion and SafestVersion fields from packages tool
