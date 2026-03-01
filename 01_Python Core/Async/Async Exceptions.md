---
type: concept
tags:
  - async
  - python
  - concept
  - asyncio
  - exceptions
  - error-handling
created: 2026-03-01
modified: 2026-03-01
status: draft
---

# Async Exceptions & Error Handling

## Overview
Exception handling in async code follows the same `try/except/finally` model as synchronous Python, but there are important differences around `CancelledError`, `ExceptionGroup` (3.11+), and how unhandled exceptions in tasks are reported.

## Key Points

- **`CancelledError`** (subclass of `BaseException`) — raised inside a task when `.cancel()` is called; **must be re-raised** after cleanup or cancellation won't propagate
- **`ExceptionGroup`** (3.11+) — raised by `TaskGroup` when one or more tasks fail; holds multiple exceptions; handled with `except*` syntax
- **Unhandled task exceptions** — exceptions in `create_task()` tasks that are never awaited are swallowed silently (Python 3.12 emits a warning); always await or add a done callback
- **`asyncio.timeout()` raises `TimeoutError`** (builtin) not `asyncio.TimeoutError` (though the latter is an alias in 3.11+)
- **`finally` blocks always run** — even when a coroutine is cancelled, allowing resource cleanup
- **Task exception callbacks** — use `task.add_done_callback()` to react to failures without awaiting

## When to Use

- Wrapping `await` calls in `try/except` for specific recoverable errors (connection resets, timeouts)
- Using `except*` with `TaskGroup` to handle partial failures from concurrent tasks
- Adding `finally` blocks for guaranteed resource cleanup in long-running coroutines
- Using done callbacks for fire-and-forget tasks you still want to log errors from

## When NOT to Use

- **Silencing `CancelledError`** — never `except CancelledError: pass`; always re-raise it
- **Bare `except Exception` in tasks** — masks failures; prefer specific exception types
- **Relying on unhandled task exception warnings** — treat them as bugs; always handle errors explicitly

## Related Concepts

- [[Tasks vs Futures]] — tasks are the source of async exceptions
- [[Async Locks]] — `finally` blocks used for guaranteed lock release
- [[Async Events]] — shutdown events triggered on exception
- [[Async Logging]] — logging exceptions from async contexts
- [[Asyncio Module]] — TaskGroup, timeout, CancelledError

## Examples

### CancelledError — Always Re-raise
```python
import asyncio

async def worker():
    try:
        while True:
            await asyncio.sleep(1)
            print("working...")
    except asyncio.CancelledError:
        print("Cleaning up before cancel...")
        await cleanup()
        raise   # ← CRITICAL: re-raise so the event loop knows we're cancelled

async def main():
    task = asyncio.create_task(worker())
    await asyncio.sleep(2.5)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Task cancelled successfully")

asyncio.run(main())
```

### ExceptionGroup with `except*` (Python 3.11+)
```python
import asyncio

async def risky(n: int):
    await asyncio.sleep(0.1)
    if n % 2 == 0:
        raise ValueError(f"Even number: {n}")
    return n

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(risky(i)) for i in range(5)]
    except* ValueError as eg:
        # eg.exceptions is a tuple of all ValueError instances
        for exc in eg.exceptions:
            print(f"Caught: {exc}")

    # Successful tasks still have results
    ok = [t.result() for t in tasks if not t.cancelled() and t.exception() is None]
    print(f"Succeeded: {ok}")

asyncio.run(main())
```

### Timeout with Cleanup
```python
import asyncio

async def fetch_with_cleanup(url: str):
    resource = await acquire_resource()
    try:
        async with asyncio.timeout(5.0):
            return await http_get(url)
    except TimeoutError:
        print(f"Timed out fetching {url}")
        return None
    finally:
        await resource.close()   # always runs, even on timeout or cancel
```

### Unhandled Task Exception — Done Callback
```python
import asyncio
import logging

def _on_task_done(task: asyncio.Task) -> None:
    if task.cancelled():
        return
    exc = task.exception()
    if exc:
        logging.error("Task %s failed: %r", task.get_name(), exc, exc_info=exc)

async def fire_and_forget(coro):
    task = asyncio.create_task(coro)
    task.add_done_callback(_on_task_done)
    return task

async def main():
    await fire_and_forget(risky_operation())
    await asyncio.sleep(1)   # let it run

asyncio.run(main())
```

### gather() with return_exceptions
```python
import asyncio

async def may_fail(n: int):
    if n == 2:
        raise RuntimeError("Oops")
    return n * 10

async def main():
    results = await asyncio.gather(
        *[may_fail(i) for i in range(4)],
        return_exceptions=True   # exceptions returned as values, not raised
    )
    for r in results:
        if isinstance(r, Exception):
            print(f"Error: {r}")
        else:
            print(f"Result: {r}")

asyncio.run(main())
```

## References

- [Python asyncio exception handling docs](https://docs.python.org/3/library/asyncio-task.html#task-cancellation)
- [PEP 654 — Exception Groups and except*](https://peps.python.org/pep-0654/)
- [Python asyncio.CancelledError](https://docs.python.org/3/library/asyncio-exceptions.html)
- [[Tasks vs Futures]]
- [[Asyncio Module]]
