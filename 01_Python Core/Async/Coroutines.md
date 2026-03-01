---
type: concept
tags:
  - async
  - python
  - concept
  - coroutine
created: 2026-02-15
modified: 2026-03-01
status: draft
---

# Coroutines

## Overview
Coroutines are special functions that can suspend and resume their execution. In Python, coroutines are created with `async def` and are the fundamental building blocks of asynchronous programming - they represent operations that can be paused without blocking the entire program.

## Key Points

- **Suspendable functions** - Can pause at `await` points and resume later
- **Lazy evaluation** - Calling an async function returns a coroutine object; it doesn't execute immediately
- **Event loop integration** - Coroutines must be scheduled on an event loop to run
- **State machines** - Under the hood, Python transforms coroutines into state machines
- **First-class objects** - Can be passed around, stored, and manipulated like any object

## Creating and Running Coroutines

```python
import asyncio

# Define a coroutine with async def
async def my_coroutine():
    print("Starting")
    await asyncio.sleep(1)  # Suspends here
    print("Resuming")
    return "done"

# Calling creates a coroutine object (doesn't execute!)
coro = my_coroutine()
print(type(coro))  # <class 'coroutine'>

# To actually run it, you need an event loop
asyncio.run(coro)  # Runs the coroutine to completion
```

## Coroutine Lifecycle

1. **Created** - When you call `async def` function
2. **Scheduled** - Added to event loop (via `await`, `asyncio.create_task()`, etc.)
3. **Running** - Actively executing until hitting an `await`
4. **Suspended** - Paused at `await`, waiting for something to complete
5. **Resumed** - Event loop wakes it up when awaited task completes
6. **Completed** - Returns a value or raises an exception

## Coroutine Objects vs Functions

```python
async def example():
    return "hello"

# This is the function
print(example)  # <function example at 0x...>

# This is a coroutine object
coro = example()
print(coro)     # <coroutine object example at 0x...>

# Running it again creates a NEW coroutine object
coro2 = example()
print(coro is coro2)  # False - different objects
```

## When to Use

- Representing asynchronous operations (I/O, network calls)
- Building concurrent workflows that need to wait for resources
- Creating reusable async components
- Implementing async generators with `async for`
- Any time you need to "pause" execution without blocking threads

## When NOT to Use

- **Immediate execution needs** - Coroutines are lazy; use regular functions for synchronous work
- **Simple data transformations** - Overhead not worth it for CPU-only operations
- **When you need true parallelism** - Coroutines are concurrent, not parallel (use multiprocessing)
- **In non-async contexts** - Can't easily mix with blocking code

## Inspecting Coroutines

```python
import inspect

async def sample():
    pass

coro = sample()

# Check if something is a coroutine
print(inspect.iscoroutine(coro))        # True
print(inspect.iscoroutinefunction(sample))  # True

# Check coroutine state
print(coro.cr_running)  # False (not currently executing)
print(coro.cr_await)    # None (not awaiting anything yet)
```

## Related Concepts

- [[Async/Await Syntax]] - How to define and use coroutines
- [[Event Loop]] - The scheduler that runs coroutines
- [[Tasks vs Futures]] - How coroutines become runnable tasks
- [[Asyncio Module]] - The library providing coroutine support
- [[Async Generators]] - Coroutines that yield values with `async for`

## Examples

### Basic Coroutine Chain
```python
async def step1():
    await asyncio.sleep(0.1)
    return "step1 result"

async def step2(data):
    await asyncio.sleep(0.1)
    return f"step2 processed: {data}"

async def main():
    # Coroutines can await other coroutines
    result1 = await step1()
    result2 = await step2(result1)
    print(result2)

asyncio.run(main())
```

### Storing Coroutines for Later
```python
async def task(name, delay):
    await asyncio.sleep(delay)
    print(f"{name} done")

async def main():
    # Create coroutines without running them
    coros = [
        ("A", 1),
        ("B", 2),
        ("C", 0.5)
    ]
    
    # Preferred (3.11+): TaskGroup — structured concurrency
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(task(name, delay)) for name, delay in coros]
    # All tasks done here; exceptions raise ExceptionGroup

asyncio.run(main())
```

### Coroutine with Exception
```python
async def failing_coro():
    await asyncio.sleep(0.1)
    raise ValueError("Something went wrong")

async def main():
    try:
        await failing_coro()
    except ValueError as e:
        print(f"Caught: {e}")

asyncio.run(main())
```

## References

- [Python coroutine documentation](https://docs.python.org/3/library/asyncio-task.html#coroutines)
- [PEP 492 - Coroutines with async and await syntax](https://peps.python.org/pep-0492/)
- [[Asyncio Module]] - For built-in coroutine utilities
