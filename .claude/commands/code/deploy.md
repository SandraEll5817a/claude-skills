# Deploy Command

Prepare and validate code for deployment with pre-flight checks.

## Usage
```
/deploy [environment]
```

## Arguments
- `environment` (optional): Target environment (`staging`, `production`). Defaults to `staging`.

## What This Command Does

1. **Pre-flight checks** — Verifies the codebase is ready for deployment
2. **Run tests** — Executes the full test suite and fails fast on errors
3. **Lint & format** — Ensures code style compliance
4. **Security scan** — Checks for known vulnerabilities in dependencies
5. **Build validation** — Confirms the build artifact is producible
6. **Deployment summary** — Outputs a checklist of what will be deployed

## Pre-flight Checklist

Before deploying, verify:
- [ ] All tests pass (`pytest` or equivalent)
- [ ] No uncommitted changes (`git status` is clean)
- [ ] Branch is up to date with `main`/`master`
- [ ] Environment variables are documented in `.env.example`
- [ ] Database migrations are included if schema changed
- [ ] `CHANGELOG.md` or release notes are updated
- [ ] Secrets are **not** hardcoded in source files

## Example Output

```
🚀 Deploy Readiness Report — staging
=====================================
✅ Tests: 42 passed, 0 failed
✅ Lint: No issues found
✅ Git: Working tree clean (main @ a3f9c12)
✅ Security: No known vulnerabilities
⚠️  Migration: 1 pending migration detected — run before deploy
❌ Env vars: DATABASE_URL missing from .env.example

Status: NOT READY — fix 2 issue(s) before deploying
```

## Notes

- For `production` deployments, require explicit confirmation before proceeding
- Always tag the release commit: `git tag -a v<version> -m "Release v<version>"`
- Rollback plan should be documented before any production deploy
- Check monitoring/alerting dashboards after deploy completes

## Example Deployment Script

```bash
#!/bin/bash
# deploy.sh — basic deployment helper

set -e

ENV=${1:-staging}

echo "Running pre-deploy checks for: $ENV"

# Ensure clean working tree
if [ -n "$(git status --porcelain)" ]; then
  echo "ERROR: Uncommitted changes detected. Commit or stash before deploying."
  exit 1
fi

# Run tests
python -m pytest --tb=short -q

# Run linter
flake8 . --count --select=E9,F63,F7,F82 --show-source

# Check for secrets (basic pattern)
if grep -rn --include="*.py" 'password\s*=\s*["\x27][^"\x27]' .; then
  echo "WARNING: Possible hardcoded password detected"
fi

echo "✅ Pre-deploy checks passed for $ENV"
```
