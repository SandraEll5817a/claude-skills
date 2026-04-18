# Database Migration Command

Generate and apply database migrations safely with rollback support.

## Usage

```
/code/migrate [--generate | --apply | --rollback] [--dry-run]
```

## What This Does

1. Analyzes current schema and model definitions
2. Detects schema drift between models and database
3. Generates migration scripts with up/down methods
4. Validates migrations before applying
5. Applies migrations in a transaction with rollback on failure

## Migration Generation

When generating migrations, Claude will:
- Compare ORM models against current database schema
- Identify added/removed/modified tables and columns
- Generate reversible migration with `upgrade()` and `downgrade()` functions
- Name migration files with timestamp prefix and descriptive slug

## Example Output

```python
"""Add user profile table

Revision ID: 3f2a1b9c4d8e
Revises: 1a2b3c4d5e6f
Create Date: 2024-01-15 10:23:45.123456
"""
from alembic import op
import sqlalchemy as sa

revision = '3f2a1b9c4d8e'
down_revision = '1a2b3c4d5e6f'
branch_labels = None
depends_on = None


def upgrade() -> None:
    op.create_table(
        'user_profiles',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('user_id', sa.Integer(), sa.ForeignKey('users.id'), nullable=False),
        sa.Column('bio', sa.Text(), nullable=True),
        sa.Column('avatar_url', sa.String(255), nullable=True),
        sa.Column('created_at', sa.DateTime(), server_default=sa.text('now()'), nullable=False),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('user_id')
    )
    op.create_index('ix_user_profiles_user_id', 'user_profiles', ['user_id'])


def downgrade() -> None:
    op.drop_index('ix_user_profiles_user_id', table_name='user_profiles')
    op.drop_table('user_profiles')
```

## Safety Checks

Before applying any migration, Claude will verify:
- [ ] Migration has a valid `downgrade()` function
- [ ] No destructive operations on columns with data (unless `--force`)
- [ ] Foreign key constraints are consistent
- [ ] Indexes are not duplicated
- [ ] Migration chain is unbroken (no missing revisions)

## Flags

| Flag | Description |
|------|-------------|
| `--generate` | Detect drift and create new migration file |
| `--apply` | Apply all pending migrations |
| `--rollback` | Revert the last applied migration |
| `--dry-run` | Show SQL that would be executed without running it |
| `--force` | Skip destructive operation warnings |
| `--target <rev>` | Migrate up or down to a specific revision |

## Notes

- Always run `--dry-run` before applying to production
- Migrations are wrapped in transactions; failure triggers automatic rollback
- Keep migrations small and focused — one logical change per file
- Never edit an already-applied migration; create a new one instead
