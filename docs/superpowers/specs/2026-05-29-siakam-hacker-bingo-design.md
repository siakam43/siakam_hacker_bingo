# siakam-hacker-bingo Skill Design

## Overview

`siakam-hacker-bingo` is a markdown-only Claude Code skill that analyzes C source files
(.c, .h) in a target project for security vulnerabilities. It uses sub-agents for
parallel analysis with checkpoint-based resume.

## Scope

- Target: Linux kernel drivers, mobile boot chain (UEFI, BL31, BL2, XLoader), mobile
  co-processor firmware (ISP, SensorHub, GPU)
- Languages: C only
- Excluded concerns: database, web, privacy leaks, memory leaks, DDoS, performance,
  stability

## File Structure

```
siakam-hacker-bingo/
├── SKILL.md                      # Main orchestrator
├── sub-agent-prompt.md           # Prompt template for sub-agents
├── task-list-template.md         # Task-list checkpoint format
└── finding-output-template.md    # Finding entry format
```

## Architecture: Linear Pipeline (Approach A)

### Phase 0 — Input Handling
- Extract `project_dir` from slash-command args, default to `.`
- Load `.siakamignore` if present (gitignore syntax)
- Set `OUT_DIR = project_dir/.siakam_out/BINGO`

### Phase 1 — Scan & Task Creation
- Enumerate all `.c` and `.h` files via `find`
- Exclude files matching `.siakamignore` patterns
- Group files by immediate parent directory; split tasks with >10 files
- Write `task-list.md` with per-file status tracking
- Write `source-file-index.md` as scan snapshot

### Phase 2 — Dispatch Loop
- CONCURRENCY parameter (default 2) at top of SKILL.md
- Pick up to CONCURRENCY pending tasks, mark `in_progress`
- Launch sub-agents via Claude Code `Agent` tool with `sub-agent-prompt.md`
- Each sub-agent writes to `findings/task-<ID>-findings.md`
- Mark `done` only after confirming result file exists

### Phase 3 — Aggregation
- Concatenate all `findings/task-*-findings.md` into `FINAL_REPORT.md`
- Include summary statistics (files analyzed, findings by severity/type)

### Resume
- On startup, check for existing `task-list.md`
- Skip `done` tasks; re-run `in_progress` tasks (assumed interrupted)
- Continue from first `pending` task

## Task List Format

Checkpoint file at `.siakam_out/BINGO/task-list.md`:

| ID | Directory | Files | Count | Status | Result |
|----|-----------|-------|-------|--------|--------|

- File lists stored in `file-list/task-<ID>-files.md` to avoid table bloat
- Status: `pending` → `in_progress` → `done`

## Sub-Agent Design

Each sub-agent receives:
1. Role context (C security auditor for low-level embedded code)
2. Scope boundaries (excluded categories)
3. 5 focus categories with concrete patterns
4. Per-file analysis rules (read fully, trace data flow, record immediately)
5. Output path: `findings/task-<ID>-findings.md`
6. Quality rule: "Better to miss a theoretical issue than flood with false positives"
7. Anti-skimming guard: "Read every function. Do not skip code."

Sub-agents are read-only (Read tool only, no Bash, no Edit).

## Finding Output Format

Each finding:

```markdown
### [<SEVERITY>] <SHORT_TITLE>
- **File**: `path/to/file.c`
- **Lines**: `123-145`
- **Category**: Input Validation | Memory Safety | System Security | Cryptography | Race Condition
- **Type**: e.g., Buffer Overflow (Stack), Use-After-Free
- **Description**: <One paragraph>
```

Severities: Critical, High, Medium, Low. Zero-finding tasks still produce a result file.

## Security Categories

1. **Input Validation**: Missing validation on external inputs (params, IPC, shared memory,
   files), trust boundary violations, missing bounds checks, type confusion, deserialization
2. **Memory Safety**: Buffer overflows, OOB read/write, UAF, double free, NULL deref,
   uninitialized memory, integer overflow, format strings
3. **System Security**: Auth bypass, privilege escalation, RCE, dynamic code execution
4. **Cryptography**: Hardcoded keys/passwords, weak algorithms, improper key storage,
   randomness issues, cert validation bypass
5. **Race Conditions**: Multi-threaded memory safety issues under concurrency
