# Cache Management

Analyze and implement caching strategies for the specified code.

## Usage

```
/code/cache [target] [strategy]
```

## What This Does

1. Identifies cacheable operations (expensive computations, repeated DB queries, API calls)
2. Recommends appropriate caching strategy (in-memory, Redis, file-based, CDN)
3. Implements cache with TTL, invalidation, and eviction policies
4. Adds cache hit/miss metrics
5. Handles cache stampede and cold-start scenarios

## Caching Strategies

- **in-memory**: `functools.lru_cache`, `cachetools` — fast, process-local
- **redis**: distributed, TTL support, pub/sub invalidation
- **file**: disk-based, survives restarts, suitable for large objects
- **cdn**: edge caching for HTTP responses

## Example

### Before

```python
def get_product_catalog(category: str) -> list[dict]:
    """Fetch all products for a category from the database."""
    with get_connection() as conn:
        rows = conn.execute(
            "SELECT * FROM products WHERE category = ? AND active = 1",
            (category,)
        ).fetchall()
    return [dict(row) for row in rows]


def get_exchange_rate(base: str, target: str) -> float:
    """Fetch live exchange rate from external API."""
    response = requests.get(
        f"https://api.exchangerate.host/convert",
        params={"from": base, "to": target},
        timeout=5,
    )
    response.raise_for_status()
    return response.json()["result"]
```

### After

```python
import time
import threading
from functools import wraps
from typing import Any, Callable


# ---------------------------------------------------------------------------
# Simple TTL cache decorator (no external dependencies)
# ---------------------------------------------------------------------------

_cache_store: dict[str, tuple[Any, float]] = {}
_cache_lock = threading.Lock()


def ttl_cache(ttl_seconds: int = 300):
    """Cache the return value of a function for *ttl_seconds* seconds.

    Thread-safe. Keys are built from the function name and all arguments.
    """
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = f"{func.__qualname__}:{args}:{sorted(kwargs.items())}"
            now = time.monotonic()

            with _cache_lock:
                if key in _cache_store:
                    value, expires_at = _cache_store[key]
                    if now < expires_at:
                        return value  # cache hit

            # cache miss — compute outside the lock to avoid blocking
            result = func(*args, **kwargs)

            with _cache_lock:
                _cache_store[key] = (result, now + ttl_seconds)

            return result
        return wrapper
    return decorator


def invalidate_cache(prefix: str) -> int:
    """Remove all cache entries whose key starts with *prefix*.

    Returns the number of entries removed.
    """
    with _cache_lock:
        keys_to_delete = [k for k in _cache_store if k.startswith(prefix)]
        for k in keys_to_delete:
            del _cache_store[k]
    return len(keys_to_delete)


# ---------------------------------------------------------------------------
# Cached versions of the original functions
# ---------------------------------------------------------------------------

@ttl_cache(ttl_seconds=60)          # catalog refreshes every minute
def get_product_catalog(category: str) -> list[dict]:
    """Fetch all products for a category from the database (cached)."""
    with get_connection() as conn:
        rows = conn.execute(
            "SELECT * FROM products WHERE category = ? AND active = 1",
            (category,)
        ).fetchall()
    return [dict(row) for row in rows]


@ttl_cache(ttl_seconds=30)          # exchange rates change frequently
def get_exchange_rate(base: str, target: str) -> float:
    """Fetch live exchange rate from external API (cached for 30 s)."""
    response = requests.get(
        "https://api.exchangerate.host/convert",
        params={"from": base, "to": target},
        timeout=5,
    )
    response.raise_for_status()
    return response.json()["result"]
```

## Cache Invalidation Hooks

Call `invalidate_cache(prefix)` after mutations so stale data is never served:

```python
def update_product(product_id: int, data: dict) -> None:
    with get_connection() as conn:
        conn.execute("UPDATE products SET ... WHERE id = ?", (product_id,))
    # Bust catalog cache for every category — simple and safe
    invalidate_cache("get_product_catalog:")
```

## Notes

- Prefer `functools.lru_cache` for pure functions with hashable args and no TTL requirement.
- Use Redis (`redis-py`) when the cache must be shared across multiple processes or hosts.
- Always set a TTL — unbounded caches are a memory leak waiting to happen.
- Log cache hit rates in production to validate that caching is actually helping.
