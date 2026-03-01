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

# Async Events

## Overview
`asyncio.Event` is a simple coroutine-safe signalling flag: one task sets it, any number of waiting tasks are unblocked. Unlike a `Lock`, it carries no ownership — any task can set or clear it.

## Key Points

- **`event.set()`** — marks the event as set; wakes all tasks currently `await`ing it
- **`event.clear()`** — resets the flag; future `await event.wait()` calls will block again
- **`event.is_set()`** — non-blocking check of the current flag state
- **`await event.wait()`** — suspends the caller until the event is set (returns immediately if already set)
- **Broadcast semantics** — all waiters are released simultaneously when `set()` is called; contrast with `Condition.notify()` which can wake a single waiter
- **One-shot or reusable** — can be re-used by calling `clear()` after each signal cycle

## When to Use

- **Startup signals** — one task signals readiness before others proceed (e.g. server started, DB connected)
- **Cancellation flags** — a shutdown event that multiple workers monitor
- **Fan-out notification** — notify many consumers that new data is available at once
- **Coordination without shared state** — pure signalling between tasks with no resource to protect

## When NOT to Use

- **Mutual exclusion** — use `asyncio.Lock` when you need only one task in a section at a time
- **Producer/consumer with queues** — `asyncio.Queue` is a better fit when data must be passed alongside the signal
- **Rate limiting** — use `asyncio.Semaphore` instead
- **When the waiter needs to know *what* changed** — use `asyncio.Condition` to combine signal + data check

## Related Concepts

- [[Async Locks]] — mutual exclusion, ownership semantics
- [[Tasks vs Futures]] — the concurrent units that await events
- [[Asyncio Module]] — module providing `asyncio.Event`
- [[Event Loop]] — dispatches wake-ups when an event is set

## Examples

### Basic Start Signal
```python
import asyncio

ready = asyncio.Event()

async def server():
    print("Server starting up...")
    await asyncio.sleep(1)           # simulate startup work
    ready.set()                      # broadcast: we're live
    print("Server ready")

async def client(name: str):
    print(f"{name}: waiting for server")
    await ready.wait()               # blocks until set()
    print(f"{name}: connected!")

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(server())
        tg.create_task(client("C1"))
        tg.create_task(client("C2"))
        tg.create_task(client("C3"))

asyncio.run(main())
```

### Reusable Cycle (set → clear → set …)
```python
import asyncio

tick = asyncio.Event()

async def clock():
    for _ in range(3):
        await asyncio.sleep(1)
        tick.set()
        tick.clear()   # reset for next cycle

async def worker(name: str):
    for _ in range(3):
        await tick.wait()
        print(f"{name} tick!")

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(clock())
        tg.create_task(worker("W1"))
        tg.create_task(worker("W2"))

asyncio.run(main())
```

### Shutdown Flag Pattern
```python
import asyncio

shutdown = asyncio.Event()

async def worker():
    while not shutdown.is_set():
        print("Working...")
        try:
            await asyncio.wait_for(shutdown.wait(), timeout=0.5)
        except TimeoutError:
            pass   # continue working
    print("Worker stopped cleanly")

async def main():
    task = asyncio.create_task(worker())
    await asyncio.sleep(2)
    shutdown.set()     # signal all workers to stop
    await task

asyncio.run(main())
```

## References

- [Python asyncio.Event docs](https://docs.python.org/3/library/asyncio-sync.html#asyncio.Event)
- [[Async Locks]] — related primitives
- [[Asyncio Module]]
