# Template: Architecture Migration (Cloud Readiness)

Use this template when making an application cloud-ready for AWS by removing platform-specific code, externalizing state to AWS managed services, containerizing, and resolving cloud blockers.

## Plan Document Structure

```
TRANSFORMATION GOAL:
[Make application X cloud-ready by resolving N cloud blockers for deployment on AWS. Eliminate platform-specific dependencies and externalize state to AWS managed services.]

SOURCE APPLICATION:
- Language: [from stats]
- Framework: [from packages]
- Size: [LOC from stats, element count]
- Current deployment: [on-premise / app server (which one) / VM — from architectural_graph and stats]
- Cloud blocker summary: [total blocker count, broken down by category from cloud-detection-patterns]
  - [Category]: [count] objects
  - [Category]: [count] objects
  - ...

TARGET STATE:
- Deployment platform: [AWS ECS / EKS / Lambda / EC2 — as specified by user]
- Containerized: [Yes/No — if yes, specify base image]
- State management: [externalized to — ElastiCache (Redis) / DynamoDB / MemoryDB]
- File storage: [S3 — replacing local file system operations]
- Configuration: [environment variables / AWS Systems Manager Parameter Store / AWS Secrets Manager]
- Logging: [CloudWatch Logs with structured JSON format]
- Service discovery: [AWS Cloud Map / ECS Service Connect — if applicable]
- Load balancing: [ALB / NLB — if applicable]

CLOUD BLOCKERS TO RESOLVE:
[For EACH blocker category. From quality_insight_violations with include_locations=True for each cloud-detection-pattern.]

Category: [category name from CAST Imaging] — [total objects affected]

Pattern: [pattern name] ([contribution: Blocker/Booster], [criticality])
  Impact: [what this means for cloud deployment]
  Locations:
    - [file:line] — [what the code does currently]
    - [file:line] — [what the code does currently]
  Fix: [specific cloud-native replacement — be precise about what service/pattern to use]

Example entries:

Category: Persistent Files — 15 objects

Pattern: CloudReady - Perform File Manipulation (Blocker, Low)
  Impact: Local file operations will fail in ephemeral container environments
  Locations:
    - src/main/java/com/app/FileService.java:34 — writes uploaded files to /tmp/uploads
    - src/main/java/com/app/ReportGen.java:89 — creates PDF reports in local directory
  Fix: Replace with S3 client (software.amazon.awssdk:s3). Use presigned URLs for downloads.
Category: Security & User Authentication — 72 objects

Pattern: CloudReady - Use of an unsecured data string (Blocker, Critical)
  Impact: Hardcoded credentials/connection strings exposed in source code
  Locations:
    - src/main/java/com/app/DBConfig.java:12 — hardcoded JDBC URL with password
    - src/main/java/com/app/MailService.java:8 — hardcoded SMTP credentials
  Fix: Move to AWS Secrets Manager. Inject via environment variables at runtime.
[Continue for every category: Stateful Sessions, Hardcoded URLs, Environment Variables, System DLLs, In-memory Caching, etc.]

INFRASTRUCTURE ARTIFACTS TO GENERATE:
[Based on AWS target compute — list each artifact to create.]

- Dockerfile — [base image, multi-stage build if applicable, port, health check]
- docker-compose.yml — [for local development with dependent services]
- [ECS task definition + service / EKS deployment+service manifests / Lambda handler + SAM template]
- [CI/CD: buildspec.yml for CodePipeline / .github/workflows/deploy.yml]
- [IaC: AWS CDK app / CloudFormation template / Terraform — if requested]

DEPENDENCIES TO UPDATE:
Remove:
- [platform-specific deps — e.g., JBoss/WebLogic client libs, Windows-specific packages]

Add:
- [AWS SDK — e.g., software.amazon.awssdk:s3, software.amazon.awssdk:secretsmanager, software.amazon.awssdk:elasticache]
- [AWS managed service clients — e.g., spring-cloud-aws, aws-sdk for caching/messaging]
- [Health check / observability — e.g., spring-boot-starter-actuator, micrometer-registry-cloudwatch, aws-xray-sdk]

ADDITIONAL INSTRUCTIONS:
- Preserve all business logic — only change infrastructure and platform concerns
- All configuration must be externalizable via environment variables (12-factor app)
- Health check endpoint required at: [path — e.g., /actuator/health or /health]
- Structured logging format: [JSON / key-value — for CloudWatch/log aggregation]
- Graceful shutdown handling required for container orchestration
- Build command: [command]
- Container build: [docker build command]
- Container test: [docker run + health check validation]
- Scope exclusions: [directories/files NOT to touch]
- [Any platform-specific constraints — e.g., "must run on Graviton/ARM64", "max 512MB memory"]

PATTERN-SPECIFIC MCP VERIFICATION:
Use CAST Imaging MCP tools throughout the transformation — not just the ones listed below. Any available MCP tool may be called if it provides useful context. These are the key verification checkpoints for this pattern:

- After resolving each cloud blocker: Call `quality_insight_violations` for the specific cloud-detection-pattern ID to verify violation count decreased
- After all cloud blockers resolved: Call `quality_insights` nature="cloud-detection-patterns" to verify total Blocker count is zero
- After externalizing state (file storage, sessions, cache): Call `objects` filtered by file I/O or session types to confirm no remaining local state patterns
- After removing platform-specific dependencies: Call `packages` to verify platform-specific libraries are gone and AWS SDK replacements are in place
- After externalizing configuration: Call `source_files` to verify no hardcoded config files remain with sensitive data
- Call `transactions` to verify all API/UI endpoints still function after infrastructure changes — endpoint count should match assessment
- Call `transaction_details` focus="type_graph" on critical transactions to verify they still reach backend systems through new cloud-native paths
- Call `data_graphs_involving_object` on modified data access objects to verify database interactions survive the externalization changes
- Call `architectural_graph` to verify the application's component structure is intact after cloud refactoring
- Call `advisors` for cloud migration rules (e.g., "Move to Amazon Web Services") to verify all rules are now satisfied
- Call `inter_applications_dependencies` to verify cross-app interfaces still function through new deployment model
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

### CLOUD BLOCKERS
- Use `quality_insights` nature="cloud-detection-patterns" to get all patterns with counts
- Use `quality_insight_violations` with id=[pattern-id] and include_locations=True for each Blocker pattern
- Focus on Blockers first, then note Boosters (things already cloud-ready)
- Use `object_details` focus="code" on affected objects to understand what the code actually does

### INFRASTRUCTURE ARTIFACTS
- Base the Dockerfile on the technology stack from `stats`
- Use `transactions` to identify exposed ports and endpoints
- Use `application_database_explorer` to identify external service dependencies for docker-compose

### ADVISORS
- Use `advisors` — specifically "Move to Amazon Web Services" advisor if available
- Get rules and violations to identify AWS-specific migration patterns
