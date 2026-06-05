# Phase 2 Transformation Definition — Reference Structure

> **This file documents the structure of the TD that Phase 1 generates automatically.**
> You do NOT need to manually create this — Phase 1 produces it. This reference is for understanding what the output looks like and what to review before execution.

---

## Generated TD Structure

Phase 1 produces a fully-structured transformation definition with these sections:

### 1. Name and Description
```
# [pattern-name]-execution

## Description
[One-line summary — e.g., "Migrate Struts 1.3 to Spring Boot 3.3 with security remediation for WebGoat application"]
```

### 2. Entry Criteria
```
## Entry Criteria

1. CAST Imaging MCP server is configured and responding (application: [APPLICATION_NAME])
2. Target repository source code is accessible at: [repo path]
3. [Language/runtime] version [X] is installed locally
4. [Pattern-specific prerequisites]
```

### 3. Implementation Steps
The actual transformation work — populated with specific files, objects, counts, and data from the CAST Imaging assessment. This is the core of the TD.

### 4. CAST Imaging MCP Usage During Execution
A generic guidance block instructing the execution agent on how to use CAST Imaging MCP tools before, during, and after each transformation step.

### 5. Pattern-Specific MCP Verification
Verification checkpoints specific to the chosen pattern (e.g., "After namespace renames, call `objects` filtered by old namespace to confirm zero remnants").

### 6. Validation / Exit Criteria
```
## Validation / Exit Criteria

1. Build command passes: [exact command]
2. Test command passes: [exact command]
3. Baseline vs target quality metrics (e.g., "CVE count: 15 critical → 0")
4. Pattern-specific MCP verification checks all satisfied
5. No regressions: all existing transactions still function
```

---

## Execution Commands

### Local (with CAST Imaging MCP — recommended):
```bash
atx custom def exec \
  -n "[td-name]" \
  -p "[/path/to/repo]" \
  -x -t \
  --mcp-config ./mcp.json
```

### Local (without MCP — fallback):
```bash
atx custom def exec \
  -n "[td-name]" \
  -p "[/path/to/repo]" \
  -x -t
```

### MCP Config File (`mcp.json`):
```json
{
  "mcpServers": {
    "cast-imaging": {
      "command": "uvx",
      "args": ["cast-imaging-mcp-server@latest"],
      "env": {
        "CAST_IMAGING_URL": "<your-instance-url>",
        "CAST_IMAGING_API_KEY": "<your-api-key>"
      }
    }
  }
}
```

---

## Key Differences from a Self-Contained Plan

| Aspect | Old Approach (Self-Contained) | New Approach (MCP-Augmented) |
|--------|-------------------------------|------------------------------|
| CAST Imaging in Phase 2 | Not required | **Required** |
| Verification | Build + test only | Build + test + MCP verification |
| Caller analysis | Pre-computed in plan | Live lookup during execution |
| Impact assessment | Static file list | Dynamic via `transactions_using_object` |
| Code retrieval | Embedded snippets | On-demand via `object_details` focus="code" |
| Accuracy | Plan may drift from reality | Always current from MCP |

---

## Notes

- CAST Imaging MCP must remain accessible throughout Phase 2 execution
- Run `--dry-run` first to preview changes before committing
- For combined/multi-pattern TDs, consider running each phase as a separate execution
- Commit the generated TD to version control alongside the code for traceability
