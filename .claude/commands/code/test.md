# Test Generation Command

Generate comprehensive tests for the specified code.

## Usage

```
/code:test [file_or_function] [--type unit|integration|e2e] [--framework pytest|unittest|jest]
```

## What This Command Does

Analyzes the target code and generates meaningful tests that cover:

1. **Happy path** — Expected inputs and outputs
2. **Edge cases** — Boundary values, empty inputs, large inputs
3. **Error handling** — Invalid inputs, exceptions, failure modes
4. **Mocking** — External dependencies, I/O, network calls

## Instructions

You are a senior QA engineer and Python developer. Your task is to generate high-quality, maintainable tests for the provided code.

### Steps

1. Read and understand the target file or function: `$ARGUMENTS`
2. Identify all public functions, methods, and classes
3. Determine the appropriate test framework (default: pytest)
4. Generate tests following these principles:
   - Each test should have a single, clear assertion focus
   - Use descriptive test names: `test_<function>_<scenario>_<expected_result>`
   - Group related tests in classes when testing a class
   - Add docstrings explaining what each test validates
   - Use fixtures for shared setup/teardown
   - Mock external dependencies (APIs, databases, file system)

### Output Format

Produce a complete test file with:
- Proper imports
- Fixtures at the top
- Tests organized logically
- Comments explaining non-obvious test logic

### Coverage Goals

- Aim for >80% branch coverage
- Every public function must have at least one test
- All exception paths should be tested
- Parametrize tests where multiple input variations apply

### Example Test Structure

```python
import pytest
from unittest.mock import patch, MagicMock

# Fixtures
@pytest.fixture
def sample_data():
    return {...}

# Unit tests
class TestFunctionName:
    def test_returns_expected_value_for_valid_input(self, sample_data):
        """Verify function returns correct result for standard input."""
        ...

    def test_raises_value_error_for_empty_input(self):
        """Ensure ValueError is raised when input is empty."""
        with pytest.raises(ValueError):
            ...

    @pytest.mark.parametrize("input,expected", [
        (1, "one"),
        (2, "two"),
    ])
    def test_handles_multiple_input_types(self, input, expected):
        ...
```

After generating the tests, provide a brief summary of:
- Number of tests generated
- Coverage areas addressed
- Any areas that may need manual testing or are difficult to automate
