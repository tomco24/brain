---
type: concept
tags:
  - async
  - python
  - concept
  - syntax
created: 2026-02-15
modified: 2026-03-01
status: draft
---

# Async/Await Syntax

## Overview
The `async`/`await` syntax in Python 3.5+ provides a clean, readable way to write asynchronous code. It allows functions to pause execution at `await` points, yielding control to the event loop until the awaited operation completes.

## Key Points

- **`async def`** - Declares an asynchronous function (coroutine)
- **`await`** - Pauses coroutine execution until the awaited task completes
- **`await` is only valid inside `async def`** - Cannot use await in regular functions
- **Non-blocking** - While awaiting, other tasks can run on the event loop
- **Sequential appearance, concurrent execution** - Code looks linear but executes asynchronously

## Syntax

```python
import asyncio

async def fetch_data():
    # async def creates a coroutine
    print("Starting fetch...")
    await asyncio.sleep(1)  # await pauses here, yields to event loop
    print("Fetch complete!")
    return "data"

async def main():
    # You can await coroutines
    result = await fetch_data()
    print(f"Got: {result}")

# Run the async program
asyncio.run(main())
```

## When to Use

- I/O-bound operations (network requests, file I/O, database queries)
- When you need to perform multiple operations concurrently
- API clients and web servers handling many simultaneous connections
- Any situation where you'd otherwise use threads for I/O parallelism

## When NOT to Use

- **CPU-bound tasks** - Use multiprocessing instead (async doesn't parallelize CPU work)
- Simple scripts with sequential I/O - Async adds complexity without benefit
- When working with blocking libraries that don't support async
- Performance-critical tight loops - Async overhead may matter

## Common Mistakes

```python
# WRONG: Calling async function without await
coro = fetch_data()  # Just creates coroutine object, doesn't run it

# WRONG: Using await outside async def
def regular_function():
    await fetch_data()  # SyntaxError!

# CORRECT: Proper await usage
async def correct():
    result = await fetch_data()  # Actually runs the coroutine
```

## Related Concepts

- [[Coroutines]] - What `async def` actually creates
- [[Event Loop]] - The mechanism that makes await work
- [[Asyncio Module]] - The standard library providing these features
- [[Tasks vs Futures]] - How to run multiple async operations concurrently

## Examples

### Sequential vs Concurrent
```python
import asyncio

async def task(name, duration):
    print(f"Task {name} starting")
    await asyncio.sleep(duration)
    print(f"Task {name} done")

# Sequential (slow)
async def sequential():
    await task("A", 1)
    await task("B", 1)  # Starts after A finishes
    # Total: 2 seconds

# Concurrent — preferred (3.11+): TaskGroup
async def concurrent_taskgroup():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(task("A", 1))
        tg.create_task(task("B", 1))
    # Total: 1 second
    # Any exception raises ExceptionGroup (not swallowed)

# Concurrent — gather() still valid, useful for return_exceptions
async def concurrent_gather():
    await asyncio.gather(
        task("A", 1),
        task("B", 1)
    )
    # Total: 1 second
```

### Error Handling
```python
async def risky_operation():
    try:
        result = await fetch_data()
    except ConnectionError:
        print("Failed to fetch")
        result = None
    return result
```

## References

- [Python asyncio documentation](https://docs.python.org/3/library/asyncio.html)
- [PEP 492 - Coroutines with async and await syntax](https://peps.python.org/pep-0492/)
- [[Literature Note - AsyncIO Book]]
