# Search & Replace Command

Perform intelligent code search and replace operations across the codebase.

## Usage

```
/code/search <pattern> [--replace <replacement>] [--scope <files>]
```

## What This Does

1. **Search** for patterns (literal, regex, or semantic)
2. **Preview** all matches with context
3. **Replace** safely with confirmation
4. **Report** a summary of changes made

## Search Modes

- `--literal`: Exact string match (default)
- `--regex`: Regular expression pattern
- `--semantic`: Find conceptually similar code

## Examples

### Find all usages of a function
```
/code/search calculate_total
```

### Rename a function across the codebase
```
/code/search calculate_total --replace compute_total
```

### Find deprecated API calls
```
/code/search "requests.get" --regex --scope "src/**/*.py"
```

### Replace with regex capture groups
```
/code/search "print\((.+)\)" --regex --replace "logger.debug(\1)"
```

## Example Output

```python
# Before search/replace
def calculate_total(items):
    return sum(item.price for item in items)

result = calculate_total(cart.items)
print(f"Total: {calculate_total(order.items)}")

# After: /code/search calculate_total --replace compute_total
def compute_total(items):
    return sum(item.price for item in items)

result = compute_total(cart.items)
print(f"Total: {compute_total(order.items)}")
```

## Instructions

When this command is invoked:

1. Parse the search pattern and options from the arguments
2. Scan the specified scope (default: entire project)
3. Display matches with file path, line number, and surrounding context (2 lines)
4. If `--replace` is provided:
   - Show a diff preview of all proposed changes
   - Ask for confirmation before applying
   - Apply changes atomically
   - Report: X files changed, Y occurrences replaced
5. Respect `.gitignore` and skip binary files
6. Highlight potential risks (e.g., replacing inside strings vs. identifiers)
