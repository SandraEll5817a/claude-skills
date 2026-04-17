# Security Audit Command

Perform a security audit on the specified code, identifying vulnerabilities and suggesting fixes.

## Usage

```
/code/security [file_or_directory]
```

## What This Command Does

1. Scans code for common security vulnerabilities
2. Checks for insecure dependencies or patterns
3. Identifies injection risks (SQL, command, XSS)
4. Reviews authentication and authorization logic
5. Suggests remediation for each finding

## Security Checks Performed

### Input Validation
- Unvalidated user inputs
- Missing sanitization before DB queries
- Path traversal vulnerabilities

### Authentication & Authorization
- Hardcoded credentials or API keys
- Weak password hashing (MD5, SHA1)
- Missing authorization checks
- Insecure token storage

### Cryptography
- Weak or deprecated algorithms
- Hardcoded secrets/keys
- Insecure random number generation

### Dependency Issues
- Known vulnerable package versions
- Unnecessary permissions

## Output Format

For each finding, provide:
- **Severity**: Critical / High / Medium / Low / Info
- **Location**: File and line number
- **Description**: What the vulnerability is
- **Risk**: What an attacker could do
- **Fix**: Concrete code change to remediate

## Example

**Input code:**
```python
import sqlite3
import hashlib

def login(username, password):
    conn = sqlite3.connect('app.db')
    query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
    cursor = conn.execute(query)
    return cursor.fetchone()

def hash_password(password):
    return hashlib.md5(password.encode()).hexdigest()

API_KEY = "sk-1234567890abcdef"
```

**Expected findings:**

- 🔴 **Critical** — SQL Injection at `login()` line 5  
  Use parameterized queries: `WHERE username=? AND password=?`

- 🔴 **Critical** — Hardcoded API key at line 12  
  Move to environment variable: `os.environ.get('API_KEY')`

- 🟠 **High** — Weak hashing (MD5) at `hash_password()` line 9  
  Use `bcrypt` or `argon2` for password hashing

## Notes

- Focus on actionable findings, not theoretical risks
- Prioritize fixes by severity
- Provide minimal working code examples for each fix
- Check OWASP Top 10 as baseline reference
