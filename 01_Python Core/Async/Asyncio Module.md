---
type: concept
tags:
  - async
  - python
  - concept
  - asyncio
  - standard-library
created: 2026-02-15
modified: 2026-03-01
status: draft
---

# Asyncio Module

## Overview
The `asyncio` module is Python's standard library for writing concurrent code using the async/await syntax. It provides the event loop, coroutines, tasks, synchronization primitives, and utilities needed for asynchronous I/O and concurrent programming.

## Key Points

- **Standard library** - Built into Python 3.4+ (async/await in 3.5+)
- **Single-threaded concurrency** - Uses cooperative multitasking, not threads
- **Event loop based** - Central loop schedules and runs all async operations
- **I/O focused** - Optimized for I/O-bound, not CPU-bound, operations
- **Growing ecosystem** - Many third-party libraries built on asyncio

## Core Components

### 1. Event Loop Management
```python
import asyncio

# Modern way - creates, runs, and closes loop
asyncio.run(main())

# Within async context, get the running loop
loop = asyncio.get_running_loop()  # Preferred over get_event_loop()

# DEPRECATED: asyncio.get_event_loop() — raises DeprecationWarning in 3.12
# when there is no current event loop
```

### 2. Running Coroutines
```python
# Run a coroutine
asyncio.run(my_coroutine())

# Create and schedule a task (give it a name for easier debugging)
task = asyncio.create_task(my_coroutine(), name="my-task")

# Run multiple coroutines concurrently — preferred in 3.11+
async with asyncio.TaskGroup() as tg:
    t1 = tg.create_task(coro1())
    t2 = tg.create_task(coro2())
# results available via t1.result(), t2.result()

# gather() still works but TaskGroup gives better error propagation
results = await asyncio.gather(coro1(), coro2(), coro3())
```

### 3. Timeouts
```python
# Modern: asyncio.timeout() context manager (Python 3.11+)
try:
    async with asyncio.timeout(5.0):
        result = await slow_operation()
except TimeoutError:  # Note: built-in TimeoutError, not asyncio.TimeoutError
    print("Operation timed out")

# asyncio.wait_for() still works (raises asyncio.TimeoutError)
try:
    result = await asyncio.wait_for(slow_operation(), timeout=5.0)
except asyncio.TimeoutError:
    print("Operation timed out")

# Wait with timeout, returns done/pending sets
done, pending = await asyncio.wait(tasks, timeout=10.0)
```

### 4. Sleeping (Non-blocking delay)
```python
# Pause without blocking other tasks
await asyncio.sleep(1)  # Sleep for 1 second

# Compare to time.sleep(1) which blocks the entire thread!
```

### 5. Gathering Results
```python
# Preferred in 3.11+: TaskGroup — structured concurrency with proper error propagation
async with asyncio.TaskGroup() as tg:
    t1 = tg.create_task(fetch_user(1))
    t2 = tg.create_task(fetch_user(2))
    t3 = tg.create_task(fetch_user(3))
# All tasks complete on exit; any exception raises ExceptionGroup
results = [t1.result(), t2.result(), t3.result()]

# gather() still valid — useful when you need return_exceptions=True
results = await asyncio.gather(
    fetch_user(1),
    fetch_user(2),
    fetch_user(3)
)
# results = [user1_data, user2_data, user3_data]

# Return_exceptions=True prevents one failure from stopping others
results = await asyncio.gather(
    risky_op(),
    risky_op(),
    return_exceptions=True
)
```

## When to Use

- **Web servers** - Handling many concurrent connections (FastAPI, Sanic)
- **API clients** - Making multiple HTTP requests concurrently
- **Database operations** - Async database drivers (asyncpg, motor)
- **Web scraping** - Fetching pages concurrently
- **Real-time applications** - WebSockets, chat servers
- **Microservices** - High-throughput I/O-bound services

## When NOT to Use

- **CPU-bound tasks** - Use `multiprocessing` instead (asyncio is single-threaded)
- **Simple scripts** - Added complexity not worth it for sequential operations
- **Blocking libraries** - Can't mix asyncio with blocking code easily
- **True parallelism needs** - asyncio is concurrent, not parallel
- **Performance-critical loops** - Event loop overhead matters in tight loops

## Common Patterns

### Basic Structure
```python
import asyncio

async def main():
    # Your async code here
    result = await some_async_operation()
    print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

### Concurrent Execution
```python
# Preferred (3.11+): TaskGroup — structured, exception-safe
async def fetch_all_urls(urls):
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch(url)) for url in urls]
    return [t.result() for t in tasks]

# Alternative: gather() — still valid
async def fetch_all_urls_gather(urls):
    return await asyncio.gather(*[fetch(url) for url in urls])
```

### Semaphore for Rate Limiting
```python
# Limit concurrent operations
sem = asyncio.Semaphore(10)  # Max 10 concurrent

async def limited_fetch(url):
    async with sem:
        return await fetch(url)
```

## Asyncio Submodules

| Submodule | Purpose |
|-----------|---------|
| `asyncio.streams` | TCP/Unix socket handling |
| `asyncio.subprocess` | Run subprocesses asynchronously |
| `asyncio.queues` | Async-safe queues |
| `asyncio.locks` | Locks, events, conditions |
| `asyncio.transports` | Low-level transport layer |

## Comparison with Alternatives

| Approach | Best For | Scaling |
|----------|----------|---------|
| asyncio | I/O-bound, many connections | Thousands of concurrent ops |
| threading | I/O-bound, blocking libraries | Limited by GIL (~hundreds) |
| multiprocessing | CPU-bound | Limited by cores/memory |

## Common Mistakes

```python
# WRONG: Blocking the event loop
def sync_function():
    time.sleep(1)  # Blocks everything!

# RIGHT: Use asyncio.sleep
async def async_function():
    await asyncio.sleep(1)  # Allows other tasks to run

# WRONG: Calling async function without await
coro = async_func()  # Just creates coroutine

# RIGHT: Actually await it
result = await async_func()
```

## Related Concepts

- [[Event Loop]] - The core mechanism asyncio uses
- [[Coroutines]] - The building blocks asyncio runs
- [[Async/Await Syntax]] - The language feature asyncio enables
- [[Tasks vs Futures]] - How asyncio manages concurrent execution
- [[Async Locks]] - Lock, Semaphore, Condition for coordinating tasks
- [[Async Events]] - asyncio.Event for signalling between tasks
- [[Async Exceptions]] - CancelledError, ExceptionGroup, error handling patterns
- [[Async Logging]] - Non-blocking logging from async code
- [[Async Context Managers]] - Resource management in async
- [[FastAPI Async Routes]] - Framework built on asyncio

## Examples

### Echo Server
```python
async def handle_client(reader, writer):
    while True:
        data = await reader.read(100)
        if not data:
            break
        writer.write(data)
        await writer.drain()
    writer.close()

async def main():
    server = await asyncio.start_server(
        handle_client, '127.0.0.1', 8888
    )
    async with server:
        await server.serve_forever()

asyncio.run(main())
```

### Concurrent API Calls
```python
async def fetch_user(user_id):
    await asyncio.sleep(0.1)  # Simulated API call
    return {"id": user_id, "name": f"User {user_id}"}

async def main():
    user_ids = [1, 2, 3, 4, 5]
    
    # Preferred (3.11+): TaskGroup
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch_user(uid)) for uid in user_ids]
    users = [t.result() for t in tasks]
    
    for user in users:
        print(user)

asyncio.run(main())
```

### Queue-based Worker
```python
async def producer(queue):
    for i in range(5):
        await queue.put(i)
        await asyncio.sleep(0.1)
    await queue.put(None)  # Signal done

async def consumer(queue):
    while True:
        item = await queue.get()
        if item is None:
            break
        print(f"Processed: {item}")

async def main():
    queue = asyncio.Queue()
    async with asyncio.TaskGroup() as tg:
        tg.create_task(producer(queue))
        tg.create_task(consumer(queue))

asyncio.run(main())
```

## References

- [Python asyncio documentation](https://docs.python.org/3/library/asyncio.html)
- [asyncio Cheat Sheet](https://superfastpython.com/asyncio-cheatsheet/)
- [Real Python asyncio guide](https://realpython.com/async-io-python/)
- [[Literature Note - AsyncIO Book]]
