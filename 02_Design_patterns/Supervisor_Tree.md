---
type: pattern
category: [design-pattern, async-pattern]
tags: [supervision, fault-tolerance, concurrency, erlang, asyncio]
created: 2026-02-22
modified: 2026-02-22
complexity: intermediate
---

# Supervisor Tree

## Problem

Long-running concurrent systems inevitably have components that crash. Without a structured recovery mechanism, a single failure cascades and takes down the whole application, or requires fragile `try/except` wrapped everywhere.[^1][^2]

## Solution

Separate processes into **workers** (do the work, allowed to crash) and **supervisors** (do nothing but watch workers and restart them). Arrange them in a hierarchy so failures are caught at the right level, recovered locally, and only escalate upward when they exceed acceptable crash frequency.[^3][^4]

## Structure

```python
import asyncio
import traceback
from collections import deque
from time import monotonic
from enum import Enum


# ─────────────────────────────────────────────
# Restart policies (what to do after each exit)
# ─────────────────────────────────────────────

class RestartPolicy(Enum):
    PERMANENT  = "permanent"   # always restart (normal or crash)
    TRANSIENT  = "transient"   # restart only after crash
    TEMPORARY  = "temporary"   # never restart


# ─────────────────────────────────────────────
# Worker: does actual business logic
# Supervisors don't care *what* it does —
# only whether it lives or dies.
# ─────────────────────────────────────────────

async def db_worker():
    """Example worker: could be a DB poll loop, queue consumer, etc."""
    while True:
        await asyncio.sleep(1)
        print("db_worker: processing...")
        # raise RuntimeError("simulated crash")  # uncomment to test restart

async def http_worker():
    """Example worker: could be a long-running HTTP listener."""
    while True:
        await asyncio.sleep(2)
        print("http_worker: serving...")


# ─────────────────────────────────────────────
# Supervisor: wraps a single worker coroutine.
# Applies restart policy + crash-frequency guard.
# ─────────────────────────────────────────────

async def supervisor(
    coro_factory,                           # no-arg callable returning a coroutine
    policy: RestartPolicy = RestartPolicy.PERMANENT,
    max_crashes: int = 3,                   # max allowed crashes...
    within_seconds: float = 30.0,           # ...within this window
    backoff: float = 0.5,                   # initial delay between restarts
):
    crash_times: deque = deque([float("-inf")], maxlen=max_crashes)

    while True:
        crash_times.append(monotonic())
        try:
            await coro_factory()

            # ── clean exit (return, not exception) ──
            if policy == RestartPolicy.PERMANENT:
                continue                    # restart even after clean exit
            return                          # TRANSIENT / TEMPORARY: stop

        except asyncio.CancelledError:
            raise                           # always honour external cancellation

        except Exception:
            traceback.print_exc()

            if policy == RestartPolicy.TEMPORARY:
                return                      # never restart

            # crash-frequency guard
            if min(crash_times) > monotonic() - within_seconds:
                print(f"[supervisor] Too many crashes — escalating to parent.")
                raise                       # bubble up to a parent supervisor

            print(f"[supervisor] Restarting in {backoff}s...")
            await asyncio.sleep(backoff)


# ─────────────────────────────────────────────
# Supervisor Tree: composes child supervisors.
# Strategy = one_for_one (default asyncio.TaskGroup)
# Strategy = one_for_all (TaskGroup cancels all on any failure)
# ─────────────────────────────────────────────

async def root_supervisor():
    """
    one_for_one: each child has its own supervisor wrapper.
    If db_worker exhausts retries, TaskGroup cancels http_worker too.
    That's one_for_all escalation at the root level.
    """
    async with asyncio.TaskGroup() as tg:
        tg.create_task(
            supervisor(db_worker, policy=RestartPolicy.TRANSIENT, max_crashes=3),
            name="db_worker",
        )
        tg.create_task(
            supervisor(http_worker, policy=RestartPolicy.PERMANENT),
            name="http_worker",
        )


# ─────────────────────────────────────────────
# Manager: adds a brain on top of a supervisor.
# Handles persistent faults the supervisor can't.
# (exponential backoff, circuit breaker, alerting)
# ─────────────────────────────────────────────

async def manager(supervisor_factory, max_retries: int = 5):
    delay = 1.0
    for attempt in range(1, max_retries + 1):
        try:
            await supervisor_factory()
            return                          # clean exit — done
        except* Exception as eg:           # Python 3.11+ ExceptionGroup
            print(f"[manager] Attempt {attempt}/{max_retries} failed: {eg}")
            if attempt == max_retries:
                raise
            await asyncio.sleep(delay)
            delay = min(delay * 2, 60)     # exponential backoff, cap at 60s


# ─────────────────────────────────────────────
# Entry point
# ─────────────────────────────────────────────

asyncio.run(manager(root_supervisor))
```


## Key Design Rules

- **Fragile things go deep** — unstable workers sit at leaf nodes; stable infrastructure sits near the root[^3]
- **Supervisors stay dumb** — the `supervisor()` function has no business logic; that's the worker's job[^2]
- **Manager adds the brain** — exponential backoff, circuit breaking, alerting all live in `manager()`, not in the supervisor[^3]
- **`CancelledError` always re-raises** — never swallow it; it's how `TaskGroup` propagates cancellation through the tree[^5]
- **`except*` on Python 3.11+** — `TaskGroup` wraps exceptions in `ExceptionGroup`, so the manager uses `except*` to unpack them[^5]


## Links & Resources

### Origin & Theory

- [Erlang/OTP Design Principles — Supervision Trees](https://www.erlang.org/doc/system/design_principles.html) — the original source of the pattern; concise and worth reading even if you never write Erlang[^1]
- [Erlang `supervisor` behaviour reference](https://www.erlang.org/doc/apps/stdlib/supervisor.html) — full spec for restart strategies, child specs, and intensity/period settings[^6]
- [Adopting Erlang — Supervision Trees](https://adoptingerlang.org/docs/development/supervision_trees/) — the best practical guide to *designing* a supervision tree; language-agnostic concepts[^3]
- [Learn You Some Erlang — Building OTP Applications](https://learnyousomeerlang.com/building-otp-applications) — friendly deep-dive into OTP; great for understanding the philosophy[^7]


### Python / asyncio

- [Python docs — `asyncio` Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html) — covers `create_task`, `TaskGroup`, `CancelledError`, and `ExceptionGroup` handling[^5]
- [Python docs — `asyncio.TaskGroup`](https://docs.python.org/3.11/library/asyncio-task.html) — introduced in Python 3.11; the closest built-in to an OTP supervisor strategy[^8]
- [taskgroup on PyPI](https://pypi.org/project/taskgroup/) — backport of `asyncio.TaskGroup` for Python < 3.11[^9]
- [Stack Overflow — asyncio restart coroutine](https://stackoverflow.com/questions/63844557/python-asyncio-restart-coroutine) — practical patterns for wrapping coroutines with restart logic[^10]


### Python Fault-Tolerance Libraries

- [**hyx**](https://github.com/roma-glushko/hyx) — lightweight asyncio-native resiliency primitives (retry, circuit breaker, bulkhead, timeout, fallback) for Python 3.9+ microservices[^11]
- [**asy**](https://github.com/sasano8/asy) — minimal asyncio supervisor library with `supervise()` API inspired directly by OTP[^12]
<span style="display:none">[^13][^14][^15][^16][^17]</span>

<div align="center">⁂</div>
