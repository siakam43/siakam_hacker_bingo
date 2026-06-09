# siakam-hacker-bingo: Low-Capability Model Robustness Improvements

## Problem Statement

When running siakam-hacker-bingo on lower-capability models (e.g., Qwen3.5-35B-A3B), two failures occur intermittently:

1. **Sub-agents don't write output files.** The sub-agent completes its analysis but does not call the Write tool to persist findings. This happens because the output instruction is buried in the prompt as passive natural language ("Write your findings to...") rather than an explicit tool-calling directive.

2. **The orchestrator stops after one batch.** After dispatching, waiting for, and verifying one batch of sub-agents, the main agent fails to loop back and dispatch the next batch. The word "Repeat" in Step 6 of the Dispatch Loop is not concrete enough for low-capability models to translate into explicit re-read-and-continue behavior.

Both issues are intermittent — the skill works sometimes but isn't reliable on Qwen3.5-35B-A3B, indicating the prompts sit at the model's capability threshold.

## Root Cause

| Problem | Root Cause | Type |
|---------|-----------|------|
| Missing output files | "Write your findings" is passive NL; model doesn't translate "write" to Write tool call | Prompt ambiguity |
| Batch loop stops | "Repeat" is a one-word instruction without observable verification step; model forgets the loop context after batch completion | Missing checkpoint mechanism |

## Design

### Change 1: sub-agent-prompt.md — Output enforcement

**Change 1a:** Append one sentence to the "Your Task" paragraph to establish the output obligation without distracting from analysis:

```
When analysis is complete, you MUST use the Write tool to save your findings. See "Output" section for the full requirements.
```

**Change 1b:** Rewrite the "Output" section to be tool-explicit and mandatory:

```
Use the **Write** tool to create `<project_dir>/.siakam_out/BINGO/findings/task-<ID>-findings.md`.

Follow the format in `finding-output-template.md` exactly.

**This step is mandatory.** The findings file must exist and be non-empty before you finish. If the Write call fails, retry it. A missing file means the task failed.
```

### Change 2: SKILL.md — Loop enforcement

**Change 2a:** Add one line after the "While there are tasks with status `pending`:" guard to anchor the loop intent:

```
Keep dispatching until the task list contains **zero** `pending` tasks. Do not stop early. Do not report "done" while any task is still `pending` or `in_progress`.
```

**Change 2b:** Replace the single-word "Repeat" step with an explicit checkpoint:

Step 6 from:

```
6. **Repeat** until all tasks are `done`.
```

To:

```
6. **Check remaining tasks:** Re-read `task-list.md` and count tasks with status `pending`.
   - If any `pending` tasks remain: go back to step 1 immediately. **Do not stop or report completion.**
   - If all tasks are `done`: exit the dispatch loop and proceed to Step 4 (Aggregate Results).
```

## Impact

- **High-capability models:** No functional change. The explicit instructions are already how these models interpret the original prompts.
- **Low-capability models:** The disambiguated tool directives and concrete checkpoint steps should raise reliability above the capability threshold.
- **No architecture changes.** Pure prompt-level modifications only.
