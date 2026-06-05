# Template: Security Remediation

Use this template when the primary goal is fixing vulnerabilities (CVEs and structural security/reliability flaws) without changing framework or architecture.

## Plan Document Structure

```
TRANSFORMATION GOAL:
[Fix N CVEs and M structural flaws in application X to achieve [target: zero critical CVEs / compliance with standard Y / pass security audit]]

SOURCE APPLICATION:
- Language: [from stats]
- Framework: [from packages — name + version]
- Size: [LOC from stats, element count]
- Current CVE count: [total from quality_insights, broken down by severity — Critical: N, High: N, Medium: N, Low: N]
- Current structural flaw count: [total objects from structural-flaws, by flaw type]
- ISO-5055 violations: [if applicable — count by characteristic from iso-5055 insights]

TARGET STATE:
- Same application architecture — no framework or language change
- CVE target: [Zero critical/high / Zero all / specific CVEs to fix]
- Structural flaws target: [which flaw types to resolve]
- ISO-5055 target: [if applicable]

CVE REMEDIATION PLAN:
[For EACH CVE in scope. From quality_insight_violations with include_locations=True. Group by remediation strategy.]

Dependency Upgrades (fixes CVE without code changes):
- [CVE-ID] ([severity]): [library name] [current version] → [target version]
  Description: [one-line impact description]
  No code changes required — dependency update only.

Dependency Removals (library no longer needed):
- [CVE-ID] ([severity]): [library name] [current version] → REMOVE
  Description: [one-line impact description]
  Replacement: [alternative library or "functionality no longer needed"]
  Code changes: [files that import/use this library]

Code Fixes Required:
- [CVE-ID] ([severity]): [library or code pattern]
  Description: [one-line impact description]
  Fix strategy: [describe the code change needed]
  Affected locations:
    - [file:line] — [what the vulnerable code does]
    - [file:line] — [what the vulnerable code does]
  Remediation pattern: [show the fix pattern — e.g., "use parameterized queries instead of string concatenation"]

STRUCTURAL FLAW FIXES:
[For EACH flaw type in scope. From quality_insight_violations with include_locations=True.]

[Flaw Name] ([CWE-ID]) — [Security/Reliability/Efficiency] — [count] occurrences:
  Fix strategy: [general approach]
  Locations:
    - [file:line] — [brief description of violation + specific fix]
    - [file:line] — [brief description of violation + specific fix]

Example entries:
SQL Injection (CWE-89) — Security — 10 occurrences:
  Fix strategy: Replace string concatenation with parameterized queries / prepared statements
  Locations:
    - src/main/java/com/app/dao/UserDAO.java:45 — concatenates user input into SELECT query → use PreparedStatement
    - src/main/java/com/app/dao/OrderDAO.java:78 — concatenates order ID into DELETE query → use PreparedStatement

Reflected XSS (CWE-79) — Security — 5 occurrences:
  Fix strategy: Apply output encoding before rendering user input
  Locations:
    - src/main/webapp/pages/search.jsp:23 — echoes request parameter directly → use JSTL <c:out> or escapeXml
    - src/main/java/com/app/servlet/ResultServlet.java:56 — writes parameter to response → use StringEscapeUtils.escapeHtml4()

DEPENDENCY CHANGES:
[Summary of all dependency modifications.]

Upgrade:
- [groupId:artifactId] [old version] → [new version] — fixes: [CVE-ID list]

Remove:
- [groupId:artifactId] [version] — reason: [vulnerability / unused / replaced]

Add:
- [groupId:artifactId] [version] — reason: [replacement for removed dep / needed for fix]

ADDITIONAL INSTRUCTIONS:
- Do NOT change application architecture, framework, or language version
- Do NOT modify business logic — only security and quality fixes
- Preserve all existing tests — they must continue to pass
- Build command: [command]
- Test command: [command]
- Security scan command: [if applicable — e.g., OWASP dependency-check, Snyk]
- Scope exclusions: [directories/files not to touch — e.g., test fixtures with intentionally vulnerable code]
- [For WebGoat-style apps: note that some "vulnerabilities" are intentional for training — clarify which to fix vs. leave]

PATTERN-SPECIFIC MCP VERIFICATION:
Use CAST Imaging MCP tools throughout the transformation — not just the ones listed below. Any available MCP tool may be called if it provides useful context. These are the key verification checkpoints for this pattern:

- After each CVE fix (dependency upgrade): Call `packages` to verify the new version is in place and `package_interactions` to confirm no code still uses vulnerable APIs from the old version
- After each CVE fix (code change): Call `quality_insight_violations` with the specific CVE ID and include_locations=True to verify the exact file/line location was addressed
- After each structural flaw fix: Call `quality_insight_violations` with the flaw ID to verify the violation count decreased by the expected amount
- After fixing SQL injection: Call `object_details` focus="code" on the fixed method AND `data_graphs_involving_object` to verify data flow integrity wasn't broken
- After fixing XSS: Call `transactions_using_object` on the fixed output handler to verify all transaction paths through it are covered
- After removing a vulnerable dependency: Call `packages` to verify it's gone, then call `objects` filtered by import types for that library to confirm no orphaned references
- Periodically: Call `quality_insights` nature="cve" and nature="structural-flaws" to track overall progress against the target counts
- Call `application_iso_5055_explorer` if ISO-5055 compliance is a target — verify weakness counts are decreasing
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

### CVE REMEDIATION
- Use `quality_insights` nature="cve" to get the full CVE list with severity and affected packages
- Use `packages` tool to get current versions and check if safer versions exist (SaferClosestVersion, SafestVersion fields)
- Use `quality_insight_violations` with id=[CVE-ID] and include_locations=True to get code locations
- Use `package_interactions` to see which application objects use the vulnerable package

### STRUCTURAL FLAWS
- Use `quality_insights` nature="structural-flaws" to get all flaw types and counts
- Use `quality_insight_violations` with id=[flaw-id] and include_locations=True for exact locations
- Use `object_details` focus="code" on affected objects to see the vulnerable code pattern

### PRIORITISATION
- Critical/High CVEs with known exploits → fix first
- Structural flaws with Security factor → fix second
- Medium CVEs and Reliability flaws → fix third
- Low CVEs and Efficiency flaws → fix last (or exclude from scope)
