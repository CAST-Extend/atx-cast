# cast-imaging-to-aws-transform-phase1

## Description
Assess an application using CAST Imaging MCP structural analysis, determine the optimal modernisation path through data-driven probing, and generate a detailed transformation plan document ready for execution in a separate Phase 2 TD. This TD performs NO code changes — it is purely analytical.

## Entry Criteria

1. CAST Imaging MCP server is configured and responding to tool calls
2. At least one application is available in the CAST Imaging environment with completed analysis
3. Target repository source code is accessible locally (for file verification only)

## Implementation Steps

### Stage A: Assessment

1. Call `applications` to list available applications in the CAST Imaging environment
2. Let the user select an application to assess
3. Once selected, gather comprehensive assessment data:
   - `stats` for size, complexity, technologies, and element types
   - `architectural_graph` at layer, component, sub-component, and technology-category levels (both nodes and links)
   - `quality_insights` across ALL natures: cve, structural-flaws, iso-5055, cloud-detection-patterns, green-detection-patterns
   - `packages` for external dependencies with versions
   - `transactions` and `transaction_profiles` for API/UI endpoints
   - `data_graphs` and `data_graph_profiles` for data entity interactions
   - `inter_applications_dependencies` for cross-app coupling
   - `application_database_explorer` for database table inventory
   - `application_iso_5055_explorer` for ISO-5055 characteristics and weaknesses
   - `object_profiles` for object type distribution
   - `advisors` with focus="list" then focus="rules" for each advisor — get all migration/modernisation rules
4. For the top 5 largest transactions, gather `transaction_details` with focus="type_graph" and focus="complexity"
5. For each structural flaw and cloud blocker, gather ALL violations using a tiered approach:
   - First call `quality_insight_violations` (without `include_locations`) to get category summaries and total counts per rule
   - Then iterate through each rule/category calling `quality_insight_violations` with `include_locations=True`, processing paginated results
   - Process in severity order: Critical first, then High, Medium, Low
   - If a single rule has more than 50 violations, batch the detailed location retrieval across multiple paginated calls and summarise patterns (e.g., "47 instances of hardcoded URLs, all following pattern X in files matching Y")
   - ALL violations must be accounted for in the final plan — none may be skipped. Group repeated patterns together with counts rather than listing each individually.
6. For up to 5 representative objects per violation category, use `object_details` with focus="code" to retrieve actual source code patterns. Select objects that represent distinct implementation patterns within that category.
7. Use `source_files` with file paths discovered in steps 5-6 to verify those files exist in the local repository. Use `source_file_details` nature="inventory" to confirm the objects at those locations match what CAST Imaging reports.
8. Synthesise all findings into a structured Application Assessment Summary

### Stage B: Probing & Target State Definition

9. Present the assessment summary to the user with quantified findings
10. Ask targeted probing questions based on what the data reveals. NEVER present a generic menu. Base questions on specifics:
    - If CVEs found: "I found X critical CVEs in [library]. Do you want to upgrade [library] or replace the framework entirely?"
    - If outdated framework: "[Framework version] is [N years] old. What is your target framework/version?"
    - If structural flaws: "There are N security/reliability flaws ([top CWE types]). Is security remediation a priority for this transformation?"
    - If cloud blockers: "There are N cloud blockers (hardcoded URLs, stateful sessions, etc.). Are you targeting cloud deployment on AWS? Which AWS services are preferred (ECS/EKS/Lambda)?"
    - If high complexity: "There are N objects with cyclomatic complexity above [threshold]. Is code quality improvement in scope?"
    - If multiple technologies: "The app spans [tech list]. Which tier(s) are in scope for transformation?"
    - If dead code detected: "[N] objects flagged as unused by advisor rules. Should dead code removal be included?"
11. Based on answers, determine the modernisation pattern. There are 4 primary categories, each with distinct sub-patterns:

    TECHNOLOGY UPGRADE:
    - Language Version Upgrade (e.g., Java 8 to 17, Python 3.8 to 3.12, Node.js 16 to 22)
    - Framework Version Upgrade (e.g., Spring Boot 2 to 3, Angular 14 to 18, Django 3 to 5)
    - Framework Migration (e.g., JBoss/WildFly to Spring Boot, Struts to Spring MVC, jQuery to React)

    SECURITY REMEDIATION:
    - CVE Remediation (vulnerable dependency upgrades, code-level vulnerability fixes)
    - Structural Flaw Fixes (CWE violations — SQL injection, XSS, resource leaks, error handling, reliability)

    CODE REFACTORING:
    - Dead Code Removal (unused methods, classes, stored procedures identified by advisor rules)
    - Complexity Reduction (extract methods, decompose conditionals, simplify high-cyclomatic-complexity hotspots)
    - Coupling Fixes (reduce inter-component dependencies, apply dependency inversion, extract interfaces)
    - Idiom Modernisation (update legacy patterns to modern language/framework idioms)

    CLOUD READINESS:
    - Cloud Blocker Resolution (hardcoded configs, platform-specific APIs, stateful sessions, local file I/O)
    - State Externalization (local file storage to S3, in-memory sessions to Redis/DynamoDB, local cache to ElastiCache)
    - Containerization (Dockerfile generation, deployment manifests, health checks, environment variable config)

    An application may require a COMBINATION of patterns across categories (e.g., Technology Upgrade + Security Remediation + Cloud Readiness). Identify ALL applicable sub-patterns.
12. Gather pattern-specific intelligence from CAST Imaging based on identified sub-patterns:

    TECHNOLOGY UPGRADE:
    - Language Version Upgrade: Use `packages` to identify all deps needing version bumps. Use `quality_insight_violations` for deprecated API usages. Use `object_details` focus="code" on objects using deprecated language features.
    - Framework Version Upgrade: Use `packages` to get current framework version and all ecosystem packages. Use `quality_insight_violations` for deprecated framework patterns. Use `advisors` for framework-specific upgrade rules. Use `object_details` focus="code" on objects using deprecated framework APIs.
    - Framework Migration: Use `objects` with type filters to get full inventory of source framework objects with file paths. Use `object_details` focus="code" on representative objects to understand current patterns. Use `object_profiles` to quantify framework-specific object types.

    SECURITY REMEDIATION:
    - CVE Remediation: Use `quality_insight_violations` with `include_locations=True` for every CVE. Use `package_interactions` to see which application objects use each vulnerable package.
    - Structural Flaw Fixes: Use `quality_insight_violations` with `include_locations=True` for every structural flaw (Security, Reliability). Use `application_iso_5055_explorer` for ISO-5055 weakness details.

    CODE REFACTORING:
    - Dead Code Removal: Use `advisors` for "Not Used" rules and `advisor_occurrences` to get specific unused objects. Use `object_details` focus="inward" to verify zero callers.
    - Complexity Reduction: Use `objects` sorted by complexity for hotspots. Use `object_details` focus="code" on top complex objects to understand patterns. Use `object_details` focus="inward"/"outward" to understand dependencies before decomposition.
    - Coupling Fixes: Use `architectural_graph` at component level with mode="links" to identify heavy coupling. Use `object_details` focus="inward"/"outward" on boundary objects. Use `transactions` to understand which flows cross component boundaries.
    - Idiom Modernisation: Use `object_details` focus="code" on representative objects to identify legacy patterns. Use `quality_insight_violations` for maintainability-related structural flaws.

    CLOUD READINESS:
    - Cloud Blocker Resolution: Use `quality_insight_violations` with `include_locations=True` for all cloud-detection-patterns. Use `advisors` rules and violations for cloud migration rules.
    - State Externalization: Use `data_graphs` and `application_database_explorer` to understand current data access patterns. Use `transactions_using_object` on stateful objects to understand session/cache usage scope.
    - Containerization: Use `architectural_graph` to understand deployment structure. Use `source_files` to find existing deployment configs (Dockerfiles, scripts, manifests).

    FOR ALL PATTERNS: Use `source_files` to verify file existence in local repository for every file path referenced.
13. Ask follow-up questions to clarify target state details and scope boundaries. Always ask for:
    - Build command (e.g., mvn clean verify, npm run build, gradle build)
    - Test command (e.g., mvn test, pytest, npm test)
    - Any directories or files explicitly out of scope

### Stage C: Plan Document Generation

14. Based on the modernisation pattern determined in step 11, load the appropriate plan template from skill resources:

    TECHNOLOGY UPGRADE:
    - Language Version Upgrade: `read_skill_resource("cast-imaging-to-aws-transform-phase1", "document_references/language-upgrade.md")`
    - Framework Version Upgrade: `read_skill_resource("cast-imaging-to-aws-transform-phase1", "document_references/framework-upgrade.md")`
    - Framework Migration: `read_skill_resource("cast-imaging-to-aws-transform-phase1", "document_references/framework-migration.md")`

    SECURITY REMEDIATION:
    - CVE Remediation: `read_skill_resource("cast-imaging-to-aws-transform-phase1", "document_references/security-remediation.md")`
    - Structural Flaw Fixes: `read_skill_resource("cast-imaging-to-aws-transform-phase1", "document_references/security-remediation.md")` (same template, focus on flaws section)

    CODE REFACTORING:
    - All Code Refactoring sub-patterns: `read_skill_resource("cast-imaging-to-aws-transform-phase1", "document_references/code-refactoring.md")`

    CLOUD READINESS:
    - All Cloud Readiness sub-patterns: `read_skill_resource("cast-imaging-to-aws-transform-phase1", "document_references/cloud-migration.md")`

    COMBINED (multiple sub-patterns across categories): Load each applicable template and merge into a phased plan using `read_skill_resource("cast-imaging-to-aws-transform-phase1", "document_references/combined-multi-pattern.md")` as the wrapper structure

15. Populate the template with CAST Imaging data. Every section MUST use real data — do NOT leave placeholders or generic descriptions. If data wasn't retrieved for a section, omit that section entirely.

    If `inter_applications_dependencies` revealed other applications depending on this application, include a DO NOT MODIFY section in the plan listing the objects/interfaces that external consumers rely on. Use `inter_app_detailed_dependencies` to identify the specific boundary objects whose signatures must be preserved.

16. Before finalizing, verify the plan's accuracy:
    - Cross-reference file paths against the local repository using source_files
    - Ensure CVE remediation strategies match actual dependency versions from packages
    - Confirm object counts match CAST Imaging data
    - Verify advisor rules have been addressed
    - Ensure no data is fabricated — only include what was retrieved from CAST Imaging

17. Present the completed plan to the user as a fully-structured Transformation Definition (TD) with the following sections:
    - Name and Description
    - Entry Criteria (including CAST Imaging MCP availability requirement, required runtimes/tools for the target state, and source code accessibility)
    - Implementation Steps (the actual transformation work, referencing specific files, objects, and data from the assessment)
    - The CAST Imaging MCP Usage During Execution guidance block (the generic block from the Phase 2 Execution Guidance section of this TD)
    - The Pattern-Specific MCP Verification section from the template (defines how to verify the pattern-specific transformation goal was achieved using MCP tools)
    - Validation / Exit Criteria, constructed from:
      - Build command passes: [exact command from step 13]
      - Test command passes: [exact command from step 13]
      - Baseline vs. target quality metrics (e.g., "CVE count reduced from 15 critical to 0", "cloud blocker count reduced from 47 to 0", "complexity hotspots above 50 reduced from 10 to 0")
      - Pattern-specific MCP verification checks all satisfied (reference the specific checks from the Pattern-Specific MCP Verification section)
      - No regressions: all existing transactions still function, no callers broken
    
    Then state:

    "Here is your Phase 2 transformation definition. To execute it:
    1. Review and edit the TD as needed (add/remove files, change versions, adjust scope)
    2. Save this as a new transformation definition (you can ask me to publish it, or use: atx custom def publish)
    3. Run the new TD against the same repository with CAST Imaging MCP configured
    
    CAST Imaging MCP must remain available during Phase 2 execution — the TD actively uses it for caller/callee verification, code retrieval, and impact analysis."

## Phase 2 Execution Guidance

The generated plan document MUST include the following section at the end to instruct the Phase 2 agent on how to use CAST Imaging MCP during execution:

```
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
   - Call `quality_insight_violations` to cross-reference that the violation's location was properly addressed (verify the fix matches the exact file/line reported)

4. When encountering ambiguity:
   - Call `objects` with filters to find related objects not listed in the plan
   - Call `source_file_details` nature="inventory" to see what else is in a file before modifying it
   - Call `object_details` focus="code" on sibling objects to understand surrounding patterns

Application name for all CAST Imaging MCP calls: [APPLICATION_NAME]
```

## Validation / Exit Criteria

1. CAST Imaging assessment was completed with ALL relevant data types gathered
2. `quality_insight_violations` with `include_locations=True` was used to get exact file paths and line numbers
3. `object_details` with focus="code" was used on representative objects to understand implementation patterns
4. Probing questions were data-driven, referencing specific counts and findings from CAST Imaging
5. User confirmed the target state, scope, and modernisation pattern
6. The generated plan document:
   - Uses the appropriate template for the selected modernisation pattern
   - Contains specific file paths verified against the repository
   - Contains exact object counts matching CAST Imaging data
   - Lists every relevant CVE with severity and remediation strategy
   - Lists every dependency with current version and action
   - Has clear scope boundaries
   - Is detailed enough to define scope and goals, but instructs the execution agent to use CAST Imaging MCP for verification and accuracy during transformation
7. NO code changes were made — this TD is assessment and planning only
