# Refactor Command

Analyze and refactor the specified code for improved readability, maintainability, and performance.

## Usage

```
/code/refactor [file_or_function] [--focus=readability|performance|structure]
```

## What This Does

1. **Analyzes** the target code for common issues:
   - Code duplication (DRY violations)
   - Long functions/methods that should be split
   - Poor naming conventions
   - Missing or outdated docstrings
   - Unnecessary complexity
   - Magic numbers/strings

2. **Suggests and applies** targeted refactors:
   - Extract repeated logic into helper functions
   - Rename variables/functions for clarity
   - Simplify conditionals and loops
   - Add type hints where missing
   - Replace magic values with named constants

3. **Validates** the refactor:
   - Ensures existing tests still pass
   - Confirms behavior is unchanged
   - Checks that new code passes linting

## Refactor Checklist

- [ ] No behavior changes introduced
- [ ] All existing tests pass
- [ ] New constants defined for magic values
- [ ] Functions are single-responsibility
- [ ] Docstrings updated to reflect changes
- [ ] Type hints added or corrected
- [ ] No new linting warnings introduced

## Example Output

```python
# BEFORE
def p(d):
    r = []
    for i in d:
        if i > 0 and i < 100:
            r.append(i * 1.08)
    return r

# AFTER
TAX_RATE = 1.08
MIN_PRICE = 0
MAX_PRICE = 100

def apply_tax_to_valid_prices(prices: list[float]) -> list[float]:
    """
    Apply tax rate to all prices within the valid range.

    Args:
        prices: List of raw price values.

    Returns:
        List of tax-applied prices for values in (MIN_PRICE, MAX_PRICE).
    """
    return [
        price * TAX_RATE
        for price in prices
        if MIN_PRICE < price < MAX_PRICE
    ]
```

## Notes

- Always run tests after refactoring: `/code/test`
- For large files, refactor one section at a time
- Commit working state before starting: `/git/cm`
- Use `/code/review` afterward to verify quality
