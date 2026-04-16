# Document Code

Generate comprehensive documentation for the specified code.

## Usage

```
/document [file_or_function] [--style=<docstring_style>] [--include-examples]
```

## Arguments

- `file_or_function` — Path to file or specific function/class to document
- `--style` — Docstring style: `google` (default), `numpy`, `sphinx`, or `plain`
- `--include-examples` — Add usage examples to docstrings

## What This Does

1. **Analyzes** the target code to understand its purpose, inputs, and outputs
2. **Generates** docstrings for all public functions, classes, and methods
3. **Documents** parameters with types and descriptions
4. **Documents** return values and raised exceptions
5. **Adds** module-level docstring if missing
6. **Optionally** includes usage examples in `Examples` sections

## Docstring Styles

### Google Style (default)

```python
def calculate_discount(price: float, rate: float) -> float:
    """Calculate the discounted price after applying a rate.

    Args:
        price: The original price before discount.
        rate: Discount rate as a decimal between 0 and 1.

    Returns:
        The final price after applying the discount.

    Raises:
        ValueError: If price is negative or rate is outside [0, 1].

    Examples:
        >>> calculate_discount(100.0, 0.2)
        80.0
    """
```

### NumPy Style

```python
def calculate_discount(price: float, rate: float) -> float:
    """
    Calculate the discounted price after applying a rate.

    Parameters
    ----------
    price : float
        The original price before discount.
    rate : float
        Discount rate as a decimal between 0 and 1.

    Returns
    -------
    float
        The final price after applying the discount.
    """
```

## Guidelines

- Preserve existing docstrings unless `--overwrite` is passed
- Skip private methods (prefixed with `_`) unless `--include-private` is set
- Keep descriptions concise but informative
- Infer types from annotations when available, otherwise from usage context
- Flag any functions where intent is ambiguous for human review

## Output

The command will:
- Show a diff of all proposed docstring additions
- Ask for confirmation before applying changes
- Report a summary: `X functions documented, Y already had docs, Z skipped`
