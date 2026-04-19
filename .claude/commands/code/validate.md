# Validate

Validate data structures, inputs, and configurations with comprehensive error reporting.

## Usage

```
/code:validate [target]
```

## What This Does

1. Analyzes the target code for validation patterns
2. Identifies missing input validation
3. Suggests schema-based validation where appropriate
4. Generates validators with clear error messages

## Example Input

```python
def create_user(data: dict):
    # No validation
    return User(
        name=data['name'],
        email=data['email'],
        age=data['age']
    )
```

## Example Output

```python
from dataclasses import dataclass
from typing import Any
import re


class ValidationError(Exception):
    """Raised when input validation fails."""

    def __init__(self, field: str, message: str):
        self.field = field
        self.message = message
        super().__init__(f"{field}: {message}")


@dataclass
class UserValidator:
    """Validates user creation data."""

    EMAIL_PATTERN = re.compile(r'^[\w.+-]+@[\w-]+\.[\w.]+$')
    MIN_AGE = 0
    MAX_AGE = 150
    MAX_NAME_LENGTH = 100

    def validate(self, data: dict[str, Any]) -> dict[str, Any]:
        """Validate and return cleaned user data.

        Args:
            data: Raw input dictionary.

        Returns:
            Cleaned and validated data.

        Raises:
            ValidationError: If any field fails validation.
        """
        errors = []

        name = data.get('name', '').strip()
        if not name:
            errors.append(ValidationError('name', 'Name is required'))
        elif len(name) > self.MAX_NAME_LENGTH:
            errors.append(ValidationError('name', f'Name must be under {self.MAX_NAME_LENGTH} characters'))

        email = data.get('email', '').strip().lower()
        if not email:
            errors.append(ValidationError('email', 'Email is required'))
        elif not self.EMAIL_PATTERN.match(email):
            errors.append(ValidationError('email', 'Invalid email format'))

        age = data.get('age')
        if age is None:
            errors.append(ValidationError('age', 'Age is required'))
        elif not isinstance(age, int):
            errors.append(ValidationError('age', 'Age must be an integer'))
        elif not (self.MIN_AGE <= age <= self.MAX_AGE):
            errors.append(ValidationError('age', f'Age must be between {self.MIN_AGE} and {self.MAX_AGE}'))

        if errors:
            raise ValueError({'errors': [{'field': e.field, 'message': e.message} for e in errors]})

        return {'name': name, 'email': email, 'age': age}


def create_user(data: dict):
    validator = UserValidator()
    clean_data = validator.validate(data)
    return User(**clean_data)
```

## Validation Patterns Applied

- **Required fields**: Checks for missing or empty values
- **Type checking**: Ensures correct data types
- **Format validation**: Regex patterns for emails, URLs, etc.
- **Range validation**: Numeric bounds checking
- **Length limits**: String length constraints
- **Sanitization**: Strips whitespace, normalizes casing

## Notes

- Collects all errors before raising (fail-all, not fail-fast)
- Returns cleaned/normalized data on success
- Uses structured error format for API compatibility
