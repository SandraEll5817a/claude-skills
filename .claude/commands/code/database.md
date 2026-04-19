# Database Command

Analyze and improve database interactions in the specified code.

## Usage
```
/code/database [file_or_directory]
```

## What This Does

1. **Query Analysis** - Identify N+1 queries, missing indexes, and inefficient joins
2. **ORM Usage** - Review ORM patterns and suggest optimizations
3. **Connection Management** - Check connection pooling and resource cleanup
4. **Migration Safety** - Validate schema changes for backward compatibility
5. **Transaction Handling** - Ensure proper transaction boundaries and rollback logic

## Checklist

- [ ] No raw SQL with string interpolation (use parameterized queries)
- [ ] Indexes defined for frequently queried columns
- [ ] Lazy vs eager loading used appropriately
- [ ] Connections properly closed / returned to pool
- [ ] Transactions used for multi-step writes
- [ ] Bulk operations used where applicable
- [ ] Sensitive data not logged in queries

## Example — Before

```python
import sqlite3

def get_user_orders(user_id):
    conn = sqlite3.connect("app.db")
    cursor = conn.cursor()
    # Vulnerable to SQL injection, no connection cleanup
    cursor.execute(f"SELECT * FROM orders WHERE user_id = {user_id}")
    orders = cursor.fetchall()
    # N+1: fetching items per order in a loop
    result = []
    for order in orders:
        cursor.execute(f"SELECT * FROM order_items WHERE order_id = {order[0]}")
        items = cursor.fetchall()
        result.append({"order": order, "items": items})
    return result
```

## Example — After

```python
import sqlite3
from contextlib import contextmanager
from typing import Generator

DB_PATH = "app.db"

@contextmanager
def get_connection() -> Generator[sqlite3.Connection, None, None]:
    """Context manager for safe database connection handling."""
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row  # enables dict-like access
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

def get_user_orders(user_id: int) -> list[dict]:
    """
    Fetch all orders and their items for a given user.

    Uses a single JOIN query to avoid N+1 problem.
    Parameterized query prevents SQL injection.
    """
    query = """
        SELECT
            o.id        AS order_id,
            o.created_at,
            o.status,
            oi.id       AS item_id,
            oi.product_id,
            oi.quantity,
            oi.unit_price
        FROM orders o
        LEFT JOIN order_items oi ON oi.order_id = o.id
        WHERE o.user_id = ?
        ORDER BY o.created_at DESC
    """
    with get_connection() as conn:
        rows = conn.execute(query, (user_id,)).fetchall()

    # Group items under their parent order
    orders: dict[int, dict] = {}
    for row in rows:
        oid = row["order_id"]
        if oid not in orders:
            orders[oid] = {
                "id": oid,
                "created_at": row["created_at"],
                "status": row["status"],
                "items": [],
            }
        if row["item_id"] is not None:
            orders[oid]["items"].append({
                "id": row["item_id"],
                "product_id": row["product_id"],
                "quantity": row["quantity"],
                "unit_price": row["unit_price"],
            })

    return list(orders.values())
```

## Key Improvements

| Issue | Fix |
|---|---|
| SQL injection via f-string | Parameterized query with `?` placeholder |
| Unclosed connection | `contextmanager` with `finally` block |
| N+1 queries | Single `JOIN` query |
| No transaction rollback | `try/except/rollback` in context manager |
| Raw tuple access | `row_factory = sqlite3.Row` for named columns |
