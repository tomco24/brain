---
type: concept
tags:
  - async
  - python
  - concept
  - event-loop
  - core
created: 2026-02-15
modified: 2026-03-01
status: draft
---

# Event Loop

## Overview
The event loop is the core of Python's asynchronous programming model. It's a programming construct that waits for and dispatches events or messages in a program, enabling single-threaded concurrent execution by continuously checking for work to do and running tasks until completion.

## Key Points

- **Central coordinator** - Manages all async operations in a single thread
- **Continuous cycle** - Runs indefinitely, checking for and executing tasks
- **Non-blocking I/O** - Uses OS mechanisms (epoll, kqueue, IOCP) for efficient I/O monitoring
- **Task scheduling** - Maintains queues of ready and waiting tasks
- **One loop per thread** - Each thread can have its own event loop, but typically one main loop

## How It Works

1. **Run queue** - Tasks ready to execute
2. **Select/poll** - Check for I/O readiness
3. **Callback execution** - Run callbacks for completed I/O
4. **Repeat** - Continue until stopped

```
┌─────────────────────────────────────┐
│           EVENT LOOP                │
│                                     │
│  ┌──────────────┐  ┌─────────────┐ │
│  │   Run Queue  │  │  I/O Queue  │ │
│  │  (ready to   │  │  (waiting   │ │
│  │   execute)   │  │   for I/O)  │ │
│  └──────┬───────┘  └──────┬──────┘ │
│         │                 │        │
│         └────────┬────────┘        │
│                  │                 │
│         ┌────────▼────────┐        │
│         │  Execute Tasks  │        │
│         └────────┬────────┘        │
│                  │                 │
│         ┌────────▼────────┐        │
│         │ Check I/O State │        │
│         │  (select/poll)  │        │
│         └─────────────────┘        │
└─────────────────────────────────────┘
```

## Getting and Running the Event Loop

```python
import asyncio

# Method 1: asyncio.run() - Preferred, creates and manages loop automatically
async def main():
    print("Running in event loop")

asyncio.run(main())  # Creates loop, runs coroutine, closes loop

# Method 2: Manual loop management (advanced use cases)
loop = asyncio.new_event_loop()
asyncio.set_event_loop(loop)
try:
    loop.run_until_complete(main())
finally:
    loop.close()

# Method 3: Get current loop (within async context) — preferred over get_event_loop()
async def get_loop():
    loop = asyncio.get_running_loop()  # Always use this inside async code
    print(f"Current loop: {loop}")

# DEPRECATED in 3.10+, removed behaviour in 3.12:
# asyncio.get_event_loop()  — raises DeprecationWarning if no running loop
```

## Event Loop Internals

```python
async def demonstrate_loop():
    # Get the running loop
    loop = asyncio.get_running_loop()
    
    # Loop properties
    print(f"Loop running: {loop.is_running()}")
    print(f"Loop closed: {loop.is_closed()}")
    
    # Time in loop (monotonic clock)
    now = loop.time()
    print(f"Loop time: {now}")
    
    # Schedule callback
    loop.call_later(1, lambda: print("Callback executed!"))

asyncio.run(demonstrate_loop())
```

## When to Use

- **Web servers** - Handling thousands of concurrent connections (FastAPI, aiohttp)
- **API clients** - Concurrent HTTP requests without thread overhead
- **Real-time applications** - WebSockets, chat systems
- **I/O-bound workloads** - Database queries, file operations, network calls
- **Microservices** - High-throughput service-to-service communication

## When NOT to Use

- **CPU-intensive tasks** - Event loop doesn't parallelize CPU work (use multiprocessing)
- **Simple sequential scripts** - Adds unnecessary complexity
- **Blocking code** - Mixing blocking operations freezes the entire loop
- **Applications needing true parallelism** - asyncio is concurrent, not parallel

## Blocking the Event Loop

```python
import time

async def bad_example():
    # WRONG: This blocks the ENTIRE event loop!
    time.sleep(5)  # All other tasks freeze for 5 seconds
    
async def good_example():
    # RIGHT: Yield control to other tasks
    await asyncio.sleep(5)  # Other tasks can run during wait
```

## Common Operations

### Scheduling Tasks
```python
async def main():
    loop = asyncio.get_running_loop()
    
    # Schedule to run later
    loop.call_later(2, callback_function)
    
    # Schedule at specific time
    loop.call_at(loop.time() + 5, callback_function)
    
    # Schedule immediately
    loop.call_soon(callback_function)
```

### Running in Executor (for blocking code)
```python
import concurrent.futures

async def run_blocking_code():
    loop = asyncio.get_running_loop()
    
    # Run CPU-intensive code in thread pool
    with concurrent.futures.ThreadPoolExecutor() as pool:
        result = await loop.run_in_executor(
            pool, 
            cpu_intensive_function, 
            arg1, 
            arg2
        )
    return result
```

## Event Loop Policies

```python
# Windows uses SelectorEventLoop by default (Proactor in 3.8+ for pipes)
# Linux/macOS use SelectorEventLoop

# Change policy (advanced)
asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

## Related Concepts

- [[Async-Await Syntax]] - The language features the loop enables
- [[Coroutines]] - The units of work the loop executes
- [[Tasks vs Futures]] - How the loop manages concurrent operations
- [[Asyncio Module]] - The library providing the event loop
- [[Running Sync Code in Async Context]] - How to use executors
- [[Custom Event Loop]] - Advanced: building custom loop implementations

## Examples

### Basic Event Loop Flow
```python
import asyncio

async def task(name, delay):
    print(f"Task {name} starting")
    await asyncio.sleep(delay)
    print(f"Task {name} finished")

async def main():
    # Preferred (3.11+): TaskGroup for structured concurrency
    async with asyncio.TaskGroup() as tg:
        tg.create_task(task("A", 2))
        tg.create_task(task("B", 1))
    # Total time: ~2 seconds (not 3!)
    # Any exception raises ExceptionGroup

asyncio.run(main())
```

### Monitoring Loop Health
```python
async def monitor_loop():
    loop = asyncio.get_running_loop()
    
    # Check if loop is responsive
    start = loop.time()
    await asyncio.sleep(0.001)  # Tiny sleep
    elapsed = loop.time() - start
    
    if elapsed > 0.1:  # Should be ~0.001
        print(f"WARNING: Event loop lag detected: {elapsed:.3f}s")

asyncio.run(monitor_loop())
```

### Custom Loop with Cleanup
```python
async def main():
    try:
        # Your application logic
        await run_server()
    except asyncio.CancelledError:
        print("Shutting down gracefully...")
        await cleanup_resources()
        raise  # Re-raise to properly exit

# Run with proper signal handling
if __name__ == "__main__":
    asyncio.run(main())
```

## Performance Considerations

- **Latency** - Event loop adds small overhead per task switch (~microseconds)
- **Scalability** - Can handle 10,000+ concurrent connections vs ~hundreds with threads
- **Fairness** - Long-running tasks can starve others (use `await asyncio.sleep(0)` to yield)
- **Memory** - Much lower memory per "connection" than threads

## Debugging

```python
# Enable debug mode
asyncio.run(main(), debug=True)

# Or set via environment variable
# PYTHONASYNCIODEBUG=1 python script.py

# Shows:
# - Slow callbacks (>100ms)
# - Unawaited coroutines
# - Exceptions in background tasks
```

## References

- [Python Event Loop documentation](https://docs.python.org/3/library/asyncio-eventloop.html)
- [Event Loop implementation details](https://docs.python.org/3/library/asyncio-eventloops.html)
- [Real Python - Async IO in Python](https://realpython.com/async-io-python/)
- [[Asyncio Module]] - Higher-level asyncio APIs
