# Finding Output Template

Each sub-agent writes findings using this exact format. Consistency is mandatory — a follow-up LLM pass will parse these files.

## Zero Findings

If no security issues are found, write ONLY this:

```markdown
# Task <ID> Findings — <directory>
**Files analyzed**: <N>
**Findings**: 0

No security issues found.
```

## Finding Entry Format

Each finding is a single markdown block with exactly these fields:

```markdown
### [<SEVERITY>] <SHORT_TITLE>
- **File**: `<relative/path/to/file.c>`
- **Lines**: `<start>-<end>`
- **Category**: <CATEGORY>
- **Type**: <VULNERABILITY_TYPE>
- **Description**: <One paragraph — what the code does, why it is dangerous, how an attacker could trigger it, what conditions are needed.>
```

## Severity Levels

- **Critical**: Direct path to arbitrary code execution, privilege escalation, or complete system compromise with minimal attacker effort
- **High**: Significant security impact (memory corruption with exploitation potential, auth bypass, credential exposure) requiring moderate attacker sophistication
- **Medium**: Security weakness that enables or amplifies other attacks, or requires significant preconditions to exploit
- **Low**: Defense-in-depth gaps, minor information disclosure, or hardening opportunities — use sparingly

## Categories

Must be exactly one of:

- `Input Validation`
- `Memory Safety`
- `System Security`
- `Cryptography`
- `Race Condition`

## Finding Quality Rule

**Better to miss a theoretical issue than flood the report with false positives.**

Each finding must be something a security engineer would confidently raise in a PR review. If you are unsure whether something is exploitable, do NOT report it. Silence on a non-issue is better than noise that buries real findings.

## Per-File Header

Start each file's analysis section with:

```markdown
## `path/to/file.c` — Analysis
```

Then list findings for that file. If no findings for a file, write:

```markdown
## `path/to/file.c` — No findings
```

## Anti-Patterns to Avoid

| Do NOT report | Why |
|---------------|-----|
| Memory leaks | Out of scope — not a security issue for this analysis |
| DDoS / resource exhaustion | Out of scope |
| Performance issues | Out of scope |
| Stability / crash-only issues | Out of scope |
| Database / web vulnerabilities | Out of scope |
| Privacy leaks | Out of scope |
| Speculative issues ("if the caller passes X then Y could happen") | Only report issues that exist in the code as written, not hypothetical scenarios requiring broken callers |
| Style issues or best-practice deviations without security impact | This is a security audit, not a code review |
| "Missing NULL check" when the calling context guarantees non-NULL | Understand the control flow before reporting |
