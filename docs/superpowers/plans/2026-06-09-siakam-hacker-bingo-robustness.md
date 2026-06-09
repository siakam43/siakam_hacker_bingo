# siakam-hacker-bingo Robustness Improvements — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply prompt-level fixes to sub-agent-prompt.md and SKILL.md so siakam-hacker-bingo executes reliably on lower-capability models (Qwen3.5-35B-A3B).

**Architecture:** Two isolated prompt edits with no code changes. Change 1 makes the Write tool instruction explicit and mandatory for sub-agents. Change 2 replaces the vague "Repeat" loop instruction with an explicit checkpoint that forces re-reading task-list.md after each batch.

**Tech Stack:** Markdown-only. No code, no scripts, no tests.

---

### Task 1: Fix sub-agent-prompt.md — Enforce Write tool usage

**Files:**
- Modify: `siakam-hacker-bingo/sub-agent-prompt.md`

**Change 1a — Append output obligation to "Your Task" paragraph:**

- [ ] **Step 1: Add the Write tool sentence to the "Your Task" paragraph**

Find the paragraph starting with:

```
You MAY browse other files in `<project_dir>` to understand context — see "Reading Scope vs. Analysis Scope" below.
```

After that line, add:

```
When analysis is complete, you MUST use the Write tool to save your findings. See "Output" section for the full requirements.
```

**Change 1b — Rewrite the "Output" section:**

- [ ] **Step 2: Replace the Output section**

Find:

```
## Output

Write your findings to `<project_dir>/.siakam_out/BINGO/findings/task-<ID>-findings.md` using the format defined in `finding-output-template.md`.
```

Replace with:

```
## Output

Use the **Write** tool to create `<project_dir>/.siakam_out/BINGO/findings/task-<ID>-findings.md`.

Follow the format in `finding-output-template.md` exactly.

**This step is mandatory.** The findings file must exist and be non-empty before you finish. If the Write call fails, retry it. A missing file means the task failed.
```

- [ ] **Step 3: Verify the changes**

Run: `git diff siakam-hacker-bingo/sub-agent-prompt.md`

Expected: Two hunks — one in "Your Task" paragraph, one replacing the "Output" section.

- [ ] **Step 4: Commit**

```bash
git add siakam-hacker-bingo/sub-agent-prompt.md
git commit -m "fix: enforce Write tool usage in sub-agent prompt for low-capability models

Change 1a: append Write tool obligation to Your Task paragraph
Change 1b: rewrite Output section with explicit tool directive and mandatory step

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```

---

### Task 2: Fix SKILL.md — Enforce batch dispatch loop continuation

**Files:**
- Modify: `siakam-hacker-bingo/SKILL.md`

**Change 2a — Add loop intent anchor:**

- [ ] **Step 1: Add the anchor line after the "While" guard**

Find:

```
While there are tasks with status `pending`:
```

After that line, add:

```
Keep dispatching until the task list contains **zero** `pending` tasks. Do not stop early. Do not report "done" while any task is still `pending` or `in_progress`.
```

**Change 2b — Replace "Repeat" with explicit checkpoint:**

- [ ] **Step 2: Replace Step 6**

Find:

```
6. **Repeat** until all tasks are `done`.
```

Replace with:

```
6. **Check remaining tasks:** Re-read `task-list.md` and count tasks with status `pending`.
   - If any `pending` tasks remain: go back to step 1 immediately. **Do not stop or report completion.**
   - If all tasks are `done`: exit the dispatch loop and proceed to Step 4 (Aggregate Results).
```

- [ ] **Step 3: Verify the changes**

Run: `git diff siakam-hacker-bingo/SKILL.md`

Expected: Two hunks — one after the "While" guard, one replacing Step 6.

- [ ] **Step 4: Commit**

```bash
git add siakam-hacker-bingo/SKILL.md
git commit -m "fix: enforce batch dispatch loop continuation for low-capability models

Change 2a: add loop intent anchor after the While guard
Change 2b: replace vague 'Repeat' with explicit checkpoint instruction

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>"
```
