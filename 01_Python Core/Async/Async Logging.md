---
type: concept
tags:
  - async
  - python
  - concept
  - asyncio
  - logging
  - observability
created: 2026-03-01
modified: 2026-03-01
status: draft
---

# Async Logging

## Overview
Python's built-in `logging` module is **synchronous** — calling it from async code is safe but will briefly block the event loop if handlers do I/O (e.g. writing to a remote log sink). For lightweight file/console logging this is fine; for high-throughput async services, use a `QueueHandler` to offload blocking I/O to a background thread.

## Key Points

- **`logging` is thread-safe but not async-native** — the standard `logging.getLogger().info(...)` works from coroutines, but `StreamHandler`/`FileHandler` write synchronously
- **`QueueHandler` + `QueueListener`** — the recommended pattern for non-blocking logging: async code enqueues log records (fast), a background thread dequeues and writes them
- **`logging.captureWarnings(True)`** — routes Python `warnings` into the logging system, including deprecation warnings from async APIs
- **Structured logging** — pass extra context via `extra={}` or use `structlog` / `python-json-logger` for JSON output in production
- **`asyncio` debug mode** — `asyncio.run(main(), debug=True)` logs slow callbacks, unawaited coroutines, and exception details via the `asyncio` logger
- **Task name in logs** — `asyncio.current_task().get_name()` available inside a coroutine; add to `LogRecord` with a custom `Filter`

## When to Use

- All async services — at minimum use `QueueHandler` so a slow log backend doesn't stall task scheduling
- Debugging concurrency issues — enable `asyncio` debug mode and watch for slow-callback warnings
- Production services — structured JSON logs with task/request IDs for tracing concurrent operations
- Catching silent task failures — log inside done callbacks (see [[Async Exceptions]])

## When NOT to Use

- **Logging inside tight loops** — even cheap string formatting adds up; use a counter + periodic summary instead
- **`print()` as a logging substitute** — not configurable, not filterable, not thread-safe in all contexts
- **Blocking HTTP/DB log handlers on the main loop** — always use `QueueHandler` + a dedicated thread for any remote sink

## Related Concepts

- [[Async Exceptions]] — done callbacks as the hook for logging task failures
- [[Event Loop]] — the single thread that logging must not block
- [[Asyncio Module]] — `asyncio` logger and debug mode
- [[Tasks vs Futures]] — accessing `task.get_name()` for log context

## Examples

### Basic Logging in Async Code
```python
import asyncio
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s %(levelname)s %(name)s: %(message)s",
)
log = logging.getLogger(__name__)

async def fetch(url: str) -> str:
    log.info("Fetching %s", url)          # safe from a coroutine
    await asyncio.sleep(0.1)              # simulated I/O
    log.debug("Done fetching %s", url)
    return f"data:{url}"

async def main():
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch(f"url-{i}")) for i in range(3)]

asyncio.run(main())
```

### Non-blocking Logging with QueueHandler (recommended for production)
```python
import asyncio
import logging
import logging.handlers
import queue

def setup_async_logging() -> None:
    """Route all log records through a thread-safe queue."""
    log_queue: queue.Queue = queue.Queue(maxsize=10_000)

    # QueueHandler — fast, non-blocking enqueue from any thread/coroutine
    queue_handler = logging.handlers.QueueHandler(log_queue)
    root = logging.getLogger()
    root.setLevel(logging.DEBUG)
    root.addHandler(queue_handler)

    # QueueListener — runs in a background thread, does the actual I/O
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(
        logging.Formatter("%(asctime)s %(levelname)s %(name)s: %(message)s")
    )
    listener = logging.handlers.QueueListener(
        log_queue, console_handler, respect_handler_level=True
    )
    listener.start()
    return listener   # caller must call listener.stop() on shutdown

async def main():
    listener = setup_async_logging()
    log = logging.getLogger(__name__)
    try:
        log.info("App started")
        await asyncio.sleep(0.5)
        log.info("App done")
    finally:
        listener.stop()   # flush remaining records before exit

asyncio.run(main())
```

### Adding Task Name to Every Log Record
```python
import asyncio
import logging

class AsyncTaskFilter(logging.Filter):
    """Inject the current asyncio task name into every LogRecord."""
    def filter(self, record: logging.LogRecord) -> bool:
        task = asyncio.current_task()
        record.task_name = task.get_name() if task else "main"
        return True

def setup_logging():
    handler = logging.StreamHandler()
    handler.setFormatter(
        logging.Formatter("%(asctime)s [%(task_name)s] %(levelname)s: %(message)s")
    )
    handler.addFilter(AsyncTaskFilter())
    logging.getLogger().addHandler(handler)
    logging.getLogger().setLevel(logging.DEBUG)

async def worker(n: int):
    log = logging.getLogger(__name__)
    log.info("Worker %d starting", n)
    await asyncio.sleep(0.1)
    log.info("Worker %d done", n)

async def main():
    setup_logging()
    async with asyncio.TaskGroup() as tg:
        for i in range(3):
            tg.create_task(worker(i), name=f"worker-{i}")

asyncio.run(main())
# output: 2026-03-01 17:00:00 [worker-0] INFO: Worker 0 starting
```

### Logging Unhandled Task Exceptions via Done Callback
```python
import asyncio
import logging

log = logging.getLogger(__name__)

def _task_error_logger(task: asyncio.Task) -> None:
    if task.cancelled():
        log.warning("Task %s was cancelled", task.get_name())
        return
    exc = task.exception()
    if exc:
        log.error(
            "Task %s raised an unhandled exception",
            task.get_name(),
            exc_info=exc,
        )

async def unreliable():
    await asyncio.sleep(0.1)
    raise RuntimeError("something broke")

async def main():
    task = asyncio.create_task(unreliable(), name="unreliable-task")
    task.add_done_callback(_task_error_logger)
    await asyncio.sleep(1)   # keep loop alive

asyncio.run(main())
```

### Enable asyncio Debug Logging
```python
import asyncio
import logging

# The 'asyncio' logger emits slow-callback warnings, unawaited coros, etc.
logging.getLogger("asyncio").setLevel(logging.DEBUG)
logging.basicConfig(level=logging.DEBUG)

async def main():
    await asyncio.sleep(0.001)

asyncio.run(main(), debug=True)
# PYTHONASYNCIODEBUG=1 python script.py  # equivalent via env var
```

## References

- [Python logging docs](https://docs.python.org/3/library/logging.html)
- [Python logging.handlers.QueueHandler](https://docs.python.org/3/library/logging.handlers.html#queuehandler)
- [Python logging cookbook — logging from multiple threads/processes](https://docs.python.org/3/howto/logging-cookbook.html)
- [asyncio debug mode](https://docs.python.org/3/library/asyncio-dev.html#debug-mode)
- [[Async Exceptions]] — logging done-callback pattern
- [[Asyncio Module]]
