# Sub-Agent Reading Scope Design

## Context

Each sub-agent receives a file list and must audit every file in that list for security vulnerabilities. The current prompt is silent on whether sub-agents may read files outside their list. This creates ambiguity: sub-agents may assume they can only read listed files (missing context from callers/callees), or may drift into auditing everything they read (scope creep).

## Design Decision

Add a clear two-tier scope distinction to the sub-agent prompt:

- **Analysis scope** — files listed in "Files to Analyze". Must be read completely, every function audited.
- **Browsing scope** — any file within `<project_dir>`. May be read for context understanding only. Vulnerabilities found in browsed files are NOT reported.

The two actions use distinct verbs throughout: **analyze** (assigned files, full audit) vs **browse** (context files, scan for understanding).

## Changes to `sub-agent-prompt.md`

### 1. Your Task paragraph

Clarify "assigned files" and forward-reference the new scope section.

```
Analyze the files listed below for security vulnerabilities. For each
**assigned file**, read it completely — do not skip any function or any
code path. If a file is long, work through it methodically section by
section.

You MAY browse other files in `<project_dir>` to understand context —
see "Reading Scope vs. Analysis Scope" below.
```

### 2. New section: Reading Scope vs. Analysis Scope

Inserted after "Files to Analyze" block.

```
## Reading Scope vs. Analysis Scope

You MAY browse any file within `<project_dir>` to understand how the
assigned files fit into the larger codebase — trace call chains, check
data structures, or read included headers.

Your vulnerability findings, however, MUST only cover files explicitly
listed in "Files to Analyze" above. If you notice a vulnerability in a
file you browsed for context, do NOT report it — the sub-agent
responsible for that file will catch it.

| Scope | Files | Method |
|-------|-------|--------|
| **Analysis** | Listed in "Files to Analyze" | Full audit — read every function, trace every path |
| **Browsing** | Any file in `<project_dir>` | Context lookup — scan, trace, understand logic |
```

### 3. Rule 1

Title changed from "Read Every Function" to "Analyze Every Assigned File". Body scoped to assigned files only.

```
### Rule 1: Analyze Every Assigned File
Go through every function in every **assigned** file (listed in "Files
to Analyze"). Do not skim. Do not skip. If an assigned file has 500
lines, read all 500 lines. This rule applies to analysis targets only
— browsing context files may be done at a lighter pace.
```

### 4. Anti-Rationalization table

Existing anti-scan entry scoped to assigned files. New entry added to prevent browsing from turning into auditing.

```
| "I'll just quickly scan this function" (in an assigned file) | You must ANALYZE it, not scan it. Assigned files require full attention. |
| "I should fully audit this context file I'm browsing" | Browsed files are for context only. Scan for what you need and move on. Do not audit them. |
```

### 5. Task Completion Checklist

`read completely` replaced with `analyzed completely`.

```
- [ ] Every file in the list has been analyzed completely
```

## Scope Boundary

| Dimension | Limit |
|-----------|-------|
| Reading | `<project_dir>`, respecting `.siakamignore` (excluded files never appear in any task list) |
| Analysis | Only files in the task's "Files to Analyze" list |
| Reporting | Only files in the task's "Files to Analyze" list |

`.siakamignore` is handled by the orchestrator at scan time — sub-agents never see excluded files, so no need to pass it.

## Compatibility

- No changes to SKILL.md (orchestrator) — file lists, dispatch, and aggregation are unchanged
- No changes to task-list-template.md or finding-output-template.md
- `<project_dir>` placeholder is already substituted by the orchestrator during dispatch
