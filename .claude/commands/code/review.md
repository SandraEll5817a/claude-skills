# Code Review

Perform a thorough code review of the specified files or recent changes.

## Usage

```
/code/review [files or scope]
```

## What This Does

This command performs a comprehensive code review focusing on:

1. **Correctness** — Logic errors, edge cases, off-by-one errors
2. **Security** — Injection vulnerabilities, insecure defaults, exposed secrets
3. **Performance** — Inefficient algorithms, unnecessary re-renders, N+1 queries
4. **Readability** — Naming clarity, function length, complexity
5. **Maintainability** — DRY violations, tight coupling, missing abstractions
6. **Test coverage** — Untested paths, missing edge case tests

## Steps

1. If `$ARGUMENTS` is provided, review those specific files. Otherwise, review all files changed since the last commit (`git diff HEAD`).
2. For each file, analyze the code against the criteria above.
3. Categorize findings by severity:
   - 🔴 **Critical** — Must fix before merging (bugs, security issues)
   - 🟡 **Warning** — Should address (performance, maintainability)
   - 🔵 **Suggestion** — Nice to have (style, minor improvements)
4. Provide a summary section at the end with overall assessment.
5. If no issues are found, explicitly confirm the code looks good.

## Output Format

```
## Code Review: <scope>

### <filename>

🔴 Critical: <description>
   Line <N>: <code snippet>
   Reason: <explanation>
   Fix: <suggested fix>

🟡 Warning: <description>
   ...

🔵 Suggestion: <description>
   ...

---

## Summary
- Critical issues: N
- Warnings: N
- Suggestions: N
- Overall: <LGTM / Needs Work / Major Issues>
```

## Arguments

`$ARGUMENTS` — Optional. File paths or glob patterns to review. Defaults to `git diff HEAD`.
