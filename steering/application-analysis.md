---
inclusion: manual
---

# Application Analysis Guide

## Purpose

This guide provides detailed instructions for performing the deep assessment of a selected application using CAST Imaging MCP tools. Follow this when executing Step 3 of the main workflow.

## Assessment Data Collection

### Architecture (Structural Understanding)

```
architectural_graph(application, mode="nodes", level="layer")
architectural_graph(application, mode="links", level="layer")
architectural_graph(application, mode="nodes", level="component")
architectural_graph(application, mode="links", level="component")
architectural_graph(application, mode="nodes", level="sub-component")
architectural_graph(application, mode="links", level="sub-component")
architectural_graph(application, mode="nodes", level="technology-category")
```

**What to look for:**
- Number of layers and their sizes (User Interaction, Business Logic, Data Access, etc.)
- Component coupling — which components have the most links between them
- Technology spread — how many different technologies are in play
- Monolithic indicators — single large component with everything coupled

### Quality Insights (Risk Assessment)

```
quality_insights(application, nature="cve")
quality_insights(application, nature="structural-flaws")
quality_insights(application, nature="iso-5055")
quality_insights(application, nature="cloud-detection-patterns")
quality_insights(application, nature="green-detection-patterns")
```

**What to look for:**
- Critical/High CVEs — these drive urgency
- Structural flaw distribution — Security vs Reliability vs Efficiency vs Maintainability
- Cloud blocker count and categories — determines cloud readiness effort
- ISO-5055 violations — compliance requirements

### Violation Locations (Precision)

For each significant quality insight:
```
quality_insight_violations(application, nature="...", id="...", include_locations=True)
```

**Why this matters:** File paths and line numbers are what make the plan actionable. Without locations, the plan is just a list of problems without solutions.

### Dependencies (Upgrade Path)

```
packages(application)
```

**What to look for:**
- Framework packages and their versions (how old?)
- Packages with known CVEs (SaferClosestVersion, SafestVersion fields)
- Build tool dependencies
- Test framework versions

### Transactions (Business Flows)

```
transactions(application)
transaction_profiles(application)
```

For the top 5 largest:
```
transaction_details(application, id="...", focus="type_graph")
transaction_details(application, id="...", focus="complexity")
```

**What to look for:**
- Transaction count and size distribution
- Which components are involved in critical transactions
- Complexity hotspots within transactions

### Data Flows (Data Ownership)

```
data_graphs(application)
data_graph_profiles(application)
application_database_explorer(application)
```

**What to look for:**
- Table count and groupings
- Which transactions access which tables (data ownership boundaries)
- Shared tables (coupling indicators for microservice decomposition)

### Advisors (Migration Rules)

```
advisors(application, focus="list")
advisors(application, focus="rules", advisor_id="...")  # for each advisor
```

**What to look for:**
- Which advisors are available (Move to AWS, Database Migration, .NET Modernization, etc.)
- Rule counts and violation counts per advisor
- Specific rules that match the user's likely modernisation path

### Object Profiles (Type Distribution)

```
object_profiles(application)
```

**What to look for:**
- Framework-specific object types and their counts
- Ratio of business logic objects to infrastructure objects
- Dead code indicators (types with zero references)

### Code Inspection (Pattern Understanding)

For representative objects identified in violations:
```
object_details(application, focus="code", filters="id:eq:...")
```

**What to look for:**
- Current coding patterns (how the framework is used)
- Deprecated API usage patterns
- Complexity patterns (nested loops, large switch statements)

### File Verification

```
source_files(application, file_path="...")
source_file_details(application, file_path="...", nature="inventory")
```

**What to look for:**
- Confirm files mentioned in violations actually exist in the repository
- Understand file-to-object mapping
- Identify build/config files that need modification

## Assessment Summary Template

After gathering all data, synthesise into this structure:

```
## Application Assessment: [Name]

### Overview
- Language: [X], Framework: [Y v.Z], Size: [N] LOC, [M] objects
- Build: [tool], Database: [engine], Deployment: [current state]

### Architecture
- [N] layers, [M] components, [P] sub-components
- Key coupling: [Component A] ↔ [Component B] ([N] links)
- Technology spread: [list technologies with object counts]

### Quality Posture
- CVEs: [Critical: N, High: N, Medium: N, Low: N]
- Structural Flaws: [Security: N, Reliability: N, Efficiency: N, Maintainability: N]
- Cloud Blockers: [N total — Category1: N, Category2: N, ...]
- ISO-5055: [characteristic violations if applicable]

### Key Dependencies
- [Framework] v[X] — [status: EOL/deprecated/current]
- [Library] v[X] — [CVE count if any]
- [N] total packages, [M] with known vulnerabilities

### Business Flows
- [N] transactions, largest: [name] ([M] objects)
- [N] data graphs, [M] database tables

### Advisor Findings
- [Advisor name]: [N] rules, [M] violations
- Key rules: [list most relevant]

### Recommended Modernisation Path(s)
[Based on findings — what patterns are indicated and why]
```
