# CAST Imaging → AWS Transform Custom Power

A Kiro Power that bridges deep structural application assessment (CAST Imaging) with automated code transformation (AWS Transform Custom). It eliminates the manual handoff between knowing what's wrong and fixing it at scale.

## What It Does

This power operates in two phases:

1. **Phase 1 — Assessment & TD Generation**: Connects to CAST Imaging via MCP, performs a comprehensive structural assessment, asks data-driven probing questions, and generates a fully-structured Transformation Definition (TD) for Phase 2.

2. **Phase 2 — Execution**: The generated TD is executed via AWS Transform Custom CLI, with CAST Imaging MCP remaining connected for live caller verification, blast radius analysis, and violation resolution tracking.

No code changes happen in Phase 1 — it's purely analytical. Human review happens between phases.

## Supported Modernisation Patterns

| Category | Sub-Patterns |
|----------|-------------|
| **Technology Upgrade** | Language version upgrade, framework version upgrade, framework migration |
| **Security Remediation** | CVE remediation, structural flaw fixes (CWE violations) |
| **Code Refactoring** | Dead code removal, complexity reduction, coupling fixes, idiom modernisation |
| **Cloud Readiness** | Cloud blocker resolution, state externalisation, containerisation |
| **Combined** | Any combination of the above in phased execution |

## Prerequisites

- **Kiro IDE** — [kiro.dev](https://kiro.dev)
- **CAST Imaging instance** — With at least one application analysed
- **CAST Imaging API key** — For Imaging authentication
- **CAST Imaging MCP Server** — For MCP server authentication
- **AWS CLI** — Configured with `AWSTransformCustomFullAccess` IAM policy (for Phase 2)
- **ATX CLI** — AWS Transform Custom CLI installed (for Phase 2)

## Installation

1. Install this power in your Kiro workspace
2. On first use, the power will prompt you for your CAST Imaging URL and API key
3. Configure the MCP server (the power will guide you through this)

## Configuration

The power requires the CAST Imaging MCP server to be configured. Add this to your Kiro MCP configuration (`.kiro/settings/mcp.json` or `~/.kiro/settings/mcp.json`):

```json
{
  "mcpServers": {
    "cast-imaging": {
      "type": "http",
      "url": "<cast_imaging_mcp_url>",
      "headers": {
        "x-api-key": "${input:imaging-key}"
      }
    }
  }
}
```

The power will prompt you for these values if the MCP server is not already configured.

## Usage

### Phase 1: Assessment

1. Start a conversation with the power active
2. The power verifies the CAST Imaging MCP connection (prompts for credentials if needed)
3. Select an application from the CAST Imaging environment
4. The power performs a deep assessment and presents quantified findings
5. Answer data-driven probing questions to determine the modernisation path
6. The power generates a complete Phase 2 Transformation Definition

### Phase 2: Execution

1. Review and edit the generated TD as needed
2. Publish the TD: `atx custom def publish -n "my-transformation" --sd ./td/`
3. Execute with CAST Imaging MCP:

```bash
atx custom def exec \
  -n "my-transformation" \
  -p ./my-repo \
  -x -t \
  --mcp-config ./mcp.json
```

The `--mcp-config` flag gives the execution agent live access to CAST Imaging for verification during transformation.

## Project Structure

```
├── POWER.md                    # Power metadata and capabilities documentation
├── power.json                  # Power configuration (name, version, keywords)
├── README.md                   # This file
├── transformation_definition.md # Phase 1 TD (the assessment workflow)
├── phase2_td_template.md       # Reference structure for generated Phase 2 TDs
├── steering/
│   ├── getting-started.md      # Main workflow guide (auto-included)
│   ├── application-analysis.md # Deep assessment guidance
│   ├── modernisation-paths.md  # Pattern-specific data collection & TD generation
│   └── phase2-execution.md     # Phase 2 review and execution guide
├── document_references/
│   ├── language-upgrade.md     # Template: Language version upgrade
│   ├── framework-upgrade.md    # Template: Framework version upgrade
│   ├── framework-migration.md  # Template: Framework migration
│   ├── code-refactoring.md     # Template: Code refactoring (all sub-patterns)
│   ├── security-remediation.md # Template: Security remediation
│   ├── cloud-migration.md      # Template: Cloud readiness (all sub-patterns)
│   └── combined-multi-pattern.md # Template: Combined / multi-pattern wrapper
└── blog/
    ├── post.md                 # Blog post draft
    └── figures/                # Diagram source files and SVGs
```

## How It Works

### Phase 1 Data Collection

The power uses CAST Imaging MCP tools to gather:

- **Architecture** — Multi-level decomposition (layer, component, sub-component, technology)
- **Quality** — CVEs, structural flaws, ISO-5055 violations, cloud blockers
- **Dependencies** — External packages with versions and vulnerability status
- **Transactions** — API/UI endpoints and business flow complexity
- **Data Flows** — Entity interactions and database structure
- **Advisors** — Migration and modernisation rules with violation counts
- **Code** — Source snippets for representative objects

### Phase 2 MCP Verification

During execution, the transformation agent uses CAST Imaging MCP to:

- Check all callers before changing method signatures
- Quantify blast radius via transaction analysis on every modified object
- Verify violations are resolved after each fix
- Retrieve current implementation on-demand
- Prevent cross-layer coupling during refactoring

## Key Design Principles

1. **Data drives decisions** — Every recommendation backed by specific CAST Imaging findings
2. **Probing, not menus** — Targeted questions referencing actual data, never generic option lists
3. **Specificity** — "14 files using javax.servlet" not "deprecated APIs found"
4. **File paths are mandatory** — The transformation engine needs exact locations
5. **MCP in both phases** — Assessment and execution both use live structural intelligence
6. **Human oversight** — User reviews the generated TD before any code is touched
7. **All violations accounted for** — None skipped; repeated patterns grouped with counts

## License

This project is licensed under the MIT License.

## Contributing

Contributions are welcome. Please open an issue or submit a pull request on the [CAST-Extend/atx-cast](https://github.com/CAST-Extend/atx-cast) repository.
