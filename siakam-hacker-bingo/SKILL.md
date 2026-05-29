---
name: siakam-hacker-bingo
description: Use when the user invokes /siakam-hacker-bingo or asks to perform a security audit of C source code (.c/.h files) in a project directory, focusing on embedded/systems-level vulnerabilities (kernel drivers, boot chain, firmware). Triggers on requests to find security bugs in C codebases, analyze C files for vulnerabilities, or audit low-level C projects for memory safety, input validation, crypto, race condition, or system security issues.
---

# siakam-hacker-bingo

Security vulnerability analysis for C codebases using parallel sub-agents with checkpoint-based resume. Targets low-level embedded/systems code: Linux kernel drivers, mobile boot chain (UEFI, BL31, BL2, XLoader), and co-processor firmware (ISP, SensorHub, GPU).

## Configuration

Set at the top of this file. Edit to adjust behavior.

```
CONCURRENCY = 2   # Number of sub-agents running in parallel
```

## Usage

```
/siakam-hacker-bingo <project_dir>
```

If `project_dir` is omitted, defaults to the current working directory.

## Workflow

```
SCAN → TASK LIST → DISPATCH LOOP → AGGREGATE
  │                    │
  └── .siakamignore    └── RESUME: skip done, re-run in_progress
```

### Step 1: Check for Resume

Before starting a fresh scan, check whether `<project_dir>/.siakam_out/BINGO/task-list.md` exists.

**If task-list.md exists:** Read it. Skip all tasks with status `done`. Find the first task with status `pending` or `in_progress`. Jump to Step 3 (Dispatch Loop) starting from that task. Tasks marked `in_progress` were interrupted — re-run them.

**If task-list.md does not exist:** Start from Step 2.

### Step 2: Scan and Create Task List

1. **Enumerate source files:** Run `find <project_dir> -type f \( -name "*.c" -o -name "*.h" \)` to list all C source and header files.

2. **Apply .siakamignore:** If `<project_dir>/.siakamignore` exists, read it and exclude files matching its patterns. The file uses gitignore syntax:
   - Each line is a glob pattern relative to `project_dir`
   - `#` starts a comment
   - `!` negates a previous pattern
   - Patterns without `/` match at any depth
   - Trailing `/` matches only directories
   - `*` matches anything except `/`
   - `**` matches anything including `/`

3. **Group files into tasks:** Group the remaining files by their immediate parent directory. Each directory becomes one task. If a directory has more than 10 files, split it into sub-tasks of roughly equal size (e.g., alphabetically by filename).

   A parent directory that contains both source files AND subdirectories: create one task for the parent's own source files, and separate tasks for each subdirectory.

4. **Create output directories:**
   ```
   mkdir -p <project_dir>/.siakam_out/BINGO/file-list
   mkdir -p <project_dir>/.siakam_out/BINGO/findings
   ```

5. **Write file lists:** For each task, write the list of files to `<project_dir>/.siakam_out/BINGO/file-list/task-<ID>-files.md`. One file path per line, relative to `project_dir`. Use backtick-quoted paths.

6. **Write task-list.md:** Create the checkpoint file at `<project_dir>/.siakam_out/BINGO/task-list.md`. Follow the format in `task-list-template.md`. All tasks start with status `pending`.

### Step 3: Dispatch Loop

While there are tasks with status `pending`:

1. **Pick tasks:** Take up to `CONCURRENCY` tasks with status `pending` from the task list (in order by ID).

2. **Mark in_progress:** Update each task's status to `in_progress` in task-list.md. Update the `Last updated` timestamp.

3. **Dispatch sub-agents:** For each task, launch a sub-agent using the `Agent` tool with:
   - `description`: Short description like "Security analysis of <directory>"
   - `subagent_type`: `general-purpose`
   - `prompt`: The exact content from `sub-agent-prompt.md`, with the file list inserted and `<project_dir>` / `<ID>` replaced with actual values.

   **CRITICAL:** Dispatch all `CONCURRENCY` sub-agents in parallel in a single message. Each sub-agent call is independent — they do not share state.

4. **Wait for completion:** All sub-agents in the batch must finish before the next batch starts.

5. **Verify and mark done:** For each completed sub-agent:
   - Confirm the findings file exists at `<project_dir>/.siakam_out/BINGO/findings/task-<ID>-findings.md`
   - If the file is missing, re-run that sub-agent once
   - If the file exists, mark the task status as `done` in task-list.md
   - Update `Completed` and `Remaining` counts and `Last updated` timestamp

6. **Repeat** until all tasks are `done`.

### Step 4: Aggregate Results

1. **Concatenate findings:** Collect all `<project_dir>/.siakam_out/BINGO/findings/task-*-findings.md` files. Concatenate them into a single `<project_dir>/.siakam_out/BINGO/FINAL_REPORT.md`.

2. **Add summary header:**

   ```markdown
   # Security Analysis Report — <project_dir>
   **Completed**: <ISO 8601 timestamp>
   **Total files analyzed**: <N>
   **Total tasks**: <N>
   **Total findings**: <N>
   **Critical**: <N> | **High**: <N> | **Medium**: <N> | **Low**: <N>

   ---
   ```

   Count findings by parsing `### [<SEVERITY>]` headings in each task file.

3. **Report completion:** Tell the user the report is ready at `<project_dir>/.siakam_out/BINGO/FINAL_REPORT.md` with the summary counts.

## Required Permissions

The skill needs these tools to be allowed without prompts:

| Tool | Reason |
|------|--------|
| `Bash` | `find` for file enumeration, `mkdir` for output directories |
| `Read` | Reading source files, .siakamignore, task-list.md |
| `Write` | Writing task-list.md, file lists, findings files, FINAL_REPORT.md |
| `Agent` | Dispatching sub-agents for parallel security analysis |

## Rules for the Main Agent

1. **Do not analyze code yourself.** Your job is orchestration only — scan, dispatch, aggregate. Sub-agents do the analysis.

2. **Do not skip tasks.** Every task in the list must be dispatched, even if there are dozens of them. Quality over speed.

3. **Update the task list atomically.** Status changes must be written to task-list.md immediately.

4. **Do not dispatch more than CONCURRENCY sub-agents at once.** Wait for the batch to complete before starting the next.

5. **On resume, re-run in_progress tasks.** Assume they were interrupted mid-analysis.

6. **Verify output exists before marking done.** Check the findings file is present and non-empty.

## Orchestrator Anti-Rationalization

When you catch yourself thinking these thoughts, STOP:

| Rationalization | Reality |
|-----------------|---------|
| "There are 50 tasks, I'll bump concurrency to 10" | CONCURRENCY is set deliberately. Do not override it. |
| "This project is small, I'll just analyze it myself" | Your job is orchestration only. Sub-agents do the analysis. |
| "These files look similar, I'll combine tasks" | Each task is scoped for quality. Combining reduces thoroughness. |
| "The task list is too long, I'll simplify" | Every task must be dispatched. No skipping. Quality over speed. |
| "I can dispatch the next batch while waiting" | Wait for ALL in-flight sub-agents to complete before starting the next batch. |
| "The result file is empty, but the sub-agent probably found nothing" | Re-run the task. An empty file means an error, not zero findings. |
| "I'll skip the .siakamignore check, the user probably doesn't need it" | Always check for .siakamignore. It is a documented feature. |

## Red Flags — STOP and Re-evaluate

- You are about to analyze code yourself instead of dispatching a sub-agent
- You are about to dispatch more than CONCURRENCY sub-agents at once
- You are about to mark a task `done` without verifying the result file exists
- You are about to skip a task because "it's just a few files"

**All of these mean: Follow the workflow. Do not take shortcuts.**

## Resume Example

```
User: /siakam-hacker-bingo /path/to/kernel-tree
Agent: Found existing task list. 15/45 tasks done, 30 remaining.
       Resuming from task 16 (in_progress — re-running).
       [Dispatches tasks 16 and 17]
```
