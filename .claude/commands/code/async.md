# Async Patterns Command

Convert synchronous code to async/await patterns or improve existing async code.

## Usage

```
/code/async [file or code snippet]
```

## What This Does

1. Identifies blocking synchronous operations
2. Converts to async/await patterns
3. Adds proper error handling for async contexts
4. Implements concurrent execution where beneficial
5. Avoids common async pitfalls (forgotten awaits, blocking the event loop)

## Examples

### Before: Synchronous HTTP calls

```python
import requests

def fetch_user(user_id: int) -> dict:
    response = requests.get(f"https://api.example.com/users/{user_id}")
    response.raise_for_status()
    return response.json()

def fetch_all_users(user_ids: list[int]) -> list[dict]:
    return [fetch_user(uid) for uid in user_ids]
```

### After: Async with concurrent execution

```python
import asyncio
import aiohttp
from typing import Any

async def fetch_user(session: aiohttp.ClientSession, user_id: int) -> dict[str, Any]:
    """Fetch a single user by ID using an existing aiohttp session."""
    async with session.get(f"https://api.example.com/users/{user_id}") as response:
        response.raise_for_status()
        return await response.json()

async def fetch_all_users(user_ids: list[int]) -> list[dict[str, Any]]:
    """Fetch multiple users concurrently."""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_user(session, uid) for uid in user_ids]
        return await asyncio.gather(*tasks, return_exceptions=False)
```

### Async context manager pattern

```python
import asyncio
from contextlib import asynccontextmanager
from typing import AsyncGenerator

class AsyncDatabasePool:
    """Async database connection pool."""

    def __init__(self, dsn: str, max_connections: int = 10):
        self.dsn = dsn
        self.max_connections = max_connections
        self._pool = None

    async def connect(self) -> None:
        """Initialize the connection pool."""
        import asyncpg
        self._pool = await asyncpg.create_pool(
            self.dsn,
            max_size=self.max_connections
        )

    async def disconnect(self) -> None:
        """Close all connections in the pool."""
        if self._pool:
            await self._pool.close()

    @asynccontextmanager
    async def acquire(self) -> AsyncGenerator:
        """Acquire a connection from the pool."""
        async with self._pool.acquire() as connection:
            yield connection

async def get_user_by_email(pool: AsyncDatabasePool, email: str) -> dict | None:
    """Look up a user record by email address."""
    async with pool.acquire() as conn:
        row = await conn.fetchrow(
            "SELECT id, name, email, created_at FROM users WHERE email = $1",
            email
        )
        return dict(row) if row else None
```

### Background task pattern

```python
import asyncio
from collections.abc import Callable, Coroutine
from typing import Any

class BackgroundTaskManager:
    """Manage fire-and-forget background tasks safely."""

    def __init__(self):
        self._tasks: set[asyncio.Task] = set()

    def spawn(self, coro: Coroutine) -> asyncio.Task:
        """Schedule a coroutine as a background task."""
        task = asyncio.create_task(coro)
        self._tasks.add(task)
        task.add_done_callback(self._tasks.discard)
        return task

    async def wait_all(self) -> None:
        """Wait for all pending background tasks to complete."""
        if self._tasks:
            await asyncio.gather(*self._tasks, return_exceptions=True)
```

## Guidelines Applied

- Use `asyncio.gather` for concurrent independent tasks
- Prefer `async with` for resource management
- Avoid `asyncio.get_event_loop()` — use `asyncio.run()` at top level
- Use `asyncio.create_task` to avoid blocking on coroutines
- Keep CPU-bound work in `loop.run_in_executor` to avoid blocking the event loop
- Type-hint coroutines with `Coroutine` or `Awaitable` from `collections.abc`
