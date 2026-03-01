---
type: concept
tags:
  - async
  - python
  - concept
  - asyncio
  - concurrency
  - synchronisation
created: 2026-03-01
modified: 2026-03-01
status: draft
---

# Async Locks & Synchronisation Primitives

## Overview
`asyncio` provides coroutine-safe synchronisation primitives — `Lock`, `Semaphore`, `BoundedSemaphore`, and `Condition` — that allow async tasks sharing resources to coordinate access without blocking the event loop.

> [!NOTE]
> These are **not** thread-safe. For cross-thread synchronisation use `threading.Lock` + `loop.run_in_executor`.

## Key Points

- **`asyncio.Lock`** — mutual exclusion; only one task may hold the lock at a time
- **`asyncio.Semaphore(n)`** — allows up to `n` concurrent holders; ideal for rate limiting
- **`asyncio.BoundedSemaphore(n)`** — like Semaphore but raises `ValueError` on over-release
- **`asyncio.Condition`** — combines a lock with the ability to wait for a predicate
- **Always use `async with`** — ensures the lock is released even if an exception is raised
- **Locks are not reentrant** — a task that already holds the lock will deadlock if it tries to acquire it again

## When to Use

- Protecting shared mutable state accessed by multiple concurrent tasks (e.g. a shared cache, counter, or buffer)
- Rate-limiting outgoing requests with `Semaphore` (e.g. max 10 concurrent HTTP calls)
- Producer/consumer coordination where a consumer must wait until data is ready (`Condition`)
- Guarding initialisation so only one task performs setup even when many arrive simultaneously

## When NOT to Use

- **CPU-bound critical sections** — use `multiprocessing` locks instead; asyncio locks don't cross process boundaries
- **Cross-thread access** — `asyncio` primitives are not thread-safe; mixing sync threads requires `threading` primitives
- **Simple sequential code** — no shared mutable state means no locks needed
- **When `TaskGroup` already serialises access** — structured concurrency can eliminate the need for explicit locks

## Related Concepts

- [[Event Loop]] — the single thread all async tasks share
- [[Tasks vs Futures]] — the concurrent units that locks coordinate
- [[Async Events]] — simpler signalling without mutual exclusion
- [[Asyncio Module]] — the module providing these primitives

## Examples

### Basic Lock
```python
import asyncio

counter = 0
lock = asyncio.Lock()

async def increment():
    global counter
    async with lock:          # acquire → do work → release
        value = counter
        await asyncio.sleep(0)  # yield to show interleavings are safe
        counter = value + 1

async def main():
    async with asyncio.TaskGroup() as tg:
        for _ in range(5):
            tg.create_task(increment())
    print(counter)  # always 5

asyncio.run(main())
```

### Semaphore — Rate Limiting
```python
import asyncio
import aiohttp

sem = asyncio.Semaphore(5)  # max 5 concurrent requests

async def fetch(session: aiohttp.ClientSession, url: str) -> str:
    async with sem:
        async with session.get(url) as resp:
            return await resp.text()

async def main():
    urls = [f"https://example.com/{i}" for i in range(20)]
    async with aiohttp.ClientSession() as session:
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(fetch(session, url)) for url in urls]
    results = [t.result() for t in tasks]

asyncio.run(main())
```

### Condition — Wait for Predicate
```python
import asyncio

queue: list[int] = []
condition = asyncio.Condition()

async def producer():
    for i in range(3):
        await asyncio.sleep(0.5)
        async with condition:
            queue.append(i)
            condition.notify_all()   # wake waiting consumers

async def consumer(name: str):
    async with condition:
        await condition.wait_for(lambda: len(queue) > 0)
        item = queue.pop(0)
        print(f"{name} got {item}")

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(producer())
        tg.create_task(consumer("C1"))
        tg.create_task(consumer("C2"))

asyncio.run(main())
```

### Avoiding Deadlock — Non-reentrant Warning
```python
async def outer(lock: asyncio.Lock):
    async with lock:
        await inner(lock)   # ❌ deadlock — lock already held by this task

async def inner(lock: asyncio.Lock):
    async with lock:        # blocks forever waiting for itself
        ...
```

## References

- [Python asyncio synchronisation docs](https://docs.python.org/3/library/asyncio-sync.html)
- [PEP 3156 — asyncio](https://peps.python.org/pep-3156/)
- [[Asyncio Module]]
