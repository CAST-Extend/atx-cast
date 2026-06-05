---
inclusion: auto
---

# CAST Imaging → AWS Transform Custom: Getting Started

## What This Power Does

This power performs **Phase 1** of application modernisation: assessment and TD generation using CAST Imaging structural analysis. It produces a fully-structured Phase 2 transformation definition that is executed via AWS Transform custom with CAST Imaging MCP available for verification.

**Phase 1 (this power)**: Assessment → Probing → Phase 2 TD Generation  
**Phase 2 (generated TD)**: Execute the transformation via AWS Transform custom (with CAST Imaging MCP for verification)

## Workflow

### Step 0: Verify CAST Imaging MCP Configuration

Before doing anything else, check whether the CAST Imaging MCP server is configured and accessible:

1. Attempt to call `applications` to test the connection
2. **If the call fails or the MCP server is not configured**, prompt the user for their CAST Imaging credentials:

   Ask the user:
   > "I need your CAST Imaging connection details to proceed. Please provide:
   > 1. **CAST Imaging URL** — the base URL of your CAST Imaging instance (e.g., `https://imaging.example.com`)
   > 2. **CAST Imaging API Key** — your API key for authenticating with the CAST Imaging REST API
   >
   > These credentials are used to connect to the CAST Imaging MCP server. They are only stored in your local MCP configuration and are never sent anywhere else."

3. Once the user provides their credentials, instruct them to add the CAST Imaging MCP server to their Kiro MCP configuration (`.kiro/settings/mcp.json` or `~/.kiro/settings/mcp.json`):

   ```json
   {
     "mcpServers": {
       "cast-imaging": {
         "command": "uvx",
         "args": ["cast-imaging-mcp-server@latest"],
         "env": {
           "CAST_IMAGING_URL": "<user-provided-url>",
           "CAST_IMAGING_API_KEY": "<user-provided-api-key>"
         }
       }
     }
   }
   ```

4. After configuration, retry the `applications` call to confirm the connection is working
5. **If the call succeeds on the first attempt**, skip this step and proceed directly to Step 1

### Step 1: Verify Connection

Confirm the CAST Imaging MCP server is responding:
- Call `applications` to list available applications
- If it fails, return to Step 0 to reconfigure credentials

### Step 2: Application Selection

- Present the list of available applications to the user
- Let them select one (or ask if they want a portfolio overview first)
- Confirm the selection before proceeding

### Step 3: Deep Assessment

Gather ALL of the following for the selected application (make parallel calls where possible):

| Tool | What It Provides |
|------|-----------------|
| `stats` | Size, complexity, technologies, element types |
| `architectural_graph` (nodes + links, all levels) | Architecture structure and coupling |
| `quality_insights` (all 5 natures) | CVEs, structural flaws, ISO-5055, cloud blockers, green patterns |
| `packages` | External dependencies with versions |
| `transactions` + `transaction_profiles` | API/UI endpoints and business flows |
| `data_graphs` + `data_graph_profiles` | Data entity interactions |
| `inter_applications_dependencies` | Cross-application coupling |
| `application_database_explorer` | Database tables |
| `application_iso_5055_explorer` | ISO-5055 characteristics and weaknesses |
| `object_profiles` | Object type distribution |
| `advisors` (list + rules for each) | Migration/modernisation rules |

Then drill deeper using a tiered approach:
- `transaction_details` (type_graph + complexity) for top 5 largest transactions
- `quality_insight_violations` (without locations first for summaries, then with `include_locations=True` per rule, paginated, in severity order)
- `object_details` (focus="code") for up to 5 representative objects per violation category
- `source_files` + `source_file_details` nature="inventory" to verify files and objects match

### Step 4: Present Assessment Summary

Synthesise findings into a structured summary with:
- Application size and technology stack
- Architecture overview (layers, components, coupling)
- Quality posture (CVE count by severity, flaw count by type, cloud blocker count)
- Key dependencies and their status
- Transaction/data flow complexity
- Advisor findings (migration rules triggered)

### Step 5: Data-Driven Probing

**CRITICAL**: Never present a generic menu of options. Ask targeted questions based on SPECIFIC findings:

- "I found 12 critical CVEs, 8 of which are in Apache Struts 1.3.10. Do you want to upgrade Struts or migrate to Spring Boot entirely?"
- "The app has 47 cloud blockers: 15 hardcoded file paths, 12 stateful sessions, 20 hardcoded URLs. Are you targeting AWS deployment? Which compute service?"
- "Java 8 is EOL. The app uses 23 deprecated APIs that were removed in Java 17. What's your target JDK version?"

### Step 6: Determine Pattern & Gather Focused Data

Based on user answers, identify the modernisation pattern(s) and gather pattern-specific intelligence. See `modernisation-paths.md` for detailed guidance per pattern.

### Step 7: Generate Phase 2 TD

Load the appropriate template from `document_references/` and populate it with real CAST Imaging data. See `modernisation-paths.md` for template selection logic.

**Rules for TD generation:**
- Every data point must come from CAST Imaging — no fabrication
- Every file path must be verified against the repository
- Every count must match what was retrieved
- If data wasn't gathered for a section, omit it entirely
- Include the CAST Imaging MCP Usage During Execution block
- Include pattern-specific MCP verification checkpoints
- If `inter_applications_dependencies` found cross-app consumers, include a DO NOT MODIFY section

### Step 8: Handoff to Phase 2

Present the generated TD and explain:
1. Review and edit the TD as needed
2. Save as a new transformation definition (or ask to publish via `atx custom def publish`)
3. Run the TD against the same repository with CAST Imaging MCP configured

**CAST Imaging MCP must remain available during Phase 2 execution.**

## Key Principles

1. **Data drives decisions** — Every recommendation is backed by specific CAST Imaging findings
2. **Probing, not menus** — Ask targeted questions referencing actual data
3. **Specificity over generality** — "14 files using javax.servlet" not "deprecated APIs found"
4. **File paths are mandatory** — The transformation engine needs exact locations
5. **No code changes in Phase 1** — Phase 1 is purely analytical
6. **MCP in both phases** — Phase 2 uses CAST Imaging MCP for verification and impact analysis
7. **Human oversight** — User reviews and edits the generated TD before Phase 2 execution
8. **All violations accounted for** — None may be skipped; group repeated patterns with counts
