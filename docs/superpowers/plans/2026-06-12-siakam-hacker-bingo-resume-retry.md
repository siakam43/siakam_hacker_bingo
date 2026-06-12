# Resume Retry Rule — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a retry rule to Step 1 of SKILL.md so the orchestrator never skips resume based on an inconclusive check.

**Architecture:** Prompt-level change only — replace the binary "exists / does not exist" branch in Step 1 with a retry table and two-stage confirmation gate. No file structure changes, no new files.

**Tech Stack:** Markdown (SKILL.md prompt content)

---

### Task 1: Replace Step 1 with retry rule

**Files:**
- Modify: `siakam-hacker-bingo/SKILL.md:57-63`

- [ ] **Step 1: Replace Step 1 content**

Replace lines 57-63:

```
### Step 1: Check for Resume

Before starting a fresh scan, check whether `<project_dir>/.siakam_out/BINGO/task-list.md` exists.

**If task-list.md exists:** Read it. Skip all tasks with status `done`. Find the first task with status `pending` or `in_progress`. Jump to Step 3 (Dispatch Loop) starting from that task. Tasks marked `in_progress` were interrupted — re-run them.

**If task-list.md does not exist:** Start from Step 2.
```

With:

```
### Step 1: Check for Resume

Before starting a fresh scan, check whether `<project_dir>/.siakam_out/BINGO/task-list.md` exists.

**Retry rule:** Only a definitive "file exists" result can skip retry. In every other case
you MUST try a second time with a different method (different command or different tool):

| Result | Action |
|--------|--------|
| File exists | Read task-list.md and resume from Step 3 |
| File not found (explicit, like "NOT_FOUND" or "No such file") | Try one more time with a different method |
| No output / empty result | Try one more time with a different method |
| Tool error | Try one more time with a different method |

Only when **both attempts** confirm the file does not exist, proceed to Step 2. If both
attempts produce no output or errors, do NOT proceed — report the issue to the user and
stop. Never assume the file is missing based on an inconclusive check.

**If task-list.md exists:** Read it. Skip all tasks with status `done`. Find the first task
with status `pending` or `in_progress`. Jump to Step 3 (Dispatch Loop) starting from that
task. Tasks marked `in_progress` were interrupted — re-run them.
```

- [ ] **Step 2: Verify the diff**

```bash
git -C /home/admin/cc/wksp/siakam_security_skills/siakam_hacker_bingo diff siakam-hacker-bingo/SKILL.md
```

Expected: Shows exactly the Step 1 replacement, with retry table and two-stage-confirmation rule in place.

- [ ] **Step 3: Commit**

```bash
git -C /home/admin/cc/wksp/siakam_security_skills/siakam_hacker_bingo add siakam-hacker-bingo/SKILL.md
git -C /home/admin/cc/wksp/siakam_security_skills/siakam_hacker_bingo commit -m "fix: add resume retry rule to prevent false-negative task-list overwrite"
```

Commit message: `fix: add resume retry rule to prevent false-negative task-list overwrite`
