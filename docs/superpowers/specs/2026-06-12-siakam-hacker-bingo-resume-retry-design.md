# siakam-hacker-bingo: Resume Check Retry Rule

## Problem Statement

When running siakam-hacker-bingo on lower-capability models, the Step 1 "Check for Resume" sometimes produces inconclusive results: Bash tool calls return no output, empty results, or errors. The orchestrator interprets these as "file does not exist" and proceeds to Step 2 (fresh scan), overwriting the existing `task-list.md` and losing all prior progress.

The root cause is that Step 1 only has two branches — "exists" and "does not exist" — with no handling for the ambiguous third case: "I couldn't determine whether it exists."

## Design

Modify SKILL.md Step 1 to add a retry rule with a different method before concluding the file does not exist.

### Current

```
### Step 1: Check for Resume

**If task-list.md exists:** Resume. **If not:** Start from Step 2.
```

### New

```
### Step 1: Check for Resume

Before starting a fresh scan, check whether `<project_dir>/.siakam_out/BINGO/task-list.md`
exists.

Only a definitive "file exists" result can skip retry. In every other case you MUST try a
second time with a different method (different command or different tool):

| Result | Action |
|--------|--------|
| File exists | Read task-list.md and resume from Step 3 |
| File not found (explicit, like "NOT_FOUND" or "No such file") | Try one more time with a different method |
| No output / empty result | Try one more time with a different method |
| Tool error | Try one more time with a different method |

Only when **both attempts** confirm the file does not exist, proceed to Step 2. If both
attempts produce no output or errors, do NOT proceed — report the issue to the user and
stop.

**If task-list.md exists:** Read it. Skip all tasks with status `done`. Find the first task
with status `pending` or `in_progress`. Jump to Step 3 (Dispatch Loop) starting from that
task. Tasks marked `in_progress` were interrupted — re-run them.
```

## Impact

- **High-capability models:** No functional change. They already get definitive results on the first try.
- **Low-capability models:** The retry rule prevents false-negative resume failures. A second attempt with a different method (e.g., `ls` after `test -f`, Read after Bash) catches the ambiguous case.
- **Both checks inconclusive:** The orchestrator stops and reports the problem instead of silently destroying work.
- **No architectural changes.** Pure Step 1 prompt-level modification.
