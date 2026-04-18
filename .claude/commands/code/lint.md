# Lint & Format Code

Analyze the provided code for style issues, formatting problems, and linting errors. Apply fixes following best practices for the detected language.

## Usage

```
/code/lint [file_or_code] [--fix] [--strict]
```

## What This Does

1. Detects the programming language
2. Identifies style and formatting issues
3. Checks for common linting errors
4. Suggests or applies fixes automatically
5. Reports a summary of changes made

## Lint Checks Performed

### Python
- PEP 8 compliance (indentation, line length, whitespace)
- Import ordering (isort conventions)
- Unused imports and variables
- Naming conventions (snake_case for functions/variables, PascalCase for classes)
- Missing type hints (if strict mode)
- Docstring presence and format

### JavaScript / TypeScript
- ESLint standard rules
- Prettier formatting
- Unused variables and imports
- Consistent semicolons and quotes
- Arrow function consistency

### General
- Trailing whitespace
- Mixed tabs/spaces
- Missing newline at end of file
- Lines exceeding 120 characters

## Example

**Before:**
```python
import os
import sys
import json

def calculateTotal( items,tax_rate ):
    Total=0
    for i in items:
        Total=Total+i['price']
    total_with_tax=Total*(1+tax_rate)
    return total_with_tax

class shoppingCart:
    def __init__(self,items=[]):
        self.items=items
```

**After:**
```python
import json
import os
import sys
from typing import List


def calculate_total(items: List[dict], tax_rate: float) -> float:
    """Calculate the total price of items including tax."""
    total = sum(item['price'] for item in items)
    return total * (1 + tax_rate)


class ShoppingCart:
    def __init__(self, items: List[dict] | None = None) -> None:
        self.items = items or []
```

## Lint Report Format

After analysis, provide a structured report:

```
Lint Report
===========
File: example.py
Language: Python

Issues Found: 8
  - [LINE 4]  E302 expected 2 blank lines, found 1
  - [LINE 6]  E211 whitespace before '('
  - [LINE 7]  N806 variable 'Total' should be lowercase
  - [LINE 10] E225 missing whitespace around operator
  ...

Auto-fixed: 8/8
Remaining: 0
```

## Instructions

When `/code/lint` is invoked:
1. Read the target file or code snippet
2. Run through all applicable checks for the detected language
3. If `--fix` flag is present, apply all safe automatic fixes
4. If `--strict` flag is present, also enforce type hints and docstrings
5. Present the before/after diff for significant changes
6. Summarize total issues found and fixed
7. List any issues that require manual intervention
