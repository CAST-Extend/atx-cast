# Template: Code Refactoring

Use this template when the primary goal is improving code quality, reducing complexity, removing dead code, fixing coupling issues, and applying modern design patterns — WITHOUT changing framework, language version, or architecture.

This is NOT for fixing CVEs/security vulnerabilities (use security-remediation.md) or changing frameworks (use framework-migration.md).

## Plan Document Structure

```
TRANSFORMATION GOAL:
[Improve code quality of application X by addressing N structural flaws, reducing complexity in M hotspot methods, removing dead code, and applying modern idioms. No framework or architecture change.]

SOURCE APPLICATION:
- Language: [from stats]
- Framework: [from packages — name + version — stays the same]
- Size: [LOC from stats, element count]
- Complexity hotspots: [count of high-complexity objects from objects tool sorted by cyclomatic complexity]
- Structural flaws: [total from structural-flaws insights — Efficiency: N, Reliability: N, Maintainability: N]
- Dead code: [count from advisors — unused stored procedures, unreachable methods, etc.]
- Coupling issues: [from architectural_graph links — tightly coupled components]
- ISO-5055 violations: [if applicable — by characteristic: Maintainability: N, Reliability: N, Efficiency: N]

TARGET STATE:
- Same framework, language, and architecture
- Complexity target: [e.g., "No method exceeds cyclomatic complexity of 20" / "Reduce top 10 hotspots by 50%"]
- Structural flaws target: [which flaw types to resolve — Efficiency / Reliability / Maintainability]
- Dead code: [removed / flagged for review]
- Coupling: [specific decoupling goals]

COMPLEXITY REDUCTION:
[From objects tool sorted by cyclomatic complexity (highest first) + object_details focus="code" on hotspots.]

Hotspot 1: [object name] — Cyclomatic complexity: [N]
  File: [filepath:line]
  Current pattern: [brief description of why it's complex — nested loops, large switch, deep conditionals]
  Refactoring strategy: [extract method / replace conditional with polymorphism / decompose into smaller methods / introduce strategy pattern]

Hotspot 2: [object name] — Cyclomatic complexity: [N]
  File: [filepath:line]
  Current pattern: [description]
  Refactoring strategy: [strategy]

[Continue for each hotspot in scope]

STRUCTURAL FLAW FIXES:
[For EACH non-security flaw type in scope. From quality_insight_violations with include_locations=True. Group by factor: Efficiency, Reliability, Maintainability.]

Efficiency Flaws:

[Flaw Name] ([CWE-ID]) — [count] occurrences:
  Impact: [why this hurts performance]
  Fix strategy: [general approach]
  Locations:
    - [file:line] — [description + specific fix]
    - [file:line] — [description + specific fix]

Example entries:
SQL queries inside loops (CWE-1050) — 7 occurrences:
  Impact: N+1 query problem causing excessive DB round-trips
  Fix strategy: Replace iterative queries with batch/set-oriented queries using JOINs or IN clauses
  Locations:
    - src/main/java/com/app/dao/OrderDAO.java:45 — loops over customers, queries orders per customer → use single JOIN query
    - src/main/java/com/app/service/ReportService.java:78 — fetches details in loop → use batch fetch

Reliability Flaws:

[Flaw Name] ([CWE-ID]) — [count] occurrences:
  Impact: [why this hurts reliability]
  Fix strategy: [general approach]
  Locations:
    - [file:line] — [description + specific fix]

Example entries:
Empty catch blocks (CWE-1069) — 3 occurrences:
  Impact: Exceptions swallowed silently, masking failures
  Fix strategy: Add proper logging + rethrow or handle appropriately
  Locations:
    - src/main/java/com/app/service/PaymentService.java:92 — catches SQLException, does nothing → log error + rethrow as ServiceException

Maintainability Flaws:

[Flaw Name] — [count] occurrences:
  Impact: [why this hurts maintainability]
  Fix strategy: [general approach]
  Locations:
    - [file:line] — [description + specific fix]

DEAD CODE REMOVAL:
[From advisors rules — "Not Used Stored Procedures", "Not Used Methods", unreachable code. Also from quality_insight_violations.]

Unused objects to remove:
- [object type]: [object name] at [filepath] — reason: [no callers / unreachable / orphaned]
- [object type]: [object name] at [filepath] — reason

Example entries:
- Stored Procedure: sp_legacy_report at db/procedures/sp_legacy_report.sql — no callers in application
- Java Method: UserDAO.getByLegacyId at src/main/java/com/app/dao/UserDAO.java:145 — zero inward references
- Java Class: OldMigrationHelper at src/main/java/com/app/util/OldMigrationHelper.java — entire class unused

COUPLING REDUCTION:
[From architectural_graph links between components + object_details focus="inward"/"outward" on tightly coupled objects.]

Coupling issue 1: [Component A] ↔ [Component B] — [link count] dependencies
  Problem: [why this coupling is harmful — circular dependency / god class / feature envy]
  Affected objects:
    - [object] in [Component A] calls [N] methods in [Component B]
    - [object] in [Component B] references [N] objects in [Component A]
  Refactoring strategy: [extract interface / introduce mediator / move method / apply dependency inversion]

[Continue for each coupling issue in scope]

IDIOM MODERNISATION:
[Modern language/framework idioms to apply. From object_details focus="code" on representative objects. Only include if user requested.]

- [Old idiom] → [Modern idiom]: [count] occurrences in [files/directories]
  Priority: [Must (deprecated) / Should (cleaner) / Optional (style preference)]

Example entries (Java):
- Callbacks/anonymous classes → Lambda expressions: 15 occurrences in src/main/java/com/app/services/
- Manual null checks → Optional: 8 occurrences in src/main/java/com/app/dao/
- StringBuilder loops → String.join / Collectors.joining: 5 occurrences
- Imperative collection processing → Stream API: 12 occurrences

Example entries (Python):
- Manual file open/close → context managers (with): 8 occurrences
- Type comments → type annotations: throughout
- String formatting (%) → f-strings: 20 occurrences

Example entries (JavaScript/TypeScript):
- var declarations → const/let: 30 occurrences
- Callback chains → async/await: 12 occurrences
- CommonJS require → ES module import: 15 occurrences

ISO-5055 VIOLATIONS TO FIX:
[If ISO-5055 compliance is in scope. From application_iso_5055_explorer + quality_insight_violations.]

Characteristic: [Maintainability / Reliability / Efficiency / Security]
  Weakness: [weakness name] — [count] violations
  Locations:
    - [file:line] — [fix description]

FILES IN SCOPE:
[Every file being modified or deleted. From source_files + violation locations.]

- [filepath] — [Refactor: reduce complexity / Fix: flaw type / Delete: dead code / Modernise: idioms]

ADDITIONAL INSTRUCTIONS:
- Do NOT change application framework, language version, or architecture
- Do NOT change public API signatures or interfaces (preserve backward compatibility)
- All existing tests must continue to pass after refactoring
- Preserve all business logic — refactoring must be behavior-preserving
- Build command: [command]
- Test command: [command]
- Code coverage: [must not decrease / target percentage]
- Scope exclusions: [files/directories not to touch]
- [Whether to add new unit tests for refactored code]
- [Whether to update comments/documentation for refactored code]
- [Maximum method length / complexity thresholds to enforce]

PATTERN-SPECIFIC MCP VERIFICATION:
Use CAST Imaging MCP tools throughout the transformation — not just the ones listed below. Any available MCP tool may be called if it provides useful context. These are the key verification checkpoints for this pattern:

- After refactoring a complexity hotspot: Call `objects` filtered by the refactored method name to check if cyclomatic complexity decreased (results are sorted by complexity, so position should drop)
- After removing dead code: Call `advisors` with focus="violations" for "Not Used" rules to verify the dead object is no longer reported. Call `object_details` focus="inward" to double-check zero callers before deletion.
- After fixing coupling: Call `architectural_graph` at component level with mode="links" to verify the link count between decoupled components decreased
- After extracting methods: Call `object_details` focus="inward" on the new extracted method to verify it's properly called, and focus="outward" on the original to verify it now delegates correctly
- After idiom modernisation: Call `object_details` focus="code" on modernised objects to verify patterns are correct
- Periodically: Call `quality_insight_violations` for structural-flaw patterns (Efficiency, Reliability, Maintainability) to track progress against target flaw counts
- Call `transactions_using_object` on every refactored object to verify no business flows were broken
- Call `data_graphs_involving_object` on refactored data access methods to ensure data entity interactions are preserved
- Call `application_iso_5055_explorer` if ISO-5055 compliance is a target — verify weakness counts are decreasing
- Call `pathfinder_hierarchy_details` focus="hierarchy" on refactored classes to verify the new structure is clean
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

### COMPLEXITY HOTSPOTS
- Use `objects` tool (results are sorted by cyclomatic complexity, highest first)
- Use `object_details` focus="code" on top N complex objects to understand the code
- Use `object_details` focus="inward" and "outward" to understand dependencies before refactoring

### STRUCTURAL FLAWS
- Use `quality_insights` nature="structural-flaws" — filter for non-security factors (Efficiency, Reliability)
- Use `quality_insight_violations` with include_locations=True for exact locations
- Use `object_details` focus="code" on affected objects to see the actual pattern

### DEAD CODE
- Use `advisors` — look for rules like "Not Used Stored Procedures", unused methods/classes
- Use `advisor_occurrences` to get specific unused objects
- Use `object_details` focus="inward" to verify zero callers

### COUPLING
- Use `architectural_graph` at component level with mode="links" to identify heavy coupling
- Use `object_details` focus="inward"/"outward" on objects at component boundaries
- Use `transactions` to understand which transaction flows cross component boundaries

### ISO-5055
- Use `application_iso_5055_explorer` to list characteristics and weaknesses
- Use `quality_insight_violations` nature="iso-5055" with include_locations=True
