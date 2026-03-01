---
type: concept
tags:
  - async
  - python
  - concept
  - task
  - future
created: 2026-02-15
modified: 2026-03-01
status: draft
---

# Tasks vs Futures

## Overview
Tasks and Futures are the concurrent execution primitives in asyncio. While both represent the result of an asynchronous operation, **Tasks** wrap coroutines and schedule them for execution, while **Futures** are lower-level objects representing a value that will exist in the future.

## Key Points

### Tasks
- **Wrapper around coroutines** - Schedules coroutines to run on the event loop
- **High-level** - The primary way to run things concurrently
- **Eager execution** - Once created, the coroutine starts running
- **Cancellation support** - Can be cancelled gracefully
- **Result/exception access** - Can retrieve return value or caught exception

### Futures
- **Low-level primitive** - Represents a future result
- **Manual completion** - Must be set with `set_result()` or `set_exception()`
- **Used internally** - asyncio uses Futures under the hood
- **Callback support** - Can attach callbacks that fire on completion
- **Rarely used directly** - Most code uses Tasks instead

## Creating Tasks

```python
import asyncio

async def my_coroutine():
    await asyncio.sleep(1)
    return "completed"

async def main():
    # Preferred: asyncio.create_task() — named for easier debugging
    task1 = asyncio.create_task(my_coroutine(), name="my-task")
    
    # asyncio.ensure_future() is functionally equivalent but discouraged —
    # create_task() is more explicit and always returns a Task
    # task2 = asyncio.ensure_future(my_coroutine())  # avoid
    
    # Both start executing immediately (eager)
    result1 = await task1
    print(f"Result: {result1}")

asyncio.run(main())
```

## Task States

```python
async def example():
    task = asyncio.create_task(asyncio.sleep(1))
    
    print(task.done())      # False - not finished
    print(task.cancelled()) # False - not cancelled
    
    await task
    
    print(task.done())      # True - completed
    print(task.result())    # None - return value
```

## Task Cancellation

```python
async def long_task():
    try:
        await asyncio.sleep(10)
        return "finished"
    except asyncio.CancelledError:
        print("Task was cancelled!")
        raise  # Re-raise to properly propagate cancellation

async def main():
    task = asyncio.create_task(long_task())
    await asyncio.sleep(0.1)
    
    # Cancel the task
    task.cancel()
    
    try:
        await task
    except asyncio.CancelledError:
        print("Main caught cancellation")

asyncio.run(main())
```

## Working with Multiple Tasks

```python
async def fetch_data(url, delay):
    await asyncio.sleep(delay)
    return f"Data from {url}"

async def main():
    # Preferred (3.11+): TaskGroup — structured concurrency
    # Exceptions from any task raise ExceptionGroup (not silently swallowed)
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch_data("api1.com", 1))
        t2 = tg.create_task(fetch_data("api2.com", 2))
        t3 = tg.create_task(fetch_data("api3.com", 0.5))
    results = [t1.result(), t2.result(), t3.result()]
    print(results)
    
    # gather() still valid — preferable when you need return_exceptions=True
    results = await asyncio.gather(
        fetch_data("api1.com", 1),
        fetch_data("api2.com", 2),
        return_exceptions=True
    )
    
    # Wait for first to complete
    tasks = [
        asyncio.create_task(fetch_data("api1.com", 1)),
        asyncio.create_task(fetch_data("api2.com", 2)),
    ]
    done, pending = await asyncio.wait(
        tasks, 
        return_when=asyncio.FIRST_COMPLETED
    )
    for p in pending:
        p.cancel()  # Always cancel pending tasks!

asyncio.run(main())
```

## Futures (Low-Level)

```python
async def main():
    # Create a Future manually (rarely needed)
    future = asyncio.Future()
    
    # Set the result later (simulating completion)
    future.set_result("Hello from future!")
    
    # Await the future
    result = await future
    print(result)  # Hello from future!
    
    # Check state
    print(future.done())     # True
    print(future.result())   # Hello from future!

asyncio.run(main())
```

## When to Use

### Use Tasks when:
- Running multiple coroutines concurrently
- Need to cancel operations
- Want to fire-and-forget background work
- Need to wait for multiple operations with timeouts
- Building concurrent workflows

### Use Futures when:
- Implementing low-level async primitives
- Bridging callback-based code to async/await
- Creating custom synchronization mechanisms
- Writing library code that needs manual result setting

## When NOT to Use

### Don't use Tasks when:
- You just need sequential execution - use `await` directly
- Working with CPU-bound code (use processes instead)
- The overhead of task management isn't worth it

### Don't use Futures when:
- High-level application code - Tasks are almost always better
- You can use `asyncio.gather()` or `asyncio.wait()` instead

## Task vs Future Comparison

| Feature | Task | Future |
|---------|------|--------|
| Creation | `asyncio.create_task()` | `asyncio.Future()` |
| Execution | Auto-schedules coroutine | Manual completion |
| Level | High-level | Low-level |
| Common Use | 95% of async code | Library internals |
| Cancellation | Yes | Yes |
| Callbacks | Yes | Yes |

## Related Concepts

- [[Coroutines]] - What Tasks wrap to make runnable
- [[Event Loop]] - Where Tasks are scheduled and run
- [[Async/Await Syntax]] - How to create coroutines for Tasks
- [[Gathering Tasks]] - Pattern for running multiple tasks
- [[Timeouts in Asyncio]] - How to add timeouts to tasks
- [[Cancellation]] - Graceful task termination

## Examples

### Timeout Pattern
```python
# Modern (3.11+): asyncio.timeout() context manager
async def fetch_with_timeout():
    try:
        async with asyncio.timeout(5.0):
            result = await fetch_data()
        return result
    except TimeoutError:  # built-in TimeoutError
        print("Request timed out")
        return None

# asyncio.wait_for() still works:
async def fetch_with_wait_for():
    task = asyncio.create_task(fetch_data())
    try:
        result = await asyncio.wait_for(task, timeout=5.0)
        return result
    except asyncio.TimeoutError:
        print("Request timed out")
        return None
```

### Fire and Forget
```python
async def background_logger():
    asyncio.create_task(log_to_remote())  # Don't await - runs in background
    return "Request accepted"
```

### Task Groups (Python 3.11+)
```python
async def main():
    # Preferred over gather() for most concurrent patterns
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch_data(), name="fetch-1")
        task2 = tg.create_task(fetch_data(), name="fetch-2")
        task3 = tg.create_task(fetch_data(), name="fetch-3")
    
    # All tasks complete when exiting context
    print(f"Results: {task1.result()}, {task2.result()}, {task3.result()}")

    # Exception handling with ExceptionGroup:
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(risky_task())
    except* ValueError as eg:  # except* is 3.11+ syntax
        for exc in eg.exceptions:
            print(f"Caught: {exc}")
```

## References

- [Python Task documentation](https://docs.python.org/3/library/asyncio-task.html)
- [Python Future documentation](https://docs.python.org/3/library/asyncio-future.html)
- [[Asyncio Module]] - For task creation functions
