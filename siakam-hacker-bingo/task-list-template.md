# Task List Template

The task list is both the work plan and the checkpoint file for interrupt/resume.

## File Location

`<project_dir>/.siakam_out/BINGO/task-list.md`

## Format

```markdown
# Task List — <project_dir>
- **Created**: <ISO 8601 timestamp>
- **Last updated**: <ISO 8601 timestamp>
- **CONCURRENCY**: 2
- **Total tasks**: <N>
- **Completed**: <N>
- **Remaining**: <N>

| ID | Directory | Files Ref | Count | Status | Result |
|----|-----------|-----------|-------|--------|--------|
| 01 | src/kernel/ | file-list/task-01-files.md | 5 | pending | — |
| 02 | src/uefi/   | file-list/task-02-files.md | 8 | pending | — |
| 03 | src/sensor/ | file-list/task-03-files.md | 3 | pending | — |
```

## Columns

- **ID**: Zero-padded sequential number (01, 02, ...)
- **Directory**: Relative path from project_dir
- **Files Ref**: Path to the file list for this task, relative to `.siakam_out/BINGO/`
- **Count**: Number of files in this task
- **Status**: `pending` → `in_progress` → `done`
- **Result**: Path to findings file (`findings/task-<ID>-findings.md`), or `—` if not yet run

## Status Lifecycle

```
pending → in_progress → done
```

- **pending**: Not yet dispatched
- **in_progress**: Sub-agent is running or was interrupted (re-run on resume)
- **done**: Sub-agent completed and result file confirmed to exist

## Atomic Updates

Always update the task list immediately after a status change:
1. Mark `in_progress` BEFORE dispatching the sub-agent
2. Mark `done` only AFTER confirming the result file exists and is non-empty
3. Update `Last updated` timestamp on every change

## File List Per Task

Each task's file list goes in `<project_dir>/.siakam_out/BINGO/file-list/task-<ID>-files.md`:

```markdown
# Task <ID> Files — <directory>
1. `path/to/file1.c`
2. `path/to/file2.c`
3. `path/to/file3.c`
```
