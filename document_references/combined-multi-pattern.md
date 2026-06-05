# Template: Combined / Multi-Pattern

Use this template when the transformation involves multiple patterns (e.g., Framework Migration + Security Remediation + Cloud Readiness). This template provides the wrapper structure — load and embed relevant sections from the individual pattern templates.

## When to Use

- User wants Technology Upgrade AND Security Remediation (e.g., upgrade framework AND fix CVEs)
- User wants Technology Upgrade AND Cloud Readiness (e.g., upgrade language AND containerize)
- User wants Security Remediation AND Cloud Readiness (e.g., fix flaws AND resolve cloud blockers)
- User wants all three categories combined
- Any combination of 2+ sub-patterns across different categories in a single transformation

## Plan Document Structure

```
TRANSFORMATION GOAL:
[Overall goal covering ALL patterns being applied in this transformation. Be specific about what changes and what the end result looks like.]

SOURCE APPLICATION:
[Single combined source description — merge relevant fields from all applicable templates. Do not duplicate information.]

- Language: [from stats]
- Framework: [from packages — current framework + version]
- Size: [LOC, elements, key object type counts]
- Build system: [from source_files]
- Database: [from application_database_explorer + packages]
- Current deployment: [if cloud migration is in scope]
- Current CVE count: [if security remediation is in scope]
- Cloud blocker count: [if cloud migration is in scope]
- Architecture: [current architecture description]

TARGET STATE:
[Single combined target description — the final desired state after ALL phases complete.]

- Language: [target version if upgrading]
- Framework: [target framework if migrating]
- Deployment: [target platform if cloud migrating]
- Security posture: [target CVE state if remediating]
- Architecture: [target architecture — e.g., containerized, serverless]
- [All other target state attributes]

---

PHASE 1: [Pattern name — e.g., "Framework Migration: Struts 1.x → Spring Boot 3.x"]

[Load and populate the relevant sections from the pattern-specific template. Include:]
- PATTERNS TO TRANSFORM (if framework migration)
- DEPRECATED APIs TO REPLACE (if language upgrade)
- CVE REMEDIATION PLAN (if security remediation)
- CLOUD BLOCKERS TO RESOLVE (if cloud readiness)

[Include FILES IN SCOPE for this phase]
[Include DEPENDENCIES changes for this phase]

---

PHASE 2: [Pattern name — e.g., "Security Remediation: Fix remaining CVEs and structural flaws"]

[Load and populate relevant sections from the pattern-specific template]

[Include FILES IN SCOPE for this phase — may overlap with Phase 1]
[Include DEPENDENCIES changes for this phase]

---

PHASE 3: [Pattern name — e.g., "Cloud Migration: Containerize for AWS ECS"]

[Load and populate relevant sections from the pattern-specific template]

[Include FILES IN SCOPE for this phase]
[Include INFRASTRUCTURE ARTIFACTS for this phase]
[Include DEPENDENCIES changes for this phase]

---

COMBINED DEPENDENCY CHANGES:
[Merge all dependency changes across phases into a single consolidated list. Resolve conflicts — if Phase 1 adds a dep and Phase 3 upgrades it, show the final target version.]

Remove:
- [dep] [version] — reason (removed in Phase N)

Upgrade:
- [dep] [old] → [new] — reason (Phase N)

Add:
- [dep] [version] — reason (Phase N)

EXECUTION ORDER:
[Specify the order phases MUST be executed in and why.]

1. Phase [N] first because: [reason — e.g., "framework migration must happen before cloud blockers can be fixed in the new code"]
2. Phase [N] second because: [reason]
3. Phase [N] last because: [reason — e.g., "containerization requires the final state of the code"]

ADDITIONAL INSTRUCTIONS:
[Combined constraints from all phases. Deduplicate and organize.]

Preservation:
- [What to preserve across all phases]

Validation:
- After Phase 1: [build/test command]
- After Phase 2: [build/test + security scan command]
- After Phase 3: [container build + health check validation]

Scope exclusions:
- [Combined list of out-of-scope items]

[Any cross-phase constraints — e.g., "Phase 2 security fixes must be applied to the NEW code from Phase 1, not the old code"]

PATTERN-SPECIFIC MCP VERIFICATION:
Use CAST Imaging MCP tools throughout the transformation — not just the ones listed below. Any available MCP tool may be called if it provides useful context. For combined transformations, verification must cover each phase:

- After each phase completes: Run the phase-specific verifications from the individual pattern template (Technology Upgrade, Security Remediation, Cloud Readiness, or Code Refactoring)
- Between phases: Call `architectural_graph` at component level to verify structural integrity before proceeding to next phase
- Call `transactions` after each phase to verify no API/UI endpoints were lost
- Call `quality_insights` (all natures) after the final phase to get a complete quality snapshot and compare against assessment baseline
- Call `packages` after the final phase to verify all dependencies are at their intended final versions (no intermediate-state deps left behind)
- Call `advisors` with focus="rules" to verify all applicable migration/modernisation rules are satisfied
- Call `inter_applications_dependencies` after the final phase to confirm cross-app interfaces survived the full transformation
- If phases modify the same files: Call `object_details` focus="code" before Phase N+1 touches a file that Phase N already modified — verify the Phase N changes are intact
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

## Guidance for Using This Template

### PHASING DECISIONS
- Order phases by dependency: if one phase's output is another's input, it goes first
- Framework migration typically goes first (new code is what gets secured/containerized)
- Security remediation can go before or after framework migration depending on scope:
  - If removing the framework eliminates the CVEs → framework migration first
  - If CVEs are in shared code that survives the migration → fix them in whichever phase modifies those files
- Cloud migration typically goes last (operates on final code state)
- Language upgrade typically goes first or alongside framework upgrade

### AVOIDING DUPLICATION
- A file should appear in FILES IN SCOPE for the phase that makes the MOST significant change to it
- If a file is modified in multiple phases, note it in the first phase and add "(also modified in Phase N)" 
- Dependencies should be listed in the phase that introduces the need for them

### CONFLICT RESOLUTION
- If Phase 1 adds dependency X version 2.0 and Phase 3 needs version 3.0, list only version 3.0 in COMBINED DEPENDENCY CHANGES with a note
- If Phase 1 creates a file and Phase 2 modifies it, the creation is in Phase 1's scope and the modification is in Phase 2's scope
