# Monitor

Analyze and set up monitoring, logging, and observability for the codebase.

## Usage

```
/code:monitor [target]
```

## What This Does

1. **Reviews existing logging** — checks current log coverage and quality
2. **Identifies blind spots** — finds code paths with no observability
3. **Suggests metrics** — recommends key performance indicators to track
4. **Adds structured logging** — improves log format and context
5. **Health check endpoints** — adds or improves health/readiness probes

## Example

Given this service:

```python
import time
import logging
from functools import wraps
from typing import Callable, Any

logger = logging.getLogger(__name__)

def timed(func: Callable) -> Callable:
    """Decorator to measure and log function execution time."""
    @wraps(func)
    def wrapper(*args, **kwargs) -> Any:
        start = time.perf_counter()
        try:
            result = func(*args, **kwargs)
            elapsed = (time.perf_counter() - start) * 1000
            logger.info(
                "function_completed",
                extra={
                    "function": func.__name__,
                    "duration_ms": round(elapsed, 2),
                    "status": "success",
                },
            )
            return result
        except Exception as exc:
            elapsed = (time.perf_counter() - start) * 1000
            logger.error(
                "function_failed",
                extra={
                    "function": func.__name__,
                    "duration_ms": round(elapsed, 2),
                    "status": "error",
                    "error_type": type(exc).__name__,
                    "error_message": str(exc),
                },
            )
            raise
    return wrapper


class HealthCheck:
    """Simple health check aggregator."""

    def __init__(self):
        self._checks: dict[str, Callable[[], bool]] = {}

    def register(self, name: str, check_fn: Callable[[], bool]) -> None:
        """Register a named health check function."""
        self._checks[name] = check_fn

    def run(self) -> dict[str, Any]:
        """Run all checks and return aggregated status."""
        results = {}
        overall = True
        for name, fn in self._checks.items():
            try:
                passed = fn()
            except Exception as exc:
                passed = False
                logger.warning("health_check_error", extra={"check": name, "error": str(exc)})
            results[name] = "ok" if passed else "fail"
            if not passed:
                overall = False
        return {"status": "ok" if overall else "degraded", "checks": results}
```

The command will:
- Confirm structured logging is present and consistent
- Suggest adding a `request_id` or `trace_id` to log context
- Recommend exposing `HealthCheck.run()` via a `/healthz` HTTP endpoint
- Propose counters for error rates and latency histograms (e.g., Prometheus)
- Flag any background tasks or DB calls lacking `@timed` coverage

## Output

- Summary of current observability gaps
- Patched code with improved logging and metrics hooks
- Example health check endpoint integration
- Recommended alerting thresholds where applicable
