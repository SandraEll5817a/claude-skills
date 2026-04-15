# Debug Command

Analyze and debug code issues systematically.

## Usage

```
/code/debug [file_or_error]
```

## What This Does

This command helps you identify and fix bugs by:

1. **Analyzing the error** — Parses stack traces, error messages, and symptoms
2. **Identifying root cause** — Traces the issue back to its origin
3. **Suggesting fixes** — Provides concrete, actionable solutions
4. **Explaining the bug** — Helps you understand what went wrong and why

## Steps

### 1. Gather Context

- Read the full error message or stack trace
- Identify the file(s) and line number(s) involved
- Note the expected vs actual behavior
- Check recent changes that may have introduced the bug

### 2. Reproduce the Issue

```python
# Minimal reproduction example
def reproduce_bug():
    # Isolate the failing case
    pass
```

### 3. Analyze Root Cause

Common categories to check:
- **Type errors** — Mismatched types, None values, unexpected data shapes
- **Logic errors** — Off-by-one, wrong conditionals, incorrect algorithms
- **State errors** — Mutated state, race conditions, stale data
- **Integration errors** — API misuse, missing dependencies, version conflicts
- **Environment errors** — Missing env vars, wrong paths, permission issues

### 4. Apply Fix

- Make the minimal change needed to fix the issue
- Avoid introducing new complexity
- Preserve existing behavior for unaffected code paths

### 5. Verify

- Confirm the original error no longer occurs
- Run existing tests to check for regressions
- Add a test case that covers the bug scenario

## Example Prompts

```
/code/debug
I'm getting: AttributeError: 'NoneType' object has no attribute 'split'
In file: src/parser.py line 42
```

```
/code/debug src/api/client.py
The API call works locally but fails in CI with a timeout error
```

## Output Format

The debug response will include:

- **Bug Summary** — One-line description of the issue
- **Root Cause** — Detailed explanation of why it happens
- **Fix** — Code change with before/after diff
- **Prevention** — How to avoid similar bugs in the future
- **Test Case** — A test that would catch this regression

## Tips

- Paste the **full** stack trace, not just the last line
- Include the code surrounding the error, not just the error line
- Mention what you've already tried
- Share relevant environment info (Python version, OS, dependencies)
