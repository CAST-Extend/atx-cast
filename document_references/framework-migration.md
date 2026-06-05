# Template: Framework Migration

Use this template when migrating from one framework to another (e.g., Struts → Spring Boot, Angular → React, Express → Fastify, Django → FastAPI).

## Plan Document Structure

```
TRANSFORMATION GOAL:
[What is being migrated, from what framework/version to what framework/version, and primary motivation (CVE elimination, EOL framework, modernisation, etc.)]

SOURCE APPLICATION:
- Language: [from stats]
- Framework: [from packages + stats — exact name and version]
- Size: [LOC from stats, object counts by type from object_profiles — specifically list counts of framework-specific objects: Actions, Controllers, Forms, Views, Services, etc.]
- Build system: [from source_files — tool name and config file path]
- Backend integration: [from architectural_graph + object_details — what external systems does this app connect to]
- Database: [from application_database_explorer + packages — engine name, access pattern (raw SQL/ORM/stored procs)]
- Config files: [list framework config files from source_files — e.g., struts-config.xml, web.xml, applicationContext.xml]

TARGET STATE:
- Backend: [exact framework + version — e.g., Spring Boot 3.3.x]
- Frontend: [if applicable — framework + version + build tool — e.g., React 18 + TypeScript + Vite]
- Auth: [mechanism replacing current auth pattern — e.g., Spring Security with JWT]
- Database: [connection/ORM strategy — e.g., Spring Data JPA with HikariCP]
- Build: [build tools for each module — e.g., Maven (backend) + npm/Vite (frontend)]
- [Any other target-specific components — messaging, caching, API gateway, etc.]

PATTERNS TO TRANSFORM:
[For EACH source framework concept, list the target equivalent and every affected object with file path. Get these from CAST Imaging objects tool filtered by type + source_files for paths.]

- [Source framework class type] → [Target framework equivalent]: [count] objects in [directory path]
  Files: [comma-separated list of specific file names]

Example entries:
- Struts Action classes → Spring @RestController: 15 objects in J2EE/WEB-INF/src/com/myapp/actions/
  Files: LogonAction.java, UserAction.java, OrderAction.java, ...
- Struts Form Beans → Request/Response DTOs: 31 objects
  Files: logonform, userform, orderform, ...
- JSP pages → React components: 24 pages in J2EE/pages/
  Files: login.jsp, dashboard.jsp, userdetail.jsp, ...
- Struts Action Mappings → @RequestMapping annotations: 33 mappings in struts-config.xml
  Mappings: /logon, /user, /order, ...
- [Continue for every framework concept: Forwards→Routes, Taglibs→Components, Tiles→Layouts, Interceptors→Filters, etc.]

FILES IN SCOPE:
[Every file that will be modified, created, or deleted. Compiled from source_files, quality_insight_violations locations, and object file paths.]

- [filepath] — [action: Convert to X / Delete after migration / Modify for Y / Create new] — [brief reason]

Example entries:
- src/main/webapp/WEB-INF/struts-config.xml — DELETE — replaced by Spring annotations
- src/main/java/com/myapp/actions/LogonAction.java — CONVERT to AuthController.java
- src/main/webapp/pages/login.jsp — CONVERT to React Login component
- pom.xml — MODIFY — remove Struts deps, add Spring Boot starters
- CREATE: src/main/java/com/myapp/config/SecurityConfig.java — Spring Security configuration
- CREATE: frontend/src/App.tsx — React app root

QUALITY ISSUES TO FIX DURING TRANSFORMATION:
[Every CVE and structural flaw that is resolved by this migration or needs explicit fixing. From quality_insight_violations with include_locations=True.]

- [CVE-ID] ([severity]): [affected library or code pattern] — [fix strategy: remove dep / upgrade to version X / rewrite code at path:line]
- [Structural flaw name] ([CWE-ID]): [count] occurrences — [fix strategy with specific locations]

Example entries:
- CVE-2016-1000031 (CRITICAL): Apache Commons FileUpload RCE — REMOVE by eliminating Struts dependency
- CVE-2012-0391 (CRITICAL): Struts OGNL injection — REMOVE by eliminating Struts entirely
- Reflected XSS (CWE-79): 5 occurrences at src/lessons/XSS/*.java — Fixed by React auto-escaping + Spring output encoding

DEPENDENCIES TO UPDATE:
[From packages tool — current deps with actions, then new deps to add.]

Remove:
- [groupId:artifactId] [current version] — reason: [replaced by X / transitive of removed framework / EOL]

Keep/Upgrade:
- [groupId:artifactId] [current version] → [target version] — reason

Add:
- [groupId:artifactId] [version] — purpose: [what it provides]

ADDITIONAL INSTRUCTIONS:
[Constraints, preservation requirements, execution guidance. Include ALL of the following that apply:]

- What to preserve and NOT modify (backend integrations, specific directories, external system connectors)
- Build/test commands to validate after transformation (e.g., mvn clean package, npm run build, npm test)
- Project structure conventions (package naming, module layout)
- API documentation requirements (OpenAPI/Swagger generation)
- Error handling patterns (@ControllerAdvice, Error Boundaries)
- Authentication/authorization implementation details
- Test generation requirements (framework, minimum coverage, test types)
- CORS/security configuration requirements
- Health check / actuator endpoints
- Logging framework/pattern
- Scope exclusions (directories/files NOT to touch)

PATTERN-SPECIFIC MCP VERIFICATION:
Use CAST Imaging MCP tools throughout the transformation — not just the ones listed below. Any available MCP tool may be called if it provides useful context. These are the key verification checkpoints for this pattern:

- After converting each source framework object: Call `objects` filtered by old framework types (e.g., type:contains:Struts) to track remaining count — it should decrease to zero by end
- After completing migration: Call `object_profiles` to verify old framework object types are gone from the application profile
- Call `source_files` to confirm old framework config files (struts-config.xml, web.xml with framework-specific entries, etc.) are removed or cleaned
- Call `packages` to verify old framework dependencies are fully removed and no transitive references remain
- Call `transactions` to verify all API/UI endpoints are preserved — compare against the original endpoint count from assessment. No endpoints should be lost.
- Call `transaction_details` focus="type_graph" on critical transactions to verify their call graphs still reach the same backend systems
- Call `data_graphs_involving_object` on converted data access objects to ensure database interactions are preserved
- Call `architectural_graph` to verify the new framework's component structure is clean and properly layered
- Call `inter_applications_dependencies` to verify cross-app interfaces weren't broken during migration
- Call `quality_insight_violations` for any CVEs that should have been eliminated by removing the old framework — confirm they're gone
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

### PATTERNS TO TRANSFORM
- Use `objects` tool filtered by framework-specific types (e.g., type:contains:Struts, type:contains:Action, type:contains:Form Bean)
- Use `object_profiles` to get exact counts per type
- Use `source_files` to get file paths
- Use `object_details` focus="code" on 3-5 representative objects to understand the current pattern

### FILES IN SCOPE
- Start with files containing framework-specific objects (from objects tool)
- Add files flagged by quality_insight_violations
- Add framework config files (from source_files)
- Add build files that need dependency changes
- List CREATE entries for new files that the target framework requires

### QUALITY ISSUES
- Only include issues that are IN SCOPE for this transformation
- If removing a framework eliminates a CVE, state that explicitly
- For structural flaws, only include those in files being modified

### DEPENDENCIES
- Get current deps from `packages` tool
- Determine which are transitive of the framework being removed
- Research compatible versions for the target framework
