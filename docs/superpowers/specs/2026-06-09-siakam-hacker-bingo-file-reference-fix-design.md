# siakam-hacker-bingo: File Reference Path Fix for Low-Capability Models

## Problem Statement

When running siakam-hacker-bingo on lower-capability models (e.g., Qwen3.5-35B-A3B), the orchestrator and sub-agents fail to find helper markdown files. SKILL.md references `sub-agent-prompt.md`, `task-list-template.md`, and `finding-output-template.md` with bare filenames. High-capability models infer these files live in the skill directory (sibling to SKILL.md); low-capability models search the project being audited (`project_dir`) and fail.

## Root Cause

Bare filenames in references with no indication of where the files actually live. The model defaults to searching `project_dir` (the user's codebase under audit), not the skill installation directory.

## Design

### Change 3a: Add Helper Files section to SKILL.md

After the Configuration block and before Usage, add a section that explicitly names all helper files and states their location — the skill directory, **not** the target project directory.

```markdown
## Helper Files

The following files are in the **same directory as this SKILL.md**:

- `sub-agent-prompt.md` — the full sub-agent analysis prompt
- `task-list-template.md` — format for the task list checkpoint file
- `finding-output-template.md` — format for per-task findings output

When you need to read one of these files, look in the skill directory — **not** in the target project directory.
```

### Change 3b: Annotate references in SKILL.md

**Step 2.6 reference to task-list-template.md:**

From:

```
Follow the format in `task-list-template.md`.
```

To:

```
Follow the format in `task-list-template.md` (in the skill directory).
```

**Step 3.3 reference to sub-agent-prompt.md:**

From:

```
`prompt`: The exact content from `sub-agent-prompt.md`, with the file list inserted
```

To:

```
`prompt`: The exact content from `sub-agent-prompt.md` (in the skill directory), with the file list inserted
```

### Change 3c: Annotate reference in sub-agent-prompt.md

**Output section reference to finding-output-template.md:**

From:

```
Follow the format in `finding-output-template.md` exactly.
```

To:

```
Follow the format in `finding-output-template.md` exactly. This file is in the skill directory.
```

The orchestrator, when constructing the sub-agent prompt, should replace the bare filename with the actual path (e.g., `siakam-hacker-bingo/finding-output-template.md`) since it knows the skill directory location.

### Change 3d: Orchestrator path substitution (SKILL.md Step 3.3)

In Step 3.3 of the dispatch loop, add a note that the orchestrator should resolve the `finding-output-template.md` reference to a concrete path before sending the sub-agent prompt:

After the "prompt" line in Step 3.3, add:

```
Before sending the prompt, replace the reference to `finding-output-template.md` with the actual path to the file in the skill directory (e.g., `siakam-hacker-bingo/finding-output-template.md`).
```

## Impact

- **High-capability models:** No functional change. These models already resolve the references correctly.
- **Low-capability models:** The explicit "skill directory" annotation + concrete path substitution removes ambiguity about where to find helper files.
- **No architecture changes.** Pure prompt-level modifications only.
