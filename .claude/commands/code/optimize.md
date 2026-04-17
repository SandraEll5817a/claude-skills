# Optimize Code Performance

Analyze and optimize the provided code for performance, memory usage, and efficiency.

## Usage

```
/code/optimize [file_or_code] [--focus=speed|memory|readability]
```

## What This Command Does

1. **Profiles** the code to identify bottlenecks
2. **Analyzes** time and space complexity (Big O notation)
3. **Suggests** concrete optimizations with explanations
4. **Rewrites** the optimized version with comments
5. **Benchmarks** before/after comparison where possible

## Optimization Checklist

- [ ] Replace nested loops with more efficient algorithms
- [ ] Use appropriate data structures (sets for lookups, deques for queues)
- [ ] Avoid redundant computations — cache or memoize repeated calls
- [ ] Prefer list comprehensions over explicit loops where readable
- [ ] Use generators for large datasets to reduce memory footprint
- [ ] Minimize I/O operations and batch where possible
- [ ] Leverage built-in functions (they're implemented in C)
- [ ] Avoid global variable lookups in hot paths

## Example

**Before:**
```python
def find_duplicates(items: list) -> list:
    duplicates = []
    for i in range(len(items)):
        for j in range(i + 1, len(items)):
            if items[i] == items[j] and items[i] not in duplicates:
                duplicates.append(items[i])
    return duplicates
# Time: O(n^3), Space: O(n)
```

**After:**
```python
from collections import Counter

def find_duplicates(items: list) -> list:
    """Return items that appear more than once. O(n) time and space."""
    counts = Counter(items)
    return [item for item, count in counts.items() if count > 1]
# Time: O(n), Space: O(n)
```

## Output Format

For each optimization, Claude will provide:

```
### Issue: <short description>
- **Location**: line X–Y
- **Current complexity**: O(?)
- **Optimized complexity**: O(?)
- **Explanation**: why this change helps
- **Optimized code**: <rewritten snippet>
```

## Notes

- Readability is preserved unless `--focus=speed` is explicitly set
- Micro-optimizations are flagged but not applied by default
- Thread-safety implications are noted when relevant
