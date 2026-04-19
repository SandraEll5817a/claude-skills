# Format Code

Automatically format code to follow project style conventions and best practices.

## Usage

```
/code/format [file_or_directory] [--style=<style>] [--check-only]
```

## What This Does

1. **Detects language** from file extension or content
2. **Applies formatting rules** based on project config or defaults
3. **Reports changes** made or needed
4. **Optionally checks only** without modifying files

## Formatting Rules Applied

### Python
- Line length: 88 characters (Black default)
- String quotes: double quotes
- Import ordering: stdlib → third-party → local (isort)
- Trailing commas in multi-line structures
- Blank lines between top-level definitions

### JavaScript / TypeScript
- Semicolons: as configured
- Quotes: single or double (consistent)
- Indent: 2 spaces
- Trailing commas: ES5 compatible

### General
- Remove trailing whitespace
- Ensure newline at end of file
- Consistent line endings (LF)

## Examples

### Before Formatting

```python
import os
import sys
from myapp import utils
import json

def calculate_price(base_price,tax_rate,discount=0):
    """Calculate final price."""
    if discount<0 or discount>1:
        raise ValueError('discount must be between 0 and 1')
    discounted=base_price*(1-discount)
    tax=discounted*tax_rate
    return discounted+tax

class OrderProcessor:
    def __init__(self,order_id,items):
        self.order_id=order_id
        self.items=items
    def total(self):
        return sum(item['price']*item['qty'] for item in self.items)
```

### After Formatting

```python
import json
import os
import sys

from myapp import utils


def calculate_price(base_price, tax_rate, discount=0):
    """Calculate final price."""
    if discount < 0 or discount > 1:
        raise ValueError("discount must be between 0 and 1")
    discounted = base_price * (1 - discount)
    tax = discounted * tax_rate
    return discounted + tax


class OrderProcessor:
    def __init__(self, order_id, items):
        self.order_id = order_id
        self.items = items

    def total(self):
        return sum(item["price"] * item["qty"] for item in self.items)
```

## Check-Only Mode

Use `--check-only` to validate formatting without modifying files:

```
/code/format src/ --check-only
```

Output example:
```
❌ src/models/order.py — 12 formatting issues
❌ src/utils/helpers.py — 3 formatting issues
✅ src/api/routes.py — already formatted

2 files would be reformatted, 1 file would be left unchanged.
```

## Configuration Files Respected

| File | Tool |
|------|------|
| `pyproject.toml` | Black, isort |
| `.flake8` | Flake8 |
| `.prettierrc` | Prettier |
| `eslint.config.js` | ESLint |
| `.editorconfig` | General |

## Notes

- Always review changes before committing
- Combine with `/code/lint` for full style compliance
- Run before opening a pull request
