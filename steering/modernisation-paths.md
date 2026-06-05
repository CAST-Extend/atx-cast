---
inclusion: manual
---

# Modernisation Paths Guide

## Purpose

This guide provides pattern-specific instructions for gathering focused intelligence and generating the transformation plan document. Use this after the assessment is complete and the modernisation pattern has been determined.

## Pattern Selection Logic

Based on assessment findings, determine the pattern category and sub-patterns:

| Finding | Indicated Category | Sub-Pattern |
|---------|-------------------|-------------|
| Language EOL / deprecated APIs removed in newer version | Technology Upgrade | Language Version Upgrade |
| Framework version outdated, same framework has newer major version | Technology Upgrade | Framework Version Upgrade |
| Framework EOL / fundamentally wrong framework for needs | Technology Upgrade | Framework Migration |
| Critical CVEs in dependencies | Security Remediation | CVE Remediation |
| Structural security/reliability flaws (CWE violations) | Security Remediation | Structural Flaw Fixes |
| High complexity hotspots | Code Refactoring | Complexity Reduction |
| Dead code detected by advisor rules | Code Refactoring | Dead Code Removal |
| Tight coupling between components | Code Refactoring | Coupling Fixes |
| Legacy coding patterns | Code Refactoring | Idiom Modernisation |
| Cloud blockers, platform-specific code | Cloud Readiness | Cloud Blocker Resolution |
| Local file/session/cache state | Cloud Readiness | State Externalization |
| Targeting container deployment | Cloud Readiness | Containerization |
| Multiple of the above | Combined | Multi-Pattern (use combined template) |

## Pattern-Specific Data Collection

### Language Version Upgrade

**Additional data to gather:**
```
packages(application)  # all deps — check version compatibility with target language
quality_insight_violations(application, nature="cloud-detection-patterns", id="deprecated-version-pattern-id", include_locations=True)
advisors(application, focus="rules", advisor_id="...")  # language-specific rules
object_details(application, focus="code", filters="...")  # deprecated API usage samples
source_files(application, file_path="pom.xml")  # or build.gradle, package.json, pyproject.toml
```

**Template:** `document_references/language-upgrade.md`

---

### Framework Upgrade

**Additional data to gather:**
```
packages(application)  # framework package + ecosystem versions
objects(application, filters="type:contains:[framework-type]")  # all framework objects
quality_insight_violations(application, nature="cloud-detection-patterns", id="...", include_locations=True)  # deprecated patterns
object_details(application, focus="code", filters="...")  # current framework usage patterns
source_files(application, file_path="[config-files]")  # framework config files
```

**Template:** `document_references/framework-upgrade.md`

---

### Framework Migration

**Additional data to gather:**
```
objects(application, filters="type:contains:[source-framework-type]")  # full inventory of source framework objects
object_profiles(application)  # counts per framework-specific type
object_details(application, focus="code", filters="...")  # 3-5 representative objects for pattern understanding
source_files(application, file_path="[framework-config]")  # framework config files
quality_insight_violations(application, nature="cve", id="...", include_locations=True)  # CVEs fixed by migration
transactions(application)  # to understand endpoint mapping
```

**Template:** `document_references/framework-migration.md`

---

### Code Refactoring

**Additional data to gather:**
```
objects(application, filters="type:contains:[main-type]")  # sorted by complexity (highest first)
quality_insight_violations(application, nature="structural-flaws", id="...", include_locations=True)  # for each flaw type
advisors(application, focus="rules", advisor_id="...")  # dead code rules
advisor_occurrences(application, id="...")  # specific unused objects
object_details(application, focus="code", filters="...")  # hotspot code
object_details(application, focus="inward", filters="...")  # verify zero callers for dead code
architectural_graph(application, mode="links", level="component")  # coupling analysis
```

**Template:** `document_references/code-refactoring.md`

---

### Security Remediation

**Additional data to gather:**
```
quality_insight_violations(application, nature="cve", id="...", include_locations=True)  # for EVERY CVE
quality_insight_violations(application, nature="structural-flaws", id="...", include_locations=True)  # security-factor flaws
packages(application)  # current versions + SaferClosestVersion/SafestVersion
package_interactions(application, component="...", version="...")  # which objects use vulnerable packages
object_details(application, focus="code", filters="...")  # vulnerable code patterns
```

**Template:** `document_references/security-remediation.md`

---

### Architecture Migration (Cloud)

**Additional data to gather:**
```
quality_insight_violations(application, nature="cloud-detection-patterns", id="...", include_locations=True)  # ALL cloud blockers with locations
advisors(application, focus="rules", advisor_id="...")  # "Move to AWS" advisor rules
advisors(application, focus="violations", advisor_id="...", rule_id="...")  # AWS-specific violations
object_details(application, focus="code", filters="...")  # platform-specific code patterns
transactions(application)  # exposed endpoints (for port/routing config)
application_database_explorer(application)  # external service dependencies
source_files(application, file_path="Dockerfile")  # existing container config
source_files(application, file_path="docker-compose")  # existing compose config
```

**Template:** `document_references/cloud-migration.md`

---

### Combined / Multi-Pattern

**Additional data to gather:** Combine the data collection from each applicable pattern above.

**Template:** `document_references/combined-multi-pattern.md` as the wrapper, with sections from each individual pattern template.

**Phasing rules:**
1. Framework migration/upgrade → first (new code is what gets secured/containerized)
2. Language upgrade → first or alongside framework upgrade
3. Security remediation → after framework changes (unless CVEs are in shared code)
4. Code refactoring → after framework/language changes
5. Cloud migration → last (operates on final code state)

## Plan Generation Rules

1. **Every data point must come from CAST Imaging** — no fabrication, no assumptions
2. **Every file path must be verified** — use `source_files` to confirm existence
3. **Every count must match** — cross-reference with original tool responses
4. **Omit sections without data** — don't include empty or placeholder sections
5. **Be specific** — "javax.servlet.http.HttpSession in 8 files" not "session management"
6. **Include remediation strategies** — not just problems, but how to fix them
7. **Include MCP verification guidance** — the generated TD must instruct the execution agent to use CAST Imaging MCP
8. **Scope boundaries must be explicit** — what's in scope AND what's excluded
9. **All violations accounted for** — none may be skipped; group repeated patterns with counts
10. **Always ask for build/test commands** — include in the TD's validation criteria
