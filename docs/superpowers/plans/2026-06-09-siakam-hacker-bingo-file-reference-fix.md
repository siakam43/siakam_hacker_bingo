# File Reference Path Fix — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix all helper file references so low-capability models find them in the skill directory instead of the target project directory.

**Architecture:** Add a "Helper Files" section to SKILL.md listing all supplementary files and their location. Annotate every bare filename reference with "in the skill directory". Add orchestrator instruction to substitute concrete paths in sub-agent prompts.

**Tech Stack:** Markdown-only. No code, no scripts, no tests.

---

### Task 1: Add Helper Files section to SKILL.md (Change 3a)

**Files:**
- Modify: `siakam-hacker-bingo/SKILL.md`

- [ ] **Step 1: Insert the Helper Files section**

Insert after the Configuration block (after the ``` fence closing the config block) and before `## Usage`:

```markdown
## Helper Files

The following files are in the **same directory as this SKILL.md**:

- `sub-agent-prompt.md` — the full sub-agent analysis prompt
- `task-list-template.md` — format for the task list checkpoint file
- `finding-output-template.md` — format for per-task findings output

When you need to read one of these files, look in the skill directory — **not** in the target project directory.
```

- [ ] **Step 2: Verify the insertion**

Run: `git diff siakam-hacker-bingo/SKILL.md | head -30`

Expected: One hunk inserting the "Helper Files" section between Configuration and Usage.

- [ ] **Step 3: Commit**

```bash
git add siakam-hacker-bingo/SKILL.md
git commit -m "docs: add Helper Files section to SKILL.md for file location clarity

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 2: Annotate references in SKILL.md (Changes 3b, 3d)

**Files:**
- Modify: `siakam-hacker-bingo/SKILL.md`

- [ ] **Step 1: Annotate task-list-template.md reference in Step 2.6**

Find:

```
Follow the format in `task-list-template.md`.
```

Replace with:

```
Follow the format in `task-list-template.md` (in the skill directory).
```

- [ ] **Step 2: Annotate sub-agent-prompt.md reference in Step 3.3**

Find:

```
`prompt`: The exact content from `sub-agent-prompt.md`, with the file list inserted and `<project_dir>` / `<ID>` replaced with actual values.
```

Replace with:

```
`prompt`: The exact content from `sub-agent-prompt.md` (in the skill directory), with the file list inserted and `<project_dir>` / `<ID>` replaced with actual values.
```

- [ ] **Step 3: Add path substitution instruction in Step 3.3 (Change 3d)**

After the "prompt" line block, append a note:

```
Before sending the prompt, replace the reference to `finding-output-template.md` with the actual path to the file in the skill directory (e.g., `siakam-hacker-bingo/finding-output-template.md`).
```

- [ ] **Step 4: Verify**

Run: `git diff siakam-hacker-bingo/SKILL.md`

Expected: Three hunks — Step 2.6 annotation, Step 3.3 annotation, and the new path substitution note.

- [ ] **Step 5: Commit**

```bash
git add siakam-hacker-bingo/SKILL.md
git commit -m "fix: annotate file references with skill directory location hints

Annotate task-list-template.md and sub-agent-prompt.md references.
Add orchestrator instruction to substitute concrete paths in sub-agent prompts.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 3: Annotate reference in sub-agent-prompt.md (Change 3c)

**Files:**
- Modify: `siakam-hacker-bingo/sub-agent-prompt.md`

- [ ] **Step 1: Annotate finding-output-template.md reference in Output section**

Find:

```
Follow the format in `finding-output-template.md` exactly.
```

Replace with:

```
Follow the format in `finding-output-template.md` exactly. This file is in the skill directory.
```

- [ ] **Step 2: Verify**

Run: `git diff siakam-hacker-bingo/sub-agent-prompt.md`

Expected: One hunk appending "This file is in the skill directory." to the finding-output-template.md reference.

- [ ] **Step 3: Commit**

```bash
git add siakam-hacker-bingo/sub-agent-prompt.md
git commit -m "fix: annotate finding-output-template.md reference with skill directory hint

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```
